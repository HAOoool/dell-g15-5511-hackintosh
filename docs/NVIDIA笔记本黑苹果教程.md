# 在 NVIDIA 笔记本上安装 macOS Tahoe
## —— Dell G15 5511（i7-11800H + RTX 3060 Laptop）实战教程

> 本教程基于 2026 年 10 月的一次真实安装过程整理，所有步骤与参数均为实测通过。
> 适用于：11 代 Intel + NVIDIA Ampere 笔记本；思路对其他 NVIDIA 笔记本同样适用。
>
> **免责声明**
> - 在非 Apple 硬件上安装 macOS 属于个人研究行为，请自行评估合规性与风险。
> - 刷写引导、抹盘等操作**有数据丢失风险**，操作前务必备份。
> - 社区驱动（NullMoth、OpenCore、VoodooI2C 等）均为第三方作品，未经 Apple/NVIDIA 认可，稳定性不作保证。

---

## 目录

1. 开工前的三件事
2. 准备工具与材料
3. 制作安装盘
4. BIOS 设置
5. 安装 macOS
6. 安装显卡驱动
7. 逐项修复（声音 / WiFi / 触控板 / CPU 名称）
8. 摆脱 U 盘：EFI 搬到内部磁盘
9. 已知限制一览
10. 故障排查速查表
11. 更新策略

---

## 1. 开工前的三件事

顺序不能反。任何一项没确认就做盘，很可能白装一遍。

### 1.1 显示通路（笔记本的生死线）

- 内屏是挂在**核显**（Optimus）还是**独显**（MUX / 独显直连）？
- **Optimus 情况下，macOS 内屏拿不到加速**，只能用外接口（HDMI/DP 通常直连独显）。
- Dell G15 系列的 MUX 在 BIOS 里，Advanced 页的 `Enable Hybrid graphics`（BIOS 1.3.3 起才放开）；关掉它内屏才归独显。
- **判断方法**：BIOS 关掉 Hybrid 后进 Windows，看设备管理器里核显是否消失；或 NVIDIA 控制面板 → 配置 Surround/PhysX 看显示器连在哪块 GPU。

### 1.2 显卡驱动路线

| 显卡 | 路线 |
|---|---|
| NVIDIA Turing 及以后（GTX16 / RTX20/30/40/50） | 只有 NullMoth 这条社区路线 |
| AMD RX 5000/6000、Intel UHD 630 | 原生可用，走标准 OpenCore 流程 |
| 11 代 Intel（Tiger Lake）核显 | **macOS 完全无驱动**，别指望 |

### 1.3 macOS 版本

- **选驱动官方验证过的版本，不是最新的。**
- 截至本文，NullMoth 的实机验证重点是 Sequoia 15 与 Tahoe 26 的部分版本。
- **macOS 27 起不再支持 Intel**，Intel 机器上限就是 Tahoe 26。
- SMBIOS 要跟版本匹配：例如 `iMacPro1,1` 不在 Tahoe 支持列表里。

---

## 2. 准备工具与材料

### 2.1 硬件

- **一台 Windows 电脑**（用来跑 1401 做盘）
- **U 盘**：
  - ≥32GB 用于**全量离线安装盘**（推荐）
  - 16GB 只能做 **Recovery 引导盘**（安装时在线下载 18GB+ 系统包，网不稳会反复失败）
- 目标磁盘（**建议单独一块盘，不要和 Windows 同盘**）

### 2.2 软件

| 工具 | 用途 | 地址 |
|---|---|---|
| 1401（Windows 版） | 检测硬件、生成 EFI、写安装盘 | github.com/nullmoth/1401 |
| 1401（Mac 版） | 装 NVIDIA 驱动 | 随驱动包提供 |
| NullMoth 驱动 | NVIDIA 显卡驱动 | github.com/nullmoth/nvidia-macos-driver |
| AppleALC | 声卡驱动 | github.com/acidanthera/AppleALC |
| SimpleHDA | Tahoe 补回 AppleHDA | github.com/laobamac/SimpleHDA |
| AirportItlwm | Intel 无线网卡（原生接口） | github.com/laobamac/itlwm |
| VoodooI2C | I2C 触控板 | github.com/VoodooI2C/VoodooI2C |

