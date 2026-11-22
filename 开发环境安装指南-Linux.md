# 开发环境安装指南（Linux版）

> 适用平台：**Linux**。本系列按平台拆分为三份：`开发环境安装指南-Linux.md` / `开发环境安装指南-Mac.md` / `开发环境安装指南-Windows.md`。
> 涵盖工具：Git、Node.js、NVM、Python、Miniconda、Go、GVM、Claude Code、OpenClaw、Docker。

## 目录

- Git 安装
  - Linux 安装
- Node.js 和 NVM 安装
  - Linux 安装
- Python 安装
  - Linux 安装
- Miniconda 安装
  - Linux 安装
- Go 安装
  - Linux 安装
- GVM (Go Version Manager) 安装
  - Linux 安装
- Claude Code 安装
  - 安装前检查
  - Linux 安装
- OpenClaw 安装
  - 安装前检查
  - Linux 安装
- Docker 安装
  - Linux 安装
  - Docker 常用命令
- 参考链接
- 安装检查清单

---

# Git 安装

所有平台的基础版本控制工具。

## Linux 安装

> **最新版本**：以 https://git-scm.com 为准（约 2.52.x）
>
> 各发行版可通过包管理器安装。

### Debian/Ubuntu

```bash
# 安装最新稳定版
sudo apt-get update
sudo apt-get install git

# Ubuntu PPA（获取最新上游版本）
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install git
```

### Fedora

```bash
# Fedora 21 及之前
sudo yum install git

# Fedora 22 及之后
sudo dnf install git
```

### Red Hat Enterprise Linux / Oracle Linux / CentOS / Scientific Linux

