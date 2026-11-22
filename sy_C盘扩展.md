# remote-win 主机 C 盘扩展全记录

> **整理时间**：2026-11-22
> **操作对象**：`remote-win`（Windows 主机，Administrator，192.168.x.x；Windows 11，16 GB 内存）
> **文档性质**：只读分析 + 用户逐项授权下的分区/迁移操作，全程未破坏系统
> **文档状态**：全部操作已完成；SSH 教学部分已移至《开发环境安装指南-Windows.md》
> **来源**：《2026-11-22 remote-win 磁盘空间分析与分区调整》实操记录

## 总目录

- 一、背景与连接环境 —— 问题 1～3（SSH 连接方式、命令引号、超时）
- 二、阶段一：D 盘"没文件却占空间"问题分析
- 三、阶段二：清空 D 盘 —— 问题 4（pagefile）、问题 5（sshd 服务缺失）
- 四、阶段三：删除 D 盘并扩展 C 盘
- 五、阶段四：C 盘空间清理分析
- 六、阶段五：重新分出 D 盘 —— 问题 6（参数名）、问题 7（空间扩展方向）
- 七、最终状态
- 八、遇到的问题速查表
- 九、可复用的命令/脚本位置

---

## 实操记录 —— remote-win 磁盘空间分析与分区调整

主线：分析 D 盘"没文件却占空间" → 清空 D 盘（迁移 pagefile）→ 删除 D 扩展 C → 重新分出 D 盘备用。全程遇到 7 个技术问题，其中 SSH 服务问题最曲折。

---

### 一、背景与连接环境

`~/.ssh/config` 中已配置 `remote-win`：

```
Host remote-win
    HostName 192.168.x.x
    User Administrator
```

目标机器是 **Windows（cmd.exe / PowerShell）**，不是 Linux。这点很关键：所有磁盘操作都要用 Windows 的方式（PowerShell / diskpart / bcdedit），不能用 Linux 工具。

#### 问题 1：无法非交互 SSH（没有 sshpass / paramiko）

**现象**：本机（WSL，普通用户 youruser，无 sudo/pip）没有 `sshpass`，Python 没有 `paramiko`，无法在非交互脚本里传密码。

**解决**：本机有 `expect`。写了一个 expect 脚本，spawn ssh 后自动匹配 password 提示并发送密码：

```tcl
#!/usr/bin/expect -f
set timeout 30
set cmd [lindex $argv 0]
spawn ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null remote-win $cmd
expect {
    -re ".*password.*:" { send "<密码>\r"; exp_continue }
    -re ".*Password.*:" { send "<密码>\r"; exp_continue }
    eof
}
```

（密码不在此文档明文记录，放在脚本里由本人保管。）

#### 问题 2：PowerShell 命令被 bash / 引号层层破坏

**现象**：直接把 PowerShell 命令通过多层引号传进去，出现两类问题：

- bash 把 `$_` 当成变量展开，导致 `if(!$_.PSIsContainer)` 变成 `if(!].PSIsCon...)`，语法报错
- cmd.exe 的编码导致中文目录名显示成乱码（`吖吖` / `߹߹`）

**解决**：用 **base64 UTF-16LE 编码 PowerShell 命令**，再通过 `powershell -EncodedCommand <base64>` 执行。base64 串里没有任何特殊字符，彻底绕开引号地狱：

```bash
ENC=$(python3 -c 'import base64,sys; print(base64.b64encode(sys.argv[1].encode("utf-16-le")).decode())' "$1")
/tmp/ssh_sy.exp "powershell -NoProfile -EncodedCommand $ENC"
```

中文乱码则在脚本开头加 `[Console]::OutputEncoding=[Text.Encoding]::UTF8` 解决。

#### 问题 3：递归扫描大目录触发 expect 超时

**现象**：递归统计 `C:\Windows` 等大目录，30 秒超时导致无输出。

**解决**：单独做一份超时 300 秒的 expect 脚本（`ssh_sy_long.exp` / `sy_ps_long.sh`）专门跑慢命令。

---

### 二、阶段一：D 盘"没文件却占空间"问题分析

#### 现象
D 盘 251 GB，已用 4.2 GB，但资源管理器里看不到文件。

