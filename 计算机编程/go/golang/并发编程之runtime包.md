# golang并发编程之runtime包

runtime包里面定义了一些协程管理相关的api

## runtime.Gosched()

让出CPU时间片，重新等待安排任务



```go
package main

import (
	"fmt"
	//"runtime"
)


func show(s string){
	for i := 0; i < 2; i++{
		fmt.Println(s)
	}		

}	

func main() {
	go show("java")
	for i := 0; i < 2; i++ {
   runtime.Gosched()       //让出cpu的时间片，让其他子协程来执行
	 fmt.Println("golang")
		
	}
	
}
```





## runtime.Goexit()

退出当前协程

```go
package main

import (
	"fmt"
	"time"
	"runtime"
)



func show(){
	for i := 0; i < 10; i++ {
		if i >= 5 {
			runtime.Goexit()     //退出当前协程
		
		}
		fmt.Printf("i: %v\n", i)
	}
	
}
func main() {
	go show()
	time.Sleep(time.Second)
}
```





`i: 0`

`i: 1`

`i: 2`

`i: 3`

`i: 4`



## runtime.GOMAXPROCS

```golang
package main

import (
	"fmt"
	"runtime"
	"time"
)

func a() {
	for i := 1; i < 30; i++ {
		fmt.Println("A:", i)
	}
}

func b() {
	for i := 1; i < 20; i++ {
		fmt.Println("B:", i)
	}
}

func main() {
	fmt.Printf("runtime.NumCPU(): %v\n", runtime.NumCPU())  //输出cpu线程数
	runtime.GOMAXPROCS(2) //最大cpu线程数是1 修改为1查看效果
	go a()
	go b() 
	time.Sleep(time.Second)
}
```

