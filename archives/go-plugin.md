---
title: Go Plugin 浅析
date: 2023-04-03T10:58:40+08:00
Description:
Tags: 
  - go
Categories:
DisableComments: false
---

`go plugin` 支持将 `go包` 编译为`共享库` 的形式单独发布，主程序可以在运行时动态加载这些编译为动态共享库文件的 `go plugin`，从中提取导出 `变量` 或 ` 函数` 的符号并在主程序的包中使用

`go plugin` 的这种特性为Go开发人员提供更多的灵活性，我们可以用之实现支持热插拔的插件系统。



## 基本使用

go官方文档明确说明 **go plugin只支持Linux, FreeBSD和macOS** ，这算是go plugin的第一个约束。

主程序通过 `plugin` 包加载 `动态库` 并提取 `动态库文件` 中的符号的过程与C语言应用运行时加载动态链接库并调用库中函数的过程如出一辙。下面我们就来看一个直观的例子。

下面是例子的结构布局

```
└─demo01                      
   ├─main.go          主程序       
   ├─pkg              主程序
   │  └─pkg.go   
   ├─plugin           插件包
   │  └─plugin.go    
```

插件代码示例

```go
package main

import (
	"fmt"
	"log"
)

func init() {
	log.Println("plugin init")
}

var PluginInt int

func F() {
	fmt.Printf("plugin: public integer variable PluginInt=%d\n", PluginInt)
}

type private struct{}

func (private) M1() {
	fmt.Println("plugin: invoke private.M1")
}
```

`plugin包` 和普通的`go包` 没太多区别，只是 `plugin包` 有一个约束：其包名必须为 `main`，我们使用下面命令编译该plugin：

```
go build -buildmode=plugin -o plugin.so plugin.go
```

如果 `plugin` 源代码没有放置在 `main包` 下面，我们在编译plugin时会遭遇如下编译器错误：

```
-buildmode=plugin requires exactly one main package
```

接下来，我们来看主程序

```go
package main

import (
	"log"
	"plugins/demo1/pkg"
)

func init() {
	log.Println("main")
}

func main() {
	if err := pkg.LoadPlugin("./plugin/plugin.so"); err != nil {
		panic(err)
	}
}
```

其中 `pkg/pkg.go`文件内容如下

```go
package pkg

import (
	"errors"
	"log"
	"plugin"
)

type MyInterface interface {
	M1()
}

func init() {
	log.Println("pkg init")
}

func LoadPlugin(pluginPath string) error {
	p, err := plugin.Open(pluginPath)
	if err != nil {
		return err
	}

	// 导出整型变量
	pluginInt, err := p.Lookup("PluginInt")
	if err != nil {
		return err
	}
	*pluginInt.(*int) = 15

	// 导出函数变量
	f, err := p.Lookup("F")
	if err != nil {
		return err
	}
	f.(func())()

	// 导出自定义类型变量
	f1, err := p.Lookup("Public")
	if err != nil {
		return err
	}
	i, ok := f1.(MyInterface)
	if !ok {
		return errors.New("f1 does not implement MyInterface")
	}
	i.M1()
	return nil
}
```

通过`plugin` 包提供的 `Plugin` 类型提供的 `Lookup` 方法在加载的动态库中查找相应的导出符号，比如上面的 `PluginInt` 、`F` 和 `Public`等。`Lookup` 方法返回`plugin.Symbol` 类型，而 `Symbol` 类型定义如下

```go
// $GOROOT/src/plugin/plugin.go
type Symbol interface{}
```

我们看到 `Symbol` 的底层类型是`interface{}`，因此它可以承载从 `plugin` 中找到的任何类型的`变量`、`函数` 的符号。而 `plugin` 中定义的类型则是不能被主程序查找的，通常主程序也不会依赖 `plugin` 中定义的类型。

一旦 `Lookup` 成功，我们便可以将符号通过 `类型断言` 获取到其真实类型的实例，通过这些 `实例` (`变量` 或 `函数`)，我们可以调用 `plugin` 中实现的逻辑。编译`plugin` 后，运行上述主程序，我们可以看到如下结果：

```ini
2023/04/03 14:14:48 pkg init
2023/04/03 14:14:48 main
2023/04/03 14:14:48 plugin init
plugin: public integer variable PluginInt=15
plugin: invoke private.M1
```

