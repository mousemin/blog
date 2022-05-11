---
title: Go内嵌静态资源
date: 2021-10-11T14:47:43+08:00
Description:
Tags: 
 - go
Categories:
DisableComments: false
---

把静态资源嵌入在程序里，原因无外乎以下几点：
- 布署程序更简单。传统部署要么需要把静态资源和编译好的程序一起打包上传，要么使用docker和dockerfile自动化.
- 保证程序完整性。运行中发生静态资源损坏或丢失往往会影响程序的正常运行.
- 可以自主控制程序需要的静态资源.

最常见的,比如一个混编网址的后端程序,本来需要把程序与它所需要的静态资源(html模版、css、js、图片)一起上传至生产服务器,同时还需要正确配置静态资源在服务器中的路径让程序能正常访问.现在我们将这些资源全部嵌入到程序中,部署的时候只需要部署一个`二进制`文件,配置也只针对这个程序本身,部署的流程大大简化.

## go 1.16前如何内嵌静态资源

在`go 1.16`之前, 我们需要借助第三方工具来实现. 这些工具都是借助代码生成来完成资源的嵌入. 我们拿 [go-bindata] 举例.

- [go-bindata]
- [pkger](https://github.com/markbates/pkger)

[go-bindata]: https://github.com/go-bindata/go-bindata

首先我们创建一个项目：

```bash
mkdir embed-demo && cd embed-demo
go mod init embed/demo
# 安装打包工具
go get -u github.com/go-bindata/go-bindata/... 
```

然后我们复制一个png图片进images文件夹，整个项目看起来如下：

![image-20211011151413667](https://cdn.mousemin.com/img/2021-10-15-ecd057a852a0a440b0262372abd80626.png)

然后是我们的代码

```go
package main

import "log"

//go:generate go-bindata -fs -nomemcopy -pkg=main -ignore="\\.DS_Store|less" -prefix=./images -debug=false -o=images_gen.go ./images/...

func main() {
	fp, err := Asset("2233.png")
	if err != nil {
		log.Fatal(err)
		return
	}
	log.Print(len(fp))
}
```

想要完成资源嵌入,我们需要运行 `go generate` 命令, 之后直接运行 `go build` 即可, 顺利运行后项目如下:

![image-20211011152308163](https://cdn.mousemin.com/img/2021-10-15-15c0a5455ce6bdc23befb4090221bb74.png)

`go-bindata` 的思路就是将资源文件编码成合法的golang源文件，然后利用golang把这些代码化的资源编译进程序里。这是比较主流的嵌入资源实现方案。

从上面的例子我们可以看出这类方法有不少缺点:

- 需要安装额外工具
- 会生成超大体积的生产代码(是静态文件的3倍多, 因为需要对二进制文件进行一定的编码才能正常存储在go源文件中)
- 编译完成的程序体积也锁资源文件的两倍多

## golang1.16的官方内置版静态资源

想要嵌入静态资源，首先我们得利用`embed`这个新的标准库。在声明静态资源的文件里我们需要引入这个库。

对于我们想要嵌入进程序的资源，需要使用`//go:embed`指令进行声明，注意`//`之后不能有空格。具体格式如下

```go
//go:embed pattern
// pattern是path.Match所支持的路径通配符
```

具体的通配符如下，

| 通配符        | 释义                                                         |
| ------------- | ------------------------------------------------------------ |
| ?             | 代表任意一个字符（不包括半角中括号）                         |
| *             | 代表0至多个任意字符组成的字符串（不包括半角中括号）          |
| [...]和[!...] | 代表任意一个匹配方括号里字符的字符，!表示任意不匹配方括号中字符的字符 |
| [a-z]、[0-9]  | 代表匹配a-z任意一个字符的字符或是0-9中的任意一个数字         |
| **            | 部分系统支持，*不能跨目录匹配，**可以，不过目前个golang中和*是同义词 |

我们可以在 `embed` 的 `pattern` 里自由组合这些通配符。

golang的embed默认的根目录从module的目录开始，路径开头不可以带`/`，不管windows还是其他系统路径分割副一律使用`/`。如果匹配到的是目录，那么目录下的所有文件都会被嵌入（有部分文件夹和文件会被排除，后面详细介绍），如果其中包含有子目录，则对子目录进行递归嵌入。

下面举一些例子，假设我们的项目在`/data/project`：

```go
//go:embed resources
这是匹配所有位于/data/project/resources及其子目录中的文件

//go:embed resources/images/2233.png
匹配/data/project/resources/images/2233.png这一个文件

//go:embed a.txt
匹配/data/project/a.txt

//go:embed resources/js/*.min.js
匹配/data/project/resources/js/下所有 `.min.js` 文件

//go:embed /data/project/resources/images/a?.jpg
匹配/data/project/resources/images/下的 `a1.jpg` `a2.jpg` `ab.jpg`等

//go:embed resources/images/*.*
/data/project/resources/images/*.*的文件夹里的所有有后缀名的文件，例如2233.png jpg/a.jpeg

//go:embed *
直接匹配整个/data/project

//go:embed a.txt
//go:embed *.png *.jpg
//go:embed aa.jpg
可以指定多个//go:embed指令行，之间不能有空行，也可以用空格在一行里写上对个模式匹配，表示匹配所有这些文件，相当于并集操作
可以包含重复的文件或是模式串，golang对于相同的文件只会嵌入一次，很智能
```

另外，通配符的默认目录和源文件所在的目录是同一目录，所以我们只能匹配同目录下的文件或目录，不能匹配到父目录。举个例子：

![image-20211011161931857](https://cdn.mousemin.com/img/2021-10-16-8d41387e96d4aa59aa9b109b63bc7adc.png)

考虑如上的目录结构。 `code/main.go`可见资源只有 `code` 目录及其子目录里的文件, 而`resources`里的文件是无法匹配的.

### 如何使用嵌入的静态资源

对于一个完整的嵌入资源,代码中的申明是这样的

```go
//go:embed resources/images/**
var images embed.FS

//go:embed resources/css/bootstrap.css
var css []byte

//go:embed resources/texts/zh.txt
var txt string
```

一共有三种数据格式可选：

| 数据类型 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| []byte   | 表示数据存储为二进制格式，如果只使用[]byte和string需要以`import (_ "embed")`的形式引入embed标准库 |
| string   | 表示数据被编码成utf8编码的字符串，因此不要用这个格式嵌入二进制文件比如图片，引入embed的规则同[]byte |
| embed.FS | 表示存储多个文件和目录的结构，[]byte和string只能存储单个文件 |

实际上接受嵌入文件数据的变量也可以是string和[]byte的类型别名或基于他们定义的新类型，例如下面的代码那样：

```
type StringAlias = string

//go:embed resources/texts/zh.txt
var txt StringAlias

type NewBytes []byte

//go:embed resources/css/bootstrap.css
var css NewBytes
```

这一变化是[issue 43602](https://github.com/golang/go/issues/43602)中提出的，并在[commit ec94701](https://github.com/golang/go/commit/ec9470162f26819abd7b7bb86dd36cfe87f7f5bc)中实现。

下面我们看个更具体例子，目录结构如下：

![image-20211011161931857](https://cdn.mousemin.com/img/2021-10-16-8d41387e96d4aa59aa9b109b63bc7adc.png)

目录包含了一些静态图片、`javascript`、`css`文件，一个国际化的文件。当然还有我们的测试代码。

### 单个文件

我们先来看用`[]byte`和`string`嵌入单个文件的例子：

```go
package main

import (
	_ "embed"
	"log"
)

//go:embed resources/css/bootstrap.css
var css []byte

//go:embed resources/texts/zh.txt
var txt string

func main() {
	log.Println(len(css)) // bootstrap.css文件的总字符数
	log.Panicln(txt)
}
```

如你所见，声明嵌入内容的变量一定要求使用`var`声明。

我们直接用`go run main.go`或`go build main.go && ./main`即可完成编译运行，过程中不会生成任何中间代码。另外变量是否是公开的（首字母是否大小写）并不会对资源的嵌入产生影响。



在[issue 43216](https://github.com/golang/go/issues/43216)中，基于如下的矛盾golang取消了对本地作用域变量的嵌入资源声明的支持：

- 如果嵌入资源只初始化一次，那么每次函数调用都将共享这些资源，考虑到任何函数都可以作为goroutine运行，这会带来严重的潜在风险；
- 如果每次函数调用时都重新初始化，这样做会产生昂贵的性能开销。



因此最后golang官方在[commit 54198b0](https://github.com/golang/go/commit/54198b04dbdf424d8aec922c1f8870ce0e9b7332)中关闭了本地作用域的静态资源嵌入功能。现在你的代码应该这样写：

```go
//go:embed resources/texts/zh.txt
var txt string

func Print() {
	// //go:embed resources/texts/zh.txt
	// var txt string
}
```

再来看看二进制文件的例子，`main.go`如下所示：

```go
package main

import (
	_ "embed"
	"log"
)

//go:embed resources/images/2233.png
var image []byte

func main() {
	log.Println(len(image))
}
```

如果编译运行这个程序，你会发现二进制文件的大小是`4.5M`（不同系统会有差异），比我们之前使用`go-bindata`创建的要小了许多。



### 多个文件和目录

如果你 `go doc embed` 的话会发现整个标准库里只有一个 `FS` 类型（之前按提案被命名为Files，后来考虑到用目录结构组织多个资源更类似新的io/fs.FS接口，故改名），而我们对静态资源的操作也全都依赖这个`FS`。下面接着用例子说明：

```go
package main

import (
	"embed"
	"log"
)

//go:embed resources/texts/**
var txts embed.FS

func main() {
	zh, err := txts.ReadFile("resources/texts/zh.txt")
	if err != nil {
		log.Fatal("read zh.txt error:", err)
	} else {
		log.Println("zh.txt: ", string(zh))
	}
}

```

运行结果：

```
2021/10/11 17:26:26 zh.txt:  other = "分类"
```

我们想读取单个文件需要用 `ReadFile` 方法，它接受一个path字符串做参数，从中查找对应的文件然后返回 `([]byte, error)`。

要注意的是文件路径必须要明确写出自己的父级目录，否则会报错，*因为嵌入资源是按它存储路径相同的结构存储的，和通配符怎么指定无关*。

`Open`是和ReadFile类似的方法，只不过返回了一个`fs.File`类型的`io.Reader`，因此这里就不再赘述，需要使用`Open`还是`ReadFile`可以由开发者根据自身需求决定。

`embed.FS`自身是只读的，所以我们不能在运行时添加或删除嵌入的文件，`fs.File`也是只读的，所以我们不能修改嵌入资源的内容。

如果只是提供了一个查找读取资源的能力，那未免小看了embed。在golang1.16里任意实现了`io/fs.FS`接口的类型都可以表现的像是真实存在于文件系统中的目录一样，哪怕它其实是在内存里的类map数据结构。因此我们也可以像遍历目录一样去处理`embed.FS`:

```go
package main

import (
	"embed"
	"log"
)

//go:embed resources/images/**
var images embed.FS

func main() {
	dirs, err := images.ReadDir("resources/images")
	if err != nil {
		log.Fatal(err)
	}
	for _, dir := range dirs {
		info, _ := dir.Info()
		log.Println("filename: ", info.Name(), "\t isDir: ", info.IsDir(), "\tsize: ", info.Size())
	}
}
```

运行结果：

```ini
2021/10/11 17:23:21 filename:  026b232abe59d6e2bce9513cefedc5f70c4db615.jpg      isDir:  false  size:  5391201
2021/10/11 17:23:21 filename:  2233.png          isDir:  false  size:  2663844
2021/10/11 17:23:21 filename:  ae8b83d9598534fa43f791a5ac688fecf0253009.jpg      isDir:  false  size:  3467751
2021/10/11 17:23:21 filename:  gkypn.jpg         isDir:  false  size:  78569
```

唯一和真实的目录不一样的地方是目录文件的大小，在ext4等文件系统上目录会存储子项目的元信息，所以大小通常不为0。

如果想要内嵌整个module，则在引用的时候需要使用`"."`这个名字，但除了单独使用之外路径里不可以包含`..`或者`.`，换而言之，`embed.FS`不支持相对路径，把上面的代码稍加修改：

```go
package main

import (
	"embed"
	"log"
)

//go:embed *
var files embed.FS

func main() {
	dirs, err := files.ReadDir(".")
	if err != nil {
		log.Fatal(err)
	}
	for _, dir := range dirs {
		info, _ := dir.Info()
		log.Println("filename: ", info.Name(), "\t isDir: ", info.IsDir(), "\tsize: ", info.Size())
	}
}
```

程序输出

```ini
2021/10/11 17:28:59 filename:  .DS_Store         isDir:  false  size:  6148
2021/10/11 17:28:59 filename:  code      isDir:  true   size:  0
2021/10/11 17:28:59 filename:  go.mod    isDir:  false  size:  27
2021/10/11 17:28:59 filename:  main      isDir:  false  size:  4707136
2021/10/11 17:28:59 filename:  main.go   isDir:  false  size:  310
2021/10/11 17:28:59 filename:  resources         isDir:  true   size:  0
```

因为使用了错误的文件名或路径会在运行时`panic`，所以要格外小心。（当然 `//go:embed` 是在编译时检查的，而且同样不支持相对路径，同时也不支持超出了module目录的任何路径，比如`go module`在`/data/project`，我们指定了`/data/project2`）

### 一些陷阱

方便的功能背后往往也会有陷阱相随，golang的内置静态资源嵌入也不例外。

#### 隐藏文件的处理

根据2020年11月21日的[issue](https://github.com/golang/go/issues/42328)，现在golang在对目录进行递归嵌入的时候会忽略名字以下划线（_）和点（.）开头的文件或目录。这些文件名在部分文件系统中为隐藏文件，issue的提出者认为默认不应该包含这些文件，隐藏文件通常包含对程序来说没有意义的元数据，或是用户的隐私配置，除非明确声明，否则嵌入资源中包含隐藏文件是不妥的。

举个例子，假设我们有个images文件夹，底下有`a.jpg`，`.b.jpg`两个常规文件，以及`_imgs`和`imgs`两个子目录，根据[commit](https://github.com/golang/go/commit/37588ffcb221c12c12882b591a16243ae2799fd1)，以下的嵌入资源指令的效果如注释中的解释：

```go
//go:embed images
var images embed.FS // 不包含.b.jpg和_imgs目录

//go:embed images/*
var images embed.FS // 注意！！！ 这里包含.b.jpg和_imgs目录

//go:embed images/.b.jpg
var bJPG []byte // 明确给出文件名也不会被忽略
```

注意第二条。使用`*`相当于明确给出了目录下所有文件的名字，因此点和下划线开头的文件和目录也会被包含。

当然，隐藏文件不止文件名特殊这么简单，在部分文件系统上拥有正常文件名的文件通过增加某些flag或者attribute也可以变为隐藏，目前怎么处理此类情况还没有定论。官方暂且按照社区的习惯使用文件名进行区分。

另外对于`*`是否应该包含隐藏文件的争论也没有停止，官方暂且认为应该包含隐藏文件，这点要多加注意。

#### 资源是否应该被压缩

静态资源嵌入的提案被接受后争论最多的就是是否应该对资源采取压缩，压缩后的资源更紧凑，不会浪费太多存储空间，特别是一些大文本文件。同时更大的程序运行加载时间越长，cpu缓存利用率可能会变低。

而反对意见认为压缩和运行时的解压一个浪费编译的时间一个浪费运行时的效率，在用户没有明确指定的情况下用户需要为自己不需要的功能花费代价。

目前官方采用的实现是不压缩嵌入资源，并预计在后续版本加入控制是否启用压缩的选项。

### 会被忽略的目录

前面说过，embed会递归处理目录，出来以下的几个：

- `.bzr`
- `.hg`
- `.git`
- `.svn`

这些都是版本控制工具的目录，资源里理应不包含他们，因此是被忽略的。会被忽略的目录列在`src/cmd/go/internal/load/pkg.go`的`isBadEmbedName`函数里`(line: 2094)`。

> 注意: `.idea`不在此列

另外不像隐藏文件可以明确指定嵌入，这些目录你是无法用任何正常手段嵌入的，golang都会忽略他们。

## 参考资料

- [https://go.googlesource.com/proposal/+/master/design/draft-embed.md](https://go.googlesource.com/proposal/+/master/design/draft-embed.md)
- [https://github.com/golang/go/issues/41191](https://github.com/golang/go/issues/41191)

- [https://github.com/mattn/go-embed-example](https://github.com/mattn/go-embed-example)
