

## shell脚本中使用

- `-e`  yi'hang

### 类似awk的用法

```shell
# @F 类似awk的 $0
ll|perl -lane 'print "@F" if  /n/'

# $F[0]类似awk的$1    =~表示满足正则表达式  
# 下面则是筛选出目录
ll|perl -lane 'print "@F[0,1,2,3,4,5,6,7,8]" if   $F[0]  =~/^d/'
```
