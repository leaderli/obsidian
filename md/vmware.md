
centos7共享目录

```shell
yum install open-vm-tools

sudo /usr/bin/vmhgfs-fuse .host:/ /mnt/hgfs -o subtype=vmhgfs-fuse,allow_other

```