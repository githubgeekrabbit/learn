# 方法接收者类型

结构体实例，有值类型和指针类型，那么方法的接收者是结构体，那么也有值类型和指针类型。区别就是接收者是否复制结构体副本。值类型复制，指针类型不复制。

## **值类型结构体和指针类型结构体**

```go
package main

import "fmt"

type xbt struct{                       //声明小白兔结构体
	name string
}

func dbt(per xbt)  {                  //声明方法dbt,per是接收该方法的结构体，xbt是结构体
	fmt.Printf("per: %p\n", &per)
	per.name="小黑兔"                  //给结构体成员赋值
	fmt.Printf("per: %v\n", per)
	
}
func lbt(per *xbt){                   //声明方法lbt,per是接收该方法的结构体，*xbt是结构体指针
	fmt.Printf("per: %p\n", per)
	per.name="老黑兔"
	fmt.Printf("per: %v\n", per)
}

func main() {
	x1:=xbt{name:"geekrabbit"}        //给短变量x1赋值为结构体xbt，并给结构体赋值(geekrabbit)
	fmt.Printf("x1: %p\n", &x1)       //输出x1的地址
	dbt(x1)                           //调用方法dbt
	fmt.Printf("x1: %v\n", x1)        //输出x1的值
	fmt.Println("--------")           
	x2:=&xbt{name:"geekrabbit"}       //给短变量赋值为结构体的地址，并为结构体赋值为（geekrabbit)
	fmt.Printf("x2: %p %v\n", x2,x2)        //输x2的地址
	lbt(x2)                           //调用方法lbt
	fmt.Printf("x2: %v\n", x2)        //输出x2d的值
}
```

运行结果：

`x1: 0xc00009e210`

`per: 0xc00009e220`

`per: {小黑兔}`

`x1: {geekrabbit}`

`\--------`

`x2: 0xc00009e250 &{geekrabbit}`

`per: 0xc00009e250`

`per: &{老黑兔}`

`x2: &{老黑兔}`



从运行结果，我们看到x1是值传递，拷贝了副本，地址发生了改变，而x2是指针类型，地址没有改变。