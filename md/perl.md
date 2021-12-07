

## shell脚本中使用

- `-e`  以一行执行
- `-l`  遇到换行终止执行
- `-n` 假定使用`<>{...}`执行
- `-a` 分割字符串模式，将字符串数组赋值给`@F`


### 一些变量

1. `$_` 当前行
2. `$.` 当前行数

### 类似awk的用法

```shell
# @F 类似awk的 $0
ll|perl -lane 'print "@F" if  /n/'

# $F[0]类似awk的$1    =~表示满足正则表达式  
# 下面则是筛选出目录
ll|perl -lane 'print "@F[0,1,2,3,4,5,6,7,8]" if   $F[0]  =~/^d/'

# perl 数组可以使用正则来表示，与上一行等价
ll|perl -lane 'print "@F[0..8]" if   $F[0]  =~/^d/'
```