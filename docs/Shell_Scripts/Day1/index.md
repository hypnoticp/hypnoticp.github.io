```
###写一个脚本，遍历/data/目录下的txt文件
###将这些txt文件做一个备份
###备份的文件名增加一个年月日的后缀，比如将tom.txt备份为tom.txt_20240424
```

***

```
#!/bin/bash
##定义后缀变量，注意下面``(反引号)的含义
suffix =  `date +%Y%m%d`

##找到/data/目录下的txt文件，用for循环遍历
for f in `find /data/ -type f -name "*.txt"`
do
    echo "备份文件$f"
    cp ${f} ${f}_${suffix}
done 
```

### 总结：

1. date命令用法，可以根据日期，时间获取到想要的字符
2. for循环如何遍历文件