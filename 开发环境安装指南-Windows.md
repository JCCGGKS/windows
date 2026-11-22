# 开发环境安装指南（Windows版）

> 适用平台：**Windows**。本系列按平台拆分为三份：`开发环境安装指南-Linux.md` / `开发环境安装指南-Mac.md` / `开发环境安装指南-Windows.md`。
> 涵盖工具：Git、Node.js、NVM、Python、Miniconda、Go、GVM、Claude Code、OpenClaw、Docker、WSL、Windows OpenSSH。

## 目录

- Git 安装
  - Windows 安装
- Node.js 和 NVM 安装
  - Windows 安装
- Python 安装
  - Windows 安装
- Miniconda 安装
  - Windows 安装
- Go 安装
  - Windows 安装
- GVM (Go Version Manager) 安装
  - Windows 安装
- Claude Code 安装
  - 安装前检查
  - Windows 安装
- OpenClaw 安装
  - 安装前检查
  - Windows 安装
- WSL 安装 (Windows)
- Docker 安装
  - Windows 安装
  - Docker 常用命令
- 常见问题排查
  - Windows 问题
  - WSL 问题
  - Docker 问题
- 参考链接
- 安装检查清单
- Windows OpenSSH 安装与配置完全指南

---

# Git 安装

所有平台的基础版本控制工具。

## Windows 安装

**官方下载地址：** https://git-scm.com/download/win

**安装步骤：**

1. 访问官网下载页面，下载会自动开始（或选择 32/64 位版本）
2. 运行下载的 `.exe` 安装程序
3. 安装选项建议：
   - **Select Components**: 保持默认，建议勾选 "Windows Explorer integration"
   - **Choosing the default editor**: 选择你常用的编辑器（如 VS Code）
   - **Adjusting the name of the initial branch**: 选择 "Let Git decide" 或自定义为 `main`
   - **Adjusting your PATH environment**: 选择 **"Git from the command line and also from 3rd-party software"**（推荐）
   - **Choosing HTTPS transport backend**: 选择 "Use the OpenSSL library"
   - **Configuring the line ending conversions**: 选择 **"Checkout Windows-style, commit Unix-style line endings"**
   - **Configuring the terminal emulator**: 选择 **"Use Windows' default console window"** 或 MinTTY
4. 完成安装，打开 CMD 或 PowerShell 验证：
   ```cmd
   git --version
   ```

**配置 Git：**
```cmd
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

## Windows 安装

### 方法一：直接安装 Node.js

**官方下载地址：** https://nodejs.org/

1. 下载 LTS 版本的 Windows 安装程序
2. 运行 `.msi` 安装程序，按向导完成安装
3. 验证安装：
   ```cmd
   node --version
   npm --version
   ```

### 方法二：使用 NVM for Windows（推荐）

**前置条件**：必须先卸载所有已存在的 Node.js 安装

**GitHub 地址：** https://github.com/coreybutler/nvm-windows

**下载地址：** https://github.com/coreybutler/nvm-windows/releases

**安装步骤：**

1. 下载 `nvm-setup.exe` 并运行
2. 完成安装后，**重启终端**

**常用命令：**
```cmd
# 列出可安装版本
nvm list available

