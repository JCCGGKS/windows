# 2026-11-19 磁盘整理与迁移清单

> 操作对象：Windows 系统盘 C:\（119 GB）
> 本次为只读分析 + 用户授权下的清理/迁移，未破坏系统。
> 文档状态：**已同步至全部操作完成**（2026-11-19 最终更新）。

---

## 一、本次做了哪些操作

### 模块 A：系统缓存清理（已释放约 6.3 GB，均在 C 盘）

清理的都是可重建的临时/缓存数据，未触碰任何系统组件。

| 项目 | 释放 | 说明 |
|---|---|---|
| AppData\Local\Temp 临时文件 | 1756 MB | 含一个 758 MB 的 VMware 日志；386 MB 被占用留待重启后清 |
| 各软件 updater 安装包残留（9 项） | ~2238 MB | kimi / genieworkbuddy / obsidian / tabby / apifox 等更新包 |
| 开发缓存（pip / uv / npm / go-build） | 1734 MB | 删后下次拉依赖需重新下载 |
| Chrome + Edge 浏览器缓存 | 330 MB | 浏览器自动重建 |
| .cache 通用缓存 | 211 MB | — |
| 回收站 | 233 MB | 已清空 |

### 模块 B：应用数据迁移（C 盘 → D 盘，约 16.4 GB）

