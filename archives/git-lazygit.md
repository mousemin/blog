---
title: Lazygit 一个用于 Git 命令行的简单终端 UI
date: 2021-07-22T16:49:15+08:00
Description:
Tags: 
Categories:
 - 工具
DisableComments: false
---

**Git的强大是所有开发者都心知肚明的事情，但是其多样的命令令人很是难受。**

不过在Github上有着这么一个开源项目[lazygit](https://github.com/jesseduffield/lazygit)简化git命令操作。



![21051709-6a18cab0d76ef403](https://cdn.mousemin.com/img/2021-07-16-a186ef67608b5f7e2905fd9e6aef2a43.gif)

特性:
- 轻松添加文件。
- 解决合并冲突。
- 轻松查看最近的分支机构。
- 滚动分支/提交/存储的日志/差异。
- 快速推/拉。
- 压缩并重命名提交。

如果你想要了解更多有关**Lazygit**的特性，请访问[https://youtu.be/CPLdltN7wgE](https://links.jianshu.com/go?to=https%3A%2F%2Fyoutu.be%2FCPLdltN7wgE)。

## 安装

**Lazygit**给出了多种安装方式和平台支持，你可以使用如下命令去尝试安装该应用程序。

**以下的安装方式来自官方文档，详细信息请参考[Lazygit](https://github.com/jesseduffield/lazygit/blob/master/README.md)。**

### Binary Releases

For Windows, Mac OS or Linux, you can download a binary release [here](https://github.com/jesseduffield/lazygit/releases).

### Ubuntu

```bash
sudo add-apt-repository ppa:lazygit-team/release
sudo apt-get update
sudo apt-get install lazygit
```

### centos

```bash
sudo dnf copr enable atim/lazygit -y
sudo dnf install lazygit
```

### Go

```bash
go get github.com/jesseduffield/lazygit
```

## Lazygit的基本操作

在安装完成后，你可以在某个本地的Git仓库中使用lazygit命令来打开**Lazygit**控制台。 如果你认为这条命令有点麻烦，你可以添加**alias**别名`echo "alias lg='lazygit'" >> ~/.bashrc`，后面的文件取决于你所使用中的终端。

在打开**Lazygit**之后我们很容易就能看到最下方的帮助信息。

- `PgUp`键向上滚动
- `PaDn`键向下滚动
- `x`键打开开单
- 方向键来控制关闭光标（可以使用鼠标来控制）

### Undo/Redo

See the [docs](https://github.com/jesseduffield/lazygit/blob/master/docs/Undoing.md)

### 互动基础

![互动基础](https://cdn.mousemin.com/img/2021-07-16-d719dd5d211c8297649e55739602905b.gif)


### 解决合并冲突

![解决合并冲突](https://cdn.mousemin.com/img/2021-07-16-7e1ab67d121f80cceea05de6cdb55f9f.gif)