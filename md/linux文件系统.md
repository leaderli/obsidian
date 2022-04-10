
#linux 




## df

查看文件系统以及它们的相关信息

## cp

复制文件

- -v 显示复制的详情

## tar

```shell
#压缩
tar -zcvf myfile.tgz file1 file2
# 解压
tar -xvf file.tar
```

## rsync

同步命令，可用于备份应用。该命令做数据的同步备份，会对比数据源目录和数据备份目录的数据，并把不同的数据同步到备份目录。其也可以同步到其他服务器，用法类似 scp

- -v 显示同步详情
- -p 显示同步百分比
- --delete 默认情况下 source 中被删除的文件不会同步到 target 中，需要加上该参数才会同步删除命令
- --exclude 不同步符合[pattern]的文件，[pattern]表达式要匹配的相对路径下的文件名称，包含路径
  例如

  ```shell
  rsync -avp   --exclude='dir/*' /etc/ /data/etc/
  ```

- --include 要配置`--exclude`一起使用，表示不执行符合[pattern]的`--exclude`
  例如

  ```shell
  # 表示不同步/etc/log下的文件，除了/etc/log/important相关文件
  rsync -avp  --include='/etc/*' --include='/etc/log/important*' --exclude='/etc/log/*' /etc/ /data/etc/
  ```



## proc 目录

Linux 内核提供了一种通过 proc 文件系统，在运行时访问内核内部数据结构、改变内核设置的机制。proc 文件系统是一个伪文件系统，它只存在内存当中，而不占用外存空间。它以文件系统的方式为访问系统内核数据的操作提供接口。

1. cmdline 启动时传递给 kernel 的参数信息
2. cpuinfo cpu 的信息
3. filesystems 内核当前支持的文件系统类型
4. locks 内核锁住的文件列表
5. meminfo RAM 使用的相关信息
6. swaps 交换空间的使用情况
7. version Linux 内核版本和 gcc 版本
8. self 链接到当前正在运行的进程

proc 目录下一些以数字命名的目录，它们是进程目录。系统中当前运行的每一个进程都有对应的一个目录在 proc 下，以进程的 PID 号为目录名，它们是读取进程信息的接口。而 self 目录则是读取进程本身的信息接口，是一个 link。

1. `/proc/[pid]/cmdline` 该进程的命令及参数
2. `/proc/[pid]/comm` 该进程的命令
3. `/proc/[pid]/cwd` 进程工作目录
4. `/proc/[pid]/environ` 该进程的环境变量
5. `/proc/[pid]/exe` 该进程命令的实际命令地址
6. `/proc/[pid]/fd` 该进程打开的文件描述符
7. `/proc/[pid]/maps` 显示进程的内存区域映射信息
8. `/proc/[pid]/statm` 显示进程所占用内存大小的统计信息 。包含七个值，度量单位是 page(page 大小可通过 getconf PAGESIZE 得到)。举例如下：

   ```shell
   $ cat statm
   26999 154 130 15 0 79 0

   ```

   - a）进程占用的总的内存
   - b）进程当前时刻占用的物理内存
   - c）同其它进程共享的内存
   - d）进程的代码段
   - e）共享库(从 2.6 版本起，这个值为 0)
   - f）进程的堆栈
   - g）dirty pages(从 2.6 版本起，这个值为 0)

9. `/proc/[pid]/status` 包含进程的状态信息。

   ```shell
   $ cat /proc/2406/status
   Name:   frps
   State:  S (sleeping)
   Tgid:   2406
   Ngid:   0
   Pid:    2406
   PPid:   2130
   TracerPid:  0
   Uid:    0   0   0   0
   Gid:    0   0   0   0
   FDSize: 128
   Groups: 0
   NStgid: 2406
   NSpid:  2406
   NSpgid: 2406
   NSsid:  2130
   VmPeak:    54880 kB
   VmSize:    54880 kB
   VmLck:         0 kB
   VmPin:         0 kB
   VmHWM:     34872 kB
   VmRSS:     10468 kB
   VmData:    47896 kB
   VmStk:       132 kB
   VmExe:      2984 kB
   VmLib:         0 kB
   VmPTE:        68 kB
   VmPMD:        20 kB
   VmSwap:        0 kB
   HugetlbPages:          0 kB
   Threads:    11
   SigQ:   0/31834
   SigPnd: 0000000000000000
   ShdPnd: 0000000000000000
   SigBlk: 0000000000000000
   SigIgn: 0000000000000000
   SigCgt: fffffffe7fc1feff
   CapInh: 0000000000000000
   CapPrm: 0000003fffffffff
   CapEff: 0000003fffffffff
   CapBnd: 0000003fffffffff
   CapAmb: 0000000000000000
   Seccomp:    0
   Cpus_allowed:   f
   Cpus_allowed_list:  0-3
   Mems_allowed:   00000000,00000001
   Mems_allowed_list:  0
   voluntary_ctxt_switches:    2251028
   nonvoluntary_ctxt_switches: 18031
   ```

## mdls

查看文件的 meta 信息