> 国内直连 GitHub 常被污染，准备一个可用的代理或镜像。

---

## 3. 制作安装盘

### 3.1 用 1401 生成 EFI + 写盘（Windows 下）

1. 解压 `1401-Windows-x.x.x.zip`，保持文件夹完整
2. 右键 `1401.exe` → **以管理员身份运行**
3. 它会读取硬件信息，选择目标 macOS 版本（Tahoe 26 / Sequoia 15）
4. 插入 U 盘，让它写入
5. **它会同时把 EFI 和驱动包放到 U 盘里**——`EFI/`、`NullMoth/` 两个目录

> ⚠️ 1401 官方只在**一台**机器上端到端验证过。你的机器属于"没人走过的路"，做好心理准备。

### 3.2 关于全量离线盘

如果 U 盘 ≥32GB，建议改用完整安装器，避免安装时在线下载：

**在 Mac 上**：
```bash
sudo /Applications/Install\ macOS\ Tahoe.app/Contents/Resources/createinstallmedia --volume /Volumes/你的U盘
```
然后把 1401 生成的 `EFI` 文件夹拷贝到新 U 盘的 EFI 分区里覆盖。

**判断你手上是什么盘**：
- U 盘根目录有 `Install macOS xxx.app` → 全量离线盘
- 只有 `com.apple.recovery.boot` → Recovery 盘，安装时要联网

---

## 4. BIOS 设置

开机按 **F2** 进入 BIOS：

| 设置项 | 值 | 说明 |
|---|---|---|
| Secure Boot | **Disabled** | 必须关 |
| Fast Boot | Minimal / Off | 关闭，否则可能忽略启动顺序改动 |
| SATA / Storage 模式 | **AHCI** | 如果是 RAID On，macOS 认不到硬盘 |
| Hybrid Graphics | **Disabled**（如需要） | 关掉后内屏归独显，笔记本必看 |
| Above 4G Decoding | On（若可设置） | Windows 里 Resizable BAR 显示"是"即已开启 |

> ⚠️ **改存储模式为 AHCI 后，原本的 Windows 可能无法启动**。如果 Windows 是 RAID 模式装的，需要先在 Windows 里进安全模式切换驱动，或者干脆用另一块盘装 macOS。

---

## 5. 安装 macOS

### 5.1 进入安装环境

1. 插入 U 盘，重启
2. 开机看到 Dell Logo 时连续按 **F12**
3. 选择 `UEFI: 你的U盘`
4. 出现 OpenCore 菜单 → 选择 `macOS Installer` 或 `Install macOS Tahoe`

### 5.2 抹盘

1. 选「磁盘工具」
2. 菜单栏 **显示 → 显示所有设备**（重要！否则只看得到分区层级）
3. 选中**目标磁盘的物理设备层**（不是它下面的分区）
4. 抹掉：名称 `Macintosh HD`，格式 **APFS**，方案 **GUID 分区图**

> ⚠️ 动手前**核对容量和型号**，不要按分区名判断。Windows 盘千万别碰。

### 5.3 安装

1. 关闭磁盘工具 → 选「重新安装 macOS」
2. 选刚抹好的 `Macintosh HD`
3. **会重启 3～5 次**，每次重启仍然从 U 盘进 OpenCore：
   - 前几轮选 `macOS Installer`
   - 最后选 `Macintosh HD`
4. 期间黑屏、进度条长时间不动都属正常，**别断电**

### 5.4 遇到下载失败怎么办

在线安装要下 18GB 左右的安装包，日志里出现这个就是网络断了：

```
Retrying ... after N failure(s): PKDownloadError Code=8 (underlying Code=14)
```

**判断带宽还是断流**：看 `/Volumes/Macintosh HD/macOS Install Data` 的大小增长。
如果 20 分钟下了 17GB，说明**带宽没问题，是长连接被掐断**——这时候挂代理反而可能更慢。

