# 开发环境安装指南（Mac版）

> 适用平台：**macOS**。本系列按平台拆分为三份：`开发环境安装指南-Linux.md` / `开发环境安装指南-Mac.md` / `开发环境安装指南-Windows.md`。
> 涵盖工具：Git、Node.js、NVM、Python、Miniconda、Go、GVM、Claude Code、OpenClaw、Docker。

## 目录

- Git 安装
  - macOS 安装
- Node.js 和 NVM 安装
  - macOS 安装
- Python 安装
  - macOS 安装
- Miniconda 安装
  - macOS 安装
- Go 安装
  - macOS 安装
- GVM (Go Version Manager) 安装
  - macOS 安装
- Claude Code 安装
  - 安装前检查
  - macOS 安装
- OpenClaw 安装
  - 安装前检查
  - macOS 安装
- Docker 安装
  - macOS 安装
  - Docker 常用命令
- 常见问题排查
  - macOS 问题
  - Docker 问题
- 参考链接
- 安装检查清单

---

# Git 安装

所有平台的基础版本控制工具。

## macOS 安装

> **最新版本**：以 https://git-scm.com 为准（约 2.52.x）
>
> macOS 自带 Git，但版本较旧。推荐使用 Homebrew 安装最新版。

### 方法一：Homebrew（推荐）

**先安装 Homebrew（如未安装）：**
```bash
curl -fsSL https://brew.sh | bash
```

**安装 Git：**
```bash
brew install git

# 安装 git-gui 和 gitk（可选）
brew install git-gui

# 验证
git --version
```

### 方法二：MacPorts

```bash
# 如果尚未安装 MacPorts，先安装
# https://www.macports.org

# 安装 Git
sudo port install git
```

### 方法三：Xcode Command Line Tools

```bash
# 安装 Xcode Command Line Tools
xcode-select --install
```

> Apple 自带 Git 二进制包，包含在 Xcode 命令行工具中。

### 方法四：Binary Installer（已过时）

Tim Harper 提供的安装程序仅支持到 2.33.0 版本（2021年），已不再更新，官方已移除链接。

> **注意**：非源码分发方式由第三方提供，可能不是最新版本。建议优先使用 Homebrew，可获得最新版本。

**配置 Git：**
```bash
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱"
git config --global init.defaultBranch main

# 启用颜色显示
git config --global color.ui auto

# 设置默认编辑器
git config --global core.editor "code --wait"
```

# Node.js 和 NVM 安装

**重要性**：Node.js 是 Claude Code 和 OpenClaw 的前置依赖，必须先安装。

## macOS 安装

### 方法一：使用 NVM（推荐）

**安装 NVM：**
```bash
# 使用官方安装脚本
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

**配置环境变量：**

将以下内容添加到 `~/.zshrc` 或 `~/.bash_profile`：

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

**重新加载配置：**
```bash
source ~/.zshrc
```

**使用 NVM 安装 Node.js：**
```bash
# 查看可用版本
nvm list-remote

# 安装 LTS 版本
nvm install --lts

# 安装指定版本
nvm install 20.11.0
nvm install 18.19.0

# 切换版本
nvm use 20.11.0

# 设置默认版本
nvm alias default 20.11.0

# 验证
node --version
npm --version
```

### 方法二：使用 Homebrew

```bash
brew install node

# 验证
node --version
npm --version
```

**NVM 常用命令：**
```bash
nvm --version              # 查看 NVM 版本
nvm list                   # 列出已安装版本
nvm list-remote           # 列出可安装版本
nvm install <version>     # 安装指定版本
nvm use <version>         # 切换到指定版本
nvm alias default <version>  # 设置默认版本
nvm uninstall <version>   # 卸载指定版本
```

# Python 安装

## macOS 安装

macOS 自带 Python 2.7，建议安装 Python 3：

**通过 Homebrew 安装：**
```bash
brew install python

python3 --version
pip3 --version
```

**配置国内镜像源：**
```bash
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

# Miniconda 安装

