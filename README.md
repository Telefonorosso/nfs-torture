# nfs-torture
Docker powered torture suite for NFS servers!
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
apt-get install docker-ce docker-ce-cli containerd.io parallel
```

nano torture-mount.sh

```
#!/bin/bash
brake=".1"
docker run -itd --privileged=true d3fk/nfs-client
sleep $brake
container=$(docker ps -a | head -n2 | tail -n1 | awk '{print $1}')
sleep $brake
docker container exec -u 0 $container sh -c "mount 192.168.1.100:/ /mnt/nfs-1/"
sleep $brake
docker container exec -u 0 $container sh -c "mount | grep nfs4"
sleep $brake
free -h
```

- spawn 50 clients:

```
seq 50 | xargs -I -- ./torture-mount.sh
```

This will eat up around 3.5 gigs of RAM!

- put some stress on a randomly chosen container:

nano md5stres.sh

```
#!/bin/bash
casuale=$(docker ps -a | sed '1d' | awk '{print $1}' | shuf -n1)
echo $casuale
docker container exec -u 0 $casuale sh -c "find /mnt/nfs-1/ -not -path '*/\.*' -type f -print0 2>/dev/null | xargs -0 md5sum"
```

This will md5sum every single file found in the remote share!

nano straw.sh

```
#!/bin/bash
docker ps -a | sed '1d' | awk '{print $1}' >/tmp/strawsh.lst
while read c; do
echo $c
com="dd if=/dev/urandom of=/mnt/nfs-1/stress/garbage-$(pwgen 11 1) bs=32K count=1"
docker container exec -u 0 $c sh -c "$com"
done </tmp/strawsh.lst
```

- write test:

```
seq 50 | parallel -n0 ./strawsh
```

This will deluge-write 32K garbage files in the root directory of the remote share, with 50x parallelism

(quite stressful for the client, too!)

- to clean up all this mess:

nano abandon.sh
```
#!/bin/bash
docker ps -a | sed '1d' | awk '{print $1}' >/tmp/remove.lst
while read c; do
echo $c
docker rm -f $c
done </tmp/remove.lst
echo -e "y\n" | docker system prune -a
```

***Be careful! All your containers will be vaporized!***