**处理顺序**：
1. 直接点「重新安装」重试（安装器支持断点续传）
2. 插网线、换 DNS（`networksetup -setdnsservers "Wi-Fi" 223.5.5.5 119.29.29.29`）
3. 实在不行，改做全量离线盘

### 5.5 首次进入系统

设置助手里几项必须选对：

- **数据传输** → 选「不传输任何信息」
- **Apple ID** → 选「稍后设置」（序列号是伪造的，现在登录容易触发风控）
- **用户名** → 用纯英文小写（后面命令行要用）
- **FileVault** → **绝对不要开**（黑苹果上开了容易把自己锁在外面）
- **自动更新** → 进去之后第一件事就是关掉

---

## 6. 安装显卡驱动

### 6.1 用 1401 Mac 版

1. 从 U 盘的 `NullMoth/` 目录拷出 `1401-Mac-x.x.x.zip`，解压到本地（**别在 U 盘里直接跑**）
2. 首次打开可能被系统拦：右键 → 打开，或 `xattr -dr com.apple.quarantine 路径`
3. 打开后它会自动找到 U 盘上的 OpenCore 配置
4. **先点 `Show me the changes`**，看清它要改哪些项：
   - `Kernel → Block: exclude IONDRVSupport`
   - `NVRAM Delete: add boot-args`
   - 启动菜单加两个回滚项
5. 再点安装 → **U 盘全程插着**
6. 重启

### 6.2 重启注意事项

- OpenCore 菜单里选正常启动（**别选** `1401: Start once without NVIDIA driver`）
- 第一次带驱动启动会在 `PCI configuration end` 停**最多一分钟**，别断电
- 进桌面后如果提示「系统扩展被阻止」→ 系统设置 → 隐私与安全性 → 允许 → **再重启一次**

### 6.3 验证

```bash
kmutil showloaded --list-only | grep nullmoth      # 应有 4 行
system_profiler SPDisplaysDataType                  # VRAM 应显示真实容量，Metal Support: Metal 3
```

### 6.4 驱动升级

不要只看版本号，**先看 Releases 说明**。例如 v2.3.7 修复了 Tahoe 上图标缺前景层的问题，值得升。

手动升级：
```bash
tar -xzf nullmoth-nvidia-X.X.X.tar.gz && cd pkgroot
shasum -a 256 -c SHA256SUMS
sudo ./install.sh
sudo shutdown -r now
```

回滚：
```bash
# 安装器会在 /Library/NullMoth/ 下留备份
sudo ./uninstall.sh /Library/NullMoth/backup-XXXX
```
OpenCore 菜单里也有 `1401: Remove NVIDIA driver` 应急项。

---

## 7. 逐项修复

### 7.1 声音（Tahoe 必做）

**背景**：macOS Tahoe 从 beta 2 起**删除了 AppleHDA.kext**，而 AppleALC 只是它的补丁——宿主没了就没声音。

**症状**：`system_profiler SPAudioDataType` 里 `Devices:` 是空的。

**解决**：先用 **AppleALC + SimpleHDA** 把声音救回来；如果发现**睡眠唤醒后失声**（SimpleHDA 方案的常见问题），换 **VoodooHDA**。

#### 方案一：AppleALC + SimpleHDA

1. 下载 AppleALC 1.9.7+ 和 SimpleHDA 1.0.0+
2. 两个 kext 拷进 `EFI/OC/Kexts/`
3. `Kernel → Add` 里添加，**顺序必须是 Lilu → AppleALC → SimpleHDA**
4. boot-args 确保有 `keepsyms=1`
5. 设置 `alcid=<layout>`
6. 可选但推荐：安装 `SimpleHDA-HAL-x.x.x.pkg`
7. 重启

**layout 怎么选**：查 AppleALC 支持列表里你的 codec 对应的可用值，然后一个一个试。
例如 ALC295/ALC3254 可用 `1, 3, 11, 13, 14, 15, 21, 22, 23, 24, 28, 33, 69, 75, 77`，本机实测 **21** 可用。

#### 方案二：VoodooHDA（睡眠唤醒稳定）

