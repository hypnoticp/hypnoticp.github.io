- 写一个监控脚本，监控系统负载，如果系统负载超过10，需要记录系统状态信息。
- 系统负载命令使用uptime查看，过去一分钟的平均负载
- 系统状态使用如下工具标记：top，vmstat，ss
- 要求每隔20s监控一次
- 系统状态信息需要保存到/opt/logs/下面，保留一个月，文件名建议带有`date +%s`后缀或者前缀
  ```
  #!/bin/bash
  
  ##首先看/opt/logs目录在不在，不在就创建
  [ -d /opt/logs ] || mkdir -p /opt/logs
  ##while死循环
  while :
  do 
      ## 获取系统1分钟的负载，并且只取小数点前面的数字
      load=`uptime |awk -F 'average:' '{print $2}'|cut -d',' -f1|sed 's/ //g' |cut -d. -f1`
      # 使用 uptime 命令获取系统负载信息，然后用 awk 从输出中提取出负载值，
      # cut 命令用于移除逗号后的部分，sed 命令用于去除空格，最后 cut 命令用于提取小数点前的整数部分，
      # 赋值给变量 load
      if [ $load -gt 10 ]
      then 
          ##分别记录top，vmstat和ss命令的执行结果
          top -bn1 | head -n 100 > /opt/logs/top.`date +%s`
          # 执行 top 命令并将前100行输出保存到以当前时间戳命名的文件中
          vmstat 1 10 > /opt/logs/vmstat.`date +%s`
          # 执行 vmstat 命令，每秒采集一次数据，共采集10次，并将输出保存到以当前时间戳命名的文件中
          ss -an > /opt/logs/ss.`data +%s`
          # 执行 ss 命令以获取当前网络连接信息，并将输出保存到以当前时间戳命名的文件中
      fi
      ##休眠20秒
      sleep 20
      ##找到30天以前的日志文件删除掉
      find /opt/logs \( -name "top*" -o -name "vmstat*" -o -name "ss*" \) -mtime +30 |xargs rm -f
  done
  ```

![截图](8b8d3f9d8ce60b6efde41d092b044277.png)

![截图](b7ff7313db23fd4f1546468a0acc419d.png)

## 总结

1. ||用在两条命令中间，可以起到这样的效果，当前面命令不成功就会执行后面的命令
2. 死循环可以使用while ：+ sleep组合
3. 边写脚本边在命令行里调试
4. find里可以使用小括号将多个条件组合起来当成一个整体来处理