#### 数据
| 项目 | 大小 | 说明 |
|---|---|---|
| `D:\pagefile.sys` | 3456 MB（3.4 GB） | 虚拟内存页面文件，隐藏 + 受保护系统文件 |
| NTFS MFT 元数据 | 692 MB | 文件系统账本，随文件增删增长，不会自动缩小 |
| `D:\吖吖\` | 6.6 MB | 唯一实际数据（OpenSSH msi 安装包） |
| 保留区等 | ~17 MB | — |

#### 结论
"消失"的空间是 **pagefile.sys（3.4 GB）+ MFT（0.7 GB）**，都是隐藏系统文件，默认不显示。pagefile 之所以在 D 盘，是因为 C 盘当时只剩 4 GB，Windows 自动管理模式下把页面文件放到了空间充裕的 D 盘。实际 pagefile 当前只用到 ~500 MB，但系统一次性预留了 3.4 GB。

---

### 三、阶段二：清空 D 盘（为扩展 C 盘做准备）

用户目标：把 D 盘清空并删除，空间并入 C 盘。用户逐项授权：pagefile 迁到 **E 盘**，先只做"备份 + 迁移"两步，暂不删分区。

#### 前置检查
- 磁盘布局：Disk 0（GPT）上有 EFI(0.1) / MSR(0.02) / C(80) / **D(251)** / E(130.8) / 3 个 Recovery 分区；Disk 1（MBR）是独立的 F 盘。
- **D 盘正好紧邻 C 盘之后** → 删掉 D 后空间可直接被 C 扩展。
- BitLocker 全部关闭，无重定向点。
- C 盘当前最大只能扩到 85.9 GB（因为 D 挡在后面，无相邻空闲空间）。

#### 步骤 1：备份文件
把 `D:\吖吖\OpenSSH-Win64-v10.4.0.0.msi`（6.6 MB）复制到 `E:\D_backup\`。

#### 问题 4：页面文件改位置时把注册表 PagingFiles 清空了

**现象**：原配置是自动管理（`?:\pagefile.sys`）。执行：

```powershell
$cs = Get-CimInstance Win32_ComputerSystem
$cs.AutomaticManagedPagefile = $false
Set-CimInstance -InputObject $cs
Get-CimInstance Win32_PageFileSetting | Remove-CimInstance   # 清掉了旧设置
New-CimInstance -ClassName Win32_PageFileSetting -Property @{Name="E:\pagefile.sys";InitialSize=3456;MaximumSize=3456}  # 报错
```

`New-CimInstance` 因 `InitialSize` 报错失败，结果旧设置被删、新设置没建成，注册表 `PagingFiles` 变成空 —— 系统可能没有页面文件了。

**解决**：直接用注册表写入，绕过 WMI：

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" `
  -Name PagingFiles -Value @("E:\pagefile.sys 3456 3456") -Type MultiString
```

修复后 `reg query` 显示 `E:\pagefile.sys 3456 3456`，WMI 也能读到正确设置。

#### 步骤 2：重启释放 pagefile
`shutdown /r /t 20` 重启。第一次 `/t 15` 的命令似乎没生效（ LastBootUpTime 没变），重发一次后主机正常重启。

#### 问题 5（严重）：重启后 SSH 连不上 —— "找不到 sshd 服务"的完整排查与手动注册

这是本次操作中最曲折的问题。单独展开记录，因为"找不到服务"和"手动注册"两步都容易踩坑。

##### 现象

重启后 SSH 彻底连不上：

- `ping 192.168.x.x` 通，445 / 135 / 139 端口都开
- **唯独 22 端口关闭**（`ssh` 报 `Connection refused`）
- 本机（WSL）没有 smbclient / impacket / WinRM，**没有任何远程通道能拉起服务**，只能指导用户在本机操作

##### 第一轮：确认服务是否存在

让用户在本机管理员 PowerShell 执行：

```powershell
Get-Service | Where-Object { $_.Name -match 'ssh' -or $_.DisplayName -match 'ssh' } |
  Format-Table Name,DisplayName,Status,StartType -AutoSize
