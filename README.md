# ZeroNSD Quickstart

## Conceptual Prerequisites

* When Zerotier joins a network, it creates a virtual network interface.
* When Zerotier joins mutiple networks, there will be multiple network interfaces.
* When Zeronsd starts, it binds to udp/53 on the zerotier network interface of the specified network
* When Zerotier is joined to multiple networks, it need multiple zeronsds, one for each interface.

This means:

* Zeronsd will be accessable from the node it is running on
* Zeronsd will be accessable from other nodes on the zerotier network
* Zeronsd will be isolated from other networks a node might be on

## OSX

### cargo

sudo -E zeronsd start b6079f73c620a232
sudo -E zeronsd start abfd31bd47ae4c2d

nslookup, host, and dig are broken
ping works

scutil --dns

dns-sd -G v4 zt-eff05def90.domain
dns-sd -G v6 zt-eff05def90.domain

dns-sd -G v4 derp.domain
dns-sd -G v6 derp.domain

### docker

docker run -p -d nginxdemos/hello
docker pull zerotier/zeronsd:0.1.1
docker run -v /var/lib/zerotier-one:/var/lib/zerotier-one:ro --net host zerotier/zeronsd:0.1.1 start b6079f73c620a232
    

## Ubuntu
### systemd
### docker

