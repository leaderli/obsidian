
### 修改hosts

```
C:\Windows\System32\drivers\etc

编辑 hosts 文件，编辑后清空dns缓存

192.168.111.133 centos7
```
### 清空dns缓存
```shell
ipconfig /flushdns
```