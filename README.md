# Claude Code + Codex Dev Container

一个面向 Claude Code 和 OpenAI Codex 的通用 VS Code 开发容器模板。它基于 Debian Bookworm，预装 Node.js 22、Git、Claude Code、Codex CLI，以及可连接宿主机 Docker daemon 的 Docker CLI。

## 功能

- Claude Code CLI `2.1.152` 和对应 VS Code 扩展
- OpenAI Codex CLI 和 `openai.chatgpt` 扩展
- Node.js 22、Git、OpenSSH、ripgrep、jq、Python 3 和常用构建工具
- Docker CE CLI、Docker Compose 插件和 Buildx
- 简体中文 UTF-8 环境，时区为 `Asia/Shanghai`
- 非 root 的 `vscode` 开发用户
- Claude、Codex、Git 和 SSH 配置持久化

## 前置要求

- Docker Engine 或 Docker Desktop
- Visual Studio Code
- VS Code Dev Containers 扩展

本模板在容器内独立维护 Git 用户信息和 SSH 密钥，并保存到持久化 Docker volume，不依赖宿主机 Git 配置或 SSH Agent。

## 使用

```bash
git clone git@github.com:buffmio/Claude-Code-Codex-Dev-Container.git
cd Claude-Code-Codex-Dev-Container
code .
```

在 VS Code 命令面板中执行：

```text
Dev Containers: Reopen in Container
```

首次启动需要构建镜像和下载 Dev Container Feature，耗时取决于网络情况。

修改 `.devcontainer/devcontainer.json` 或 `.devcontainer/Dockerfile` 后，执行：

```text
Dev Containers: Rebuild Container
```

## 初始化 Claude Code 和 Codex

进入开发容器后分别运行：

```bash
claude
codex
```

根据终端提示完成登录。相关配置保存在持久化 Docker volume 中，重建容器后仍会保留。

## 配置 Git 和 SSH

请直接在容器内配置 Git 身份和 SSH 密钥：

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
ssh-keygen -t ed25519 -C "你的邮箱"
```

查看公钥并添加到 GitHub、GitLab 或其他代码托管平台：

```bash
cat ~/.ssh/id_ed25519.pub
```

Git 配置和 SSH 密钥位于 `/home/vscode/.persist` 对应的 Docker volume。删除该 volume 会同时删除这些持久化数据，请妥善保管私钥并自行备份。

## Docker 使用方式

本模板使用 Docker-outside-of-Docker（DooD），把宿主机 Docker socket 提供给开发容器使用，并不会在开发容器中启动第二个 Docker daemon。

验证命令：

```bash
docker version
docker compose version
docker buildx version
```

通过 socket 创建的容器、镜像、网络和卷都属于宿主机 Docker daemon，与宿主机共享。

> [!WARNING]
> 能访问 Docker socket 的进程基本拥有宿主机 root 级别的控制能力。请只在可信代码和可信工作区中使用本模板。

## 持久化内容

命名 Docker volume 挂载到：

```text
/home/vscode/.persist
```

其中保存：

- Claude Code 配置
- Codex 配置
- Git 全局配置
- SSH 密钥和配置

项目源代码由 VS Code 挂载，不保存在该用户状态 volume 中。

## 常用检查

```bash
claude --version
codex --version
node --version
npm --version
git --version
docker version
```

如果 `docker version` 提示权限不足，请先确认宿主机 Docker daemon 正在运行，然后执行 `Dev Containers: Rebuild Container`，让官方 Docker-outside-of-Docker Feature 重新配置 socket 访问权限。

## 版本更新

- Claude Code CLI 版本由 Dockerfile 中的 `CLAUDE_CODE_VERSION` 控制。
- Codex CLI 在重建镜像时安装当时的 `latest` 版本。
- Docker CLI、Compose 和 Buildx 由官方 `docker-outside-of-docker:1` Feature 提供。
- 本仓库作为通用模板，不提交 `devcontainer-lock.json`；不同时间首次构建可能解析到不同的兼容 Feature 版本。