# 安装指定版本
nvm install 20.11.0
nvm install lts/*

# 使用指定版本
nvm use 20.11.0

# 列出已安装版本
nvm list

# 设置默认版本
nvm alias default 20.11.0

# 卸载指定版本
nvm uninstall 20.11.0

# 设置国内镜像（加速下载）
nvm node_mirror https://npmmirror.com/mirrors/node/
```

**安装后配置：**

1. 以**管理员身份**运行 PowerShell 或 CMD
2. 安装全局工具：
   ```cmd
   npm install -g npm
   ```

### 方法三：使用包管理器安装

**通过 Winget 安装：**
```powershell
winget install OpenJS.NodeJS.LTS
```

**通过 Chocolatey 安装：**
```powershell
choco install nodejs-lts
```

# Python 安装

## Windows 安装

**官方下载地址：** https://www.python.org/downloads/windows/

**安装步骤：**

1. 下载最新版 Python 安装程序
2. **重要**：运行安装程序时，勾选 **"Add Python to PATH"**
3. 选择 "Install Now" 或 "Customize installation"
4. 验证安装：
   ```cmd
   python --version
   pip --version
   ```

**配置国内镜像源：**
```cmd
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

# Miniconda 安装

Python 环境管理工具。

## Windows 安装

**官方下载地址：** https://docs.conda.io/en/latest/miniconda.html

**安装步骤：**

1. 下载 Windows 安装程序（`Miniconda3-latest-Windows-x86_64.exe`）
2. 运行安装程序：
   - 同意许可协议
   - 选择 "Just Me"（推荐）
   - **勾选 "Add Miniconda3 to my PATH environment variable"**（可选，但推荐）
   - **勾选 "Register Miniconda3 as my default Python"**（可选）
3. 完成安装，验证：
   ```cmd
   conda --version
   ```

**配置国内镜像源：**
```cmd
conda config --set show_channel_urls yes
conda config --add channels conda-forge
conda config --set channel_priority strict
# 注：清华 Anaconda 镜像 2024-08 起已停止同步官方 defaults，改用 conda-forge
conda config --set channel_priority strict
```

**常用 Conda 命令：**
```cmd
# 创建虚拟环境
conda create -n myenv python=3.11

# 激活环境
conda activate myenv

# 退出环境
conda deactivate

# 查看所有环境
conda env list

# 查看已安装包
conda list

# 安装包
conda install numpy pandas

# 导出环境配置
conda env export > environment.yml

# 从配置文件创建环境
conda env create -f environment.yml
```

# Go 安装

## Windows 安装

**官方下载地址：** https://go.dev/dl/
**官方安装文档：** https://go.dev/doc/install

**安装步骤：**

1. 下载 Windows 安装包（`go1.xx.x.windows-amd64.msi`）
2. 运行 `.msi` 安装程序，按向导完成安装
3. 默认会安装到 `C:\Program Files\Go` 并自动配置环境变量
4. **关闭并重新打开所有命令提示符窗口**，使环境变量生效
5. 验证安装：
   ```cmd
   go version
   ```

**配置 Go 环境变量：**
```cmd
# 设置 GOPROXY 加速下载
setx GOPROXY "https://goproxy.cn,direct"

# 开启 Go Modules
setx GO111MODULE "on"

# 设置 GODEBUG（可选，解决某些问题）
setx GODEBUG "asyncpreemptoff=1"
```

**验证环境变量：**
```cmd
go env GOPATH
go env GOPROXY
go env GO111MODULE
```

# GVM (Go Version Manager) 安装

Go 版本管理工具，需要先安装 Go。

## Windows 安装

**注意：** GVM 主要面向 Linux/macOS，Windows 推荐使用官方安装包或 **goenv**（需要 WSL）

**Windows 替代方案 - 使用 Scoop 安装多版本 Go：**

```powershell
# 安装 Scoop（如果未安装）
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex

# 通过 Scoop 安装 Go
scoop install go

# 安装特定版本
scoop install go@1.21.6

# 查看已安装版本
scoop list go

# 切换版本
scoop reset go@1.21.6
```

**或在 WSL 中使用 GVM（推荐）：**
```bash
# 在 WSL Ubuntu 中执行
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
source ~/.gvm/scripts/gvm
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

## Windows 安装

**官方文档：** https://docs.anthropic.com/en/docs/claude-code/overview

**系统要求：**
- Windows 10/11 64位
- 原生安装：无需 Node.js
- npm 方式：Node.js 22+
- Git（建议）

**安装步骤：**

1. 确保前置条件已满足：
   ```cmd
   node --version  # 应显示 v20.x.x 或更高
   git --version   # 应显示版本号
   ```

2. 安装 Claude Code（推荐原生安装器，无需 Node.js）：
   ```powershell
   irm https://claude.ai/install.ps1 | iex
   ```
   > 备选（需 Node.js 22+）：`npm install -g @anthropic-ai/claude-code@latest`

3. 验证安装：
   ```cmd
   claude --version
   ```

4. 首次运行：
   ```cmd
   claude
   # 按提示完成登录授权
   ```

**Windows PowerShell 执行策略设置（如遇到错误）：**
```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**设置 npm 国内镜像（可选）：**
```powershell
npm config set registry https://registry.npmmirror.com
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

## Windows 安装

**GitHub 地址：** https://github.com/openclaw/openclaw

**安装步骤：**

```powershell
# 推荐：一键安装脚本（自动装 Node 并完成配置）
curl -fsSL https://openclaw.ai/install.sh | bash
# 或手动 npm 安装（需 Node 24.16+/26.1+）：npm install -g openclaw@latest

# 或使用 npx 直接运行（无需安装）
npx openclaw
```

**首次运行配置：**
```cmd
openclaw onboard
# 按提示配置 API 密钥等信息
```

**验证安装：**
```cmd
openclaw --version
```

# WSL 安装 (Windows)

**WSL (Windows Subsystem for Linux)** 是 Windows 10/11 上的 Linux 子系统，让你在 Windows 上原生运行 Linux 环境。

**官方文档：** https://learn.microsoft.com/zh-cn/windows/wsl/install
**官方配置指南：** https://learn.microsoft.com/zh-cn/windows/wsl/setup/environment

## 系统要求

- Windows 10 版本 1903 或更高（建议 2004+）
- Windows 11（推荐）
- 64 位处理器，支持虚拟化

## 自动安装（推荐，Windows 10 2004+ / Windows 11）

以管理员身份打开 PowerShell，运行：

```powershell
wsl --install
```

这将默认安装：
- WSL 2
- Ubuntu 最新 LTS 版本

安装完成后**重启电脑**，然后继续：

```powershell
# 设置默认 WSL 版本为 2
wsl --set-default-version 2

# 查看已安装的发行版
wsl --list --verbose

# 进入 Ubuntu
wsl
```

**首次启动设置：**

首次启动新安装的 Linux 分发版时，需要：
1. 等待文件解压完成
2. 设置用户名和密码
3. 记住这是 WSL 内的用户，与 Windows 用户分开

## 手动安装（旧版 Windows）

**步骤1：启用 WSL**

```powershell
# 以管理员身份运行 PowerShell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 启用虚拟机功能
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 重启电脑
```

**步骤2：下载并安装 Linux 内核更新包**

下载地址：https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi

**步骤3：设置 WSL 默认版本**

```powershell
wsl --set-default-version 2
```

**步骤4：安装 Linux 发行版**

1. 打开 Microsoft Store
2. 搜索 "Ubuntu" 或其他发行版（Debian、openSUSE、Kali 等）
3. 点击安装
4. 从开始菜单启动 Ubuntu，设置用户名和密码

## 安装特定发行版

```powershell
# 查看可用发行版
wsl --list --online

# 安装特定发行版
wsl --install -d Ubuntu-22.04
wsl --install -d Debian
wsl --install -d openSUSE-Leap-15.5
wsl --install -d Kali-Linux
```

## 常用命令

```powershell
# 查看所有发行版
wsl --list --verbose
wsl -l -v

# 启动特定发行版
wsl -d Ubuntu-22.04

# 设置默认发行版
wsl --set-default Ubuntu-22.04

# 关闭所有 WSL 发行版
wsl --shutdown

# 终止特定发行版
wsl -t Ubuntu-22.04

# 导出/导入发行版（备份）
wsl --export Ubuntu-22.04 D:\backup\ubuntu.tar
wsl --import Ubuntu-Backup D:\WSL\Ubuntu-Backup D:\backup\ubuntu.tar

# 卸载发行版
wsl --unregister Ubuntu-22.04

# 更新 WSL
wsl --update

# 从 WSL 1 升级到 WSL 2
wsl --set-version Ubuntu-22.04 2

# 在 Windows 中运行 WSL 命令
wsl ls -la
wsl python3 script.py
wsl git commit -m "message"
```

## 配置优化

**创建/编辑配置文件** `C:\Users\用户名\.wslconfig`：

```ini
[wsl2]
# 分配给 WSL 的最大内存
memory=8GB

# 分配给 WSL 的最大处理器核心数
processors=4

# 交换空间大小
swap=4GB

# 不使用时释放内存的时间（毫秒）
localhostForwarding=true

# 是否使用 Windows 的 DNS 服务器
dnsTunneling=true

# 是否启用镜像网络模式（Windows 11 22H2+）
networkingMode=mirrored

# 是否自动释放内存
autoMemoryReclaim=gradual

# 是否启用 GUI 应用支持（WSLg）
guiApplications=true
```

**修改后执行：**
```powershell
wsl --shutdown
# 然后重新启动 WSL
```

**发行版内配置** `/etc/wsl.conf`：

```ini
[boot]
systemd=true

[user]
default=your-username

[interop]
enabled=true
appendWindowsPath=true

[automount]
enabled=true
mountFsTab=true
options="metadata,umask=22,fmask=11"
```

## 在 WSL 中安装开发工具

进入 WSL 后，参考本文的 **Linux 安装** 章节安装 Git、Go、Node.js 等工具。

```bash
# 进入 WSL
wsl

# 更新包管理器
sudo apt update && sudo apt upgrade -y

# 安装基础工具
sudo apt install -y curl wget git vim build-essential

# 安装 Node.js（通过 NVM）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
source ~/.bashrc
nvm install --lts

# 安装 Go
wget https://go.dev/dl/go1.25.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.25.0.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

## WSL 与 Windows 互操作

```bash
# 在 WSL 中访问 Windows 文件（挂载在 /mnt/）
cd /mnt/c/Users/用户名/Desktop

# 在 Windows 中访问 WSL 文件
# 在资源管理器地址栏输入：
\\wsl$\Ubuntu

# 在 WSL 中运行 Windows 程序
notepad.exe file.txt
code .
explorer.exe .

# 在 Windows 中运行 WSL 命令
wsl ls -la
wsl python3 script.py
wsl git commit -m "message"
```

## VS Code 集成 WSL

1. 在 Windows 上安装 VS Code
2. 安装 "WSL" 扩展
3. 在 WSL 终端中运行：
   ```bash
   code .
   ```
4. 或点击 VS Code 左下角 `><` 图标，选择 "Connect to WSL"

**推荐安装的 VS Code 扩展：**
- WSL
- Remote - SSH
- Docker
- GitLens

---

# Docker 安装

## Windows 安装

**官方文档：** https://docs.docker.com/desktop/install/windows-install/

**系统要求：**
- Windows 10/11 64位：专业版、企业版或教育版（Build 19041+）
- 或 Windows 11 家庭版（Build 22000+）
- 启用 WSL 2 或 Hyper-V
- 8GB 内存
- 启用 BIOS/UEFI 硬件虚拟化

**下载地址：** https://www.docker.com/products/docker-desktop/

**安装步骤：**

1. 下载 Docker Desktop for Windows 安装程序
2. 运行安装程序（`Docker Desktop Installer.exe`）
3. 安装向导中选择：
   - **Use WSL 2 instead of Hyper-V**（推荐，性能更好）
   - 或根据需求选择 Hyper-V
4. 完成安装后**重启电脑**
5. 启动 Docker Desktop，等待 "Engine started"
6. 打开 PowerShell 验证：
   ```powershell
   docker --version
   docker-compose --version
   ```

**Docker Desktop 配置：**

1. 右键点击系统托盘 Docker 图标 → Settings
2. **General**：
   - 勾选 "Start Docker Desktop when you log in"
   - 勾选 "Use the WSL 2 based engine"
3. **Resources → WSL Integration**：
   - 启用 "Enable integration with my default WSL distro"
   - 为特定 WSL 发行版启用集成
4. **Docker Engine**：可修改 daemon 配置（镜像加速等）

**配置国内镜像加速：**

1. 点击 Docker 图标 → Settings → Docker Engine
2. 在编辑窗口中添加：
   ```json
   {
     "registry-mirrors": [
       "https://docker.1ms.run",
       "https://docker.xuanyuan.me",
       "https://mirror.baidubce.com"
     ]
   }
   ```
3. 点击 Apply & Restart

**命令行静默安装（可选）：**
```powershell
# 下载安装程序后执行
Start-Process 'Docker Desktop Installer.exe' -Wait install --quiet
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

## Windows 问题

**1. 'git' 不是内部或外部命令**
- 原因：Git 未添加到 PATH
- 解决：重新运行 Git 安装程序，选择 "Git from the command line and also from 3rd-party software"

**2. 'claude' 命令找不到**
```powershell
# 查看 npm 全局安装路径
npm config get prefix

# 将该路径添加到系统 PATH
# 如：C:\Users\用户名\AppData\Roaming\npm
```

**3. NVM 切换版本失败**
- 以管理员身份运行 PowerShell 或 CMD
- 检查是否有其他程序占用了 Node.js

**4. GVM 在 Windows 上无法使用**
- GVM 官方不支持 Windows
- 使用 WSL（Windows Subsystem for Linux）运行 GVM
- 或使用 Scoop 管理 Go 版本

## WSL 问题

**1. WSL 安装失败（Windows）**
```powershell
# 确保虚拟化已启用（BIOS 中）
# 确保 Windows 功能已启用
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 安装 WSL 更新包后重启
```

**2. WSL 2 启动失败**
```powershell
# 检查 Hyper-V 是否启用
# 或检查虚拟机平台
wsl --set-default-version 2

# 查看日志
eventvwr.msc  # Windows 事件查看器
```

**3. WSL 内存占用过高**
```ini
# 创建 C:\Users\用户名\.wslconfig
[wsl2]
memory=4GB
processors=2
autoMemoryReclaim=gradual
```
修改后执行：
```powershell
wsl --shutdown
```

**4. WSL 中无法访问 Windows 文件**
```bash
# Windows 文件挂载在 /mnt/
ls /mnt/c/

# 如无法访问，检查 wsl.conf
cat /etc/wsl.conf
# 确保 automount 已启用
```

**5. Docker Desktop WSL 集成失败**
```powershell
# 确保 WSL 2 已安装
wsl --list --verbose

# 在 Docker Desktop 设置中启用 WSL 集成
# Settings → Resources → WSL Integration

# 重启 WSL
wsl --shutdown
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

# Windows OpenSSH 安装与配置完全指南

> 整理时间：2026-11-22
> 适用系统：Windows 10 / 11 / Server 2012 R2 及以上
> 覆盖场景：服务端安装启动、离线安装、代理环境绕过 Windows Update、配置文件结构、免密登录、WSL 访问排查

---

## 目录

1. [先分清两种场景](#1-先分清两种场景)
2. [安装 OpenSSH 服务端](#2-安装-openssh-服务端)
3. [Add-WindowsCapability 卡住不动的排查](#3-add-windows-capability-卡住不动的排查)
4. [仅有代理可上网的环境怎么处理](#4-仅有代理可上网的环境怎么处理)
5. [版本与架构的选择](#5-版本与架构的选择)
6. [msiexec 的正确用法](#6-msiexec-的正确用法)
7. [SSH 配置文件全清单](#7-ssh-配置文件全清单)
8. [免密登录完整配置](#8-免密登录完整配置)
9. [WSL 访问 Windows 仍要密码的排查](#9-wsl-访问-windows-仍要密码的排查)
10. [启动 / 停止 Windows sshd](#10-启动--停止-windows-sshd)
11. [常见问题速查总表](#11-常见问题速查总表)
12. [附录：关键路径速查](#附录关键路径速查)

---

## 1. 先分清两种场景

| 需求 | 装什么 | 说明 |
|---|---|---|
| 别人 ssh 连进这台 Windows | **OpenSSH 服务端**（sshd） | 本文主线 |
| 从这台 Windows 连出去 | **OpenSSH 客户端**（ssh） | Win10 1809+ 默认已自带 |

客户端检查：

```powershell
ssh -V
# 未安装则：
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

判断是否已有服务端：

```powershell
Get-Service sshd
Get-WindowsCapability -Online -Name OpenSSH.Server*
```

---

## 2. 安装 OpenSSH 服务端

### 2.1 Windows 10 / 11：系统自带功能（FoD）方式

**管理员 PowerShell** 依次执行：

```powershell
# 1. 查看可安装组件（能看到 OpenSSH.Client / OpenSSH.Server 即系统源可用）
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

# 2. 安装服务端
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 3. 启动服务
Start-Service sshd

# 4. 设为开机自启
Set-Service -Name sshd -StartupType Automatic
```

安装过程一般会自动创建防火墙入站规则 `OpenSSH-Server-In-TCP`（放行 TCP 22）。若缺失或被禁用，手动补：

```powershell
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' `
  -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

验证：

```powershell
Get-Service sshd                      # Status 应为 Running
netstat -ano | findstr :22            # 应有 0.0.0.0:22 LISTENING
ssh 用户名@localhost                   # 本机自连，首次问指纹输 yes
```

### 2.2 Windows Server 2012 R2 / 2016 / 2019+

Server 上 `Add-WindowsCapability` 不一定可用，备选方式：

```powershell
# 2019+ 可用
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 上面报错时用 DISM
DISM /Online /Add-Capability /CapabilityName:OpenSSH.Server~~~~0.0.1.0
```

旧版（2012 R2）通常下载官方 `OpenSSH-Win64.zip`，解压到 `C:\Program Files\OpenSSH` 后注册服务：

```powershell
cd "C:\Program Files\OpenSSH"
powershell -ExecutionPolicy Bypass -File install-sshd.ps1
Start-Service sshd
Set-Service sshd -StartupType Automatic
```

### 2.3 推荐方式：官方 MSI / ZIP 离线安装

**这条路完全不依赖 Windows Update**，在代理受限、精简系统、WSUS 环境下最稳，30 秒可完成。

下载地址（浏览器下载即可，无需命令行联网）：

```
https://github.com/PowerShell/Win32-OpenSSH/releases/latest
```

MSI 安装：

```powershell
msiexec /i "$env:USERPROFILE\Downloads\OpenSSH-Win64-v10.4.0.0.msi" /qb /l*v "$env:TEMP\openssh-msi.log"

Start-Service sshd
Set-Service sshd -StartupType Automatic
```

ZIP 安装（无 MSI 安装权限时）：

```powershell
Expand-Archive "$env:USERPROFILE\Downloads\OpenSSH-Win64-v10.4.0.0.zip" `
  -DestinationPath 'C:\Program Files\OpenSSH' -Force
cd 'C:\Program Files\OpenSSH'
powershell -ExecutionPolicy Bypass -File .\install-sshd.ps1
Start-Service sshd
Set-Service sshd -StartupType Automatic
```

> MSI 版与系统 FoD 版功能等价，无需为了 `Get-WindowsCapability` 显示 `Installed` 再去折腾 Windows Update。

---

## 3. Add-WindowsCapability 卡住不动的排查

**根本原因**：OpenSSH 不是镜像内置功能，该命令需要从 `download.windowsupdate.com` 下载 FoD 包。网络、代理、WSUS 策略、`wuauserv` 服务任一环节有问题，就会长时间"转圈"不报错（默认超时 30 分钟以上）。

### 3.1 先另开窗口确认它到底卡在哪

不要强杀原窗口（可能留下 CBS 中间态）。

```powershell
Get-WindowsCapability -Online -Name OpenSSH.Server* | Select Name,State
Get-Service wuauserv,bits,cryptsvc,TrustedInstaller,AppXSvc,ClipSVC | Select Name,Status,StartType
```

| 结果 | 含义 | 下一步 |
|---|---|---|
| `State: Installed` | 其实已装好，只是进度条没刷 | 直接 `Start-Service sshd` |
| `wuauserv` / `bits` 为 Stopped 或 Disabled | 服务被关，卡住主因 | 见 3.2 |
| `NotPresent` 且服务正常 | 策略或网络问题 | 见 3.3 / 3.4 |

### 3.2 Windows Update 相关服务没起来

装 FoD 依赖 `wuauserv`（下载）+ `bits`（传输）+ `cryptsvc`（签名校验）。

```powershell
sc.exe config wuauserv start= demand
sc.exe config bits start= demand
net start cryptsvc
net start bits
net start wuauserv
```

关闭原卡住窗口，重新执行安装。精简版镜像可能已删除相关注册表/dll，只能走 2.3 节离线安装。

### 3.3 被 WSUS / 组策略劫持（企业环境最常见）

组策略把更新源指向内网 WSUS，而 WSUS 未同步该 FoD，表现为无限等待后报 `0x800f0954`。

```powershell
Get-ItemProperty 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate' -EA SilentlyContinue
Get-ItemProperty 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU' -EA SilentlyContinue
```

看到 `WUServer` / `UseWUServer = 1`，两条路：

```powershell
# 方案一：临时绕过 WSUS，直接走公网 WU
Set-ItemProperty 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU' -Name UseWUServer -Value 0
Restart-Service wuauserv

# 方案二：DISM 加 /LimitAccess 绕过
DISM /Online /Add-Capability /CapabilityName:OpenSSH.Server~~~~0.0.1.0 /LimitAccess
```

> 装完记得把 `UseWUServer` 改回 1，否则其他补丁行为也会变化。

### 3.4 有未完成的更新事务 / 组件存储损坏

CBS 存在挂起操作时，任何 Capability 操作都会卡在第一步。

```powershell
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending' -EA SilentlyContinue
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\RebootRequired' -EA SilentlyContinue
```

有输出 → **先重启** 再装。无输出但 CBS 日志有 corruption：

```powershell
DISM /Online /Cleanup-Image /RestoreHealth
sfc /scannow
```

### 3.5 抓日志，不要盲猜

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0 -LogPath C:\openssh.log

Select-String -Path C:\Windows\Logs\CBS\CBS.log  -Pattern 'OpenSSH' | Select-Object -Last 20
Select-String -Path C:\Windows\Logs\DISM\dism.log -Pattern '0x8'     | Select-Object -Last 20
```

---

## 4. 仅有代理可上网的环境怎么处理

### 4.1 为什么"浏览器能上网"但命令还是卡

| 层面 | 代理配置来源 | 受影响的程序 |
|---|---|---|
| WinINET | 代理软件勾"系统代理"写入注册表 | 浏览器、部分 .NET 应用 |
| WinHTTP | `netsh winhttp` 单独配置 | **Windows Update、DISM、`Add-WindowsCapability`**（且以 SYSTEM 身份运行） |

`Add-WindowsCapability` 走的是 WinHTTP 层，且跑在 SYSTEM 会话里，**读不到用户态代理软件设置**，因此在等一条永远连不出去的连接。

### 4.2 方案 A：离线 MSI 安装（推荐，最快最稳）

浏览器能走代理 → 浏览器下载 → 本地安装，完全绕开 WU 链路。见 2.3 节。

### 4.3 方案 B：给 WinHTTP 配代理

```powershell
# 现状（默认多为 Direct access (no proxy server)）
netsh winhttp show proxy

# 方式一：从 IE/系统代理一键导入（代理软件勾了"系统代理"时首选）
netsh winhttp import proxy source=ie

# 方式二：手动指定（Clash 常见 7890，v2rayN 常见 10809，按实际端口）
netsh winhttp set proxy proxy-server="http=127.0.0.1:7890;https=127.0.0.1:7890" bypass-list="<local>;*.local"

# 让 WU 服务重新读网络配置
Restart-Service wuauserv
Restart-Service bits

Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 用完还原
netsh winhttp reset proxy
```

要点：

- `http=` / `https=` 一般写同一个端口（代理软件的 mixed / http 端口）。
- SYSTEM 账户访问 `127.0.0.1` 是允许的（回环不受防火墙限制），前提是代理软件不是只监听某个特定网卡。
- **WinHTTP 不支持 SOCKS 代理**，只能写 HTTP CONNECT 代理；只有 SOCKS 端口时走方案 C。
- `netsh winhttp set proxy` **不支持用户名密码**，需认证的企业代理只能走 TUN 或认证型网关。

### 4.4 方案 C：开 TUN / 全局模式

代理软件开启 **TUN 模式**（需管理员权限）后，系统层截获所有 IP 流量，SYSTEM 下的 Windows Update 服务也会走代理，`Add-WindowsCapability` 即可正常下载。

### 4.5 验证代理链路

```powershell
netsh winhttp show proxy
Test-NetConnection download.windowsupdate.com -Port 443

# 显式用代理发请求（curl.exe 为 Win10 1803+ 自带）
curl.exe -x http://127.0.0.1:7890 -I https://download.windowsupdate.com

# 确认代理监听端口
Get-NetTCPConnection -State Listen |
  Where-Object LocalPort -in 7890,10809,1080,8080,8888 |
  Select LocalAddress,LocalPort,OwningProcess
```

`curl.exe -x` 返回 `HTTP/1.1 200/302` 说明代理可用；提示 `Failed to connect to 127.0.0.1 port 7890` 说明端口写错。

---

## 5. 版本与架构的选择

`Win32-OpenSSH` 是**项目名**（源自 Win32 API），与位数无关。真正的架构看文件名：

| 文件名 | 实际架构 | 适用 |
|---|---|---|
| `OpenSSH-Win64-v10.4.0.0.msi/.zip` | x64 | 64 位系统用这个 |
| `OpenSSH-Win32-v10.4.0.0.msi/.zip` | x86（32 位） | 仅 32 位系统 |
| `OpenSSH-ARM64-...`（如有） | ARM64 | 骁龙 / ARM 设备 |

确认本机架构：

```powershell
$env:PROCESSOR_ARCHITECTURE      # AMD64 = x64, ARM64 = ARM, x86 = 32 位
[Environment]::Is64BitOperatingSystem
```

返回 `AMD64` + `True` → 直接用 Win64 包。

> Win32 包在 x64 上能靠 WoW64 运行，但没必要，且可能遇到 `System32` / `SysWOW64` 路径重定向问题。ARM64 设备不要用 Win64 包。

---

## 6. msiexec 的正确用法

**错误写法**：`msiexec 文件名` —— Windows Installer 会把首个参数当开关解析，返回 `错误 1639：无效命令行参数`。

**正确形式**：先给 `/i`，再给带引号的包路径。

```powershell
msiexec /i "C:\Users\你的账号\Downloads\OpenSSH-Win64-v10.4.0.0.msi" /qn
```

推荐带进度与日志：

```powershell
msiexec /i "$env:USERPROFILE\Downloads\OpenSSH-Win64-v10.4.0.0.msi" /qb /l*v "$env:TEMP\openssh-msi.log"
```

### 常用开关

| 开关 | 作用 |
|---|---|
| `/i <包路径>` | 安装（必需） |
| `/x <包路径>` | 卸载 |
| `/qn` | 全静默，无界面 |
| `/qb` | 仅显示基本进度条 |
| `/qf` / `/qb!` | 完整 / 基本界面，带取消按钮 |
| `/l*v <日志路径>` | 输出详细日志，排错必加 |
| `/norestart` | 抑制安装过程中自动重启 |
| `/p <补丁.msp>` | 打补丁 |

### 两个细节

1. **路径必须加双引号**：`$env:USERPROFILE` 展开后若含空格（如用户名 `Zhang San`、`Program Files`），不加引号会被拆成两个参数。
2. **msiexec 是异步的**：命令返回不等于安装完成。需等待时用：

```powershell
Start-Process msiexec.exe -ArgumentList '/i', "$env:USERPROFILE\Downloads\OpenSSH-Win64-v10.4.0.0.msi", '/qn' -Wait
```

### 退出码对照

| 退出码 | 含义 | 处理 |
|---|---|---|
| 0 | 成功 | — |
| 1603 | 致命错误（权限不足 / 已装同版本 / 安全软件拦截） | 管理员权限运行，查 `/l*v` 日志 |
| 1619 | 包无法打开（文件损坏、下载不完整） | 重新下载，核对文件大小 |
| 1639 | 命令行参数无效 | 漏了 `/i` |
| 3010 | 成功但需重启 | 重启即可 |

---

## 7. SSH 配置文件全清单

Windows 上 OpenSSH 的文件分三处：服务端配置根、用户侧目录、程序目录。

### 7.1 服务端配置根：`C:\ProgramData\ssh\`

| 文件 | 作用 | 能否修改 |
|---|---|---|
| `sshd_config` | 服务端主配置：端口、认证方式、允许用户、子系统 | 可以，主要改这个 |
| `sshd_config_default` | 出厂模板，用于对照/恢复 | 不要动 |
| `ssh_host_rsa_key` + `.pub` | 主机 RSA 密钥对（服务器身份） | 可删让 sshd 重新生成 |
| `ssh_host_ecdsa_key` + `.pub` | 主机 ECDSA 密钥对 | 同上 |
| `ssh_host_ed25519_key` + `.pub` | 主机 Ed25519 密钥对（默认首选） | 同上 |
| `administrators_authorized_keys` | **管理员组**的公钥授权文件（不在用户目录） | 可以，加公钥 |
| `moduli` | DH 密钥交换群参数（对应 Linux `/etc/ssh/moduli`） | 不要动 |
| `ssh_config` | 系统级**客户端**配置（部分版本才放此处） | 可选 |

> OpenSSH 9.x 起默认不再生成 DSA 主机密钥。

**权限是硬要求**，不对会导致 sshd 启动失败或静默忽略文件：

```powershell
# 查看
icacls C:\ProgramData\ssh\ssh_host_ed25519_key
icacls C:\ProgramData\ssh\administrators_authorized_keys

# 修复：主机私钥
icacls C:\ProgramData\ssh\ssh_host_*_key /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"

# 修复：管理员公钥文件
icacls C:\ProgramData\ssh\administrators_authorized_keys /inheritance:r /grant "Administrators:F" /grant "SYSTEM:F"
```

### 7.2 用户侧目录：`%USERPROFILE%\.ssh\`

| 文件 | 作用 |
|---|---|
| `id_rsa` / `id_rsa.pub` | 用户 RSA 密钥对（私钥绝不外传） |
| `id_ed25519` / `id_ed25519.pub` | 用户 Ed25519 密钥对（推荐） |
| `authorized_keys` | 普通用户的公钥授权列表（**服务端侧**使用） |
| `known_hosts` | 连过的主机指纹（**客户端侧**使用，防中间人） |
| `known_hosts.old` | 上一版本备份，`ssh-keygen -R` 重写时生成 |
| `config` | 客户端配置：按 Host 别名预设端口/用户/密钥/跳板 |
| agent 相关 | Windows 上由 `ssh-agent` **服务**托管，不落文件 |

> 注意权限规则与 7.1 相反：普通用户的 `authorized_keys` 只需属主可写，**不能**让 `Administrators` / `SYSTEM` 出现在里面。

### 7.3 程序目录：`C:\Program Files\OpenSSH\`

| 文件 | 作用 |
|---|---|
| `sshd.exe` | 服务端守护进程 |
| `ssh.exe` | 客户端 |
| `ssh-keygen.exe` | 生成/转换密钥 |
| `ssh-agent.exe` / `ssh-add.exe` | 密钥代理 |
| `sftp-server.exe` | SFTP 子系统 |
| `scp.exe` | SCP 传输 |
| `install-sshd.ps1` / `uninstall-sshd.ps1` | zip 版服务注册与卸载脚本 |
| `FixHostFilePermissions.ps1` | 一键修权限（比手动 icacls 省事） |

### 7.4 日志位置（与 Linux 最大差异）

Windows 没有 `/var/log/auth.log`，日志全在事件查看器：

| 位置 | 内容 |
|---|---|
| 事件查看器 → 应用程序和服务日志 → OpenSSH → Operational | 连接、认证、密钥拒绝明细 |
| `C:\Windows\System32\winevt\Logs\OpenSSH%4Operational.evtx` | 上面的实体文件 |

```powershell
Get-WinEvent -LogName 'OpenSSH/Operational' -MaxEvents 10 |
  Select-Object TimeCreated, Id, Message | Format-Table -Wrap
```

### 7.5 速查命令

```powershell
# 服务端家底
Get-ChildItem C:\ProgramData\ssh -Force | Select Name, Length, LastWriteTime

# 用户侧家底
Get-ChildItem "$env:USERPROFILE\.ssh" -Force | Select Name, Length
```

日常真正会动手的只有 4 个：`sshd_config`、`administrators_authorized_keys` 或 `authorized_keys`、`~/.ssh/config`、`known_hosts`。

---

## 8. 免密登录完整配置

以 `Administrator` 账户为例，配置文件位于 `C:\Users\Administrator\.ssh\config`。整套分三步：**本机生成密钥 → 公钥放入目标机 → 写 config**。

### 8.1 生成密钥

```powershell
# 确保目录存在
if (-not (Test-Path "$env:USERPROFILE\.ssh")) { New-Item -ItemType Directory "$env:USERPROFILE\.ssh" }

# 生成 Ed25519 密钥
ssh-keygen -t ed25519 -C "Administrator@win" -f "$env:USERPROFILE\.ssh\id_ed25519"

# 确认产物
Get-ChildItem "$env:USERPROFILE\.ssh" | Select Name, Length
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub"
```

passphrase 处理：
- **想真正免密**：两次直接回车，留空。
- 想更安全：设口令，后续用 `ssh-agent` 托管（见 8.6）。

### 8.2 分发公钥

#### 目标为 Linux 服务器

```powershell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub" |
  ssh root@192.168.1.10 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

或手动两步更保险：

```powershell
scp "$env:USERPROFILE\.ssh\id_ed25519.pub" root@192.168.1.10:/tmp/
ssh root@192.168.1.10 "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat /tmp/id_ed25519.pub >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && rm /tmp/id_ed25519.pub"
```

#### 目标是 Windows 的管理员账户

管理员组公钥**不能**放用户目录，必须放：

```powershell
Add-Content 'C:\ProgramData\ssh\administrators_authorized_keys' 'ssh-ed25519 AAAA... Administrator@win'
icacls 'C:\ProgramData\ssh\administrators_authorized_keys' /inheritance:r /grant 'Administrators:F' /grant 'SYSTEM:F'
Restart-Service sshd
```

#### 目标是 Windows 的普通用户

放 `C:\Users\<该用户>\.ssh\authorized_keys`，权限只留该用户可写。

另外目标机 `sshd_config` 需允许公钥认证：

```
PubkeyAuthentication yes
PasswordAuthentication no      # 建议确认密钥能通之后再关密码
```

### 8.3 config 文件

新建/编辑：

```powershell
notepad "$env:USERPROFILE\.ssh\config"
```

内容示例（支持多 Host）：

```
Host gpu
    HostName 192.168.1.10
    User root
    Port 22
    IdentityFile C:/Users/Administrator/.ssh/id_ed25519
    IdentitiesOnly yes
    ServerAliveInterval 30
    ServerAliveCountMax 3

Host winbox
    HostName 192.168.1.20
    User Administrator
    Port 22
    IdentityFile C:/Users/Administrator/.ssh/id_ed25519
    IdentitiesOnly yes
    StrictHostKeyChecking accept-new

Host *
    AddKeysToAgent yes
    IdentitiesOnly yes
```

字段说明：

| 字段 | 含义 | 注意 |
|---|---|---|
| `Host` | 别名，之后 `ssh gpu` 即可 | 可任意命名 |
| `HostName` | 真实 IP / 域名 | 必填 |
| `User` | 登录用户名 | 必填 |
| `Port` | 端口，改过 22 写实际端口 | 默认 22 可省 |
| `IdentityFile` | 私钥路径 | **Windows 上必须用正斜杠 `/`**，写反斜杠会失败 |
| `IdentitiesOnly yes` | 只用指定密钥 | 防止挨个试密钥触发 `Too many authentication failures` |
| `AddKeysToAgent yes` | 私钥自动加入 agent | 配了 passphrase 才需要 |
| `ServerAliveInterval` | 心跳间隔（秒） | 防 NAT/云环境掉线 |
| `StrictHostKeyChecking accept-new` | 首次自动记指纹，之后强校验 | 比 `no` 安全且少一次交互 |

### 8.4 权限

```powershell
# 私钥权限（出现 UNPROTECTED PRIVATE KEY FILE 就是这步没做）
icacls "$env:USERPROFILE\.ssh\id_ed25519" /inheritance:r /grant "Administrator:F"
```

`config` 本身只需当前用户可写。

### 8.5 测试

```powershell
ssh gpu                                          # 走 config
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" root@192.168.1.10   # 直连验证
ssh -v gpu                                       # 调试，重点看 "Offering public key" / "Server accepts key"
```

成功标志：**没有提示输入密码** 直接进入 shell。Linux 侧日志对应 `Accepted publickey for root`。

### 8.6 带口令密钥 + ssh-agent

```powershell
Set-Service ssh-agent -StartupType Automatic
Start-Service ssh-agent

ssh-add "$env:USERPROFILE\.ssh\id_ed25519"   # 输入一次 passphrase
ssh-add -l                                    # 确认已加载
```

> Windows 的 `ssh-agent` 是系统服务，但密钥**按登录用户隔离**，其他用户会话看不到。

---

## 9. WSL 访问 Windows 仍要密码的排查

WSL 只是客户端，认证完全发生在 Windows 的 sshd 侧。按以下顺序排查。

### 9.1 最高概率：账户属于 Administrators 组

`sshd_config` 末尾默认存在：

```
Match Group administrators
       AuthorizedKeysFile __PROGRAMDATA__/ssh/administrators_authorized_keys
```

含义：**管理员组账户的用户目录 `authorized_keys` 被完全忽略**，只认 `C:\ProgramData\ssh\administrators_authorized_keys`。

```powershell
whoami
whoami /groups | findstr /i "administrators"
```

命中 `BUILTIN\Administrators` → 公钥必须搬到 `C:\ProgramData\ssh\administrators_authorized_keys` 并按 8.2 设置权限。

### 9.2 看日志，定位真因

```powershell
Get-WinEvent -LogName 'OpenSSH/Operational' -MaxEvents 20 |
  Select-Object TimeCreated, Message | Format-List
```

| 日志关键字 | 原因 |
|---|---|
| `Authentication refused: bad ownership or modes for file ...` | 文件权限不对（继承权限混了其他用户） |
| `Failed publickey for Administrator` 后接 `Accepted password` | 公钥未匹配：位置错 / 内容不对 / 行尾问题 |
| 完全没有 `publickey` 记录 | WSL 根本没发出公钥（客户端侧问题，见 9.4） |
| `Accepted publickey for Administrator` | 已成功，看到的密码提示可能是别的 Host/账户 |
| `user ... not allowed because shell ... does not exist` | 该账户登录 shell 配置异常 |

### 9.3 指纹比对（30 秒定位是否同一把钥匙）

WSL 侧：

```bash
whoami                       # 确认是哪个用户跑的 ssh
ls -l ~/.ssh                 # root 跑的话密钥在 /root/.ssh
ssh-keygen -lf ~/.ssh/id_ed25519.pub
# 输出类似 256 SHA256:xxxxxx user@host (ED25519)
```

Windows 侧：

```powershell
ssh-keygen -lf 'C:\ProgramData\ssh\administrators_authorized_keys'
```

**两边 SHA256 指纹必须一致**。不一致说明贴错文件或贴了另一把密钥。

### 9.4 排除 WSL 客户端侧干扰

```bash
ssh -v Administrator@localhost 2>&1 | grep -Ei "Offering|Server accepts|Authentications that can continue|no mutual"
```

| 输出 | 含义 |
|---|---|
| `Offering public key: ED25519 SHA256:xxx` | 客户端已发出公钥 |
| `Server accepts key:` | 服务端认可，进入签名验证 |
| `Authentications that can continue: publickey,password` | 服务端允许公钥，但这把没通过 |
| `no mutual signature algorithm` | 算法不匹配（老 RSA 密钥常见） |
| 完全没有 `Offering` | WSL 里没找到私钥：确认 `whoami`、`~` 指向 |

算法不匹配（OpenSSH 9+ 默认禁 SHA-1），优先换 Ed25519 重新生成；或在 Windows `sshd_config` 加：

```
PubkeyAcceptedAlgorithms +ssh-rsa
HostKeyAlgorithms +ssh-rsa
```

> WSL 里如果用 `sudo` / root 执行 `ssh`，用的是 `/root/.ssh/id_*`，不是你的 `~/.ssh`。

### 9.5 检查文件两处硬伤

**行尾 CRLF / UTF-8 BOM**（用记事本编辑后最常见）：

```powershell
Format-Hex -Path 'C:\ProgramData\ssh\administrators_authorized_keys' -Count 80
```

看开头是否有 `EF BB BF`，行尾是否 `0D 0A`。修复（统一为 LF、无 BOM、单行）：

```powershell
$f = 'C:\ProgramData\ssh\administrators_authorized_keys'
$c = (Get-Content -Raw $f) -replace "`r", '' -replace [char]0xFEFF, ''
$c = ($c -split "`n" | Where-Object { $_.Trim() }) -join "`n"
[System.IO.File]::WriteAllText($f, $c.TrimEnd() + "`n", (New-Object System.Text.UTF8Encoding($false)))
icacls $f /inheritance:r /grant 'Administrators:F' /grant 'SYSTEM:F'
Restart-Service sshd
```

**权限继承未切断**：`icacls` 只 grant 不 `inheritance:r`，父目录继承来的 `Users`、`Authenticated Users` 仍在，sshd 会拒绝。

```powershell
icacls 'C:\ProgramData\ssh\administrators_authorized_keys'
# 健康输出只应有 BUILTIN\Administrators:(F) 和 NT AUTHORITY\SYSTEM:(F)
# 不能出现 (I) 继承标记
```

顺带确认配置未关闭公钥认证：

```powershell
Select-String -Path 'C:\ProgramData\ssh\sshd_config' -Pattern 'PubkeyAuthentication|AuthorizedKeysFile|Match Group|Administrators_authorized_keys'
```

### 9.6 隔离变量

在 **Windows 本机** 直连自身 sshd，绕开 WSL：

```powershell
ssh -v Administrator@localhost
```

- Windows 本机也要密码 → 100% 服务端配置/权限问题。
- Windows 本机免密、WSL 要密码 → 问题在 WSL 的密钥或路径。

### 9.7 排查优先级总结

1. `whoami /groups` —— 管理员账户就把公钥搬到 `C:\ProgramData\ssh\administrators_authorized_keys`
2. `icacls` 切断继承，只留 Administrators + SYSTEM
3. `ssh-keygen -lf` 两端比对指纹
4. `Format-Hex` 查 CRLF / BOM
5. 看 `OpenSSH/Operational` 日志，它会明说"密钥不匹配"还是"权限不对"

> 第 1 步命中率最高，先做这个。

---

## 10. 启动 / 停止 Windows sshd

### 10.1 启动

**管理员 PowerShell**：

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
Get-Service sshd | Select Name, Status, StartType
```

### 10.2 几种启动方式

| 方式 | 命令 |
|---|---|
| PowerShell（推荐） | `Start-Service sshd` |
| cmd / 老脚本 | `net start sshd` 或 `sc start sshd` |
| 图形界面 | `services.msc` → `OpenSSH SSH Server` → 右键启动，并改"启动类型"为自动 |
| 临时调试（前台运行看输出） | `& 'C:\Program Files\OpenSSH\sshd.exe' -d` |

> 必须**管理员权限**，普通用户执行会报"拒绝访问 / Access is denied"。

### 10.3 验证

```powershell
netstat -ano | findstr :22                 # 0.0.0.0:22 LISTENING
Test-NetConnection localhost -Port 22      # TcpTestSucceeded: True
ssh Administrator@localhost                # 自连测试
```

### 10.4 停止 / 重启

```powershell
Stop-Service sshd
Restart-Service sshd -Force      # 服务卡在 Stopping 状态时有用
```

### 10.5 起不来时的排查

**其一：服务根本不存在** → 没装成功：

```powershell
Get-Service sshd -ErrorAction SilentlyContinue
Test-Path 'C:\Program Files\OpenSSH\sshd.exe'
```

**其二：看事件日志**（Windows 的 sshd 报错写入事件日志）：

```powershell
Get-WinEvent -LogName Application -MaxEvents 20 |
  Where-Object ProviderName -eq 'sshd' |
  Select-Object TimeCreated, Message | Format-List
```

| 报错 | 原因 / 处理 |
|---|---|
| `sshd: no hostkeys available` | `ssh_host_*_key` 缺失或权限错，`ssh-keygen -A` 重新生成 |
| `Bad configuration option` / `sshd_config line N` | 配置语法错，用 `sshd.exe -T` 定位 |
| `bind: Address already in use` | 22 端口被占，换端口或杀进程 |
| `UNPROTECTED PRIVATE KEY FILE` | 私钥权限混了其他用户，用 icacls 修 |

**其三：先验证配置语法**（不实际起服务，只解析）：

```powershell
& 'C:\Program Files\OpenSSH\sshd.exe' -T
```

能打印最终生效配置即语法正常；报错会指到具体行。

### 10.6 端口被占用

```powershell
netstat -ano | findstr :22          # 取最后一列 PID
Get-Process -Id <PID> | Select Name, Path
```

要么停掉占用方（常见 VMware、IIS、某些远程工具），要么改端口：

```
# C:\ProgramData\ssh\sshd_config
Port 2222
```

```powershell
Restart-Service sshd
New-NetFirewallRule -Name sshd-2222 -DisplayName 'OpenSSH 2222' `
  -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 2222
```

---

## 11. 常见问题速查总表

| 现象 | 原因 | 处理位置 |
|---|---|---|
| `Add-WindowsCapability` 长时间不动 | 在等 WU 下载，网络/代理不通 | 第 3、4 节 |
| 报 `0x800f0954` | WSUS 策略拦截 | 3.3 |
| 报 `0x800f0906` / `0x800f081f` | 找不到源包 | 挂同版本 ISO 指定 `-Source`，或走 2.3 |
| 报 `0x800f0805` | 系统不识别该功能名（Server 旧版） | 换 DISM 名称或官方 MSI |
| `msiexec` 报 1639 | 命令行少了 `/i` | 第 6 节 |
| `msiexec` 报 1603 | 权限不足 / 已装同版本 / 安全软件拦截 | 管理员权限 + 查日志 |
| `Start-Service sshd` 失败 | 端口占用 / 配置语法错 / 权限错 | 10.5 |
| `no hostkeys available` | 主机密钥缺失 | `ssh-keygen -A` |
| 免密登录仍要密码（Administrator） | 公钥放错位置 | 放 `administrators_authorized_keys`，见 9.1 |
| 免密登录仍要密码（普通用户） | 文件权限含其他用户 | icacls 收紧 |
| `UNPROTECTED PRIVATE KEY FILE` | 私钥权限不对 | 8.4 |
| `Too many authentication failures` | 客户端试遍所有密钥 | config 加 `IdentitiesOnly yes` |
| `no matching host key type found` | 老客户端 + OpenSSH 9+ 禁 SHA-1 | 9.4 的算法降级 |
| 公钥贴了却不生效 | 记事本编辑导致 CRLF/BOM | 9.5 |
| config 里 `IdentityFile` 不生效 | 路径用了反斜杠 | 改正斜杠 |
| 外网连不上但本机 `localhost` 可连 | 安全组 / 端口映射 / 防火墙"公用"配置文件 | 放行规则或改端口 |
| 家庭宽带 22 端口不通 | 运营商封 22 | 换 2222 等高位端口 |

---

## 附录：关键路径速查

| 类别 | 路径 |
|---|---|
| 服务端配置 | `C:\ProgramData\ssh\sshd_config` |
| 管理员公钥 | `C:\ProgramData\ssh\administrators_authorized_keys` |
| 普通用户公钥 | `C:\Users\<用户>\.ssh\authorized_keys` |
| 主机私钥 | `C:\ProgramData\ssh\ssh_host_<算法>_key` |
| 客户端配置 | `C:\Users\<用户>\.ssh\config` |
| 客户端私钥 | `C:\Users\<用户>\.ssh\id_ed25519` |
| 已知主机指纹 | `C:\Users\<用户>\.ssh\known_hosts` |
| 程序目录 | `C:\Program Files\OpenSSH\` |
| 运行日志 | 事件查看器 → Applications and Services → OpenSSH → Operational |
| 默认 Shell 注册表 | `HKLM:\SOFTWARE\OpenSSH` → `DefaultShell` |

### 一次性环境自检脚本

```powershell
Write-Host '=== 服务状态 ==='
Get-Service sshd,ssh-agent,wuauserv,bits -ErrorAction SilentlyContinue | Select Name,Status,StartType

Write-Host '=== 监听端口 ==='
netstat -ano | findstr :22

Write-Host '=== 当前用户与管理员组 ==='
whoami
whoami /groups | findstr /i "administrators"

Write-Host '=== 服务端文件 ==='
Get-ChildItem C:\ProgramData\ssh -Force -ErrorAction SilentlyContinue | Select Name,Length

Write-Host '=== 用户侧文件 ==='
Get-ChildItem "$env:USERPROFILE\.ssh" -Force -ErrorAction SilentlyContinue | Select Name,Length

Write-Host '=== 最近 SSH 日志 ==='
Get-WinEvent -LogName 'OpenSSH/Operational' -MaxEvents 5 -ErrorAction SilentlyContinue |
  Select-Object TimeCreated, Message | Format-List
```

---

---

**文档说明**：全文由实际对话过程整理。命令默认在 Windows 管理员 PowerShell 下执行（特别标注 WSL / Linux 侧除外）。第一部分为本次 remote-win 实操的逐项记录；第二部分（Windows OpenSSH 指南）为技术参考，其中部分内容由豆包 AI 生成。
---

> 文档整理：2026-11-22。命令均按对应平台的默认终端执行。
> 通用内容（参考链接、安装检查清单、Docker 常用命令等）在各平台版本均保留。
