# nfs-torture
Docker powered torture suite for nfs servers

Based on https://hub.docker.com/r/d3fk/nfs-client/

Requisites:
- Debian 10
- a NFS server with some files on it (192.168.1.100 in this example)

How to, client side:
- Install Docker

```
apt-get update
apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io
```

- torture-mount.sh

```
#!/bin/bash

brake=".1"

#docker run -itd --privileged=true --net=host d3fk/nfs-client
docker run -itd --privileged=true d3fk/nfs-client
sleep $brake
container=$(docker ps -a | head -n2 | tail -n1 | awk '{print $1}')
sleep $brake
docker container exec -u 0 $container sh -c "mount 10.1.0.1:/ /mnt/nfs-1/"
sleep $brake
docker container exec -u 0 $container sh -c "mount | grep nfs4"
sleep $brake
free -h
```



