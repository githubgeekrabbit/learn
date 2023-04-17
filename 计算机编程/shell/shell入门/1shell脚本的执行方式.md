
# `#!/bin/Bash`


这个不是注释，这是标称这是一个shell脚本，不能省略


# 脚本执行

-  赋予执行权限，直接运行`
```bash
chmod 755 hello.sh

./hello.sh
```

- 通过Bash调用直接运行
```bash
bash hello.sh
```


# windows和linux下编写的shell格式不一样
![Pasted image 20230402183230](http://qn.qu1u1.cn/202304172310542.png)
在windows下编的直接放到linux直接运行会报错
> 因为windows的回车符识别为`M$`,而linux的回车符为`$`,所以回报这个错

### 可以使用`dos2unix`命令格式把win转linux

```bash
yum install dos2unix

dos2unix  [被转换的文件]
```

### 可以使用`unix2dos`命令格式把linux转windows

```bash
yum install unix2dos

unix2dos  [被转换的文件]
```