SimpleHDA 的已知问题是**睡眠唤醒后声音消失，且重启 coreaudiod 救不回来**
（`CodecCommander` 在 macOS 12.3+ 已失效，`alc-verb` 也连不上 SimpleHDA 的 provider）。

换成 VoodooHDA 3.7（`CloverHackyColor/VoodooHDA` releases）后睡眠唤醒正常，麦克风也在：

1. kext 拷进 `EFI/OC/Kexts/`，建议目录名 `VoodooHDA.kext`
2. ⚠️ **`ExecutablePath` 必须写它真实的二进制名**——v3.7.0 的包里叫 `VoodooHDA_BL`，
   写成 `Contents/MacOS/VoodooHDA` 会让 OpenCore 开机 **`Halting on critical error`**
3. `DeviceProperties → Add → PciRoot(0x0)/Pci(0x1f,0x3)` 加 **`voodoo-layout-id`**（4 字节 Data，layout 21 = `<15 00 00 00>`）
   - 不设：只有输出设备、**麦克风不出现**，默认输出还会选成 Headphones（听起来像没声音）
   - 设了：设备名为 `Internal Speakers` / `Internal Microphone`
4. 把 AppleALC / SimpleHDA 的 `Enabled` 设为 `false`（保留便于回滚）
5. 重启

### 7.2 WiFi

**Intel 网卡在 macOS 上有两条路**：

**A. itlwm + HeliPort**（兼容性好）
- WiFi 会被伪装成"以太网"，**系统设置里永远没有 WiFi 选项**
- 必须装 HeliPort 这个菜单栏小工具连网
- kext：`itlwm.kext`

**B. AirportItlwm**（原生接口，推荐）
- 系统设置里有原生 WiFi 菜单，不用第三方客户端
- 但不支持 AWDL → **隔空投送、接力、通用剪贴板都用不了**
- kext：`AirportItlwm.kext`

**两者不能同时启用**：在 `Kernel → Add` 里把不用的那个 `Enabled` 设成 `false`。

### 7.3 触控板（最容易卡住的一项）

**症状**：触控板能移动、能点击，但多指手势失效、系统设置里没有「触控板」面板。

**先判断通道**：
```bash
ioreg -rc VoodooPS2Trackpad          # 有输出 = 走 PS/2 模拟
ioreg -p IOACPIPlane -w0 | grep -iE "TPD|I2C"
```

如果 ACPI 里有 `I2C1@150001 → TPD0`，说明是真正的 I2C-HID 触控板，需要 VoodooI2C。

**核心问题**：VoodooI2C 的 PCI 匹配表**不含所有代际**。以 Tiger Lake-H 为例，控制器是 `8086:43e8~43eb`，而驱动只支持到 Ice Lake 的 `34e8~34eb`——**必须做 device-id 伪装**。

在 `DeviceProperties → Add` 里加：
```
PciRoot(0x0)/Pci(0x15,0x1)        ← I2C1 对应的 PCI 位置
    device-id = <e8 34 00 00>     ← 4 字节 Data，伪装成 Ice Lake
```

> ⚠️ **本教程最重要的一条坑**：
> 用 `PlistBuddy` 写这个值会写成**字符串**（`<"E8340000">`），OpenCore 静默忽略，
> 表现是「配置改了、重启了、什么变化都没有」。
>
> 正确做法是把 plist 转成 XML 后直接改：
> ```bash
> sudo plutil -convert xml1 config.plist
> # 把对应 <data> 内容替换为 6DQAAA==   （即 e8 34 00 00）
> sudo plutil -lint config.plist
> ```
> 验证：`plutil -p config.plist | grep -A2 "Pci(0x15,0x1)"`
> 应显示 `device-id => {length = 4, bytes = 0xe8340000}`

**成功标志**：
```bash
kmutil showloaded --list-only | grep -i alexandred
# 应有 VoodooI2C / VoodooGPIO / VoodooI2CHID / VoodooInput

ioreg -l -w0 | grep -A2 "TPD0"
# 应看到 TPD0 → VoodooI2CDeviceNub → VoodooI2CHIDDevice
```

