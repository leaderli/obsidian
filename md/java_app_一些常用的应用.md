
## [sonar](https://www.sonarqube.org/)

###  下载

可以下载[zip包](https://www.sonarqube.org/downloads/)，解压后即可运行

### 启动

```shell
# On other operating systems, as a non-root user execute:
/opt/sonarqube/bin/[OS]/sonar.sh console
# 例如在linux下
/opt/sonarquebe/bin/linux-x86-64/sonar.sh start
```
	
###  查看
	默认情况下，可以通过`localhost:9000`进行访问，默认密码为`admin:admin`
	
### 一些配置

- 配置扫描引擎的jvm大小
	修改`conf/sonar.properties`中的配置项
	```properties
	# 默认大小为512m
	sonar.ce.javaOpts=Xmx2048m -Xms512m -XX:+HeadDumpOnOutOfMemeroyError
	```