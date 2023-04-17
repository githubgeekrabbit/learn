```go
package main
import "fmt"
func main() {
	c:=make(chan int)
	go func(){
		for i := 0; i < 2; i++ {       //循环2次
			c<- i
			
		}
		
}() //自己调用自己

	

r:=<-c 
fmt.Printf("r: %v\n", r)    
r =<-c 
fmt.Printf("r: %v\n", r)
	
}
```

`r: 0`

`r: 1`



```go
package main
import "fmt"
func main() {
	c:=make(chan int)
	go func(){
		for i := 0; i < 2; i++ {       //循环2次
			c<- i
			
		}
		
}() //自己调用自己

	
r:=<-c 
fmt.Printf("r: %v\n", r)    
r =<-c 
fmt.Printf("r: %v\n", r)
r =<-c 
fmt.Printf("r: %v\n", r)
	
}
```



输出结果： 产生了死锁

`r: 0`

`r: 1`

`fatal error: all goroutines are asleep - deadlock!`



`goroutine 1 [chan receive]:`

`main.main()`

​    `/root/golang/并发编程之channel的遍历.go:21 +0x165`

`exit status 2`





```go
package main
import "fmt"
func main() {
	c:=make(chan int)
	go func(){
		for i := 0; i < 2; i++ {       //循环2次
			c<- i
			
		}
		close(c)     //关闭通道	
}() //自己调用自己

r:=<-c 
fmt.Printf("r: %v\n", r)    
r =<-c 
fmt.Printf("r: %v\n", r)
r =<-c 
fmt.Printf("r: %v\n", r) 	
}
```



`r: 0`

`r: 1`

`r: 0`





## 方法2 for range

```golang
package main

import "fmt"

func main() {
	c:=make(chan int)
	go func(){
		for i := 0; i < 5; i++ {       //循环2次
			c<- i
			
		}
		close(c)     //关闭通道,不关闭通道容易产生死锁	
}() //自己调用自己

	
for v:=range c {             //range
	fmt.Printf("v: %v\n", v)
}
   

}
```

