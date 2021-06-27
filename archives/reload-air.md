---
title: Air实现Go程序实时热重载
date: 2021-06-27T12:35:50+08:00
Description:
Tags: 
Categories:
 - 工具
DisableComments: false
---

Air能够实时监听项目的代码文件，在代码发生变更之后自动重新编译并执行

## 为什么需要实时加载？

在使用Go语言在本地做web开发调试的时候，经常需要在修改代码之后频繁的按下`Crtl+C`停止程序并重新编译执行，这样就比较麻烦

## Air介绍

怎样才能在基于web开发实现实时加载功能呢？
Github上有一个现成的工具: [Air](https://github.com/cosmtrek/air)。它支持以下特性:

- 彩色日志输出
- 自定义构建或二进制命令
- 支持忽略子目录
- 启动后支持监听新目录
- 更好的构建过程

### 安装Air

**Go**

这也是最经典的安装方式：

```bash
go get -u github.com/cosmtrek/air
```

**Mac、Linux、Window**

```bash
# binary will be $(go env GOPATH)/bin/air
curl -sSfL https://raw.githubusercontent.com/cosmtrek/air/master/install.sh | sh -s -- -b $(go env GOPATH)/bin
 
# or install it into ./bin/
curl -sSfL https://raw.githubusercontent.com/cosmtrek/air/master/install.sh | sh -s
```

**Docker**

```bash
docker run -it --rm \
    -w "<PROJECT>" \
    -e "air_wd=<PROJECT>" \
    -v $(pwd):<PROJECT> \
    -p <PORT>:<APP SERVER PORT> \
    cosmtrek/air
    -c <CONF>
```

然后按照下面的方式在docker中运行你的项目：

```bash
docker run -it --rm \
    -w "/go/src/github.com/cosmtrek/hub" \
    -v $(pwd):/go/src/github.com/cosmtrek/hub \
    -p 9090:9090 \
    cosmtrek/air
```

### 使用Air

为了敲命令时更简单更方便，你应该把**`alias air='~/.air'`**加到你的**`.bashrc`**或**`.zshrc`**中。

首先进入你的项目目录：

```bash
cd /path/to/your_project
```

最简单的用法就是直接执行下面的命令：

```bash
# 首先在当前目录下查找 `.air.toml`配置文件，如果找不到就使用默认的
air -c .air.toml
```

推荐的使用方法是：

```bash
# 1. 在当前目录创建一个新的配置文件.air.toml
touch .air.toml
 
# 2. 复制 `air_example.toml` 中的内容到这个文件，然后根据你的需要去修改它
 
# 3. 使用你的配置运行 air, 如果文件名是 `.air.toml`，只需要执行 `air`。
air
```

### air_example.toml完整示例

`air_example.toml`示例配置如下，可以根据自己的需要修改。

```toml
# [Air](https://github.com/cosmtrek/air) TOML 格式的配置文件
 
# 工作目录
# 使用 . 或绝对路径，请注意 `tmp_dir` 目录必须在 `root` 目录下
root = "."
tmp_dir = "tmp"
 
[build]
# 只需要写你平常编译使用的shell命令。你也可以使用 `make`
# Windows平台示例: cmd = "go build -o tmp\main.exe ."
cmd = "go build -o ./tmp/main ."
# 由`cmd`命令得到的二进制文件名
# Windows平台示例：bin = "tmp\main.exe"
bin = "tmp/main"
# 自定义执行程序的命令，可以添加额外的编译标识例如添加 GIN_MODE=release
# Windows平台示例：full_bin = "tmp\main.exe"
full_bin = "APP_ENV=dev APP_USER=air ./tmp/main"
# 监听以下文件扩展名的文件.
include_ext = ["go", "tpl", "tmpl", "html"]
# 忽略这些文件扩展名或目录
exclude_dir = ["assets", "tmp", "vendor", "frontend/node_modules"]
# 监听以下指定目录的文件
include_dir = []
# 排除以下文件
exclude_file = []
# 如果文件更改过于频繁，则没有必要在每次更改时都触发构建。可以设置触发构建的延迟时间 单位: ms
delay = 1000
# 发生构建错误时，停止运行旧的二进制文件。
stop_on_error = true
# air的日志文件名，该日志文件放置在你的`tmp_dir`中
log = "air_errors.log"
 
[log]
# 显示日志时间
time = true
 
[color]
# 自定义每个部分显示的颜色。如果找不到颜色，使用原始的应用程序日志。
main = "magenta"
watcher = "cyan"
build = "yellow"
runner = "green"
 
[misc]
# 退出时删除tmp目录
clean_on_exit = true
```