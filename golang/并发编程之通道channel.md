# golang并发编程之通道channel

Go 提供了一种称为通道的机制，用于在 goroutine 之间**共享数据**。当您作为 goroutine 执行并发活动时，需要在 goroutine 之间共享资源或数据，通道充当 goroutine 之间的管道（管道）并提供一种机制来保证同步交换。

需要在声明通道时指定**数据类型**。我们可以共享内置、命名、结构和引用类型的值和指针。数据在通道上传递：在任何给定时间只有一个 goroutine 可以访问数据项：因此按照设计不会发生数据竞争。

根据数据交换的行为，有两种类型的通道：无缓冲通道和缓冲通道。无缓冲通道用于执行 goroutine 之间的同步通信，而缓冲通道用于执行异步通信。无缓冲通道保证在发送和接收发生的瞬间执行两个 goroutine 之间的交换。缓冲通道没有这样的保证。

**通道由 make 函数创建，该函数指定 chan 关键字和通道的元素类型。**

### 这是创建无缓冲和缓冲通道的代码块：

### 语法

```jsx
Unbuffered := make(chan int) // 整型无缓冲通道
buffered := make(chan int, 10)	// 整型有缓冲通道
```

使用内置函数`make`创建无缓冲和缓冲通道。`make`的第一个参数需要关键字`chan`，然后是通道允许交换的数据类型。

### 这是将值发送到通道的代码块需要使用 `<-` 运算符：

### 语法

```jsx
goroutine1 := make(chan string, 5) // 字符串缓冲通道
goroutine1 <- "Australia" // 通过通道发送字符串
```

一个包含 5 个值的缓冲区的字符串类型的 goroutine1 通道。然后我们通过通道发送字符串“Australia”。

### 这是从通道接收值的代码块：

### 语法

```jsx
data := <-goroutine1 // 从通道接收字符串
```

`<-` 运算符附加到通道变量（goroutine1）的左侧，以接收来自通道的值。

## 无缓冲通道

在无缓冲通道中，在接收到任何值之前没有能力保存它。在这种类型的通道中，发送和接收 goroutine 在任何发送或接收操作完成之前的同一时刻都准备就绪。如果两个 goroutine 没有在同一时刻准备好，则通道会让执行其各自发送或接收操作的 goroutine 首先等待。同步是通道上发送和接收之间交互的基础。没有另一个就不可能发生。

## 缓冲通道

在缓冲通道中，有能力在接收到一个或多个值之前保存它们。在这种类型的通道中，不要强制 goroutine 在同一时刻准备好执行发送和接收。当发送或接收阻塞时也有不同的条件。只有当通道中没有要接收的值时，接收才会阻塞。仅当没有可用缓冲区来放置正在发送的值时，发送才会阻塞。

## 通道的发送和接收特性

1. 对于同一个通道，发送操作之间是互斥的，接收操作之间也是互斥的。
2. 发送操作和接收操作中对元素值的处理都是不可分割的。
3. 发送操作在完全完成之前会被阻塞。接收操作也是如此。

### 实例

```jsx
package main

import (
	"fmt"
	"math/rand"
	"time"
)



var values=make(chan int) //创建int类型创建了无缓冲的 channel，只能传入int类型值

func send(){
	rand.Seed(time.Now().UnixNano())  //rand获取随机数，rand.Seed设置随机数种子，加上这行代码，可以保证每次随机都是随机的
	value:=rand.Intn(100)             //rand.Intn(100),从100里面选个随机数赋值给value
	fmt.Printf("value: %v\n", value)  //输出value的值
	// time.Sleep(time.Second*5)
	values <- value       //把值放到通道里
}

func main() {
	defer close(values)        //defer延迟接收，关闭通道values,只有主函数退出时才会执行关闭通道
	go send()                  //启动协程
	fmt.Println("wait...")
	value:= <-values            //从通道接收字符
    fmt.Printf("receive: %v\n", value)
	fmt.Println("end...")
}

```



![image-20230301083441025](https://raw.githubusercontent.com/githubgeekrabbit/image/master/202303010834057.png)