# 并发编程之Timer

Timer顾名思义，就是定时器的意思，可以实现一些定时操作，内部也是通过channel来实现的。

**实例演示**



```go
package main

import (
	"fmt"
	"time"
)

func main() {
	timer1 := time.NewTimer(time.Second * 2)     //等待2秒
	t1 := time.Now()                        //当前时间
	fmt.Printf("t1:%v\n", t1)                //输出当前时间

	t2 := <-timer1.C                         //C是channel  ,阻塞的，等待两秒钟
	fmt.Printf("t2:%v\n", t2)

	//如果只是想单纯的等待的话，可以使用 time.Sleep 来实现
	timer2 := time.NewTimer(time.Second * 2)       //
	<-timer2.C
	fmt.Println("2s后")

	time.Sleep(time.Second * 2)               //单纯睡两秒
	fmt.Println("再一次2s后")

	<-time.After(time.Second * 2) //time.After函数的返回值是chan Time ，程序阻塞两秒钟在执行
	fmt.Println("再再一次2s后")

	timer3 := time.NewTimer(time.Second)       // 等待1秒，在执行
	go func() {
		<-timer3.C                            //阻塞1秒
		fmt.Println("Timer 3 expired")
	}()

	s := timer3.Stop() //停止定时器
	////阻止timer事件发生，当该函数执行后，timer计时器停止，相应的事件不再执行
	if s {
		fmt.Println("Timer 3 stopped")
	}

	fmt.Println("before")
	timer4 := time.NewTimer(time.Second * 5) //原来设置5s
	timer4.Reset(time.Second * 2)            //重新设置时间,即修改NewTimer的时间为2秒
	<-timer4.C                               //阻塞2秒
	fmt.Println("after")
}
```



`t1:2023-03-04 16:51:00.105563968 +0800 CST m=+0.000391367`

`t2:2023-03-04 16:51:02.106433903 +0800 CST m=+2.001261322`

`2s后`

`再一次2s后`

`再再一次2s后`

`Timer 3 stopped`

`before`

`after`