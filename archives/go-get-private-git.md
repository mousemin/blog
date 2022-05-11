---
title: Go拉取私有仓库的问题
date: 2021-06-28T16:13:25+08:00
Description:
Tags: 
 - go
Categories:
DisableComments: false
---

现在项目开发有很多私有仓库，直接`git clone`的方式使用，不是怎么方便。

查询`go`源码发现`go get`支持的协议除了`https`还支持`git+ssh`, `bzr+ssh`, `svn+ssh`, `ssh`

`$GOSRC/cmd/go/internal/get/vsc.go`
```go
var defaultSecureScheme = map[string]bool{
	"https":   true,
	"git+ssh": true,
	"bzr+ssh": true,
	"svn+ssh": true,
	"ssh":     true,
}
```

## 简单 - 直接使用git/ssh方式
直接在`go get gitlab.com/****/****`时,在后面加上`.git`, go会自动使用`git/ssh`的方式拉取`git仓库`.
>注意: 正常的拉取方式，会生成`$GOPATH/git.gitlab.com/****/****`目录接口, 使用`.git`方式拉取会生成`$GOPATH/gitlab.com/****/****.git`的目录接口



## 修改配置的方式
1. 私有仓库一般没方法`sum`校验，我们先把`sum`校验去除掉

配置环境变量使拉取代码不走代理与sum校验
```bash
export GOPRIVATE="gitlab.com"
```

这个配置后, 拉取仓库，可以发现`gitlab.com/user***/repo`, 这种私有仓库我们能正常的拉取, 但是类似`gitlab.com/gourp1/gourp2/repo`不能正常拉取，

使用`go get -v gitlab.com/gourp1/gourp2/repo`后能发现, `go`认为仓库的真实地址是`gitlab.com/gourp1/gourp2`,并不是`gitlab.com/gourp1/gourp2/repo`

这个问题我们通过查看源码依旧能发现
`$GOSRC/cmd/go/internal/get/vsc.go`
```go
var vcsPaths = []*vcsPath{
	// Github
	{
		prefix: "github.com/",
		regexp: lazyregexp.New(`^(?P<root>github\.com/[A-Za-z0-9_.\-]+/[A-Za-z0-9_.\-]+)(/[\p{L}0-9_.\-]+)*$`),
		vcs:    "git",
		repo:   "https://{root}",
		check:  noVCSSuffix,
	},

	// Bitbucket
	{
		prefix: "bitbucket.org/",
		regexp: lazyregexp.New(`^(?P<root>bitbucket\.org/(?P<bitname>[A-Za-z0-9_.\-]+/[A-Za-z0-9_.\-]+))(/[A-Za-z0-9_.\-]+)*$`),
		repo:   "https://{root}",
		check:  bitbucketVCS,
	},
    // .....
    {
		regexp: lazyregexp.New(`(?P<root>(?P<repo>([a-z0-9.\-]+\.)+[a-z0-9.\-]+(:[0-9]+)?(/~?[A-Za-z0-9_.\-]+)+?)\.(?P<vcs>bzr|fossil|git|hg|svn))(/~?[A-Za-z0-9_.\-]+)*$`),
		schemelessRepo: true,
	},
}
```
2. 配置获取仓库授权


配置`~/.netrc`(window中配置`~/_netrc`)完成gitlab授权，获取真实的git路径
```ini
machine gitlab.com login 账号 password 密码或者访问令牌
```
>使用访问令牌请勾选api的权限

3. 修改`git`拉取`https`替换 `ssh`

我们知道`go get`默认会使用`https`的方式拉取代码，由于`git-remote-https`走的验证是用户名，密码, 不怎么方便，我们来通过更改`git`的全局配置来使用`ssh`的方式拉取。
下面是配置`https`转换为`ssh`的命令
```bash
git config --global url."git@gitlab.com:".insteadOf https://gitlab.com/
```

## 参考资料
- [gitlab](https://gitlab.com/gitlab-org/gitlab-foss/-/issues/1337)
- [go](https://github.com/golang/go)