Python 环境管理工具。

## macOS 安装

**下载地址：** https://docs.conda.io/en/latest/miniconda.html

**安装步骤：**

```bash
# 下载安装脚本（Intel Mac）
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-x86_64.sh

# 下载安装脚本（Apple Silicon Mac）
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh

# 运行安装脚本
bash Miniconda3-latest-MacOSX-arm64.sh

# 按提示完成安装，重新加载 shell
source ~/.zshrc

# 验证
conda --version
```

**配置国内镜像源：**
```bash
conda config --set show_channel_urls yes
conda config --add channels conda-forge
conda config --set channel_priority strict
# 注：清华 Anaconda 镜像 2024-08 起已停止同步官方 defaults，改用 conda-forge
conda config --set channel_priority strict
```

**或通过 Homebrew 安装：**
```bash
brew install --cask miniconda
```

# Go 安装

## macOS 安装

**通过 Homebrew 安装（推荐）：**
```bash
brew install go

go version
```

**配置环境变量：**

将以下内容添加到 `~/.zshrc`：

```bash
# Go 环境变量
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
export GOPROXY=https://goproxy.cn,direct
export GO111MODULE=on

# 启用 Go 模块感知
export GOFLAGS=-mod=mod
```

```bash
source ~/.zshrc

# 验证
go version
go env GOPROXY
```

**或下载官方安装包：**
- 下载地址：https://go.dev/dl/
- 下载 `.pkg` 文件并双击安装

# GVM (Go Version Manager) 安装

Go 版本管理工具，需要先安装 Go。

## macOS 安装

**前置条件**：需要先安装 Go（见上节）

**官方仓库：** https://github.com/moovweb/gvm

**安装步骤：**

```bash
# 安装 GVM
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)

# 加载 GVM（或重新打开终端）
source ~/.gvm/scripts/gvm

# 验证安装
gvm version
```

**配置环境变量：**

将以下内容添加到 `~/.zshrc`：

```bash
# GVM
[[ -s "$HOME/.gvm/scripts/gvm" ]] && source "$HOME/.gvm/scripts/gvm"
```

```bash
source ~/.zshrc
```

**常用 GVM 命令：**
```bash
# 查看可安装的 Go 版本
gvm listall

# 安装特定版本（源码编译）
gvm install go1.25.0

# 安装二进制版本（更快）
gvm install go1.25.0 --binary

# 使用特定版本
gvm use go1.25.0

# 设置为默认版本
gvm use go1.25.0 --default

# 查看已安装版本
gvm list

# 卸载版本
gvm uninstall go1.25.0

# 删除 GVM（谨慎使用）
gvm implode
```

**注意：** 在 Apple Silicon Mac (M1/M2/M3) 上，可能需要先安装 Rosetta 2：
```bash
softwareupdate --install-rosetta

# 或安装二进制版本（跳过编译）
gvm install go1.25.0 --binary
```

# Claude Code 安装

**前置条件**：原生安装无需 Node.js；npm 方式需 Node.js 22+。Git 建议安装

## 安装前检查

在安装 Claude Code 之前，请先验证以下前置条件：

```bash
# 检查 Node.js 版本（必须 20+）
node --version

# 检查 npm 版本
npm --version

# 检查 Git 是否已安装
git --version
```

如果上述命令报错，请先按照本文档的 **Git 安装** 和 **Node.js 和 NVM 安装** 章节完成安装。

## macOS 安装

**系统要求：**
- macOS 10.15+
- 原生安装：无需 Node.js
- npm 方式：Node.js 22+
- Git（建议）

**安装前检查：**

```bash
# 检查 Node.js 版本（必须 20+）
node --version

# 检查 Git
git --version
```

**安装步骤：**

```bash
# 设置 npm 国内镜像（可选）
npm config set registry https://registry.npmmirror.com

# 推荐：原生安装器（无需 Node.js，自动更新）
curl -fsSL https://claude.ai/install.sh | bash
# 备选（需 Node.js 22+）：npm install -g @anthropic-ai/claude-code@latest

# 验证
claude --version

# 首次运行
claude
# 按提示完成登录授权
```

