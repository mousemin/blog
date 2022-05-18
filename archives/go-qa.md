---
title: Go 语言笔试面试题
date: 2021-12-15T14:40:52+08:00
Description:
weight: 1
Tags: 
 - go
Categories:
 - 面试题
DisableComments: false
---

日常收集的题

## 一. 基础语法

### 1 `=` 和 `:=` 的区别？

<details>
    <summary><strong>答案</strong></summary>

`:=` 声明+赋值

`=` 仅赋值
```go
var a int
a = 10
// 等价于
a := 10
```
</details>


### 2 Go 有异常类型吗？

<details>
    <summary><strong>答案</strong></summary>

Go 没有异常类型，只有错误类型（Error），通常使用返回值来表示异常状态。

```go
f, err := os.Open("/tmp/a.txt")
if err != nil {
    log.Fatal(err)
}
```
</details>

## 二. 代码输出

### 常量与变量

1. 下面代码输出

```go
func main() {
	const (
		a, b = "golang", 100
		d, e
		f bool = true
		g
	)
	fmt.Println(d, e, g)
}
```

<details>
    <summary><strong>答案</strong></summary>

`golang 100 true`

在同一个 const group 中，如果常量定义与前一行的定义一致，则可以省略类型和值。编译时，会按照前一行的定义自动补全。即等价于

```go
func main() {
	const (
		a, b = "golang", 100
		d, e = "golang", 100
		f bool = true
		g bool = true
	)
	fmt.Println(d, e, g)
}
```

</details>

2. 下面代码输出 
```go
func main() {
	const N = 100
	var x int = N

	const M int32 = 100
	var y int = M
	fmt.Println(x, y)
}
```

<details>
    <summary><strong>答案</strong></summary>

编译失败：cannot use M (type int32) as type int in assignment

Go 语言中，常量分为无类型常量和有类型常量两种，const N = 100，属于无类型常量，赋值给其他变量时，如果字面量能够转换为对应类型的变量，则赋值成功，例如，var x int = N。但是对于有类型的常量 `const M int32 = 100`，赋值给其他变量时，需要类型匹配才能成功，所以显示地类型转换

```go
var y int = int(M)
```

</details>

3. 下面代码输出 
```go
func main() {
	var a int8 = -1
	var b int8 = -128 / a
	fmt.Println(b)
}
```

<details>
    <summary><strong>答案</strong></summary>

`-128`

nt8 能表示的数字的范围是 [-2^7, 2^7-1]，即 [-128, 127]。-128 是无类型常量，转换为 int8，再除以变量 -1，结果为 128，常量除以变量，结果是一个变量。变量转换时允许溢出，符号位变为1，转为补码后恰好等于 -128。

对于有符号整型，最高位是是符号位，计算机用补码表示负数。补码 = 原码取反加一。

例如：

```ini
-1 :  11111111
00000001(原码)    11111110(取反)    11111111(加一)
-128：    
10000000(原码)    01111111(取反)    10000000(加一)

-1 + 1 = 0
11111111 + 00000001 = 00000000(最高位溢出省略)
-128 + 127 = -1
10000000 + 01111111 = 11111111
```


</details>

4. 下面代码输出 
```go
func main() {
	const a int8 = -1
	var b int8 = -128 / a
	fmt.Println(b)
}
```

<details>
    <summary><strong>答案</strong></summary>

编译失败：constant 128 overflows int8

-128 和 a 都是常量，在编译时求值，-128 / a = 128，两个常量相除，结果也是一个常量，常量类型转换时不允许溢出，因而编译失败。

</details>

### 切片

1. 下面代码输出

```go
package main

import "fmt"

func main() {
	a := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	b := a[2:5]
	c := a[9:10]
	b = append(b, 1)
	c = append(c, 1)
	fmt.Println(a)
}
```
<details>
    <summary><strong>答案</strong></summary>

`[0 1 2 3 4 1 6 7 8 9]`

</details>