# WSL 从 C 盘迁移到 F 盘 — 完整说明

> 操作日期：2026-11-18
> 迁移对象：Ubuntu-24.04（WSL2，默认发行版）
> 结果：从 `C:\Users\youruser\AppData\Local\wsl\{GUID}` 迁移至 `F:\WSL\Ubuntu-24.04`，数据完整、用户配置保留。

---

## 一、迁移背景与思路

### 背景

- C 盘容量 119 GB，迁移前仅剩 **3.1 GB**（已用 116 GB，98%）。
- WSL 的 Ubuntu-24.04 虚拟磁盘 `ext4.vhdx` 约 **24 GB**，占用 C 盘大量空间。
- F 盘 312 GB，迁移前仅用 154 MB，空间充裕。

### 为什么用「导出 → 注销 → 导入」而不是直接剪切文件

WSL2 的发行版本质是一个注册在 Windows 里的虚拟磁盘（VHDX），但它和 Windows 注册表、**默认用户 UID、发行版名称、版本标记**等绑定在一起。**直接移动或复制 vhdx 文件会导致 WSL 无法识别该发行版**。所以必须走 WSL 官方提供的 `export / unregister / import` 流程，让 WSL 重新注册发行版并绑定新路径。

整个流程分三步走：

```
① wsl --export  把发行版完整打包成 .tar 备份（此时原系统仍在，不动它）
② wsl --unregister  注销原发行版（释放 C 盘空间）
③ wsl --import  从 .tar 在 F 盘重建发行版（注册到新路径）
```

关键点：**先导出、后注销**。只要 tar 备份在，数据就不会丢；哪怕导入失败，也能从备份重来。

---

## 二、迁移前的检查

### 1. 查看发行版列表

```powershell
wsl --list --verbose
```

输出：

```
  NAME              STATE     VERSION
* Ubuntu-24.04      Running   2
  docker-desktop    Stopped   2
```

- `*` 表示默认发行版。
- `VERSION 2` 表示 WSL2（导入时要指定相同版本）。
- `docker-desktop` 是 Docker Desktop 管理的发行版，**不要手动迁移**，交由 Docker 自己管理（本次未动它）。

### 2. 确认默认用户（避免迁移后变成 root 登录）

```bash
wsl -d Ubuntu-24.04 -e cat /etc/wsl.conf
wsl -d Ubuntu-24.04 -e whoami
```

输出：

```ini
[boot]
systemd=true

[user]
default=youruser
```

```
youruser
```

`/etc/wsl.conf` 在文件系统内部，会随导出一起保留，所以导入后默认用户仍是 `youruser`，无需额外修复。如果当初是用 Microsoft Store 的 Ubuntu 启动器设置的用户（而非 wsl.conf），导入后默认用户会变 root，那才需要进系统重新设置。

### 3. 查看空间与源文件大小

```bash
df -h                       # C 盘仅剩 3.1G，F 盘 312G 可用
du -h "C:\Users\youruser\AppData\Local\wsl\{GUID}\ext4.vhdx"   # 约 24G
```

---

## 三、迁移命令逐条解析

### 步骤 0：关闭 WSL

```powershell
wsl --shutdown
```

**作用**：立即终止所有正在运行的 WSL 发行版和 WSL2 虚拟机（vmware/VHDX 后台进程）。

**为什么必须做**：导出正在运行的发行版可能导致数据不一致（类似在运行中拷贝数据库文件）。关闭后导出的是一致的文件系统快照。

---

### 步骤 1：导出发行版到 tar 备份

```powershell
wsl --export Ubuntu-24.04 F:\WSL\Ubuntu-24.04.tar
```

**作用**：把指定发行版的整个文件系统打包成一个 tar 归档文件。

**参数解析**：
- 第 1 个参数 `Ubuntu-24.04`：要导出的发行版名称，必须与 `wsl --list` 里的 NAME 完全一致（区分大小写）。
- 第 2 个参数 `F:\WSL\Ubuntu-24.04.tar`：备份文件输出路径，放在目标盘 F 盘。

**发生了什么**：WSL 挂载该发行版的 ext4.vhdx，遍历整个 Linux 根目录（`/etc`、`/home`、`/usr`、已安装的软件等），逐文件写入 tar。导出的 tar 是**未压缩的**，所以大小（23 GB）接近原 vhdx（24 GB）。

**耗时**：约 5 分钟（23 GB 的读写）。

**验证备份完整性**：

```powershell
Get-Item F:\WSL\Ubuntu-24.04.tar | Select-Object FullName, @{N='GB';E={[math]::Round($_.Length/1GB,2)}}
```

输出 23 GB，与预期一致。这一步很重要——**在注销原发行版之前，务必确认备份文件存在且大小正常**。

---

### 步骤 2：注销原发行版

```powershell
wsl --unregister Ubuntu-24.04
```

**作用**：从 Windows 注销该发行版的注册信息，并**删除**其对应的 ext4.vhdx 文件和相关注册表项。

**参数解析**：只需发行版名称。

**⚠️ 这是整个流程中唯一的破坏性操作**：
- 会立即释放 C 盘上约 24 GB 空间（执行后 C 盘可用从 3.1 GB 升到 27 GB）。
- 执行前**必须**确认步骤 1 的 tar 备份已经完整生成，否则数据真的会丢。
- 只影响指定的发行版，`docker-desktop` 不受影响。

