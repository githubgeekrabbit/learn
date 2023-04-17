
# 别名功能

### 1、别名

别名也就是把shell命令重命名，主要是用于照顾管理员使用习惯的。

命令格式：
查询系统中命令别名   

```bash
 [root@localhost ~]# alias
```

设定命令别名：
```bash
[root@localhost ~]# alias 别名='原命令'
```

  
## 用vi别名，替代vim命令 

```bash
[root@localhost ~]# alias vi = 'vim'  
```

> 就相当于用vi命令实际上执行的是vim，用命令行定义的别名只会临时生效，重启后就没了

## 让别名永久生效
```bash
vi /root/.bashrc
```
![Pasted image 20230417215239](http://qn.qu1u1.cn/202304172310702.png)

## 删除别名

```bash
unalias 别名
```

> alias查询系统中命令别名   

![Pasted image 20230417212413](http://qn.qu1u1.cn/202304172124989.png)

提示：
 命令执行顺序：
 > 用  `alias vi='vim'`  后
 
 1. 第一顺位优先级最高，用绝对路径或相对路径执行的命令如：`/usr/bin/vi install.sh`   相当于执行的是`vi install.sh` 高于 `vi install.sh`相当于执行的是`vim install.sh`
 2. 第二顺位别名的优先级是高于系统命令的。如：`vi install.sh` 实际执行的是`vim install.sh` ,所以别名高于系统命令
 3. 第三顺位执行bash的内部命令，是shell自带的，没有执行文件如`cd`
 4. 第四顺位按照$PATH环境变量定义的目录查询顺序找到第一个命令
 ```bash
 [root@ECS1133 ~]# echo $PATH
 /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```
 5. 怎么确定你定义的别名是否是系统命令，你就直接敲一下你定义的别名或这`whereis + 别名`，看看有没有找到，如果没有找到了，说明不是一个系统命令。



# 常用快捷键功能

-   `Tab`键：命令或文件补全。
-   `ctrl+a`：把光标移动到命令行开头。如果我们输入的命令过长，想要把光标移动到命令行开头时使用。
-   `ctrl+e`：把光标移动到命令行结尾。
-  <font color="#ff0000"> ctrl+c：强制终止当前的命令。</font>
-   `ctrl+l`：清屏，相当于`clear`命令。
-   <font color="#ff0000">ctrl + u：删除或剪切光标之前的命令。我输入了一行很长的命令，不用使用退格键一个一个字符的删除，使用这个快捷键会更加方便。</font>
-   <font color="#ff0000">ctrl + y：粘贴ctrl+u剪切的内容。</font>
-   <font color="#ff0000"><font color="#ff0000">ctrl + d : 退出终端</font></font>
-   <font color="#ff0000">ctrl + r :  在历史命令中搜索，按下ctrl+r后，就会出现搜索界面</font>
-   ctrl + z :  暂停并放入后台
-   ctrl + s :  暂停屏幕输出
-   ctrl + q :  恢复屏幕输出

> 提示：小写即可。这些都是Linux系统的快捷键，每个远程客户端都支持。