系统设置里出现「触控板」面板、三/四指手势可用 = 成功。

> 笔记本触控板普遍没有压力传感器，**Force Touch 压力触控是无法实现的**。

### 7.4 CPU 名称显示错误

**症状**：「关于本机」里处理器显示成 Xeon 或其它型号（因为你用别的机型 SMBIOS）。

**解决**：RestrictEvents 的两个 boot-arg

```
revpatch=sbvmm,cpuname revcpu=1
```

名字会自动从 CPUID 读取；想自定义可以再加 `revcpuname=<名字>`。

### 7.5 系统图标缺失、菜单毛玻璃异常

**症状**：系统 App 图标只剩圆角底板，前景图案没了；Dock 右键菜单毛玻璃破碎。

**原因**：驱动在 Tahoe 上的分层图标合成 bug（NullMoth 2.3.6）。

**解决**：升级到 **2.3.7 或更新**（当前最新 2.4.3）。升级后如果 Dock 图标还不对：
```bash
sudo find /private/var/folders -name "com.apple.dock*" -maxdepth 7 -exec rm -rf {} +
killall Dock
```

### 7.6 系统卡顿

按这个顺序排查：

1. **动态壁纸** → 换成静态图。4K 航拍壁纸会让 `WindowServer` 飙到 150%+
2. **iCloud 同步** → 关掉照片同步；`cloudd` 卡死可以 `sudo killall cloudd`
3. **Spotlight 索引** → 新装系统头几小时正常现象
4. 都正常还卡 → 就是 Metal 转译层的着色器编译开销，首次打开 App 慢属正常

---

## 8. 摆脱 U 盘：EFI 搬到内部磁盘

装完之后 EFI 还在 U 盘上，每次开机都要插——非常不保险。

### 步骤

```bash
# 1. 按容量认盘（进 macOS 后磁盘编号可能和 Recovery 里不同！）
diskutil list

# 2. 挂载 macOS 所在磁盘的 EFI 分区
sudo diskutil mount disk1s1

# 3. 复制（假设 U 盘挂在 /Volumes/EFI_U 之类）
sudo cp -R /Volumes/你的U盘/EFI /Volumes/EFI/

# 4. 验证结构
ls /Volumes/EFI/EFI/BOOT     # BOOTx64.efi
ls /Volumes/EFI/EFI/OC       # ACPI Drivers Kexts Resources Tools OpenCore.efi config.plist

# 5. 写入启动项
sudo bless --mount /Volumes/EFI --setBoot --file /Volumes/EFI/EFI/BOOT/BOOTx64.efi
```

### 拔掉 U 盘重启

**如果自动进了 Windows，别以为失败了**——大概率是 **OpenCore 菜单里 Windows 排在第一位，倒计时到了自动选了它**。

修法：在 OpenCore 菜单里选中 `Macintosh HD`，按 **Ctrl + Enter** 设为默认启动项。
（需要 `Misc → Security → AllowSetDefault` 为 true）

---

## 8.5 性能预期（实测参考）

CPU（i7-11800H，Geekbench 7）：单核 1723 / 多核 8559，略高于同型号平均值；
空闲降到 1.07GHz，8 线程满载稳在 4.2GHz —— **CPU 侧没有性能损失**。

GPU（RTX 3060 Laptop）：

- Metal / OpenCL **计算负载会卡死**（着色器翻译瓶颈，issue #111）
- NVENC/NVDEC 库虽然随驱动安装、进程也会加载，但 VideoToolbox 不会路由过去（#175 / #275）
- 实测 1080p→720p 转码约 1 倍实时速度，是纯软件编码

**结论：macOS 下这块卡只负责"点亮屏幕"，重活回 Windows。**

---

## 9. 已知限制一览

装完能用的功能大致如下（以本机实测为准）：

### ✅ 通常可用
显卡加速 / 内屏高刷 / 声音 / 摄像头 / 麦克风 / 有线 + 无线网络 / 蓝牙 / 触控板手势 / 亮度 / 电池 / 睡醒 / CPU 睿频 / TRIM

