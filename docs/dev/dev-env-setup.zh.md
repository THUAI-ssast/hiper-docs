# 开发环境搭建

> 开发环境搭建还算简单，故暂不使用 devcontainer。

## 前端

## 后端 - Web

安装 Go：

=== "Windows"

    ```bash
    winget add GoLang.Go
    ```

=== "macOS"

    ```bash
    brew install go
    ```

如 Dockerfile 里演示的，由于中国大陆的特殊网络环境，需要在环境变量里设置 Go 的代理站。如下：

```bash
export GOPROXY=https://goproxy.cn,direct
```

建议保存到 `~/.bashrc` 或 `~/.zshrc` 等文件中。

可直接用 VSCode 打开本项目，根据提示安装本项目需要的扩展。

请确保提交的代码都用 `go fmt`, `go mod tidy` 等工具处理过（目前没有git hook）。

## 后端 - Worker
