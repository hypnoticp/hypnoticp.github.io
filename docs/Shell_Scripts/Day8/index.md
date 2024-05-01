- 输入一个数字，然后允许对应的一个命令
- 显示命令如下：`*cmd meau** 1-date 2-ls 3-who 4-pwd`
- 当输入1时，会允许date，依此类推。

```
#!/bin/bash

##先把提示语打印出来
echo "*cmd meau** 1-date 2-ls 3-who 4-pwd"

##使用死循环，目的是为了当用户输入的字符并非要求的字符时，不能直接退出脚本，而是再次开始
while :
do
  ##然后开始使用read实现和用户交互，提示让用户输入一个数字
  read -p "please input a number 1-4: " n
  case $n in
      1)
          date
          ## 之所以要break，是因为当用户执行完命令就要退出了
          break
          ;;
      2)
          ls
          break
          ;;
      3)
          who
          break
          ;;
      4)
          pwd
          break
          ;;
      *)
          ##如果输入的不是1-4的数字，提示出错
          echo "Wrong input  try again!"
          ;;
  esac
done
```

![截图](0829a2a2b64036f54ccbedb40349c447.png)

## 总结

1. read -p 可以在shell脚本中实现和用户交互的效果
2. case ... esac 这种逻辑判断用法，非常适合做选择题，尤其是选项很多时，选项也可以有多个值，比如1|5
3. 如果想要反复和用户交互，必须使用while循环，并借助break或者continue来控制循环流程
4. break表示退出循环体，continue表示结束本次循环，进入下次循环