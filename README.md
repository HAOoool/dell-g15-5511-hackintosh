# Dell G15 5511 黑苹果（macOS Tahoe 26）配置与教程

在 **Dell G15 5511（i7-11800H + RTX 3060 Laptop）** 上安装 macOS Tahoe 26 的完整记录：
可用的 OpenCore EFI、详细配置文档、以及一份面向新手的安装教程。

> ⚠️ **本仓库由 AI（米塔 / Hermes Agent）根据实际安装过程整理生成**，并非人工逐字编写。
> 所有参数来自实机验证，但**未经过广泛测试**，不保证在其他设备上可用。
> **无支持、无担保**，使用前请先备份原 EFI 与数据。

---

## 硬件配置

| 项目 | 配置 |
|---|---|
| 机型 | Dell G15 5511 |
| CPU | Intel Core i7-11800H（8C/16T，基础 2.3GHz） |
| 显卡 | NVIDIA GeForce RTX 3060 Laptop GPU（6GB，独显直连） |
| 内存 | 16GB DDR4-3200 |
| 系统盘 | 512GB NVMe（macOS） |
| 数据盘 | 1TB NVMe（Windows，未改动） |

## 系统与驱动

| 项目 | 版本 |
|---|---|
| macOS | Tahoe 26.7.1（25G241） |
| 引导 | OpenCore（1401 生成） |
| 显卡驱动 | NullMoth NVIDIA Driver **2.4.3** |
| SMBIOS | MacBookPro16,2 |
| 声卡 | VoodooHDA 3.7.0 + `voodoo-layout-id = 21` |
| 无线 | AirportItlwm 2.4.0（Intel AX201） |
| 触控板 | VoodooI2C 2.9.1 + I2C device-id 伪装 |

## 功能状态

### ✅ 正常

显卡加速（Metal 3）、内屏 1920×1080 @165Hz、系统图标与 Liquid Glass、
扬声器与麦克风、原生 WiFi 与有线网络、蓝牙、触控板全部多指手势、
亮度调节、电池百分比、睡眠唤醒、CPU 睿频、TRIM。

### ⚠️ 有瑕疵

- 4K HEVC 视频走 CPU 软解；GPU 计算负载（Blender / Geekbench compute）会卡死
- NVENC/NVDEC 库虽随驱动安装，但 VideoToolbox 未路由（上游 issue #175 / #275）
- 温度与风扇转速读不到（VirtualSMC 限制）

### ❌ 不可用（硬件/协议限制）

- 隔空投送 / 接力 / 通用剪贴板（Intel 网卡无 AWDL，需换博通网卡）
- Force Touch 压力触控（触控板无压力传感器）
- Intel 核显加速（11 代核显 macOS 无驱动）

## 性能实测

CPU（Geekbench 7）：单核 **1723** / 多核 **8559**，略高于 i7-11800H 平均值（1687 / 8087）。
空闲 1.07GHz，8 线程满载 4.2GHz，无降频。

GPU：仅能驱动显示与桌面合成，计算类负载不可用。详见文档。

---

## 仓库内容

```
EFI/                OpenCore 配置（已脱敏）
docs/
  G15黑苹果配置文档.md / .pdf    本机完整配置与维护手册
  NVIDIA笔记本黑苹果教程.md / .pdf  面向新手的安装教程
NOTICE.md           第三方组件与许可
```

## 使用说明

> **不要直接套用本 EFI 到其他型号。** 显卡、触控板、声卡配置都与具体硬件绑定。

如果你也是 G15 5511，可以把它当作起点，但**必须做两件事**：

1. **重新生成 SMBIOS**
   本仓库的序列号 / MLB / UUID 已全部替换为占位符（`XXXXXXXXXXXX`）。
   请用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 或 OCAT 生成你自己的，
   否则 iCloud / App Store 等 Apple 服务无法正常工作。机型保持 `MacBookPro16,2`。

2. **确认 BIOS 设置**
   Secure Boot 关闭、存储模式 AHCI、Fast Boot 关闭、独显直连（MUX）开启。

### 安装步骤

详细的安装流程、驱动配置、踩坑记录，请看：

- 新手向：`docs/NVIDIA笔记本黑苹果教程.pdf`
- 本机配置与维护：`docs/G15黑苹果配置文档.pdf`

## 关键提示

- **备份优先**：改动前先备份 `EFI`，U 盘不要格式化（它是你的急救盘）。
- **一次只改一个变量**：否则出问题无法定位。
- **不要升级 macOS 大版本**：macOS 27 起不再支持 Intel。
- 驱动更新前先看上游 release notes，不要只看版本号。

## 第三方组件

所用的 OpenCore、kext、驱动均来自各自的开源项目，版权归原作者所有，
详见 [NOTICE.md](NOTICE.md)。其中 NullMoth 驱动采用 PolyForm Noncommercial 许可，
**禁止任何商业用途**。

## 免责声明

本项目仅供个人学习与研究使用。在非 Apple 硬件上安装 macOS 可能违反 Apple 的软件许可协议，
请自行评估合规风险。因使用本仓库内容造成的任何数据丢失、硬件损坏或其它后果，
作者不承担任何责任。
