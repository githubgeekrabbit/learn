# golang字符串

一个Go语言字符串是一个任意**字节的常量序列**。[] byte

# go语言字符串字面量

在Go语言中，字符串字面量使用双引号 `""` 或者反引号 `'` 来创建。双引号用来创建可解析的字符串，支持转义，但不能用来引用多行；反引号用来创建原生的字符串字面量，可能由多行组成，但不支持转义，并且可以包含除了反引号外其他所有字符。双引号创建可解析的字符串应用最广泛，反引号用来创建原生的字符串则多用于书写多行消息，HTML以及正则表达式。

**实例**

```go
package main

import "fmt"

func main() {
	var str1 string = "hello world"
	var html string = 
		<html>
			<head><title>hello golang</title>
		</html>
	`

	fmt.Printf("str1: %v\n", str1)
	fmt.Printf("html: %v\n", html)
}
```

运行结果

```go
str1: hello world
html: 
<html>
	<head><title>hello golang</title>
</html>
```

## go语言字符串连接

**使用加号**

虽然Go语言中的字符串是不可变的，但是字符串支持 `+` 级联操作和`+=`追加操作，例如：

```go
package main

import "fmt"

func main() {
	name := "tom"
	age := "20"
	msg := name + " " + age
	fmt.Printf("msg: %v\n", msg)
	fmt.Println("-------------")
	msg = ""
	msg += name
	msg += " "
	msg += age
	fmt.Printf("msg: %v\n", msg)
}
```

> golang 里面的字符串都是不可变的，每次运算都会产生一个新的字符串，所以会产生很多临时的无用的字符串，不仅没有用，还会给 gc 带来额外的负担，所以性能比较差

**使用`fmt.Sprintf()`函数**

```go
package main

import "fmt"

func main() {
	name := "tom"
	age := "20"
	msg := fmt.Sprintf("%s,%s", name, age)
	fmt.Printf("msg: %v\n", msg)
}
```

运行结果

```go
msg: tom,20
```

> 内部使用 `[]byte` 实现，不像直接运算符这种会产生很多临时的字符串，但是内部的逻辑比较复杂，有很多额外的判断，还用到了 `interface`，所以性能也不是很好

**`strings.Join()`**

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	name := "tom"
	age := "20"
	msg := strings.Join([]string{name, age}, ",")
	fmt.Printf("msg: %v\n", msg)
}
```

运行结果

```
msg: tom,20
```

> join会先根据字符串数组的内容，计算出一个拼接之后的长度，然后申请对应大小的内存，一个一个字符串填入，在已有一个数组的情况下，这种效率会很高，但是本来没有，去构造这个数据的代价也不小

**`buffer.WriteString()`**

```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	var buffer bytes.Buffer
	buffer.WriteString("tom")
	buffer.WriteString(",")
	buffer.WriteString("20")
	fmt.Printf("buffer.String(): %v\n", buffer.String())
}
```

> 这个比较理想，可以当成可变字符使用，对内存的增长也有优化，如果能预估字符串的长度，还可以用 `buffer.Grow()` 接口来设置 capacity

## go语言字符串转义字符

Go 语言的字符串常见转义符包含回车、换行、单双引号、制表符等，如下表所示。

| 转义符 | 含义                               |
| :----- | :--------------------------------- |
| `\r`   | 回车符（返回行首）                 |
| `\n`   | 换行符（直接跳到下一行的同列位置） |
| `\t`   | 制表符                             |
| `\'`   | 单引号                             |
| `\"`   | 双引号                             |
| `\\`   | 反斜杠                             |

**实例**

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Print("hello\tworld\n")
	fmt.Print("\"c:\\test\\\"")
}
```

运行结果

```
hello	world
"c:\test\"
```

## go语言字符串切片操作

```go
package main

import (
	"fmt"
)

func main() {
	str := "hello world"
	n := 3
	m := 5
	fmt.Println(str[n])   //获取字符串索引位置为n的原始字节
	fmt.Println(str[n:m]) //截取得字符串索引位置为 n 到 m-1 的字符串
	fmt.Println(str[n:])  //截取得字符串索引位置为 n 到 len(s)-1 的字符串
	fmt.Println(str[:m])  //截取得字符串索引位置为 0 到 m-1 的字符串
}
```

运行结果

```
108
lo
lo world
hello
```

## go语言字符串常用方法

| 方法                                | 介绍           |
| :---------------------------------- | :------------- |
| len(str)                            | 求长度         |
| +或fmt.Sprintf                      | 拼接字符串     |
| strings.Split                       | 分割           |
| strings.contains                    | 判断是否包含   |
| strings.HasPrefix,strings.HasSuffix | 前缀/后缀判断  |
| strings.Index(),strings.LastIndex() | 子串出现的位置 |
| strings.Join(a[]string, sep string) | join操作       |

**实例**

```go
package main

import (
	"fmt"
	"strings"
)

func main() {
	s := "hello world！"
	fmt.Printf("len(s): %v\n", len(s))
	fmt.Printf("strings.Split(s, \"\"): %v\n", strings.Split(s, " "))
	fmt.Printf("strings.Contains(s, \"hello\"): %v\n", strings.Contains(s, "hello"))
	fmt.Printf("strings.HasPrefix(s, \"hello\"): %v\n", strings.HasPrefix(s, "hello"))
	fmt.Printf("strings.HasSuffix(s, \"world！\"): %v\n", strings.HasSuffix(s, "world！"))
	fmt.Printf("strings.Index(s, \"l\"): %v\n", strings.Index(s, "l"))
	fmt.Printf("strings.LastIndex(s, \"l\"): %v\n", strings.LastIndex(s, "l"))
}
```

运行结果

```
len(s): 14
strings.Split(s, ""): [hello world！]
strings.Contains(s, "hello"): true
strings.HasPrefix(s, "hello"): true
strings.HasSuffix(s, "world！"): true
strings.Index(s, "l"): 2
strings.LastIndex(s, "l"): 9
```

## byte和rune类型

组成每个字符串的元素叫做“字符”，可以通过遍历或者单个获取字符串元素获得字符。 字符用单引号（’）包裹起来，如：

```go
package main

import "fmt"

func main() {
	var a = '华'
	var b = 'a'
	fmt.Printf("a: %v,%c\n", a, a)
	fmt.Printf("b: %v,%c\n", b, b)
}
```

运行结果

```
a: 21326,华
b: 97,a
```

Go 语言的字符有以下两种：

1. `uint8`类型，或者叫 byte 型，代表了`ASCII码`的一个字符。
2. `rune`类型，代表一个 `UTF-8字符`。

当需要处理中文、日文或者其他复合字符时，则需要用到`rune`类型。`rune`类型实际是一个`int32`。

Go 使用了特殊的 rune 类型来处理 Unicode，让基于 Unicode 的文本处理更为方便，也可以使用 byte 型进行默认字符串处理，性能和扩展性都有照顾。