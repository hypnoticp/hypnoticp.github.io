- 写一个监控脚本，监控某站点访问是否正常。
- 可以将访问的站点以参数的形式提供，例如 sh xxx.sh www.aminglinux.com
- 状态码为2xx或者3xx表示正常
- 正常时echo正常，不正常时echo不正常

```
#!/bin/bash

##检查本机有没有curl命令
if ! which curl &>/dev/null
then
    echo "本机没有安装curl"
    ## 这里假设系统为Centos/RHEL/Rocky
    yum install -y curl
    if [ $? -ne 0 ]
    then
        echo "没有安装成功curl"
        exit 1
    fi
fi
##获取状态码
code=`curl --connect-timeout 3 -I $1 2>/dev/null |grep 'HTTP' | awk '{print $2}'`
##状态码是2xx或者是3xx，则条件成立
if echo $code |grep -qE '^2[0-9][0-9]|^3[0-9][0-9]'
then
    echo "$1访问正常"
else
    echo "$1访问不正常"
fi
```

![截图](fc7ed3f026dd1703c66c285ccb8bef09.png)

![截图](58d2b4ce22c31be413fbff19160fbf85.png)