---
title: linux常用命令
date: 2020-05-13 21:53:18
categories: linux
tags:
---






































###  netstat

```shell
# 显示所有端口
netstat -tulpn
```

### ss

Linux 中的 ss 命令是 Socket Statistics 的缩写。顾名思义，ss 命令可以用来获取 socket 统计信息，它可以显示和 [[#netstat]] 类似的内容。但 ss 的优势在于它能够显示更多更详细的有关 TCP 和连接状态的信息，而且比[[#netstat]]更快速更高效。

- 查看所有网络连接
	```shell
	ss
	```
- 查看所有TCP连接
	```
	ss -ta
	```
	
### 统计

```shell
$ wc freeswitch.md
#   行数     单词数  byte字节数
    1081    2968   41953 freeswitch.md
$ wc -m freeswitch.md
#   char字数
   30744 freeswitch.md
```

### sed

sed 命令是利用脚本来处理文本文件。sed 可依照脚本的指令来处理、编辑文本文件。
语法

```shell
sed [-hnV][-e<script>][-f<script 文件>][文本文件]
```

参数说明：

- -e 指定脚本的表达式，不会修改源文件，仅将修改后的内容输出到控制台
- -f 指定脚本的文件
- -i 指定脚本的表示式，会直接修改源文件
- -n 仅显示脚本所包含模板的行

其中脚本的指令支持

- a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
- c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
- d ：删除，因为是删除啊，所以 d 后面通常不接任何咚咚；
- i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
- p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
- s ：取代，可以直接进行取代的工作哩！通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g 就是啦！
- g ： 在行内进行全局替换
- w 将所选的行写入文件
- ! 对所选行以外的行执行
- l 列出非打印字符

脚本的指令可以在指定行范围内执行，例如

```shell
# 第四行行尾  新行+newline
sed -e  4a\newline testfile
# 第一到第四行行尾  新行+newline
sed -e  1，4a\newline testfile
# 最后一行 新行+newline
sed -e  $a\newline testfile


# 使用shell变量
var=hello
sed -e  '1a'${var} testfile
```

与 grep 一样，sed 也支持特殊元字符，来进行模式查找、替换。不同的是，sed 使用的正则表达式是括在斜杠线"/"之间的模式。
如果要把正则表达式分隔符"/"改为另一个字符，比如 o，只要在这个字符前加一个反斜线，在字符后跟上正则表达式，再跟上这个字符即可。例如：sed -n '\o^Myop' datafile
sed 的正则表达式语法支持功能比较少
| 元字符| 功能|
| :---- | :---- |
|`^` |行首定位符|
|`$` |行尾定位符|
|`.` |匹配除换行符以外的单个字符 |
|`* `|匹配零个或多个前导字符 |
|`[ ]` | 匹配指定字符组内的任一字符|
|`[^]`| 匹配不在指定字符组内的任一字符|
|`&` |保存查找串以便在替换串中引用 `s/my/&/` 符号`&`代表查找串。my 将被替换为`**my**`|
|`<` | 词首定位符 `/<my/` 匹配包含以 my 开头的单词的行|
|`>` | 词尾定位符 `/my>/` 匹配包含以 my 结尾的单词的行|
|`x{m}` | 连续 m 个 x|
|`x{m,}`| 至少 m 个 x|
|`x{m,n}`| 至少 m 个，但不超过 n 个 x|

特殊字符

- `\` 新行

示例

```shell
# nl 输出行号 搜索包含root的行并打印
nl  testfile|sed -n '/root/p'
# nl 输出行号 搜索包含root的行并删除
nl  testfile|sed -n '/root/d'
# nl 输出行号 搜索包含root的行并删除
# 使用正则替换
sed 's/pattern/replace/g'

# 替换为匹配组的内容
sed "s/.*'\([^']\+\)'.*/\1/"

# 对每一行执行替换为匹配组的内容
sed -n "s/.*'\([^']\+\)'.*/\1/p"
```

### 关闭自动退出

```shell
# vi /etc/profile

unset TMOUT
# 或者
TMOUT=0

```

### 当前用户

```shell
# 当前登录终端字符设备号
tty
# 当前所有登录的用户
w


```

### tee

同时向某个文件写入内容

```shell
$ echo 123|tee 1.txt
123
$ cat 1.txt
123
```

- -a 追加

### file

猜测文件的编码

```shell
file -i *
1.txt:                           text/plain; charset=us-ascii
2.txt:                           text/plain; charset=utf-8
```

### iconv

转换文件编码

```shell
iconv -f gbk -t utf8  1.txt >  2.txt
```
### tmux

一个开启多窗口的命令

配置文件`~/.tmux.conf`
```shell
set -g status off
set -g pane-border-bg default
set -g pane-border-fg white

set -g pane-active-border-bg default
set -g pane-active-border-fg white

```

1. 输入命令tmux使用工具
2. 上下分屏：`ctrl + b  再按 "`
3. 左右分屏：`ctrl + b  再按 %`
4. 切换屏幕：`ctrl + b  再按o`
5. 关闭一个终端：`ctrl + b  再按x`
6. 上下分屏与左右分屏切换： `ctrl + b  再按空格键`
7. 左右屏幕互换`ctrl+b+o`
 
 ### ifconfig
 
 查看IP，当没有装这个命令时可使用 [[#IP]]
 
 ### IP
 
 ```shell
 # 查看当前ip
 ip addr 
 ```

### telnet

使用telent可以去测试对应服务器IP和端口是否可以访问

```shell
# 安装
yum -y install telnet
```

`telnet [hostname/ipaddress] [port number]`

例如

```
telnet 123.123.123.123 22
```


java中可以通过模拟发送`ping`消息来测试

```java
package com.leaderli.litest;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;

public class TestTelnet {

        public static void main(String[] args) throws IOException {

            Socket pingSocket = null;
            PrintWriter out = null;
            BufferedReader in = null;

            try {
                pingSocket = new Socket("192.168.111.129", 22);
                out = new PrintWriter(pingSocket.getOutputStream(), true);
                in = new BufferedReader(new InputStreamReader(pingSocket.getInputStream()));
            } catch (IOException e) {
                e.printStackTrace();
                return;
            }

            out.println("ping");
            System.out.println(in.readLine());
            out.close();
            in.close();
            pingSocket.close();
        }
}

```

### zip

```shell
zip -r  test.zip   test_folder1/  test_foler2/
```