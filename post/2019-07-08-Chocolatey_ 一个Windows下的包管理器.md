---
title: Chocolatey --- Windows下高权限的包管理器
categories: tool
---


  
## Chocolatey

## 安装要求

* Windows 7 以上 或 Windows Server 2003以上
* Power Shell v2+
* .NET Framework 4+ (the installation will attempt to install .NET 4.0 if you do not have it installed)

## 安装过程

Chocolatey installs in seconds. You are just a few steps from running choco right now!

1. First, ensure that you are using an **administrative shell** - you can also install as a non-admin, check out [Non-Administrative Installation](https://chocolatey.org/docs/installation#non-administrative-install).
2. Copy the text specific to your command shell - [cmd.exe](https://chocolatey.org/docs/installation#install-with-cmdexe) or [powershell.exe](https://chocolatey.org/docs/installation#install-with-powershellexe).
3. Paste the copied text into your shell and press Enter.
4. Wait a few seconds for the command to complete.
5. If you don't see any errors, you are ready to use Chocolatey! Type `choco` or `choco -?` now, or see [Getting Started](https://chocolatey.org/docs/getting-started) for usage instructions.

### Install with cmd

```
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

### Install with PowerShell

```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

## 卸载过程

Chocolatey 并不是真正安装在你的电脑上，而是通过环境变量进行配置；所以你只需要移除**目录**和**环境变量**即可，并不需要修改注册表；

### Folder

* C:\Chocolatey - for Chocolatey version < `0.9.8.27`
* C:\ProgramData\chocolatey > `0.9.8.27`

**NOTE:** If you upgraded from `0.9.8.26` to `0.9.8.27` it is likely that `Chocolatey` is installed at C:\Chocolatey, which was the default prior to `0.9.8.27`. If you did a fresh install of Chocolatey at version `0.9.8.27` then the installation folder will be `C:\ProgramData\Chocolatey`

### Environment Variables

* ChocolateyInstall
* ChocolateyBinRoot
* ChocolateyToolsLocation
* PATH (will need updated to remove)

## 安装一个软件

```
choco install <package name>
```

## 卸载一个软件

```
choco uninstall <package name>
```

## 安装目录

所有通过choco安装的软件被解压在`$env:ChocolateyInstall\lib`下，这个目录在C盘，而且不可修改，这是一个不太方便的地方，通常来说，C盘并没有太大的空间；

## Options

```
-?, --help, -h
     Prints out the help menu.

 -d, --debug
     Debug - Show debug messaging.

 -v, --verbose
     Verbose - Show verbose messaging. Very verbose messaging, avoid using 
       under normal circumstances.

     --trace
     Trace - Show trace messaging. Very, very verbose trace messaging. Avoid 
       except when needing super low-level .NET Framework debugging. Available 
       in 0.10.4+.

     --nocolor, --no-color
     No Color - Do not show colorization in logging output. This overrides 
       the feature 'logWithoutColor', set to 'False'. Available in 0.10.9+.

     --acceptlicense, --accept-license
     AcceptLicense - Accept license dialogs automatically. Reserved for 
       future use.

 -y, --yes, --confirm
     Confirm all prompts - Chooses affirmative answer instead of prompting. 
       Implies --accept-license

 -f, --force
     Force - force the behavior. Do not use force during normal operation - 
       it subverts some of the smart behavior for commands.

     --noop, --whatif, --what-if
     NoOp / WhatIf - Don't actually do anything.

 -r, --limitoutput, --limit-output
     LimitOutput - Limit the output to essential information

     --timeout, --execution-timeout=VALUE
     CommandExecutionTimeout (in seconds) - The time to allow a command to 
       finish before timing out. Overrides the default execution timeout in the 
       configuration of 2700 seconds. '0' for infinite starting in 0.10.4.

 -c, --cache, --cachelocation, --cache-location=VALUE
     CacheLocation - Location for download cache, defaults to %TEMP% or value 
       in chocolatey.config file.

     --allowunofficial, --allow-unofficial, --allowunofficialbuild, --allow-unofficial-build
     AllowUnofficialBuild - When not using the official build you must set 
       this flag for choco to continue.

     --failstderr, --failonstderr, --fail-on-stderr, --fail-on-standard-error, --fail-on-error-output
     FailOnStandardError - Fail on standard error output (stderr), typically 
       received when running external commands during install providers. This 
       overrides the feature failOnStandardError.

     --use-system-powershell
     UseSystemPowerShell - Execute PowerShell using an external process 
       instead of the built-in PowerShell host. Should only be used when 
       internal host is failing. Available in 0.9.10+.

     --no-progress
     Do Not Show Progress - Do not show download progress percentages. 
       Available in 0.10.4+.

     --proxy=VALUE
     Proxy Location - Explicit proxy location. Overrides the default proxy 
       location of ''. Available for config settings in 0.9.9.9+, this CLI 
       option available in 0.10.4+.

     --proxy-user=VALUE
     Proxy User Name - Explicit proxy user (optional). Requires explicity 
       proxy (`--proxy` or config setting). Overrides the default proxy user of 
       '123'. Available for config settings in 0.9.9.9+, this CLI option 
       available in 0.10.4+.

     --proxy-password=VALUE
     Proxy Password - Explicit proxy password (optional) to be used with 
       username. Requires explicity proxy (`--proxy` or config setting) and 
       user name.  Overrides the default proxy password (encrypted in settings 
       if set). Available for config settings in 0.9.9.9+, this CLI option 
       available in 0.10.4+.

     --proxy-bypass-list=VALUE
     ProxyBypassList - Comma separated list of regex locations to bypass on 
       proxy. Requires explicity proxy (`--proxy` or config setting). Overrides 
       the default proxy bypass list of ''. Available in 0.10.4+.

     --proxy-bypass-on-local
     Proxy Bypass On Local - Bypass proxy for local connections. Requires 
       explicity proxy (`--proxy` or config setting). Overrides the default 
       proxy bypass on local setting of 'True'. Available in 0.10.4+.

     --log-file=VALUE
     Log File to output to in addition to regular loggers. Available in 0.1-
       0.8+.

 -n, --name=VALUE
     Name - the name of the source. Required with some actions. Defaults to 
       empty.
```

## Document

更多信息请查询官网[文档](https://chocolatey.org/docs/)