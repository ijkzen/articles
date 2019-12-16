---
title: 解决Windows和Ubuntu双系统时间混乱的问题
categories: issues

---



使用Windows和Ubuntu双系统的同学可能会发现这样的问题，在双系统切换的时候，总会有一个系统的时间不对，这两个系统的显示时间一般是相差8小时。

## 原因

这是因为这两个系统对于硬件时间处理方法不同，Windows将机器时间视为本地时间，而Ubuntu将机器时间视为相对时间，真正用于显示的本地时间由相对时间和时区偏移组成。这样解决方案就有两种在Ubuntu中禁用UTC转而使用机器时间，或者在Windows中启用UTC。

## 解决方案

### 在Ubuntu中禁用UTC

在Ubuntu 16.04之前的版本，通过修改`/etc/default/rcs`文件内容禁用UTC；

在Ubuntu 16.04之后包括16.04的版本，时间管理交给了`timedatectl`，运行下方命令：

```shell
timedatectl set-local-rtc 1 --adjust-system-clock
```

### 在Windows中启用UTC

**打开管理员模式的cmd或者powershell**，输入下方命令：

```powershell
reg add HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation /v RealTimeIsUniversal /t REG_DWORD /d 1
```

通过修改注册表实现；

上述方法重启生效。

