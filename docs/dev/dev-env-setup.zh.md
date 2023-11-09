# 开发环境搭建

> 开发环境搭建还算简单，故暂不使用 devcontainer。

以 Windows 为例。

安装 Go：

```bash
winget add GoLang.Go
```

如 Dockerfile 里演示的，由于中国大陆的特殊网络环境，需要在环境变量里设置 Go 的代理站。如下：

```bash
export GOPROXY=https://goproxy.cn,direct
```

安装 VSCode（也可使用其他编辑器/IDE）：

```bash
winget add -i Microsoft.VisualStudioCode
```

在VSCode中搜索安装 `Go` 扩展。

可根据自己的喜好调节VSCode的设置，如 Format on Save。

请确保提交的代码都用 go fmt, go mod tidy 等工具处理过（目前没有git hook）。
