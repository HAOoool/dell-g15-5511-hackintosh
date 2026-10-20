# 第三方组件与许可

本仓库的 EFI 中包含了以下开源/社区项目。版权与许可归各自作者所有，此处仅作记录。
**建议始终从上游获取最新版本**，本仓库中的版本为 2026-10-20 的实机可用快照。

## 引导

| 组件 | 上游项目 | 说明 |
|---|---|---|
| OpenCore | https://github.com/acidanthera/OpenCorePkg | BSD-3-Clause |

## 内核扩展

| 组件 | 上游项目 | 许可 |
|---|---|---|
| Lilu | https://github.com/acidanthera/Lilu | BSD-3 |
| VirtualSMC | https://github.com/acidanthera/VirtualSMC | BSD-3 |
| WhateverGreen | https://github.com/acidanthera/WhateverGreen | BSD-3 |
| AppleALC | https://github.com/acidanthera/AppleALC | BSD-3 |
| NVMeFix | https://github.com/acidanthera/NVMeFix | BSD-3 |
| RestrictEvents | https://github.com/acidanthera/RestrictEvents | BSD-3 |
| AMFIPass | https://github.com/acidanthera/AMFIPass | BSD-3 |
| BrightnessKeys | https://github.com/acidanthera/BrightnessKeys | BSD-3 |
| USBToolBox | https://github.com/USBToolBox/kext | MIT |
| RealtekRTL8111 | https://github.com/Mieze/RTL8111_driver_for_OS_X | GPL-2.0 |
| itlwm / AirportItlwm | https://github.com/laobamac/itlwm | GPL-2.0 |
| IntelBluetoothFirmware | https://github.com/OpenIntelWireless/IntelBluetoothFirmware | GPL-2.0 |
| BlueToolFixup | https://github.com/acidanthera/BrcmPatchRAM | BSD-3 |
| VoodooPS2 | https://github.com/acidanthera/VoodooPS2 | GPL-2.0 |
| VoodooI2C | https://github.com/VoodooI2C/VoodooI2C | GPL-2.0 |
| VoodooHDA | https://github.com/CloverHackyColor/VoodooHDA | GPL-2.0 |
| SMCBatteryManager / SMCLightSensor / SMCProcessor / SMCSuperIO / SMCDellSensors | 随 VirtualSMC 分发 | BSD-3 |
| ECEnabler | https://github.com/1Revenger1/ECEnabler | MIT |
| XHCI-unsupported | https://github.com/khronokernel/XHCI-unsupported | MIT |

## 显卡驱动（重要）

| 组件 | 上游项目 | 许可 |
|---|---|---|
| NullMoth NVIDIA Driver | https://github.com/nullmoth/nvidia-macos-driver | **PolyForm Noncommercial 1.0.0** |

> NullMoth 驱动为免费、开源、**禁止商业使用**的项目。
> 它目前仍处于快速迭代阶段，部分硬件组合尚未经过实机验证，请务必阅读其官方文档。
> 本仓库不重新分发该驱动的安装包，使用前请从上游 releases 获取。

## 安装工具

| 工具 | 上游项目 |
|---|---|
| 1401（生成 EFI / 写盘 / 安装驱动） | https://github.com/nullmoth/1401 |

---

如在许可上有任何疏漏，请联系以便更正。
