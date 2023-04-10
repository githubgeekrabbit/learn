# golang中的if语句

**go语言**中的**if语句**和其他语言中的类似，都是根据给定的条件表达式运算结果来，判断执行流程。

## go语言if语句语法

```go
if 布尔表达式 {
   /* 在布尔表达式为 true 时执行 */
}
```

> 注意：在go语言中 布尔表达式不用使用括号。

## go语言if语句实例演示

**根据布尔值flag判断**

```go
import "fmt"

func test1() {
	var flag = true
	if flag {
		fmt.Println("flag is true")
	}
	fmt.Printf("程序运行结束")
}

func main() {
	test1()
}
```

程序运行结果

```go
flag is true
程序运行结束
```

**根据年龄判断是否成年**

```go
package main

import "fmt"

func test2() {
	var age = 20
	if age > 18 {
		fmt.Println("你是成年人")
	}
	fmt.Printf("程序运行结束")
}

func main() {
	test2()
}
```

程序运行结果

```go
你是成年人
程序运行结束
```

> **初始变量可以声明在布尔表达式里面，注意它的作用域**

```go
func test3() {
	if age := 20; age > 18 {
		fmt.Println("你是成年人")
	}
	fmt.Printf("程序运行结束")
}

func main() {
	// test1()
	// test2()
	test3()
}
```

程序运行结果

```
你是成年人
程序运行结束
```

> **注意：不能使用0或非0表示真假**

```go
func test4() {
	var i = 1
	if i { // 编译失败
		fmt.Println("here")
	}
	fmt.Printf("程序运行结束")
}
```

**go语言if语句使用提示：**

1. 不需使用括号将**条件包含起来**
2. 大括号`{}`必须存在，即使只有一行语句
3. 左括号必须在`if`或`else`的同一行
4. 在`if`之后，条件语句之前，可以添加变量初始化语句，使用`；`进行分隔