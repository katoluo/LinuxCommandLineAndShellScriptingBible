## 第九章 安装软件程序

​	本章将介绍Linux上能见到的各种包管理系统(package management system,PMS)，以及用来进行软件安装、管理和删除的命令行工具。

### 9.1 包管理基础

​	在深入了解Linux软件包管理之前,本章将先介绍一些基础知识。各种主流Linux发行版都采用了某种形式的包管理系统来控制软件和库的安装。PMS利用一个数据库来记录各种相关内容:

- Linux系统上已安装了什么软件包;
- 每个包安装了什么文件;
- 每个已安装软件包的版本。

​	软件包存储在服务器上,可以利用本地Linux系统上的PMS工具通过互联网访问。这些服务器称为仓库(repository)。可以用PMS工具来搜索新的软件包,或者是更新系统上已安装软件包。

​	软件包通常会依赖其他的包,为了前者能够正常运行,被依赖的包必须提前安装在系统中。PMS工具将会检测这些依赖关系,并在安装需要的包之前先安装好所有额外的软件包。

​	PMS的不足之处在于目前还没有统一的标准工具。不管你用的是哪个Linux发行版,本书到目前为止所讨论的bash shell命令都能工作,但对于软件包管理可就不一定了。

​	PMS工具及相关命令在不同的Linux发行版上有很大的不同。Linux中广泛使用的两种主要的PMS基础工具是 dpkg 和 rpm 。

​	基于Debian的发行版(如Ubuntu和Linux Mint)使用的是 dpkg 命令,这些发行版的PMS工具也是以该命令为基础的。 dpkg 会直接和Linux系统上的PMS交互,用来安装、管理和删除软件包。

​	基于Red Hat的发行版(如Fedora、 openSUSE及Mandriva)使用的是 rpm 命令,该命令是其PMS的底层基础。类似于 dpkg 命令, rmp 命令能够列出已安装包、安装新包和删除已有软件。

​	注意,这两个命令是它们各自PMS的核心,并非全部的PMS。许多使用 dpkg 或 rpm 命令的Linux发行版都有各自基于这些命令的特定PMS工具,这些工具能够助你事半功倍。随后几节将带你逐步了解主流Linux发行版上的各种PMS工具命令。

### 9.2 基于 Arch Linux 系统

​	原书上写的是基于Debian和Red Hat系统的包管理系统。这里记录Arch Linux的pacman软件包管理器。

#### 9.2.1 用法