```

输出**只有一行**：

```
ssh-agent   OpenSSH Authentication Agent   Running   Automatic
```

用户尝试启动服务，直接报错：

```powershell
Start-Service sshd
# Start-Service : 找不到任何服务名称为“sshd”的服务。
#    + CategoryInfo : ObjectNotFound: (sshd:String) [Start-Service], ServiceCommandException
#    + FullyQualifiedErrorId : NoServiceFoundForGivenName,Microsoft.PowerShell.Commands.StartServiceCommand
```

##### 根因：只有 ssh-agent，没有 sshd

- **ssh-agent** = SSH **客户端**的密钥代理服务，只负责保管私钥，跟"别人能不能连进来"无关。
- **sshd** = SSH **服务端**守护进程，监听 22 端口、接受连接。这台机器上**根本没有注册这个服务**。
- 重启之前之所以能连，是因为 `sshd.exe` 曾经被**手动当进程跑起来**过（D 盘还留着 `OpenSSH-Win64-v10.4.0.0.msi` 安装包，说明是手工装的），进程随重启消失，且从未注册为服务。
- 核心教训：**下载/放着 msi 安装包 ≠ 安装了服务**。必须运行安装包或手动注册服务，才会创建 sshd 服务项。

##### 第二轮：用系统功能安装，仍找不到服务

指导用户用 Windows 自带功能安装服务端：

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

返回成功：

```
Path          :
Online        : True
RestartNeeded : False
```

但紧接着 `Start-Service sshd` **仍报"找不到服务"**。

**原因**：`Add-WindowsCapability` 只是把 `sshd.exe` 等**文件**装进了 `C:\Windows\System32\OpenSSH\`，**没有创建 Windows 服务注册项**。服务是否存在，看的是注册表 `HKLM\SYSTEM\CurrentControlSet\Services\sshd` 这个键，而不是文件在不在。装了文件 ≠ 注册了服务，这是最容易卡住的地方。

##### 修复：手动注册 sshd 服务（实际采用 sc.exe）

在管理员 PowerShell / cmd 执行（**注意每个 `=` 后面都要有一个空格，这是 sc.exe 的硬性语法**）：

```powershell
sc.exe create sshd binPath= "C:\Windows\System32\OpenSSH\sshd.exe" start= auto DisplayName= "OpenSSH SSH Server"
Start-Service sshd
Get-Service sshd
```

返回 `Running` 后，22 端口恢复，SSH 连回。

##### 注册表层面：手动注册到底改了什么（重点）

`sc.exe create` 的本质，就是在注册表创建服务项：

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\sshd
```

关键值如下（这也是判断"服务到底注册没"的权威位置）：

| 值名 | 类型 | 实际值 | 含义 |
|---|---|---|---|
| `ImagePath` | REG_EXPAND_SZ | `"C:\Windows\System32\OpenSSH\sshd.exe"` | 服务启动时执行的程序路径 |
| `DisplayName` | REG_SZ | `OpenSSH SSH Server` | 服务显示名 |
| `Start` | REG_DWORD | `2` | 启动类型：**2=自动**，3=手动，4=禁用 |
| `Type` | REG_DWORD | `16` (0x10) | 独立进程服务 |
| `ObjectName` | REG_SZ | `LocalSystem` | 运行账户（sshd 默认以 LocalSystem 运行） |
| `ErrorControl` | REG_DWORD | `1` | 启动失败时记录日志并继续 |

如果不想用 sc.exe，也可以直接用 `.reg` 文件写入同样的注册表项（保存为 `register-sshd.reg` 双击导入，需管理员）：

```reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\sshd]
"DisplayName"="OpenSSH SSH Server"
"ImagePath"="\"C:\\Windows\\System32\\OpenSSH\\sshd.exe\""
"ObjectName"="LocalSystem"
"Start"=dword:00000002
"Type"=dword:00000010
"ErrorControl"=dword:00000001
```

> `.reg` 里 `ImagePath` 的双引号要用 `\"` 转义，路径反斜杠要写双份 `\\`。导入后用 `Start-Service sshd` 或重启生效。

**验证服务是否真正注册**（三种方式等价）：

```powershell
Get-Service sshd                                  # 有输出 = 已注册
Test-Path 'HKLM:\SYSTEM\CurrentControlSet\Services\sshd'   # True = 注册表项存在
sc.exe query sshd                                 # 显示 SERVICE_RUNNING / STOPPED
```

##### 经验与预防

- 这台机器的 OpenSSH 是"临时拼起来"的（msi 装过但服务没注册，capability 又只补了文件），**重启后不会自动起**。
- 确认开机自启：`Set-Service sshd -StartupType Automatic`（等价于把注册表 `Start` 设为 2）。
- 以后若再连不上，先在本机跑上面"验证服务是否真正注册"的命令：
  - 服务项不存在 → 用 `sc.exe create`（或 .reg 导入）重建服务；
  - 服务项存在但 Stopped → `Start-Service sshd`；
  - 都正常但 22 不通 → 查防火墙规则和 `netstat -ano | findstr :22`。

