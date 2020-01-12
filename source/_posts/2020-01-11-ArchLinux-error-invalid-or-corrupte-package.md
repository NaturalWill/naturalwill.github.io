---
layout: blog
title: ArchLinux 出现错误 invalid or corrupted package (PGP signature) 的解决方法
date: 2020-01-11 14:33:00
tags:
  - Archlinux
categories:
  - 计算机
  - Linux
---

升级系统时，有时会出现这个错误信息:

    error: vim: signature from "Levente Polyak (anthraxx) <levente@leventepolyak.net>" is unknown trust
    :: File /var/cache/pacman/pkg/vim-8.2.0100-1-x86_64.pkg.tar.zst is corrupted (invalid or corrupted package (PGP signature)).
    Do you want to delete it? [Y/n]
    error: failed to commit transaction (invalid or corrupted package)
    Errors occurred, no packages were upgraded.

可以尝试以下步骤：

1. 更新 archlinux-keyring

```bash
pacman -Sy archlinux-keyring
```

2. 更新所有密钥

```bash
pacman-key --refresh-keys
```

<!-- more -->

## pacman-key 相关命令

1. 确保正确初始化密匙环

```bash
pacman-key --init
```

2. 从 `/usr/share/pacman/keyring` 中重新加载默认密钥

```bash
pacman-key --populate
```
