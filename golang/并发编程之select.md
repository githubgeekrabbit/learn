# 并发编程之select

1. select是Go中的一个控制结构，类似于`switch`语句，用于处理异步IO操作。`select`会监听case语句中channel的读写操作，当case中channel读写操作为非阻塞状态（即能读写）时，将会触发相应的动作。

   > select中的case语句必须是一个channel操作
   >
   > select中的default子句总是可运行的。

2. 如果有多个`case`都可以运行，`select`会随机公平地选出一个执行，其他不会执行。

3. 如果没有可运行的`case`语句，且有`default`语句，那么就会执行`default`的动作。

4. 如果没有可运行的`case`语句，且没有`default`语句，`select`将阻塞，直到某个`case`通信可以运行

## 实例

```go
package main
import (
	"fmt"
	"time"	
)

var chanint =make(chan int,0)
var chanstr =make(chan string,0)
func main() {
	go func ()  {
		chanint <-100
		chanstr <-"hello"
		defer close(chanint)     //延迟关闭通道
		defer close(chanstr)     //延迟关闭通道， 刚主函数运行玩才会执行
	}()
	for{
		select{                     //select会随机选择一个case来执行
			case r:=<-chanint:
			fmt.Printf("r: %v\n", r)
			case r:=<-chanstr:
			fmt.Printf("r: %v\n", r)
			default:                  //没有可运行的case语句，且有default语句，那么就会执行default的动作
				fmt.Println("defert.....")    
		}
		time.Sleep(time.Second)      //等待1秒
	}
	
}
```





`defert.....`

`r: 100`

`r: hello`

`r:` 