#### 本阶段验证结果
- pagefile 已迁到 `E:\pagefile.sys`（3456 MB），D 盘 pagefile 消失；重启后 `Get-CimInstance Win32_PageFileUsage` 确认生效。
- D 盘已用从 4.2 GB 降到 0.83 GB（只剩 MFT 元数据和空文件夹）。
- 备份文件在 `E:\D_backup\`。

#### 补充：本地电脑 vs remote-win 的 pagefile 位置对比
后续用户确认了两台机器的最终安排，一并记录：

| 机器 | pagefile 位置 | 大小 | 管理模式 | 实际使用 |
|---|---|---|---|---|
| 本地电脑（WSL 宿主） | `C:\pagefile.sys` | 2048 MB | 自动管理（`?:\pagefile.sys`） | 129 MB |
| remote-win | `E:\pagefile.sys` | 3456 MB | 手动指定 | ~500 MB |

结论：两边都没问题。pagefile 放哪个盘系统都能正常跑，只有物理内存耗尽、频繁换页时位置才影响性能。两台机器实际用量都远低于物理内存（本地 32 GB / remote-win 16 GB），无需刻意优化。硬性要求只有一个——不能放在可移动介质（U 盘/移动硬盘）上。

#### 附带澄清：系统到底在哪个盘
用户疑问"系统不是应该在 C 盘吗"。验证：

- Windows 本体：`C:\WINDOWS`，BCD 当前启动项 `device=partition=C:`
- 启动分区：Disk 0 开头 0.1 GB EFI 分区
- WinRE 恢复环境：独立的 12 GB Recovery 分区（`harddisk0\partition7`）

pagefile 是虚拟内存文件，**可以放任意分区，不代表系统在那**。删 D 不影响开机和恢复。另发现一条无效的 "Winclone" 启动项指向 `D:\WCOS\wc10pe.wim`，但该路径不存在，属残留项，无害。

---

### 四、阶段三：删除 D 盘并扩展 C 盘

用户授权执行。安全检查：pagefile 确认不在 D 盘。

#### 如何确定各个盘的分区（关键决策依据）

执行前先用下面的命令摸清磁盘布局，再据此决定"删哪个、能不能扩、扩多少"：

```powershell
Get-Disk | Select Number,PartitionStyle,Size,OperationalStatus
Get-Partition -DiskNumber 0 | Select DiskNumber,PartitionNumber,DriveLetter,Type,Size,Offset
```

当时拿到的布局：

| Disk | Part | 盘符 | Type | Size | Offset（起始位置） |
|---|---|---|---|---|---|
| 0 | 1 | — | System (EFI) | 0.1 GB | 0 |
| 0 | 2 | — | Reserved (MSR) | 0.02 GB | 0.1 |
| 0 | 3 | **C** | Basic | 80 GB | 0.11 |
| 0 | 4 | **D** | Basic | 251.15 GB | **80.11** |
| 0 | 5 | E | Basic | 130.8 GB | 332.63 |
| 0 | 6/7/8 | — | Recovery | 0.5 / 12 / 1 GB | 463.44… |
| 1 | 1 | F | IFS | 476.94 GB | 0（独立物理盘） |

四个判断依据：

1. **确定"删哪个分区"——盘符到分区号的映射**：`DriveLetter = D` 对应 `Disk 0 / Partition 4`，所以删除命令写死 `DiskNumber 0 -PartitionNumber 4`，不会误删别的盘。
2. **确定"能不能扩展 C"——用 Offset 验证相邻性**：Windows 只能把分区向**右**扩展进紧邻的空闲空间。C 的结束位置 = Offset(0.11) + Size(80) = **80.11 GB**，恰好等于 D 的起始位置 **80.11 GB**，两者完全衔接、中间无其他分区。所以删掉 D 后，C 右侧就是连续空闲块，可直接扩展。若中间夹着 E 或恢复分区则不能直接扩——这也解释了阶段五"再给 D 加 40 GB"为何不能用同样办法（那 40 GB 落在 D 的左侧）。
3. **确定"哪些分区不能碰"——Type 字段**：`System`（EFI 启动）、`Reserved`（MSR）、`Recovery`（WinRE）都是系统分区，全部保留；只有 `Type = Basic` 且带盘符的才是数据盘。另用 `bcdedit` 和 `reagentc /info` 确认启动项在 C、WinRE 在分区 7，都不在 D，排除删 D 破坏开机的可能。
4. **确定"扩展上限"——让系统算而不是手算**：不手写目标容量，用 `Get-PartitionSupportedSize` 的 `SizeMax` 作为上限，避免算错或少扩。

删除前的三项安全检查：pagefile 已迁到 E（`Win32_PageFileUsage` 只剩 `E:\pagefile.sys`），D 盘不再被系统占用；D 盘文件已备份到 `E:\D_backup`；全盘 BitLocker 关闭、无 junction/重定位点。

#### 执行命令

```powershell
Remove-Partition -DiskNumber 0 -PartitionNumber 4 -Confirm:$false
$max = (Get-Partition -DiskNumber 0 -PartitionNumber 3 | Get-PartitionSupportedSize).SizeMax
Resize-Partition -DiskNumber 0 -PartitionNumber 3 -Size $max
```

**结果**：C 盘 **80 GB → 332.52 GB**（可用 256.4 GB）。D 盘分区删除。不用重启，在线生效。

此时全机空间：C 332.52（可用 256.39）/ E 130.8（可用 102.57）/ F 476.93（可用 95.1），**合计 940.25 GB，可用 454.06 GB**。

---

### 五、阶段四：C 盘空间清理分析（用户担心再次爆红）

用户诉求：C 盘曾爆红，想看看哪些能清。

#### C 盘已用 76 GB 的构成
| 目录 | 大小 |
|---|---|
| Users（用户数据） | 41.25 GB |
| Windows（其中 WinSxS 8.93 GB 不可删） | 24.21 GB |
| Program Files | 8.46 GB |
| Program Files (x86) | 5.94 GB |
| ProgramData | 2.22 GB |

#### 应用数据大头（Users\Administrator\AppData 下）
| 应用 | 位置 | 大小 | 能否直接删 |
|---|---|---|---|
| 微信 UWP | Local\Packages\...WeChat...\LocalCache\Roaming\Tencent\WeChatAppStore\... | 16.42 GB | ❌ 聊天文件/数据，删了丢记录 |
| 豆包 | Local\Doubao | 4.28 GB | ⚠️ 应用内清理 |
| 飞书 | Roaming\LarkShell | 3.86 GB | ⚠️ 应用内清理 |
| WPS | Roaming\kingsoft + Local\Kingsoft | 3.56 + 1.39 GB | ⚠️ 应用内清理 |
| 360 浏览器 | Roaming\360se6 | 3.18 GB | ✅ 设置里清浏览数据 |
| Tencent | Roaming + Local | ~1.9 GB | ⚠️ 应用数据 |
| 百度网盘 | Roaming\baidu | 0.79 GB | ⚠️ 应用数据 |

用户文档：Downloads 1.8 GB / Documents 0.52 GB / Desktop 0.08 GB。

#### 系统级缓存结论
Temp、更新缓存、回收站、Windows.old 基本都是 0 —— **之前已清过**。跑了 `cleanmgr /sagerun:99`（安全项）后可用空间几乎无变化（256.38 → 256.37 GB），印证系统很干净。

#### 清理原则（重要）
真正占空间的是**应用数据 30+ GB**，但**不能直接删**（会丢聊天记录/文件）。按用户另一台机器的迁移清单经验（`F:\2026_11_19-迁移清单.md`），正确做法是 **junction（目录联接）迁到 D 盘**：原路径不变，应用照常用，数据实际放 D 盘。

junction 注意事项（摘自迁移清单）：
- 删除 junction **绝对不要加 `/S`**：`rmdir "原路径"` 只删链接，`rmdir /S` 会顺着链接把 D 盘真实数据删掉。
- 目标盘必须常驻内置 NTFS，不能是移动盘，不能改盘符。
- 应用大版本更新可能在 C 盘原路径重建真实目录使 junction 失效，需重新 junction。

---

### 六、阶段五：重新分出 D 盘

用户决定先分出一个 D 盘（准备用来做 junction 迁移），建议 150 GB。

#### 步骤
```powershell
# 1. C 盘压缩 150 GB（332.52 → 182.52）
Resize-Partition -DriveLetter C -Size ($p.Size - 150*1GB)

