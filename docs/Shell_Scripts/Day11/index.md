- 使用传参的方法写个脚本，实现加减乘除的功能
- 例如： sh  a.sh  1 2,这样会分别计算加减乘除的结果
- 要求：脚本需判断提供的两个数字必须为整数；当做减法或者除法时，需判断那个数字大；减法时需要用大的数字减小的数字；除法时需要用大的数字除以小的数字，并且结果需要保留两个小数点。

```
#!/bin/bash

##先判断参数是不是2、
if [ $# -ne 2 ]
then
    echo "The number of parameter is not 2,Please useage: ./$0 1 2"
    exit 1
fi

##判断提供的数字是否是正整数
is_int()
{
    if echo "$1"|grep -q '[^0-9]'
    then
    echo "$1 is not integer number."
    exit 1
    fi
}

##找大数
max()
{
    if [ $1 -ge $2 ]
    then
    echo $1
    else
    echo $2
    fi
}
##找小数
min()
{
    if [ $1 -lt $2 ]
    then
    echo $1
    else
    ehco $2
    fi
}
## 加法
sum()
{
    echo "$1 +$2 = $[$1+$2]"
}
## 减法
{
    big=`max $1 $2`
    small=`min $1 $2`
    echo "$big - $small  = $[$big-$small]"
}
##乘法
mult()
{
    echo "$1 * $2 = $[$1*$2]"
}
##除法
div()
{
    big=`max $1 $2`
    small=`min $1 $2`
    d=`echo "scale = 2; $big / $small"|bc`
    echo "$big / $small = $d"
}

##调用各个函数
is_int $1
is_int $2
sum $1 $2
minus $1 $2
mult $1 $2
div $1 $2

```

## 总结

1. 脚本参数为`$1`, `$2`,`$3`...参数个数为$#
2. 脚本中函数用法，函数也支持参数
3. 数学运算可以借助bc，bc为Linux中一个计算器