# OpenClaw 安装

**前置条件**：推荐一键安装脚本（自动装 Node 24/26）；若手动 npm 安装需 Node 24.16+ 或 26.1+

## 安装前检查

```bash
# 检查 Node.js 是否已安装
node --version
npm --version
```

如果报错，请先安装 Node.js。

## macOS 安装

```bash
# 推荐：一键安装脚本（自动装 Node 并完成配置）
curl -fsSL https://openclaw.ai/install.sh | bash
# 或手动 npm 安装（需 Node 24.16+/26.1+）：npm install -g openclaw@latest

# 验证
openclaw --version

# 首次配置
openclaw onboard
```

# Docker 安装

## macOS 安装

**下载地址：** https://www.docker.com/products/docker-desktop/

**安装步骤（Intel Mac）：**

1. 下载 `Docker.dmg`（Intel Chip）
2. 双击打开 DMG 文件
3. 将 Docker 图标拖到 Applications 文件夹
4. 从启动台打开 Docker Desktop
5. 等待启动完成，验证：
   ```bash
   docker --version
   ```

**安装步骤（Apple Silicon Mac）：**

1. 下载 `Docker.dmg`（Apple Chip）
2. 安装步骤同上
3. 注意：某些旧镜像可能需要 `--platform linux/amd64` 参数

**或通过 Homebrew 安装：**

```bash
brew install --cask docker
```

**启动 Docker：**

```bash
# 启动 Docker Desktop 服务
open /Applications/Docker.app

# 或
brew services start docker
```

## Docker 常用命令

```bash
# 镜像操作
docker pull ubuntu:22.04          # 拉取镜像
docker images                     # 列出镜像
docker rmi image_id               # 删除镜像
docker build -t myapp:1.0 .       # 构建镜像
docker tag myapp:1.0 registry/myapp:1.0  # 标记镜像
docker push registry/myapp:1.0    # 推送镜像

# 容器操作
docker run -d -p 8080:80 nginx    # 运行容器
docker run -it ubuntu bash        # 交互式运行
docker ps                         # 列出运行中容器
docker ps -a                      # 列出所有容器
docker stop container_id          # 停止容器
docker start container_id         # 启动容器
docker rm container_id            # 删除容器
docker exec -it container_id bash # 进入容器
docker logs -f container_id      # 查看容器日志

# 数据卷
docker volume ls                  # 列出数据卷
docker volume create myvol        # 创建数据卷
docker volume rm myvol            # 删除数据卷
docker volume prune               # 清理未使用数据卷

# 网络
docker network ls                 # 列出网络
docker network create mynet       # 创建网络
docker network rm mynet           # 删除网络

# 清理
docker system prune               # 清理未使用资源
docker system prune -a            # 清理所有未使用资源
docker container prune            # 清理已停止容器
docker image prune                # 清理悬空镜像

# Docker Compose
docker compose up -d              # 后台启动服务
docker compose down               # 停止并删除服务
docker compose logs -f            # 查看服务日志
docker compose build              # 构建服务
docker compose ps                 # 列出服务状态
docker compose restart            # 重启服务
```

---

# 常见问题排查

## macOS 问题

**1. 'brew' 命令找不到**
```bash
# 重新安装 Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 添加到 PATH
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
```

**2. NVM 安装后无法使用**
```bash
# 确保在 ~/.zshrc 中有以下内容
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# 重新加载
source ~/.zshrc
```

**3. GVM 编译 Go 失败（Apple Silicon）**
```bash
# 安装 Rosetta 2
softwareupdate --install-rosetta

# 或安装二进制版本（跳过编译）
gvm install go1.25.0 --binary
```

**4. 权限问题（Permission denied）**
```bash
# 修复 npm 全局安装权限
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc
```

## Docker 问题

