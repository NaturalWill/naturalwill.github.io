---
title: 借助 Clover 使旧电脑可以用上 NVMe 固态硬盘
date: 2020-06-13 15:00:12
tags:
  - Clover
categories:
  - 400-软件使用
---

这里介绍的是一种软件实现方案——它可以使**没有 M.2 插槽**且 **BIOS 不支持 NVMe 协议**的台式电脑通过 *M.2 to PCIE 转接卡* 用上 NVMe 固态硬盘，其原理是：由 U 盘或非 NVMe 硬盘来提供最初的系统引导，借助 Clover 加载 NVMe 驱动之后将后续的系统引导交回给 NVMe 固态硬盘。

<!-- more -->

> 老主板通常没有M.2插槽，但是可以通过M.2 to PCIE转接卡将它安装上去。不过在BIOS设置引导顺序当中你却无法找到NVMe固态硬盘，也就是俗称的不认盘。这个问题是BIOS太老造成的，除了个别半旧不旧的主板可能有官方BIOS更新可以升级之外，多数UEFI BIOS可以手动注入NVMe驱动来解决，不过涉及到修改和刷新BIOS的风险较高。

> 无法作为系统盘开机，成为阻碍PCIE NVMe固态硬盘进入老电脑的首要敌人。毕竟多数人都是将固态硬盘当作系统盘使用的，尽管使用其他系统盘开机进入Windows系统之后，NVMe固态硬盘是可以识别和使用的，但好钢不能用在刀刃上，显然是非常大的遗憾。

> 对于动手能力普遍较强的数码IT玩家来说，只要有更高的性价比，多一点折腾并不麻烦。接下来就随存储极客一起实战老电脑免刷BIOS使用NVMe固态硬盘吧！

> ...

> 随着东芝推出RC100主流级NVMe固态硬盘，过去昂贵的高性能NVMe SSD开始走向平民化。M.2 2242规格、单芯片设计，这一切都预示着NVMe固态硬盘的应用范围将得到史无前例的扩展。在消灭最后一个系统启动的障碍之后，只要拥有PCIE插槽的电脑，几乎都可以使用NVMe固态硬盘，当然这需要一定的动手能力和合适的主板平台（需要PCIE 3.0接口充分发挥性能），并对自己动手能力有信心，使用软件魔改的NVMe系统盘也未尝不可。

> 引用自： https://www.sohu.com/a/241496282_615464



### 需要用到的工具：

软件：

- Archlinux 系统镜像（archiso）: https://www.archlinux.org/download/
- Clover: https://github.com/CloverHackyColor/CloverBootloader/releases
- Windows 系统镜像: https://tb.rg-adguard.net/

硬件：

- 系统安装盘（2 个 U 盘，如有另一台电脑，可在使用完 1 个后制作第 2 个系统盘，仅 1 个即可）： 用于制作 Archlinux 启动 U 盘和 Windows 安装盘。
- 引导盘： 用于安装 Clover, 引导系统启动；可以是系统支持的非 NVMe 硬盘（AHCI）或普通 U 盘，需要长期连接在电脑上。

### 步骤

#### 准备工作

1. 提前下载好 archiso 、 Clover 、以及自己需要 Windows 系统镜像。
2. 制作好 Archlinux 系统盘和 Windows 系统盘。
3. 将在 GitHub Release 页面下载到的 `Clover-*-X64.iso.7z` 解压，解压后得到的 `Clover-*-X64.iso` 可提前放置在 Archlinux 系统盘根目录。

#### 建立引导分区

1. 制作 Archlinux 系统盘，并使用该 Archlinux 系统盘引导系统启动。

可参考 https://wiki.archlinux.org/index.php/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87) 。

2. 在引导盘上建立 EFI 系统分区（以下简称 ESP ，约 512 MB），并格式化。

参考 https://wiki.archlinux.org/index.php/EFI_system_partition 。

3. 安装 Clover 。

挂载 `Clover-*-X64.iso`，可见其目录结构是这样的：

    CloverCD
      |--EFI
      |--Library
      |--usr

将其中的文件夹 `EFI` 整个复制到第 2 步建立的 ESP 中。

在 ESP 中，将 `\EFI\CLOVER\drivers\off\NvmExpressDxe.efi` 复制一份到 `\EFI\CLOVER\drivers\UEFI\` 目录下。（这是 Clover 能够识别 NVMe 固态硬盘的关键）

##### For More

如需隐藏不必要的启动项，可编辑 `\EFI\CLOVER\config.plist`，找到 Scan ，并取消注释，修改前：

    <key>#Scan</key>
    <dict>
        <key>Entries</key>
        <true/>
        <key>Legacy</key>
        <false/>
        <key>Tool</key>
        <true/>
    </dict>

修改后：

    <key>Scan</key>
    <dict>
        <key>Entries</key>
        <true/>
        <key>Legacy</key>
        <false/>
        <key>Tool</key>
        <true/>
    </dict>

#### 安装系统

使用 Windows 系统盘启动电脑，按照正常安装流程，把系统安装在支持 NVMe 协议的固态硬盘（由于在引导盘已经建立了 ESP，Windows 只会建立 MSR 分区和系统分区），重启后在 Clover 引导页可以看到新安装 Windows 系统选项，选中即可使用安装 NVMe 固态硬盘上的 Windows 系统。