> RHEL 系发行版通常提供较旧版本的 Git。可选方案：
> 1. 从源码编译：从 [kernel.org](https://www.kernel.org/pub/software/scm/git/) 下载 tarball
> 2. 使用第三方仓库：[IUS Community Project](https://ius.io/)

### Gentoo

```bash
sudo emerge --ask --verbose dev-vcs/git
```

### Arch Linux

```bash
sudo pacman -S git
```

### openSUSE

```bash
sudo zypper install git
```

### Mageia

```bash
sudo urpmi git
```

### Nix/NixOS

```bash
nix-env -i git
```

### FreeBSD

```bash
pkg install git
```

### Solaris 9/10/11 (OpenCSW)

```bash
pkgutil -i git
```

### Solaris 11 Express, OpenIndiana

```bash
pkg install developer/versioning/git
```

### OpenBSD

```bash
pkg_add git
```

### Alpine

```bash
apk add git
```

### Slackware

```bash
tazpkg get-install git
```

### 验证安装

```bash
# 验证
git --version

# 配置
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱"
git config --global init.defaultBranch main
git config --global color.ui auto
```

> **提示**：Linux 上最便捷的方式是使用各发行版的包管理器安装。

**Git 配置优化（推荐）：**
```bash
# 设置默认分支名为 main
git config --global init.defaultBranch main

# 设置拉取策略
git config --global pull.rebase false

# 设置推送策略
git config --global push.default current

# 启用长命令别名（可选）
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
```

**Git 常用命令速查：**
```bash
git init                          # 初始化仓库
git clone <url>                   # 克隆仓库
git add .                         # 添加所有文件
git commit -m "message"           # 提交
git push                          # 推送到远程
git pull                          # 拉取远程更新
git status                        # 查看状态
git log --oneline                 # 查看提交历史
git branch                        # 查看分支
git checkout -b <branch>          # 创建并切换分支
```

---

# Node.js 和 NVM 安装

**重要性**：Node.js 是 Claude Code 和 OpenClaw 的前置依赖，必须先安装。

## Linux 安装

### 方法一：使用 NVM（推荐）

```bash
# 安装 NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# 配置环境变量
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# 验证
nvm --version

# 安装 Node.js LTS
nvm install --lts
nvm alias default lts/*

# 验证
node --version
npm --version
```

### 方法二：使用包管理器

**Ubuntu/Debian：**
```bash
# 添加 NodeSource 仓库
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# 安装 Node.js
sudo apt install -y nodejs
```

**CentOS/RHEL：**
```bash
curl -fsSL https://rpm.nodesource.com/setup_20.x | sudo bash -
sudo yum install -y nodejs
```

**设置国内镜像：**
```bash
npm config set registry https://registry.npmmirror.com
```

---

# Python 安装

## Linux 安装

大多数 Linux 发行版自带 Python 3：

```bash
python3 --version
```

**如未安装：**

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3 python3-pip -y

# CentOS/RHEL 8+
sudo dnf install python3 python3-pip -y

# Arch Linux
sudo pacman -S python python-pip

# 配置国内镜像源
pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

**Python 常用命令：**
```bash
python --version           # 查看版本
python -m venv myenv       # 创建虚拟环境
source myenv/bin/activate  # 激活虚拟环境（Linux/macOS）
myenv\Scripts\activate    # 激活虚拟环境（Windows）
pip install <package>      # 安装包
pip list                   # 列出已安装包
pip freeze > requirements.txt  # 导出依赖
```

---

# Miniconda 安装

Python 环境管理工具。

## Linux 安装

```bash
# 下载安装脚本
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# 运行安装脚本
bash Miniconda3-latest-Linux-x86_64.sh

# 按提示完成安装：
# - 按 Enter 查看许可协议
# - 输入 yes 同意
# - 选择安装路径（默认 ~/miniconda3）
# - 输入 yes 初始化 conda

# 重新加载 shell 或运行
source ~/.bashrc

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

**初始化（如安装时未选择）：**
```bash
conda init bash
# 或
conda init zsh
```

---

# Go 安装

## Linux 安装

**方法1：通过包管理器（版本可能较旧）**

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install golang-go -y
```

**方法2：下载官方二进制包（推荐）**

```bash
# 下载（根据系统架构选择）
wget https://go.dev/dl/go1.25.0.linux-amd64.tar.gz

# 或使用 curl
curl -OL https://go.dev/dl/go1.25.0.linux-amd64.tar.gz

# 删除旧版本（如有）
sudo rm -rf /usr/local/go

# 解压到新目录
sudo tar -C /usr/local -xzf go1.25.0.linux-amd64.tar.gz

# 添加环境变量
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

# 验证
go version
```

**配置 Go 环境变量：**

将以下内容添加到 `~/.bashrc` 或 `~/.zshrc`：

```bash
# Go 环境变量
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
export GOPROXY=https://goproxy.cn,direct
export GO111MODULE=on

# 启用 Go 模块感知
export GOFLAGS=-mod=mod
```

```bash
source ~/.bashrc

# 验证
go env GOPATH
go env GOPROXY
```

**Go 常用命令：**
```bash
go version              # 查看版本
go mod init <module>    # 初始化模块
go mod tidy             # 整理依赖
go build                # 编译
go run                  # 运行
go test                 # 测试
go get                  # 获取依赖
go install              # 安装程序
```

---

# GVM (Go Version Manager) 安装

Go 版本管理工具，需要先安装 Go。

## Linux 安装

**前置条件**：需要先安装 Go（见上节）

**依赖安装：**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install bison gcc make -y

# CentOS/RHEL
sudo yum install bison gcc make -y
```

**安装 GVM：**
```bash
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)

# 加载 GVM
source ~/.gvm/scripts/gvm

# 添加到 ~/.bashrc
echo 'source ~/.gvm/scripts/gvm' >> ~/.bashrc
```

**使用 GVM：**
```bash
# 安装 Go 1.21.6（源码编译）
gvm install go1.25.0

# 或安装二进制版本（更快）
gvm install go1.25.0 --binary

# 设置为默认
gvm use go1.25.0 --default

# 验证
go version
```

---

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

## Linux 安装

**系统要求：**
- Linux 64位
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
```

**权限问题解决（如遇到 EACCES 错误）：**
```bash
# 方法1：配置 npm 使用自定义目录
mkdir -p ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# 方法2：使用 npx 临时运行
npx @anthropic-ai/claude-code
```

**Claude Code 常用命令：**
```bash
claude                  # 启动交互式会话
claude --version       # 查看版本
claude --help          # 查看帮助
claude --print         # 打印 Claude 的回复（不进入交互模式）
```

---

# OpenClaw 安装

**前置条件**：推荐一键安装脚本（自动装 Node 24/26）；若手动 npm 安装需 Node 24.16+ 或 26.1+

## 安装前检查

```bash
# 检查 Node.js 是否已安装
node --version
npm --version
```

如果报错，请先安装 Node.js。

## Linux 安装

```bash
# 设置 npm 国内镜像（可选）
npm config set registry https://registry.npmmirror.com

# 推荐：一键安装脚本（自动装 Node 24/26 并完成配置）
curl -fsSL https://openclaw.ai/install.sh | bash

# 或手动 npm 安装（需 Node 24.16+/26.1+）：npm install -g openclaw@latest

# 若手动安装，需运行配置向导：openclaw onboard

# 验证
openclaw --version
```

---

# Docker 安装

## Linux 安装

以 **Ubuntu/Debian** 为例：

**官方文档：** https://docs.docker.com/engine/install/ubuntu/

**步骤1：卸载旧版本**

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

**步骤2：更新 apt 并安装依赖**

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
```

**步骤3：添加 Docker 官方 GPG 密钥**

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

**步骤4：添加 Docker 软件源**

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**步骤5：安装 Docker Engine**

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**步骤6：验证安装**

```bash
sudo docker --version
sudo docker run hello-world
```

**步骤7：配置非 root 用户使用 Docker（可选但推荐）**

```bash
# 创建 docker 组
sudo groupadd docker

# 将当前用户添加到 docker 组
sudo usermod -aG docker $USER

# 重新登录或运行
newgrp docker

# 验证（无需 sudo）
docker run hello-world
```

**配置国内镜像加速：**

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.xuanyuan.me",
    "https://docker.m.daocloud.io"
  ]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
```

**Docker Compose 安装**

**Docker Desktop** 已包含 Docker Compose。

**Linux 单独安装：**

```bash
# 安装 Docker Compose 插件（推荐）
sudo apt install docker-compose-plugin

# 验证
docker compose version

# 或使用独立版本
curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
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
