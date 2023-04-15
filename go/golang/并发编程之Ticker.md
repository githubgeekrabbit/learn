# 并发编程之Ticker

Timer只执行一次，Ticker可以周期的执行。

**实例**

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ticker:=time.NewTicker(time.Second)    //定义ticker
	           
	for _ = range ticker.C {              //每过一秒钟循环一次
		fmt.Println("ticker 1")           
		
	}
	
}
```

一直循环运行，1秒输出一次，需要手动结束运行

`ticker 1`

`ticker 1`

`ticker 1`

`ticker 1`





```golang
package main

import (
	"fmt"
	"time"
)

func main() {
	ticker:=time.NewTicker(time.Second)    //1秒一周期
	    counter:=1                         
	for _ = range ticker.C {              //每过一秒钟循环一次
		fmt.Println("ticker 1")           
		counter++                         //循环一次counter+1
	
	if counter>=5{                        //当counter>=5的时候
		ticker.Stop()                     //周期结束
		break                             //退出循环
	}
	
}

} 
```





`ticker 1`

`ticker 1`

`ticker 1`

`ticker 1`





```golang
package main

import (
	"fmt"
	"time"
)

func main() {
	ticker:=time.NewTicker(time.Second)    //1秒一周期
	chanInt:=make(chan int)       //声明管道
	go func ()  {
		for _ = range ticker.C {      //1秒循环1次
			select {                  //select随机挑选case
			case chanInt <- 1:        //向管道写入1 
			case chanInt <- 2:        //2
			case chanInt <- 3:
			}
	}
	
}()                                  //自己调用自己
     
	 sum:=0                     
    for v:=range chanInt{            //从管道里面取数据给v
		fmt.Printf("收到: %v\n", v)
		sum+=v                      //sum=sum+v
		if sum>=19{                 //当sum	的值>=19
			break                   //退出循环
		}
	}
  
}
```



运行结果随机

`收到: 2`

`收到: 1`

`收到: 2`

`收到: 3`

`收到: 2`

`收到: 3`

`收到: 3`

`收到: 1`

`收到: 2`