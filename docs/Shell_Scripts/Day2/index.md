- 创建十个用户，并给他们设置随机密码，密码记录到一个文件中，文件名为userinfo_txt
- 用户从user_00到user_09
- 密码要求：包含大小写字母以及数字，密码长度为15位
  ```
  #!/bin/bash
  
  ##先查看/tmp/userinfo.txt文件是否存在，存在的话先删除，以免影响脚本结果
  if [ -f /tmp/userinfo.txt ]
  then
      rm -f /tmp/userinfo.txt
  fi
  
  ##判断mkpasswd命令在不在，我们使用该命令来生成随机字符串也就是用户的密码
  if ! which mkpasswd
  then
  ##在这里，我们假定系统是Centos或者Rocky，可以使用yum安装包
      yum install -y expect
  fi
  
  ##借助seq生成从00到09，10个数的队列
  for i in `seq -w 0 09`
  do
      ##每次生成一个随机字符串，将该字符串赋值给p变量，这个就是用户的密码
      ##mkpasswd命令默认生成的字符串会包含大小写字母和数字和特殊符号
      ##如果不要求特殊符号，可以添加-s 0 来限定不使用特殊符号
      p=`mkpasswd -l 15 -s 0`
      ##添加用户，并给该用户设置密码
      useradd user_${i} && echo "${p}"| passwd --stdin user_${i}
      echo "user_${i} ${p}" >> /tmp/userinfo.txt
  done
  ```

我的机子不是Centos，这里就不演示了。

### 总结：

1.mkpasswd可以生成随机字符串，-l指定长度，-s指定特殊字符个数，-c指定小写字母个数，-C指定大写字母个数，-d指定数字个数

2.seq可以生成序列，用法： seq 1 5 ;seq 5; seq 1 2 10; seq 10 -2 1;seq -w 1 10

3.passwd --stdin username