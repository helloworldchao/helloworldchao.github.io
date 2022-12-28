---
layout: post
title:  "Windows 环境下将 frpc 编译为 Android 的库"
date:   2022-12-28 09:31:00 +0800
categories: Android Frp Go Gomobile
permalink: /archivers/FrpcGomobileCompile
---

# 项目背景

为了在 Android 上使用 frpc 的 stcp 和 sudp 功能，我在网上搜索了一圈，发现已有的项目 frpc 版本和 Android Apk target API 都较老，因此我决定基于最新的 frpc 版本编译一个自己可以使用的 frpc 库。

# 操作步骤

## 安装 Go

去官网下载最新的 Go 版本。安装成功后，设置代理和 Module 模式

```bash
go env -w GOPROXY=https://goproxy.cn,direct
go env -w GO111MODULE=audo
```

设置完成后在系统环境变量中配置 GOPATH 到喜欢的目录，或者直接使用默认的目录也可。

## 安装 Android 开发环境

按步骤安装 Java Jdk，Android SDK，Android NDK即可，这里省略。

## 拉取 fprc 源码

在 GOPATH 目录下，按文件夹创建 GOPATH/src/github.com/fatedier/frp，其中最后一个 frp 目录放置最新的 [frp 源码 https://github.com/fatedier/frp](https://github.com/fatedier/frp)。

## 安装 Gomobile

进入上一步的 GOPATH/src/github.com/fatedier/frp 目录，打开命令行按顺序分别输入下面的命令

```bash
go get golang.org/x/mobile/cmd/gomobile
go install golang.org/x/mobile/cmd/gomobile
gomobile init
```

前面都顺利完成后，执行

```bash
gomobile version
```

出现下面这个报错，代表成功了。（这里报错的原因可能是 GoModule 配置的问题，因为对 Go 不熟悉，所以没有更深的研究了。）
```
gomobile version unknown: mobile repo git log failed: exit status 128, fatal: not a git repository (or any of the parent directories): .git
```

## 编译 frpc 库

1. 打开 GOPATH/src/github.com/fatedier/frp 源码。
2. 修改 GOPATH/src/github.com/fatedier/frp/cmd/frpc/sub/root.go，将小写的 runClient 修改为 RunClient。
3. 修改 GOPATH/src/github.com/fatedier/frp/cmd/frpc/main.go，将

先将
```go
package main
```

修改为

```go
package frpc
```

后将
```go
func main() {
	sub.Execute()
}
```

修改为

```go
func Run(cfgFilePath string) {
	sub.RunClient(cfgFilePath)
}
```

其他方法可熟悉后自行添加，例如我添加了获取版本的方法，完整示例如下
```go

package frpc

import (
	_ "github.com/fatedier/frp/assets/frpc"
	"github.com/fatedier/frp/cmd/frpc/sub"
	"github.com/fatedier/frp/pkg/util/version"
)

func Run(cfgFilePath string) {
	sub.RunClient(cfgFilePath)
}

func GetVersion() string {
	return version.Full()
}

```

原始版本如下
```go
package main

import (
	_ "github.com/fatedier/frp/assets/frpc"
	"github.com/fatedier/frp/cmd/frpc/sub"
)

func main() {
	sub.Execute()
}
```

最后执行编译命令，若没有报错则代表成功

```bash
gomobile bind -target=android github.com/fatedier/frp/cmd/frpc
```

# 参考项目

[frp 安卓客户端 https://github.com/qiuhaotc/frp_android](https://github.com/qiuhaotc/frp_android)

[frp 编译步骤 https://github.com/FrpcCluster/frpc-Android/blob/master/Compile_zh.md](https://github.com/FrpcCluster/frpc-Android/blob/master/Compile_zh.md)