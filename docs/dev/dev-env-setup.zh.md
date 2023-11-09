# 开发环境搭建

> 开发环境搭建还算简单，故暂不使用 devcontainer。

安装 Go：

```bash
# Windows
winget add GoLang.Go
# macOS
brew install go
```

如 Dockerfile 里演示的，由于中国大陆的特殊网络环境，需要在环境变量里设置 Go 的代理站。如下：

```bash
export GOPROXY=https://goproxy.cn,direct
```

建议保存到 `~/.bashrc` 或 `~/.zshrc` 等文件中；Windows 也可在操作系统的环境变量中设置。

安装 VSCode（也可使用其他编辑器/IDE）：

```bash
# Windows
winget add -i Microsoft.VisualStudioCode
```

在VSCode中搜索安装 `golang.go` 扩展。可能需要根据提示补充安装 Go 的工具链。也可直接用 VSCode 打开本项目，根据提示安装本项目需要的扩展。

可根据自己的喜好调节 VSCode 的 设置，如 `Format on Save`。

请确保提交的代码都用 `go fmt`, `go mod tidy` 等工具处理过（目前没有git hook）。