**方式**：目录联接（junction）。原路径不变，应用照常读写，数据实际存放在 `D:\AppData\` 下。对系统无影响，只迁普通桌面软件数据，未动 `Microsoft` / `Packages` / `Docker`。

| 应用 | 原路径（C:\Users\youruser\AppData\下） | 实际数据位置（D 盘） | 大小 | 状态 |
|---|---|---|---|---|
| 飞书 | Roaming\LarkShell | D:\AppData\Roaming\LarkShell | 8.3 GB | 成功 |
| WPS | Roaming\kingsoft | D:\AppData\Roaming\kingsoft | 2.4 GB | 成功 |
| lightest | Roaming\lightest | D:\AppData\Roaming\lightest | 1.9 GB | 成功 |
| Tencent | Roaming\Tencent | D:\AppData\Roaming\Tencent | 1.8 GB | 成功 |
| JetBrains | Local\JetBrains | D:\AppData\Local\JetBrains | 1.0 GB | 成功 |
| Trae | Roaming\Trae | D:\AppData\Roaming\Trae | 469 MB | 成功 |
| VS Code | Roaming\Code | D:\AppData\Roaming\Code | 449 MB | 成功 |

7 项全部验证通过：原路径已是 Junction 并指向 D 盘，数据完整。

### 备份清理（已完成）
迁移时保留的 7 个 `.bak` 备份，经严格校验（确认原路径已是 junction、指向 D 盘正确位置、D 盘数据存在）后，**已于 2026-11-19 全部删除**，C 盘空间已实际释放约 16.4 GB。删除过程：5 项直接删除成功，2 项（kingsoft / lightest）有顽固残留文件，用空目录镜像方式清除。已确认 AppData 下无 `.bak` 残留。

---

## 二、注意事项

### 1. `.bak` 备份已删除，无法再用它回滚
备份已清理，C 盘空间已释放。若应用出现异常，只能从 D 盘真实数据恢复（见第 6 条回滚方法）。因此**请保持 D:\AppData\ 目录稳定，不要删除或移动它**。

### 2. 删除 junction 的大坑
junction 在资源管理器里看着像普通文件夹。删除时**绝对不要加 `/S`**：
- 正确：`rmdir "原路径"` —— 只删链接，D 盘数据不动
- 危险：`rmdir /S "原路径"` —— 会顺着链接把 D 盘真实数据一起删掉

### 3. 应用更新可能重建目录
飞书、VS Code 等大版本更新后，偶尔会在 C 盘原路径重新生成真实目录，使 junction 失效（数据不丢，但 C 盘会重新增长）。更新后若发现原路径出现带内容的普通文件夹，按同样方法重新 junction 即可。

### 4. D 盘必须常驻
junction 目标盘不能是移动硬盘/U 盘，也不能改盘符或格式化，否则应用找不到数据。当前 D 盘是内置 NTFS 分区，符合要求。

### 5. 杀毒/白名单路径
若这些应用曾加入杀毒软件白名单，迁移后原路径变成链接，建议把 `D:\AppData\` 下对应新路径也加入白名单。

### 6. 回滚方法
`.bak` 备份已删除，若某应用异常需回滚，改为从 D 盘把数据复制回 C 盘原路径：
```
rmdir "C:\Users\youruser\AppData\Roaming\LarkShell"
robocopy "D:\AppData\Roaming\LarkShell" "C:\Users\youruser\AppData\Roaming\LarkShell" /E /COPY:DAT
```
即：先删 junction（不带 /S），再把 D 盘真实数据复制回原路径。恢复后该应用数据重新落回 C 盘。

### 7. 未触碰的部分
`hiberfil.sys`（休眠文件约 12.7 GB）仍开启。这台是笔记本且不支持 S3 睡眠，关闭休眠会失去断电保护，故保留。若基本插电当台式机用，可用管理员运行 `powercfg -h off` 释放约 13 GB。

---

## 三、可频繁手动删除的位置

以下都是会自动重建的缓存/临时数据，可定期清理。按"安全等级"分三档。

### A 档：随时可删，最安全

| 位置 | 内容 | 建议频率 |
|---|---|---|
| `%TEMP%`（即 AppData\Local\Temp） | 程序运行临时文件 | 每月，重启后清更干净 |
| `C:\Windows\Temp` | 系统临时文件（需管理员，跳过占用项） | 每月 |
| 回收站 | 已删除文件 | 需要时清空 |
| 浏览器缓存：Chrome / Edge → 设置 → 清除浏览数据（勾选"缓存的图片和文件"） | 网页缓存 | 每月 |

> 应用数据已 junction 到 D 盘，但浏览器缓存（Google / Microsoft\Edge）仍在 C 盘，属于可频繁清理项。

### B 档：可删，但下次会重新下载

| 位置 | 内容 | 影响 |
|---|---|---|
| `AppData\Local\pip\Cache` | pip 下载缓存 | 下次装包需重新下载 |
| `AppData\Local\npm-cache` | npm 缓存 | 同上 |
| `AppData\Local\uv` | uv 包缓存 | 同上 |
| `AppData\Local\go-build` | Go 编译缓存 | 重新编译稍慢 |
| `AppData\Local\ms-playwright` | Playwright 浏览器 | 自动化需重装浏览器 |

### C 档：用应用内功能清理（别直接删目录）

这些应用已迁移到 D 盘，其缓存建议用**应用自带的清理功能**，避免误删消息/文件数据：

| 应用 | 操作入口 |
|---|---|
| 飞书 | 头像 → 设置 → 通用 → 清理缓存（个人版还可"清理消息文件"按日期/大小筛选） |
| 微信 | 设置 → 通用设置 → 存储空间 → 清理缓存 |
| WPS | 设置 → 备份与清理 → 清理缓存 |
| VS Code / Trae | 命令面板执行 `Developer: Reload Window` 或清理 `CachedData` 子目录 |

### 补充：不要手动删的目录
`C:\Windows\WinSxS`、`C:\Windows\System32`、`C:\Program Files\WindowsApps`、各应用**已 junction 的原路径**（那是链接，见注意事项 2）。

---

## 四、最终磁盘状态（2026-11-19）

| 盘符 | 总容量 | 已用 | 可用 | 使用率 |
|---|---|---|---|---|
| C: | 119 GB | 75 GB | **44 GB** | 64% |
| D: | 300 GB | 24 GB | 277 GB | 8% |

对比操作前：C 盘从 **96 GB 已用 / 23 GB 可用（81%）** 降至 **75 GB 已用 / 44 GB 可用（64%）**，累计释放约 21 GB（缓存清理 6.3 GB + 迁移备份删除 16.4 GB，含少量系统自身写入波动）。

本轮产生的所有临时分析/迁移脚本已清理完毕，无残留。
