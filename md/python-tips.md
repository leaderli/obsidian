#python #tips 

## 截取字符串或数组

w = '1'
当使用 w[1:],会得到一个空串，而不会报错

## python dict 根据 value 找对应的 key

```python
dicxx = {'a':'001', 'b':'002'}
list(dicxx.keys())[list(dicxx.values()).index("001")]
#'a'
```

## 使用守护进程的方式后台启动

```shell
# 后台启动
$ python background_test.py >log.txt 2>&1 &
```

## 监听文件是否有变动

```python
# --coding:utf-8--
import os
import time

filename = '.'  # 当前路径
last = time.localtime(os.stat(filename).st_mtime)
while True:
    new_filemt = time.localtime(os.stat(filename).st_mtime)
    if last != new_filemt:
        last = new_filemt
        print('change')
        #os.system('nginx -s reload')
    time.sleep(1)

```

## 替换字符串

类似 java 的 replace

```python
import re

origin='/1/2'
re.sub('^/','',origin)
# 1/2
```

## 调用shell

```python
import subprocess

try:
  subprocess.call(['/bin/bash','xxx.sh','arg1','arg2'])
except:
  #可以忽略所有错误，比如ctrl c，终止sh运行的错误
  pass
```

获取命令行参数
[Python 命令行参数 | 菜鸟教程](https://www.runoob.com/python/python-command-line-arguments.html)

```python
#!/usr/bin/python  
# -*- coding: UTF-8 -*-  
  
import sys  
  
print '参数个数为:', len(sys.argv), '个参数。'  
print '参数列表:', str(sys.argv)  
```

执行以上代码，输出结果为：
```shell
$ python test.py arg1 arg2 arg3 
参数个数为: 4 个参数。 
参数列表: ['test.py', 'arg1', 'arg2', 'arg3']
```

