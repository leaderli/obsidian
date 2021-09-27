# go

## 安装


[官方下载网址](https://golang.google.cn/dl/)，[仓库镜像地址](https://goproxy.cn/)

### linux

[linux安装](https://golang.org/doc/install)

```shell
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.17.1.linux-amd64.tar.gz

export PATH=$PATH:/usr/local/go/bin

go version
```

## 新建工程

```shell
$ cd Hello
$ go mod init hello
go: creating new go.mod: module hello

$ cat go.mod
module hello

go 1.17

```


hello.go
```go
package main

import "fmt"

func main() {
	fmt.Println("Hello, go")
}

```


```shell
# 运行
$ go run hello.go
hello , go
# 编译
$ go build 

# 运行exe
$ ./hello.exe
hello , go
```

## 环境

```shell
$ go env 
set GO111MODULE=
set GOARCH=amd64
set GOBIN=
set GOCACHE=C:\Users\pc\AppData\Local\go-build
set GOENV=C:\Users\pc\AppData\Roaming\go\env
set GOEXE=.exe
set GOEXPERIMENT=
set GOFLAGS=
set GOHOSTARCH=amd64
set GOHOSTOS=windows
set GOINSECURE=
set GOMODCACHE=C:\Users\pc\go\pkg\mod
set GONOPROXY=
set GONOSUMDB=
set GOOS=windows
set GOPATH=C:\Users\pc\go
set GOPRIVATE=
set GOPROXY=https://proxy.golang.org,direct
set GOROOT=D:\ProgramFiles\Go
set GOSUMDB=sum.golang.org
set GOTMPDIR=
set GOTOOLDIR=D:\ProgramFiles\Go\pkg\tool\windows_amd64
set GOVCS=
set GOVERSION=go1.17.1
set GCCGO=gccgo
set AR=ar
set CC=gcc
set CXX=g++
set CGO_ENABLED=1
set GOMOD=D:\work\study\go\Hello\go.mod
set CGO_CFLAGS=-g -O2
set CGO_CPPFLAGS=
set CGO_CXXFLAGS=-g -O2
set CGO_FFLAGS=-g -O2
set CGO_LDFLAGS=-g -O2
set PKG_CONFIG=pkg-config
set GOGCCFLAGS=-m64 -mthreads -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebu
g-prefix-map=C:\Users\pc\AppData\Local\Temp\go-build2303936022=/tmp/go-build -gno-record-gcc-swit
ches

```

注意里面两个重要的环境变量`GOOS`和`GOARCH`,其中`GOOS`指的是目标操作系统，它的可用值为：

1.  aix
2.  android
3.  darwin
4.  dragonfly
5.  freebsd
6.  illumos
7.  js
8.  linux
9.  netbsd
10.  openbsd
11.  plan9
12.  solaris
13.  windows

一共支持13种操作系统。`GOARCH`指的是目标处理器的架构，目前支持的有：

1.  arm
2.  arm64
3.  386
4.  amd64
5.  ppc64
6.  ppc64le
7.  mips
8.  mipsle
9.  mips64
10.  mips64le
11.  s390x
12.  wasm

### 修改环境变量

需要用管理员权限
```shell
setx GOPATH E:\Gopath
```

报错`go: GOPATH entry is relative; must be absolute path`
```shell
$ export GOPATH="/d/resource/gopath"
```