> 主程序是如何知道导出的符号究竟是函数还是变量呢？
>
> 取决于主程序插件系统的设计，因为主程序与`plugin`间必然要有着某种 `契约` 或 `约定`。
>
> 就像上面主程序定义的 `MyInterface` 接口类型，它就是一个主程序与`plugin`之间的约定，`plugin`中只要暴露实现了该接口的类型实例，主程序便可以通过`MyInterface` 接口类型实例与其建立关联并调用 `plugin` 中的实现 。

## 包的初始化

上面的例子中我们看到，插件的初始化发生在主程序 `open` 动态库文件时。

> 按照官方文档的说法：“**当一个插件第一次被open时，plugin中所有不属于主程序的包的init函数将被调用，但一个插件只被初始化一次，而且不能被关闭**”。

我们来验证一下在主程序中多次加载同一个 `plugin` 的情况

其中 `main.go` 修改为

```go
package main

import (
	"log"
	"plugins/demo1/pkg"
)

func init() {
	log.Println("main")
}

func main() {
	if err := pkg.LoadPlugin("./plugin/plugin.so"); err != nil {
		panic(err)
	}
	log.Println("LoadPlugin ok")

	if err := pkg.LoadPlugin("./plugin/plugin.so"); err != nil {
		panic(err)
	}
	log.Println("ReLoadPlugin ok")
}
```

`pkg/pkg.go`添加包的依赖

```go
package main

import (
	"fmt"
	"log"

	_ "plugins/demo1/pkg"
)
// ....
```

运行上述代码：

```ini
2023/04/03 14:17:46 pkg init
2023/04/03 14:17:46 main
2023/04/03 14:17:46 plugin init
plugin: public integer variable PluginInt=15
plugin: invoke private.M1
2023/04/03 14:17:46 LoadPlugin ok
plugin: public integer variable PluginInt=15
plugin: invoke private.M1
2023/04/03 14:17:46 ReLoadPlugin ok
```

通过这个输出结果，我们验证了两点说法：

- 重复加载同一个plugin，不会触发多次plugin包的初始化，上述结果中**仅输出一次**：`plugin init
- plugin中依赖的包，但主程序中没有的包，在加载plugin时，这些包会被初始化，如：`pkg init`

## 使用约束

`go plugin` 应用不甚广泛的一个主因是其约束较多，这里我们来看一下究竟 `go plugin` 都有哪些约束

- 主程序与plugin的共同依赖包的版本必须一致

- 如果采用mod=vendor构建，那么主程序和plugin必须基于同一个vendor目录构建

- 主程序与plugin使用的编译器版本必须一致

- 使用plugin的主程序仅能使用动态链接

## 版本管理

使用动态链接实现插件系统，一个更大的问题就是插件的版本管理问题。

linux上的动态链接库采用soname的方式进行版本管理。soname的关键功能是它提供了兼容性的标准，当要升级系统中的一个库时，并且新库的soname和老库的soname一样，用旧库链接生成的程序使用新库依然能正常运行。这个特性使得在Linux下，升级使得共享库的程序和定位错误变得十分容易。

什么是soname呢？在 `/lib` 和 `/usr/lib` 等集中放置共享库的目录下，你总是会看到诸如下面的情况：

```ini
lrwxrwxrwx 1 root root    19 11月 15 2021 /usr/lib64/libXxf86vm.so -> libXxf86vm.so.1.0.0
lrwxrwxrwx 1 root root    19 6月  18 2021 /usr/lib64/libXxf86vm.so.1 -> libXxf86vm.so.1.0.0
-rwxr-xr-x 1 root root 23696 8月   2 2017 /usr/lib64/libXxf86vm.so.1.0.0
```

共享库的惯例中每个共享库都有多个名字属性，包括`real name`、`soname`和`linker name`：

- real name 指的是实际包含共享库代码的那个文件的名字(如上面例子中的 `libXxf86vm.so.1.0.0`)，也是在共享库编译命令行中-o后面的那个参数

- `soname` 是 `shared object name` 的缩写，也是这三个名字中最重要的一个，无论是在编译阶段还是在运行阶段，系统链接器都是通过共享库的 ` soname` (如上面例子中的libXxf86vm.so.1)来唯一识别共享库的。

  即使`real name`相同但`soname`不同，也会被链接器认为是两个不同的库。

- `linker name`是编译阶段提供给编译器的名字(如上面例子中的`libXxf86vm`)。如果你构建的共享库的`real name`是类似于上例中`libXxf86vm.so.1.0.0` 那样的带有版本号的样子，那么你在编译器命令中直接使用`-L path -lXxf86vm`是无法让链接器找到对应的共享库文件的，除非你为 `libXxf86vm.so.1.0.0` 提供了一个`linker name`。

  `linker name`一般在共享库安装时手工创建。