### ⚠️ 有瑕疵
- 4K HEVC 视频：**纯 CPU 软解**，NVDEC 未接上，风扇会明显起来
- 温度/风扇读数：VirtualSMC 不暴露 SMC 温度键，读不到
- iCloud 相关进程偶发吃满 CPU

### ❌ 不可用（硬件/协议限制）
- **隔空投送 / 接力 / 通用剪贴板**：Intel 网卡不支持 AWDL。换博通 BCM94360NG 可解决
- **Force Touch**：触控板没有压力传感器
- **Intel 核显加速 / QSV**：11 代核显无驱动
- **NVENC 硬件编码**：macOS 无对应接口

---

## 10. 故障排查速查表

| 现象 | 先查什么 |
|---|---|
| 开不了机 / 卡在苹果 logo | 上次改的配置；用 OpenCore 菜单的回滚项或 U 盘旧 EFI |
| 黑屏但有鼠标 | 显示通路问题（MUX / 显卡驱动） |
| 没有声音（设备在） | `sudo killall coreaudiod`，再不行重启 |
| 没有声音（设备都没有） | AppleHDA 栈没起来 → 查 `kmutil \| grep -i HDA` |
| 触控板没手势 | I2C 控制器 device-id 伪装（重点查类型是不是 Data） |
| 图标缺图案 | 驱动版本低于 2.3.7 |
| 系统流畅度差 | 动态壁纸 / iCloud 同步 / Spotlight 索引 |
| Windows 起不来 | 存储模式被改成 AHCI；改回 RAID 或进安全模式切驱动 |
| 磁盘工具看不到硬盘 | 存储模式是 RAID On，需改 AHCI |

### 常用诊断命令

```bash
# 驱动
kmutil showloaded --list-only | grep -iE "nullmoth|AppleALC|SimpleHDA|AirportItlwm|Voodoo"

# 显卡 / 显示
system_profiler SPDisplaysDataType

# 音频
system_profiler SPAudioDataType

# 网络接口
ifconfig -l

# CPU 功耗与频率
sudo powermetrics --samplers cpu_power -n 1 -i 1500

# 崩溃报告
ls -t /Library/Logs/DiagnosticReports | head
```

---

## 11. 更新策略

### macOS 系统更新 —— 最保守

- **关掉自动更新**
- 小版本可以升，但**升级前备份 EFI**，升级后立刻检查驱动是否还在
- **大版本（macOS 27+）绝对不要升**，Intel 支持已终止

### 驱动 / kext 更新

- NullMoth：用 1401 app 的 Update，或手动 `install.sh`
- 其它 kext：替换 `EFI/OC/Kexts/` 里的目录后重启即可（OpenCore 启动时注入，不需要重建系统缓存）
- **目录名不能改**，否则 `Kernel → Add` 里的 BundlePath 对不上

### 备份制度

黑苹果的全部配置就集中在 **EFI 那几十 MB** 里：

```bash
sudo diskutil mount disk1s1
sudo cp -R /Volumes/EFI/EFI ~/EFI-backup-$(date +%m%d)
```

**U 盘别格式化**，它就是你的急救盘。

---

## 附录：本机验证过的参考配置

| 项目 | 值 |
|---|---|
| 机型 | Dell G15 5511（i7-11800H + RTX 3060 Laptop） |
| 系统 | macOS Tahoe 26.7.1 |
| SMBIOS | MacBookPro16,2 |
| 引导 | OpenCore（1401 生成）在内部 SSD |
| 显卡驱动 | NullMoth 2.4.3 |
| 声卡 | VoodooHDA 3.7.0，`voodoo-layout-id=21`（`Internal Speakers` / `Internal Microphone`） |
| 无线 | AirportItlwm 2.4.0（AX201） |
| 触控板 | VoodooI2C 2.9.1 + device-id 伪装 |
| CPU 名称 | `revpatch=sbvmm,cpuname revcpu=1` |

---

*祝折腾顺利。记住两句话：*
*—— 改任何东西之前先备份 EFI。*
*—— 一步一步改，一次只动一个变量，否则出问题找不到原因。*
