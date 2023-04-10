# golang并发编程之Mutex互斥锁实现同步

除了使用channel实现同步之外，还可以使用Mutex互斥锁的方式实现同步。

```go
package main

import (
	"fmt"
	"sync"
)


var i int=100

var wg sync.WaitGroup     //等待组
 
var lock sync.Mutex
func add(){
	defer wg.Done()     //任务-1
	lock.Lock()         //加锁
	i+=1
	fmt.Printf("i++: %v\n", i)
	lock.Unlock()       //开锁
	
}

func sub(){
	lock.Lock()     //加锁
	defer wg.Done() //任务-1

	i-=1
	fmt.Printf("i--: %v\n", i)
	lock.Unlock()  //开锁
}
func main() {
	for i := 0; i < 100; i++ {
		
		go add()
		wg.Add(1)      //任务j+1
		go sub()
		wg.Add(1)      //任务+1
	}
    wg.Wait()
	fmt.Printf("i: %v\n", i)
} 
```