**1. Docker Desktop 启动失败（Windows，供参考）**
```powershell
# 确保 WSL 2 已启用
wsl --set-default-version 2

# 或启用 Hyper-V
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All

# 重置 Docker Desktop
# Settings → Troubleshoot → Reset to factory defaults
```

**2. Docker Desktop 启动失败（macOS）**
```bash
# 完全退出 Docker Desktop
# 然后运行
/Applications/Docker.app/Contents/MacOS/Docker --uninstall
# 重新安装
```

**3. 端口冲突**
```bash
# 查看端口占用
sudo lsof -i :8080
# 或
sudo netstat -tulpn | grep :8080

# 使用其他端口
docker run -d -p 8081:80 nginx
```

**4. 镜像拉取超时/失败**
```bash
# 配置镜像加速（见上方配置）
# 或使用代理
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080

# 然后重启 docker
```

**5. 磁盘空间不足**
```bash
# 清理未使用资源
docker system prune -a

# 清理卷
docker volume prune

# Windows/macOS 可在 Docker Desktop 中调整磁盘大小
# Settings → Resources → Disk image size
```

---

# 参考链接

## 官方文档

| 工具 | 官方文档 | 下载地址 |
|------|---------|---------|
| Git | https://git-scm.com/doc | https://git-scm.com/downloads |
| Node.js | https://nodejs.org/docs/ | https://nodejs.org/ |
| NVM | https://github.com/nvm-sh/nvm | `nvm-sh/nvm v0.40.3` |
| NVM Windows | https://github.com/coreybutler/nvm-windows | https://github.com/coreybutler/nvm-windows/releases |
| Python | https://docs.python.org/3/ | https://www.python.org/downloads/ |
| Miniconda | https://docs.conda.io/ | https://docs.conda.io/en/latest/miniconda.html |
| Go | https://go.dev/doc/ | https://go.dev/dl/ |
| GVM | https://github.com/moovweb/gvm | - |
| Claude Code | https://code.claude.com/docs/en/setup | 原生安装器或 `npm i -g @anthropic-ai/claude-code@latest` |
| OpenClaw | https://github.com/openclaw/openclaw | `openclaw.ai/install.sh` 或 `npm i -g openclaw@latest` |
| WSL | https://learn.microsoft.com/zh-cn/windows/wsl/ | - |
| Docker | https://docs.docker.com/ | https://www.docker.com/products/docker-desktop/ |

## 包管理器

| 工具 | 官方文档 |
|------|---------|
| Homebrew | https://docs.brew.sh/ |
| Scoop | https://scoop.sh/ |
| Chocolatey | https://chocolatey.org/ |
| Winget | https://learn.microsoft.com/windows/package-manager/ |

## 国内镜像源

| 服务 | 镜像地址 |
|------|---------|
| npm | https://registry.npmmirror.com |
| pip (清华) | https://pypi.tuna.tsinghua.edu.cn |
| conda-forge | https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge |
| Go Proxy | https://goproxy.cn |
| Docker (毫秒) | https://docker.1ms.run |
| Docker (轩辕) | https://docker.xuanyuan.me |
| Node (npmmirror) | https://npmmirror.com/mirrors/node/ |

---

# 安装检查清单

安装完成后，运行以下命令验证所有工具：

```bash
# 版本检查
git --version
node --version
npm --version
nvm --version  # 如安装
python --version  # 或 python3 --version
pip --version
conda --version
go version
gvm version  # 如安装
claude --version
openclaw --version

# Go 环境检查
go env GOPATH
go env GOPROXY
go env GO111MODULE

# Conda 环境检查
conda info
conda env list

# Docker 检查
docker --version
docker-compose --version  # 或 docker compose version
docker run hello-world

# WSL 检查（Windows）
wsl --version
wsl --list --verbose
wsl -d Ubuntu -e bash -c "git --version && go version && node --version"
```

---

> 文档整理：2026-11-22。命令均按对应平台的默认终端执行。
> 通用内容（参考链接、安装检查清单、Docker 常用命令等）在各平台版本均保留。
