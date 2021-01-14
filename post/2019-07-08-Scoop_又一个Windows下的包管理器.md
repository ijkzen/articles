---
title: Scoop --- Windows下绿色软件的包管理器
categories: tool
---



之前使用Chocolatey这个在Windows下的包管理器，在使用了一段时间后发现了一些问题：

1. 软件没有集中安装在一个目录下
2. 软件所安装的目录下文件无法编辑
3. 命令行软件更新出错

## 问题

### 软件没有集中安装在一个目录下

对于这个问题，官方的文档如下：

```
Where are Chocolatey packages installed to?
Chocolatey packages are installed to ChocolateyInstall\lib, but the software could go to various locations, depending on how the package maintainer created the package.

Some packages are installed under ChocolateyInstall\lib, others - especially packages that are based on Windows installers (.msi files) - install to the default path of the original installer (which is most likely within Program Files).

There are also packages for which you can set a custom installation path. These packages (like ruby) use the $env:ChocolateyBinRoot environment variable. If this variable does not exist, it will be created as c:\tools e.g. C:\tools\ruby193. To change this behaviour, you can set $env:ChocolateyBinRoot to an existing folder, e. g. C:\somestuff. Packages that use the environment variable, will then be installed in the given subfolder, f. ex. C:\somestuff\ruby193.
```

大意如下：

```
一般软件安装在 ChocolateyInstall\lib 目录下，但是根据包维护者的意愿，软件可能会安装在其他地方；
msi 文件将会安装在默认的路径下，而非ChocolateyInstall\lib目录；
还有一些包，你可以自定义安装路径；这些包大多是编程环境就像是JDK、python之类；注意这里的环境变量名已经变了，变成了ChocolateyToolsLocation：
```

关于ChocolateyInstall的目录，请查看环境变量；

### 权限问题

第二个问题的具体表现：无法修改Android Studio配置文件`studio64.exe.vmoptions`；

第三个问题的具体表现：Jetbrains系列不能在应用内部进行增量更新；使用命令行更新，会直接下载全量包安装新的版本，而旧的版本不会被卸载；

第二和第三个问题其实是同一个问题，也就是权限问题；

Chocolatey安装软件使用了管理员权限，这和Linux有一些相似，但是在Windows中权限管理做的并不好，你甚至不能通过命令行获得管理员权限；正是管理员权限的滥用，导致了上述问题；

## Scoop —— 可替代的方案

Scoop和Chocolatey究竟不同在哪里呢？官方文档表述如下：

- **Installs to ~/scoop/ by default.** You can set up your own programs and not worry that they'll interfere with other users' programs (or theirs with yours, perhaps more importantly). You can optionally choose to install programs system-wide if you have administrator rights.
- **No UAC popups, doesn't require admin rights.** Since programs are installed just for your user account, you won't be interrupted by UAC popups.
- **Doesn't pollute your path.** Where possible, Scoop puts your program shims in a single directory and just adds that to your path.
- **Doesn't use NuGet.** NuGet is a great solution to the problem of managing software library dependencies. Scoop avoids this problem altogether: each program you install is isolated and independent.
- **Simpler than packaging.** Scoop isn't a package manager, rather it reads plain JSON manifests that describe how to install a program and its dependencies.
- **Simpler app repository.** Scoop just uses Git for its app repository. You can create your own repo, or even just a single file that describes an app to install.
- **Can't always install a specific version of a program.** For some programs, scoop can install an older version of a program, via `scoop install app@version`. For example, `scoop install curl@7.56.1`. This functionality only works if the old version is still available online. Some older versions have specific installers, such as Python 2.7 and Ruby 1.9, which are commonly required. These can be installed from the [versions](https://github.com/scoopinstaller/versions/) bucket via `scoop install python27` and `scoop install ruby19`.

大意如下：

* 软件默认安装在用户目录下，避免了权限问题；
* 不再有烦人的UAC弹窗，不需要管理员权限
* 不会污染你的环境
* 不使用nuget，每个软件都是独立的
* 不需要打包，使用json来描述软件安装的过程
* 简单的软件仓库，使用git管理软件，你可以切换软件到任意的版本
* 可以安装特定版本的软件

通过上述表述，可以知道对于软件的安装，Scoop和Chocolatey是完全不同的，Chocolatey是同传统方式使用注册表，而Scoop更像是绿色软件；

### bucket

关于什么是bucket？官方文档如下：

```
What are buckets?
In Scoop, buckets are collections of apps. Or, to be more specific, a bucket is a Git repository containing JSON app manifests which describe how to install an app.
```

大意如下：

bucket是软件的集合，或者确切的说是包含许多软件安装描述信息的JSON文件的仓库；

更加直观的说法，有一点像Ubuntu中的ppa；

### 安装Scoop

```powershell
iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
```

上述命令将会执行远程脚本，将Scoop安装在用户目录(C:/Users/<username>/scoop)下，并且将该目录添加到环境变量；

如果你想将Scoop安装到其他目录，请执行下方命令：

```powershell
$env:SCOOP='<your-path>'
[environment]::setEnvironmentVariable('SCOOP',$env:SCOOP,'User')
iex (new-object net.webclient).downloadstring('https://get.scoop.sh')
```

注意替换路径；

### 安装软件

```powershell
scoop install <software>
```

执行上述命令，软件将会被安装在用户目录下；

或许有一些全局应用，你想要安装到特定目录：

```powershell
$env:SCOOP_GLOBAL='<your-path>'
[environment]::setEnvironmentVariable('SCOOP_GLOBAL',$env:SCOOP_GLOBAL,'Machine')
scoop install -g <app>
```

注意替换路径；并且在安装时加上`-g`参数；

```powershell
scoop bucket add jetbrains
scoop install IntelliJ-IDEA-Ultimate
```

上述代码，用于安装IDEA专业版；想要添加更多的bucket，使用search命令搜索关键词，将会提示添加bucket；

### 一些技巧

#### 代理

在添加bucket或者搜索包的时候，会耗费特别长的时间，这是因为scoop在访问Github上的资源，由于你懂的原因，国内访问GitHub十分缓慢，这时候就需要代理工具：

```powershell
scoop config proxy proxy.example.org:8080
```

删除代理：

```powershell
scoop config rm proxy
```

### 多线程下载

Scoop can utilize [`aria2`](https://github.com/aria2/aria2) to use multi-connection downloads. Simply install `aria2` through Scoop and it will be used for all downloads afterward.

```
scoop install aria2
```

You can tweak the following `aria2` settings with the `scoop config` command:

- aria2-enabled (default: true)
- [aria2-retry-wait](https://aria2.github.io/manual/en/html/aria2c.html#cmdoption-retry-wait) (default: 2)
- [aria2-split](https://aria2.github.io/manual/en/html/aria2c.html#cmdoption-s) (default: 5)
- [aria2-max-connection-per-server](https://aria2.github.io/manual/en/html/aria2c.html#cmdoption-x) (default: 5)
- [aria2-min-split-size](https://aria2.github.io/manual/en/html/aria2c.html#cmdoption-k) (default: 5M)

```powershell
scoop config aria2-split 32
```