**执行结果**：`Ubuntu-24.04` 从 `wsl --list` 中消失，C 盘 `%LOCALAPPDATA%\wsl\{GUID}` 目录被清理。

---

### 步骤 3：从备份导入到 F 盘

```powershell
wsl --import Ubuntu-24.04 F:\WSL\Ubuntu-24.04 F:\WSL\Ubuntu-24.04.tar --version 2
```

**作用**：用 tar 备份在指定位置重建发行版，并向 WSL 注册。

**参数解析**：
- 第 1 个参数 `Ubuntu-24.04`：新发行版的名称（可以和原来同名，因为旧的已注销）。
- 第 2 个参数 `F:\WSL\Ubuntu-24.04`：**安装目录**，WSL 会在这里创建新的 `ext4.vhdx`。这就是"把 WSL 放到 F 盘"的关键——指定一个新盘符下的路径。
- 第 3 个参数 `F:\WSL\Ubuntu-24.04.tar`：步骤 1 生成的备份文件。
- `--version 2`：指定 WSL 版本，必须和原发行版一致（原来是 VERSION 2）。如果是 WSL1 的发行版则写 `--version 1`。

**发生了什么**：WSL 在 F 盘创建 vhdx，把 tar 里的文件逐个解压写入。这是一个"解压 + 重建"的过程，不是简单复制，所以耗时较长。

**耗时**：约 11 分钟（中途 vhdx 从 0 增长到 24 GB）。

**⚠️ 重要经验**：导入命令必须**在后台运行**（或用足够长的超时），不要让它被前台超时中断。本次第一次导入时命令被超时信号（SIGTERM）打断，虽然 vhdx 已增长到 24 GB，但 WSL 检测到导入未完成，**回滚了整个操作**——发行版从列表消失、F 盘目标目录被清空。后来用后台任务重新执行才成功。回滚本身不损坏 tar 备份，所以重导即可。

**成功标志**：日志显示 `IMPORT_EXIT=0`，`wsl --list --verbose` 中重新出现 `Ubuntu-24.04`，状态为 `Stopped`，F 盘出现 24 GB 的 `ext4.vhdx`。

---

### 步骤 4：设回默认发行版

```powershell
wsl --set-default Ubuntu-24.04
```

**作用**：把 Ubuntu-24.04 设为默认。

**为什么需要**：`wsl --unregister` 注销原默认发行版后，默认标记会丢失（此时列表里 `*` 跑到了 docker-desktop 上）。导入的新发行版不会自动继承默认身份，所以要手动设回。执行后 `wsl` 命令不带参数时直接进入 Ubuntu。

---

## 四、验证结果

导入完成后逐项验证：

```bash
# 用户身份
wsl -d Ubuntu-24.04 -e whoami                  # → youruser ✅
# 默认用户配置
wsl -d Ubuntu-24.04 -e cat /etc/wsl.conf       # → default=youruser ✅
# home 目录数据
wsl -d Ubuntu-24.04 -e sh -c 'ls -la /home/youruser'
# → .bashrc / .ssh / .viminfo / .bash_history 等全部完好 ✅
```

```powershell
wsl --list --verbose
# * Ubuntu-24.04   Running   2
#   docker-desktop Stopped  2   ✅
```

空间对比：

| 位置 | 迁移前可用 | 迁移后可用 |
|---|---|---|
| C 盘 | 3.1 GB | **27 GB**（释放约 24 GB） |
| F 盘 | 311 GB | 289 GB（占用 24 GB 用于新 vhdx） |

---

## 五、收尾：清理临时备份

确认系统运行正常后，删除迁移过程的临时文件：

```powershell
Remove-Item F:\WSL\Ubuntu-24.04.tar    # 23 GB 备份
Remove-Item F:\WSL\import.log          # 导入日志
```

删除前再次确认了 `whoami` 返回 `youruser`、系统可用。删除后 F 盘只保留运行所需的 `Ubuntu-24.04` 目录，可用空间回到 **289 GB**。

> 提示：如果迁移后想再观察几天，可以先不删 tar 备份，确认日常使用无异常后再删。

---

## 六、常见问题 FAQ

**Q：可以直接把 ext4.vhdx 剪切到 F 盘吗？**
A：不行。WSL 发行版与 Windows 注册表、默认用户 UID 等绑定，直接移动文件会导致 WSL 无法识别，必须用 export/import 流程。

**Q：导出/导入会不会丢数据？**
A：不会。tar 备份是完整文件系统快照。流程是"先备份、后删除、再重建"，只要备份在，数据就安全。

**Q：docker-desktop 要不要一起迁？**
A：不要手动迁。它是 Docker Desktop 管理的，Docker 启动/设置里可以配置其磁盘位置，手动 export/import 可能破坏 Docker 的预期。

**Q：导入后默认用户变成 root 怎么办？**
A：在 WSL 内编辑 `/etc/wsl.conf`，加入：
```ini
[user]
default=你的用户名
```
然后 `wsl --shutdown` 再启动即可。本次因为原本就配置了 wsl.conf，所以没遇到这个问题。

**Q：导入很慢正常吗？**
A：正常。本质是解压重建整个 Linux 文件系统，24 GB 耗时约 11 分钟。务必让它跑完，不要中断。
