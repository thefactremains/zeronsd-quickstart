
# ZeroNSD Quickstart

<p align="center">
<b><i>
It's not DNS.<br>
There's no way it's DNS.<br>
It was DNS.<br>
</i></b>
</p>

## Status

* This is *very much* alpha software.
* Deployment options are currently Source, Cargo, and Docker.
* This may end up integrated into ZeroTier 2.0, but for now, it is
  segregated to allow us to iterate and learn as quickly as possible.
* Here be Dragons.

## Conceptual Prerequisites

* When ZeroTier joins a network, it creates a virtual network interface.
* When ZeroTier joins mutiple networks, there will be multiple network interfaces.
* When ZeroNSD starts, it binds to a ZeroTier network interface.
* When ZeroTier is joined to multiple networks, it need multiple ZeroNSDs, one for each interface.

This means:

* ZeroNSD will be accessible from the node it is running on.
* ZeroNSD will be accessible from other nodes on the ZeroTier network.
* ZeroNSD will be isolated from other networks the node might be on.

## Technical Prerequisites

This Quickstart was written using two machines - one Ubuntu virtual
machine on Digital Ocean, and one OSX laptop on a residential ISP. To
follow along step by step, you'll need to provision equivalent
infrastructure. If you use different platforms, you should be able to
figure out what to do with minimal effort.

### Create a ZeroTier Network

You may do this manually through the [ZeroTier Central WebUI](https://my.zerotier.com), 

![Create a Network](https://i.imgur.com/L6xtGKo.png)

### Install ZeroTier

ZeroTier must be installed and joined to the network you intend to provide DNS service to.
The following should work from the CLI on most plaforms. Windows users
may download the MSI from the [ZeroTier Downloads](https://www.zerotier.com/download/) page. For
the remainder of this document, please replace the example network `159924d630edb88e` with a network ID your own.

```
root@ubuntu:~# curl -s https://install.zerotier.com | bash
root@ubuntu:~# zerotier-cli join 159924d630edb88e
root@ubuntu:~# zerotier-cli  set 159924d630edb88e allowDNS=1
```

```
user@osx:~$ curl -s https://install.zerotier.com | sudo bash
user@osx:~$ zerotier-cli join 159924d630edb88e
user@osx:~$ zerotier-cli  set 159924d630edb88e allowDNS=1
```


### Authorize the Nodes

Authoriz the node to the network by clicking the "Auth" button in the
`Members` section in the
[ZeroTier Central WebUI](https://my.zerotier.com).


![Authorize the Member](https://i.imgur.com/fQTai9l.png)

# Install ZeroNSD

## Debian / Ubuntu

```
root@ubuntu:~# /usr/bin/apt-get -y install net-tools librust-openssl-dev pkg-config cargo
root@ubuntu:~# /usr/bin/cargo install zeronsd --root /usr/local
```

Write your ZeroTier Central token to a file.

```
root@ubuntu:~# echo "YOURTOKENHERE" > /var/lib/zerotier-one/central-token
root@ubuntu:~# chown zerotier-one:zerotier-one /var/lib/zerotier-one/central-token
root@ubuntu:~# chmod 600 /var/lib/zerotier-one/central-token
```

## Systemd

We have a project for per-interface DNS resolution in the works. If
you're curious, you can check it out
[here](https://github.com/zerotier/zerotier-systemd-manager).

Until then, just drop this into your systemd units directory and
replace the network number with your own.

```
root@ubuntu:~# cat <<EOF> /lib/systemd/system/zeronsd.service
[Unit]
Description=ZeroNSD
User=zerotier-one
Group=zerotier-one
Requires=zerotier-one.service

[Service]
ExecStart=/usr/local/bin/zeronsd start 159924d630edb88e -d beyond.corp -s /var/lib/zerotier-one/authtoken.secret -t /var/lib/zerotier-one/central-token -f /etc/hosts
Restart=always
KillMode=process

[Install]
WantedBy=multi-user.target
EOF
```

Finally,

```
root@ubuntu:~# systemctl daemon-reload
root@ubuntu:~# systemctl enable zeronsd
root@ubuntu:~# systemctl restart zeronsd
```

## Verify functionality

Most Linux distributions, by default, do not have per-interface DNS
resolution out of the box. To test DNS queries against ZeroNSD, find
the IP address that ZeroNSD has bound itself to, and run queries
against it explicitly.

```
root@ubuntu:~# lsof -i -n | grep ^zeronsd | grep UDP | awk '{ print $9 }' | cut -f1 -d:
172.22.245.70
```

Query the DNS server directly with the dig command

The Ubuntu machine can be queried with:
```
root@ubuntu:~# dig +short @172.22.245.70 zt-3513e8b98d.beyond.corp
172.22.245.70
root@ubuntu:~# dig +short @172.22.245.70 server.beyond.corp
172.22.245.70
```

The OSX laptop can be queried with:
```
root@ubuntu:~# dig +short @172.22.245.70 zt-eff05def90.beyond.corp
172.22.245.70
root@ubuntu:~# dig +short @172.22.245.70 laptop.beyond.corp
172.22.192.177
```

Add a line to `/etc/hosts` and query again.

```
root@ubuntu:~# echo "1.2.3.4 test" >> /etc/hosts
root@ubuntu:~# dig +short @172.22.245.70 test.beyond.corp
1.2.3.4
```

Query a domain on the public DNS to verify fall through

```
root@ubuntu:~# dig +short @172.22.245.70 example.com
93.184.216.34
```

## OSX

OSX uses `dns-sd` for DNS resolution. Unfortunately, `nslookup`, `host`, and `dig` are broken on OSX.
`ping` works.

```
user@osx:~$ ping server.beyond.corp
PING server.beyond.corp (172.22.245.70): 56 data bytes
64 bytes from 172.22.245.70: icmp_seq=0 ttl=64 time=37.361 ms
64 bytes from 172.22.245.70: icmp_seq=1 ttl=64 time=38.129 ms
64 bytes from 172.22.245.70: icmp_seq=2 ttl=64 time=37.569 ms
```

To check out the system resolver settings, use: `scutil --dns`.

The Ubuntu machine can be queried with

`dns-sd -G v4 server.beyond.corp`  
`dns-sd -G v4 zt-3513e8b98d.beyond.corp`

The OSX machine be queried with

`dns-sd -G v4 laptop.beyond.corp`  
`dns-sd -G v4 zt-eff05def90.beyond.corp`

## Windows

Are you a Windows user?  
Does this work out of the box?  
Does nslookup behave properly?  
Let us know... feedback and pull requests welcome =)

-s
