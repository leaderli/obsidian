
centos7共享目录

```shell
yum install open-vm-tools

sudo /usr/bin/vmhgfs-fuse .host:/ /mnt/hgfs -o subtype=vmhgfs-fuse,allow_other

```

设置IP的参考资料

[VMware虚拟机安装Centos7后设置静态ip - 今天不打怪 - 博客园](https://www.cnblogs.com/hsz-csy/p/9811969.html)
![[Pasted image 20211219230652.png]]
![[Pasted image 20211219230442.png]]
![[Pasted image 20211219230854.png]]

![[Pasted image 20211219230318.png]]
`/etc/sysconfig/network-scripts/ifcfg-ens33`
```diff
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
- BOOTPROTO="dhcp"
+ BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="87140c19-5b18-4db4-a259-1ea4bfdda736"
DEVICE="ens33"
ONBOOT="yes"
+ IPADDR="192.168.159.31"
+ NETMASK="192.168.159.0"
+ GATEWAY="192.168.159.2"
+ DNS1="127.0.0.1"
+ DNS2="8.8.8.8"
+ DNS3="8.8.4.4"
```

修改后重启
```shell
systemctl restart network
```