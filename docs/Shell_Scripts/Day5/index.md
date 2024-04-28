- 有一个目录/data/att/，该目录下有数百个子目录，比如/data/att/aming，/data/att/linux
- 然后再深入一层为以日期命名的目录，比如/data/att/aming/20231009，
- 每天都会生成一个日期新目录，由于/data所在磁盘快满了，
- 所以需要将老文件(一年以前的)挪到另外一个目录/data1/att下，
- 示例：mv /data/att/aming/20240428 /data1/att/aming/20240428
- 挪完之后，还需要做软链接，
- 示例：ln -s /data1/att/aming/20240428  /data/att/aming/20240428
- 写一个脚本，要求/data/att写所有目录子目录都要按此操作，
- 脚本会每天01：00执行一次，任务计划无需考虑，只需要写脚本即可
- 提醒：要确保老文件成功挪到/data1/att下之后才能做软链接，需要有日志
  ```
  #！/bin/bash
  
  ##先定义一个main函数，目的是为了后面调用函数，方便记录日志
  main()
  {
      cd /data/att
      ## 遍历第一层目录
      for dir in `ls`
      do
          ## 遍历第二层目录，用find只找当前目录一年以前的子目录
          for dir2 in `find $dir -maxdepth 1 -type d -mtime +365`
          do
              ## 将目标目录下的文件同步到/data/att/目录下，注意这里的-R可以自动创建目录结构
              rsync -aR $dir2/  /data1/att/
              if [ $? -eq 0 ]
              then
                  ## 如果同步成功，会将/data/att下的目录删除
                  rm -rf $dir2
                  echo "/data/att/$dir2 移动成功"
                  ## 做软连接
                  ln -s /data1/att/$dir2 /data/att/$dir2  && \
                  echo "/data/att/$dir2成功创建软链接"
                  echo
              else
                  echo "/data/att/$dir 未移动成功"
              fi
          done
      done
  }
  
  ## 调用main函数，并将输出写入日志里，  日志每天一个
  main &> /tmp/move_old_data_`date +%F`.log
  ```

## 总结

1. 可以通过main函数的形式来方便定义脚本日志
2. find 使用-maxdepth定义查找目录层级
3. 脚本某行很长的话，可以使用"\ + 回车"来换行，但本质上还是一行内容
4. rsync的-R选项可以自动级联创建目录层级
5. `find $dir -maxdepth 1 -type d -mtime +365`：用于查找指定目录 dir 下修改时间超过 365 天的所有目录，$dir表示要查找的目录的路径，-maxdepth 1指定搜索的最大深度为1，即只搜索指定目录下的直接子目录，不会递归地搜索子目录的子目录。
6.   ` rsync -aR $dir2/  /data1/att/`：将源目录 $dir2 下的所有文件和子目录（保持原有目录结构）同步到目标目录 /data1/att/ 下，同时保持文件的所有特性。
7. `if [ $? -eq 0 ]`：如果上一个命令的退出状态码为 0（即命令执行成功），则执行 if 语句中的代码块；否则，执行与 if 语句关联的 else 语句中的代码块。