# 2. 在腾出的空间新建分区
New-Partition -DiskNumber 0 -UseMaximumSize -DriveLetter D

# 3. 格式化
Format-Volume -DriveLetter D -FileSystem NTFS -NewFileSystemLabel "Data" -Confirm:$false
```

#### 问题 6：New-Partition 参数名错误

**现象**：`-AssignLetter D` 报错 `ParameterBindingException: NamedParameterNotFound`。

**解决**：正确参数是 `-DriveLetter D`（不是 `-AssignLetter`）。修正后 D 盘创建成功：150 GB，NTFS，卷标 Data。

#### 问题 7：C 盘再空 40 GB 给 D —— 空间在左侧无法直接扩展

**难点**：压缩 C 产生的空闲空间落在 **C 和 D 之间（D 的左侧）**，而 Windows 内置工具只能向**右**扩展分区，无法直接把左侧空间给 D。

**解决**（D 盘为空，无数据风险）：
1. 确认 D 盘为空（排除 `System Volume Information` 和 `$RECYCLE.BIN` 这两个系统目录后，实际文件数 0）
2. `Resize-Partition -DriveLetter C -Size (C.Size - 40*1GB)` → C 压到 142.52 GB
3. `Remove-Partition -DriveLetter D` → 删掉空 D
4. `New-Partition -DiskNumber 0 -UseMaximumSize -DriveLetter D` → 用连续的 150+40=190 GB 重建
5. `Format-Volume ... -FileSystem NTFS`

**结果**：C **142.52 GB**（可用 66.38）/ D **190 GB**（可用 189.9）。

---

### 七、最终状态

| 盘 | 总容量 | 可用 | 说明 |
|---|---|---|---|
| C | 142.52 GB | 66.38 GB | 系统盘 |
| D | 190 GB | 189.9 GB | NTFS，Data，准备做 junction 迁移目标 |
| E | 130.8 GB | 102.57 GB | pagefile 在此 |
| F | 476.93 GB | 95.1 GB | 独立物理盘 |

磁盘 0 布局：EFI(0.1) → MSR(0.02) → C(142.52) → D(190) → E(130.8) → Recovery(0.5/12/1)。系统启动、WinRE 均正常。

**待办**：C 盘应用数据（微信 16.4 GB、豆包 4.28 GB、飞书 3.86 GB、WPS 等共 30+ GB）尚未迁移。按迁移清单方法用 junction 迁到 D 盘即可彻底给 C 盘腾空间。

---

### 八、遇到的问题速查表

| # | 问题 | 解决 | 对应章节 |
|---|---|---|---|
| 1 | 无 sshpass/paramiko，无法非交互 SSH | expect 脚本自动发密码 | 一、问题 1 |
| 2 | 多层引号导致 PowerShell 语法错、中文乱码 | base64 UTF-16LE + `-EncodedCommand`；UTF8 输出编码 | 一、问题 2 |
| 3 | 递归扫描大目录超时 | 单独做 300 秒超时的 expect 脚本 | 一、问题 3 |
| 4 | 改 pagefile 把注册表 PagingFiles 清空 | 直接 `Set-ItemProperty -Type MultiString` 写回 | 三、问题 4 |
| 5 | 重启后 SSH 连不上（sshd 服务不存在，含 Capability 装了仍无服务） | `sc.exe create sshd ...` 或 `.reg` 导入注册，再启动 | 三、问题 5 |
| 6 | `New-Partition -AssignLetter` 参数不存在 | 改用 `-DriveLetter` | 六、问题 6 |
| 7 | C 盘左侧空闲空间无法直接扩给 D | 删空 D 后用连续空间重建更大的 D | 六、问题 7 |

---

### 九、可复用的命令/脚本位置

本机（WSL）：
- `/tmp/ssh_sy.exp`（30 秒超时）、`/tmp/ssh_sy_long.exp`（300 秒超时）
- `/tmp/sy_ps.sh`、`/tmp/sy_ps_long.sh`（base64 编码 + 执行 PowerShell）

远程 Windows：
- 页面文件：`HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management\PagingFiles`
- sshd 服务：`C:\Windows\System32\OpenSSH\sshd.exe`，用 `sc.exe create` / `Set-Service -StartupType Automatic` 管理
- 磁盘：`Get-Partition` / `Resize-Partition` / `New-Partition -DriveLetter` / `Format-Volume`

> 注意：本机临时脚本在 `/tmp`，重启可能丢失，需要时按上文重建。

---
