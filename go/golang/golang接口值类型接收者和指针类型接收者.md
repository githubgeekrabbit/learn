# golang接口值类型接收者和指针类型接收者

这个话题，本质上和方法的值类型接收者和指针类型接收者，的思考方法是一样的，值接收者是一个拷贝，是一个副本，而指针接收者，传递的是指针。

```go
package main

import "fmt"

type pet interface{            //声明pet接口
	eat()                      //方法名称eat
}

type dog struct{              //声明结构体
	name string
}

/* 实现接口方法 */
func(d dog)eat(){                                
	fmt.Printf("dog: %v\n", d)
	fmt.Println("dog eat")
	d.name="黑黑"
}

func main() {
	d:= dog{name: "花花"}
	fmt.Printf("dog: %v\n", d)
	d.eat()
	fmt.Printf("dog: %v\n", d)
	
}
```





CentOS 7版本后防火墙默认使用的是firewalld，我们可通过运行以下命令来查看防火墙状态以及关闭防火墙。

1、查看CentOS 7防火墙状态

systemctl status firewalld.service
运行上述命令后，如果看到有绿色字样标注的“active（running）”，说明防火墙是开启状态。

2、关闭运行的防火墙

systemctl stop firewalld.service
关闭后，可查看防火墙状态，当显示disavtive（dead）的字样，说明CentOS 7防火墙已经关闭。

但要注意的是，上面的命令只是临时关闭了CentOS 7防火墙，当重启操作系统后，防火墙服务还是会再次启动。如果想要永久关闭防火墙则还需要禁用防火墙服务。

3、禁用防火墙服务

systemctl disable firewalld.service

