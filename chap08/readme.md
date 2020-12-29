## 第八章 管理文件系统

​	使用Linux系统时,需要作出的决策之一就是为存储设备选用什么文件系统。大多数Linux发行版在安装时会非常贴心地提供默认的文件系统,大多数入门级用户想都不想就用了默认的那个。

​	使用默认文件系统未必就不好,但了解一下可用的选择有时也会有所帮助。本章将探讨Linux世界里可选用的不同文件系统,并向你演示如何在命令行上进行创建和管理。

### 8.1 探索Linux文件系统

​	第3章讨论了Linux如何通过文件系统来在存储设备上存储文件和目录。Linux的文件系统为我们在硬盘中存储的0和1和应用中使用的文件与目录之间搭建起了一座桥梁。

​	Linux支持多种类型的文件系统管理文件和目录。每种文件系统都在存储设备上实现了虚拟目录结构,仅特性略有不同。本章将带你逐步了解Linux环境中较常用的文件系统的优点和缺陷。

#### 8.1.1 基本的Linux文件系统

​	**1. ext文件系统**

​	Linux操作系统中引入的最早的文件系统叫作扩展文件系统 (`extended filesystem`,简记为ext)。它为Linux提供了一个基本的类Unix文件系统:使用虚拟目录来操作硬件设备,在物理设备上按定长的块来存储数据。

​	ext文件系统采用名为索引节点的系统来存放虚拟目录中所存储文件的信息。索引节点系统在每个物理设备中创建一个单独的表(称为索引节点表)来存储这些文件的信息。存储在虚拟目录中的每一个文件在索引节点表中都有一个条目。ext文件系统名称中的extended部分来自其跟踪的每个文件的额外数据,包括:

- 文件名
- 文件大小
- 文件的属主
- 文件的属组
- 文件的访问权限
- 指向存有文件数据的每个硬盘块的指针

​	Linux通过唯一的数值(称作索引节点号)来引用索引节点表中的每个索引节点,这个值是创建文件时由文件系统分配的。文件系统通过索引节点号而不是文件全名及路径来标识文件。

​	**2. ext2文件系统**

​	最早的ext文件系统有不少限制,比如文件大小不得超过2 GB。在Linux出现后不久,ext文件系统就升级到了第二代扩展文件系统,叫作ext2。

​	如你所猜测的, ext2文件系统是ext文件系统基本功能的一个扩展,但保持了同样的结构。 ext2文件系统扩展了索引节点表的格式来保存系统上每个文件的更多信息。

​	ext2的索引节点表为文件添加了创建时间值、修改时间值和最后访问时间值来帮助系统管理员追踪文件的访问情况。ext2文件系统还将允许的最大文件大小增加到了2 TB(在ext2的后期版本中增加到了32 TB),以容纳数据库服务器中常见的大文件。

​	除了扩展索引节点表外,ext2文件系统还改变了文件在数据块中存储的方式。ext文件系统常见的问题是在文件写入到物理设备时,存储数据用的块很容易分散在整个设备中(称作碎片化,fragmentation)。数据块的碎片化会降低文件系统的性能,因为需要更长的时间在存储设备中查找特定文件的所有块。

​	保存文件时,ext2文件系统通过按组分配磁盘块来减轻碎片化。通过将数据块分组,文件系统在读取文件时不需要为了数据块查找整个物理设备。

​	多年来,ext文件系统一直都是Linux发行版采用的默认文件系统。但它也有一些限制。索引节点表虽然支持文件系统保存有关文件的更多信息,但会对系统造成致命的问题。文件系统每次存储或更新文件,它都要用新信息来更新索引节点表。问题在于这种操作并非总是一气呵成的。

​	如果计算机系统在存储文件和更新索引节点表之间发生了什么,这二者的内容就不同步了。ext2文件系统由于容易在系统崩溃或断电时损坏而臭名昭著。即使文件数据正常保存到了物理设备上,如果索引节点表记录没完成更新的话,ext2文件系统甚至都不知道那个文件存在!

#### 8.1.2 日志文件系统

​	日志文件系统为Linux系统增加了一层安全性。它不再使用之前先将数据直接写入存储设备再更新索引节点表的做法,而是先将文件的更改写入到临时文件(称作日志,journal)中。在数据成功写到存储设备和索引节点表之后,再删除对应的日志条目。

​	如果系统在数据被写入存储设备之前崩溃或断电了,日志文件系统下次会读取日志文件并处理上次留下的未写入的数据。

​	Linux中有3种广泛使用的日志方法,每种的保护等级都不相同。

|   方法   |                             描述                             |
| :------: | :----------------------------------------------------------: |
| 数据模式 |     索引节点和文件都会被写入日志;丢失数据风险低,但性能差     |
| 有序模式 | 只有索引节点数据会被写入日志,但只有数据成功写入后才删除;在性能和安全性之间取得了良好的折中 |
| 回写模式 | 只有索引节点数据会被写入日志,但不控制文件数据何时写入;丢失数据风险高,但仍比不用日志好 |

​	数据模式日志方法是目前为止最安全的数据保护方法,但同时也是最慢的。所有写到存储设备上的数据都必须写两次:第一次写入日志,第二次写入真正的存储设备。这样会导致性能很差,尤其是对要做大量数据写入的系统而言。

​	这些年来,在Linux上还出现了一些其他日志文件系统。后面几节将会讲述常见的Linux日志文件系统。

​	**1. ext3文件系统**

​	2001年,ext3文件系统被引入Linux内核中,直到最近都是几乎所有Linux发行版默认的文件系统。它采用和ext2文件系统相同的索引节点表结构,但给每个存储设备增加了一个日志文件,以将准备写入存储设备的数据先记入日志。

​	默认情况下,ext3文件系统用有序模式的日志功能——只将索引节点信息写入日志文件,直到数据块都被成功写入存储设备才删除。你可以在创建文件系统时用简单的一个命令行选项将ext3文件系统的日志方法改成数据模式或回写模式。

​	虽然ext3文件系统为Linux文件系统添加了基本的日志功能,但它仍然缺少一些功能。例如ext3文件系统无法恢复误删的文件,它没有任何内建的数据压缩功能(虽然有个需单独安装的补丁支持这个功能),ext3文件系统也不支持加密文件。鉴于这些原因,Linux项目的开发人员选择再接再厉,继续改进ext3文件系统。

​	**2. ext4文件系统**

​	扩展ext3文件系统功能的结果是ext4文件系统(你可能也猜出来了)。ext4文件系统在2008年受到Linux内核官方支持,现在已是大多数流行的Linux发行版采用的默认文件系统,比如Ubuntu。

​	除了支持数据压缩和加密,ext4文件系统还支持一个称作区段(extent)的特性。区段在存储设备上按块分配空间,但在索引节点表中只保存起始块的位置。由于无需列出所有用来存储文件中数据的数据块,它可以在索引节点表中节省一些空间。

​	ext4还引入了块预分配技术(block preallocation)。如果你想在存储设备上给一个你知道要变大的文件预留空间,ext4文件系统可以为文件分配所有需要用到的块,而不仅仅是那些现在已经用到的块。ext4文件系统用 0 填满预留的数据块,不会将它们分配给其他文件。

​	**3. Reiser文件系统**

​	2001年,Hans Reiser为Linux创建了第一个称为ReiserFS的日志文件系统。ReiserFS文件系统只支持回写日志模式——只把索引节点表数据写到日志文件。 ReiserFS文件系统也因此成为Linux上最快的日志文件系统之一。

​	有两个有意思的特性被引入了ReiserFS文件系统:一个是你可以在线调整已有文件系统的大小;另一个是被称作尾部压缩(tailpacking)的技术,该技术能将一个文件的数据填进另一个文件的数据块中的空白空间。如果你必须为已有文件系统扩容来容纳更多的数据,在线调整文件系统大小功能非常好用。

​	**4. JFS文件系统**

​	作为可能依然在用的最老的日志文件系统之一,JFS(Journaled File System,日志化文件系统 )是IBM在1990年为其Unix衍生版AIX开发的。然而直到第2版,它才被移植到Linux环境中。

​	JFS文件系统采用的是有序日志方法,即只在日志中保存索引节点表数据,直到真正的文件数据被写进存储设备时才删除它。这个方法在ReiserFS的速度和数据模式日志方法的完整性之间的采取的一种折中。

​	JFS文件系统采用基于区段的文件分配,即为每个写入存储设备的文件分配一组块。这样可以减少存储设备上的碎片。

​	除了用在IBM Linux上外,JFS文件系统并没有流行起来,但你有可能在同Linux打交道的日子中碰到它。

​	**5. XFS文件系统**

​	XFS日志文件系统是另一种最初用于商业Unix系统而如今走进Linux世界的文件系统。美国硅图公司(SGI)最初在1994年为其商业化的IRIX Unix系统开发了XFS。2002年,它被发布到了适用于Linux环境的版本。

​	XFS文件系统采用回写模式的日志,在提供了高性能的同时也引入了一定的风险,因为实际数据并未存进日志文件。XFS文件系统还允许在线调整文件系统的大小,这点类似于ReiserFS文件系统,除了XFS文件系统只能扩大不能缩小。

#### 8.1.3 写时复制文件系统

​	采用了日志式技术,你就必须在安全性和性能之间做出选择。尽管数据模式日志提供了最高的安全性,但是会对性能带来影响,因为索引节点和数据都需要被日志化。如果是回写模式日志,性能倒是可以接受,但安全性就会受到损害。

​	就文件系统而言,日志式的另一种选择是一种叫作写时复制 (copy-on-write,COW)的技术。COW利用快照兼顾了安全性和性能。如果要修改数据,会使用克隆或可写快照。修改过的数据并不会直接覆盖当前数据,而是被放入文件系统中的另一个位置上。即便是数据修改已经完成,之前的旧数据也不会被重写。

​	COW文件系统已日渐流行,接下来会简要概览其中最流行的两种(Btrf和ZFS)。

​	**1. ZFS文件系统**

​	COW文件系统ZFS是由Sun公司于2005年研发的,用于OpenSolaris操作系统,从2008年起开始向Linux移植,最终在2012年投入Linux产品的使用。

​	ZFS是一个稳定的文件系统,与Resier4、Btrfs和ext4势均力敌。它最大的弱项就是没有使用GPL许可。自2013年发起的OpenZFS项目有可能改变这种局面。但是,在获得GPL许可之前,ZFS有可能终无法成为Linux默认的文件系统。

​	**2. Btrf文件系统**

​	Btrfs文件系统是COW的新人,也被称为B树文件系统。它是由Oracle公司于2007年开始研发的。Btrfs在Reiser4的诸多特性的基础上改进了可靠性。另一些开发人员最终也加入了开发过程,帮助Btrfs快速成为了最流行的文件系统。究其原因,则要归于它的稳定性、易用性以及能够动态调整已挂载文件系统的大小。OpenSUSE Linux发行版最近将Btrfs作为其默认文件系统。除此之外,该文件系统也出现在了其他Linux发行版中(如RHEL),不过并不是作为默认文件系统。

### 8.2 操作文件系统

​	Linux提供了一些不同的工具,我们可以利用它们轻松地在命令行中进行文件系统操作。可使用键盘随心所欲地创建新的文件系统或者修改已有的文件系统。本节将会带你逐步了解命令行下的文件系统交互的命令。

#### 8.2.1 创建分区

​	一开始,你必须在存储设备上创建分区来容纳文件系统。分区可以是整个硬盘,也可以是部分硬盘,以容纳虚拟目录的一部分。

​	fdisk 工具用来帮助管理安装在系统上的任何存储设备上的分区。它是个交互式程序,允许你输入命令来逐步完成硬盘分区操作。

​	要启动 fdisk 命令,你必须指定要分区的存储设备的设备名,另外还得有超级用户权限。如果在没有对应权限的情况下使用该命令,你会得到类似于下面这种错误提示。

```shell
$ fdisk /dev/sdb
Unable to open /dev/sdb
```

​	如果你拥有超级用户权限并指定了正确的驱动器,那就可以进入 fdisk 工具的操作界面了。下面展示了该命令在CentOS发行版中的使用情景。

```shell
$ sudo fdisk /dev/sdb
[sudo] password for Christine:
Device contains neither a valid DOS partition table,
nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0xd3f759b5.
Changes will remain in memory only
until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will
be corrected by w(rite)

[...]
Command (m for help):
```

​	fdisk 交互式命令提示符使用单字母命令来告诉 fdisk 做什么。

| 命令 |               描述               |
| :--: | :------------------------------: |
|  a   |         设置活动分区标志         |
|  b   |   编辑BSD Unix系统用的磁盘标签   |
|  c   |         设置DOS兼容标志          |
|  d   |             删除分区             |
|  l   |        显示可用的分区类型        |
|  m   |           显示命令选项           |
|  n   |          添加一个新分区          |
|  o   |          创建DOS分区表           |
|  p   |          显示当前分区表          |
|  q   |         退出,不保存更改          |
|  s   | 为Sun Unix系统创建一个新磁盘标签 |
|  t   |         修改分区的系统ID         |
|  u   |        改变使用的存储单位        |
|  v   |            验证分区表            |
|  w   |         将分区表写入磁盘         |
|  x   |             高级功能             |

​	尽管看上去很恐怖,但实际上你在日常工作中用到的只有几个基本命令。

​	对于初学者,可以用 p 命令将一个存储设备的详细信息显示出来。

```shell
Command (m for help): p
Disk /dev/sdb: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x11747e88
	Device Boot		Start		End		Blocks		Id		System

Command (m for help):
```

​	输出显示这个存储设备有5368 MB(5 GB)的空间。存储设备明细后的列表说明这个设备上是否已有分区。这个例子中的输出中没有显示任何分区,所以设备还未分区。

​	下一步,可以使用 n 命令在该存储设备上创建新的分区。

```shell
Command (m for help): n
Command action
	e	extended
	p	primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-652, default 1): 1
Last cylinder, +cylinders or +size{K,M,G} (1-652, default 652): +2G

Command (m for help):
```

​	分区可以按主分区(primary partition)或扩展分区(extended partition)创建。主分区可以被文件系统直接格式化,而扩展分区则只能容纳其他主分区(说法有误，扩展分区内容纳的应该是“逻辑分区”(logical partition)) 。扩展分区出现的原因是每个存储设备上只能有4个分区。可以通过创建多个扩展分区,然后在扩展分区内创建逻辑分区进行扩展。上例中创建了一个主分区,在存储设备上给它分配了分区号1,然后给它分配了2 GB的存储设备空间。你可以再次使用 p 命令查看结果。

```shell
Command (m for help): p

Disk /dev/sdb: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x029aa6af

	Device Boot		Start		End		Blocks		Id		System
	/dev/sdb1		1			262		2104483+	83		Linux
	
Command (m for help):
```

​	从输出中现在可以看到,该存储设备上有了一个分区(叫作/dev/sdb1)。 Id 列定义了Linux怎么对待该分区。 fdisk 允许创建多种分区类型。使用 l 命令列出可用的不同类型。默认类型是 83 ,该类型定义了一个Linux文件系统。如果你想为其他文件系统创建一个分区(比如Windows的NTFS分区)，只要选择一个不同的分区类型即可。

​	可以重复上面的过程,将存储设备上剩下的空间分配给另一个Linux分区。创建了想要的分区之后,用 w 命令将更改保存到存储设备上。

```shell
Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.
$
```

​	存储设备的分区信息被写入分区表中,Linux系统通过 ioctl() 调用来获知新分区的出现。设置好分区之后,可以使用Linux文件系统对其进行格式化。

#### 8.2.2 创建文件系统

​	在将数据存储到分区之前,你必须用某种文件系统对其进行格式化,这样Linux才能使用它。每种文件系统类型都用自己的命令行程序来格式化分区。

|     工具     |           用途           |
| :----------: | :----------------------: |
|   `mkefs`    |   创建一个ext文件系统    |
|   `mke2fs`   |   创建一个ext2文件系统   |
| `mkfs.ext3`  |   创建一个ext3文件系统   |
| `mkfs.ext4`  |   创建一个ext4文件系统   |
| `mkreiserfs` | 创建一个ReiserFS文件系统 |
|  `jfs_mkfs`  |   创建一个JFS文件系统    |
|  `mkfs.xfs`  |   创建一个XFS文件系统    |
|  `mkfs.zfs`  |   创建一个ZFS文件系统    |
| `mkfs.btrfs` |  创建一个Btrfs文件系统   |

​	并非所有文件系统工具都已经默认安装了。要想知道某个文件系统工具是否可用,可以使用type 命令。

```shell
$ type mkfs.ext4
mkfs.ext4 is /sbin/mkfs.ext4
$
$ type mkfs.btrfs
-bash: type: mkfs.btrfs: not found
$
```

​	据上面这个取自`Ubuntu`系统的例子显示, `mkfs.ext4` 工具是可用的。而`Btrfs`工具则不可用。请参阅第9章中有关如何在Linux发行版中安装软件和工具的相关内容。

​	每个文件系统命令都有很多命令行选项,允许你定制如何在分区上创建文件系统。要查看所有可用的命令行选项,可用 man 命令来显示该文件系统命令的手册页面(参见第3章)。所有的文件系统命令都允许通过不带选项的简单命令来创建一个默认的文件系统。

```shell
$ sudo mkfs.ext4 /dev/sdb1
[sudo] password for Christine:
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
131648 inodes, 526120 blocks
26306 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=541065216
17 block groups
32768 blocks per group, 32768 fragments per group
7744 inodes per group
Superblock backups stored on blocks:
32768, 98304, 163840, 229376, 294912
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
This filesystem will be automatically checked every 23 mounts or
180 days, whichever comes first. Use tune2fs -c or -i to override.
$
```

​	这个新的文件系统采用ext4文件系统类型,这是Linux上的日志文件系统。注意,创建过程中有一步是创建新的日志。

​	为分区创建了文件系统之后,下一步是将它挂载到虚拟目录下的某个挂载点,这样就可以将数据存储在新文件系统中了。你可以将新文件系统挂载到虚拟目录中需要额外空间的任何位置。

```shell
$ ls /mnt
$
$ sudo mkdir /mnt/my_partition
$
$ ls -al /mnt/my_partition/
$
$ ls -dF /mnt/my_partition
/mnt/my_partition/
$
$ sudo mount -t ext4 /dev/sdb1
$
$ ls -al /mnt/my_partition/
total 24
drwxr-xr-x. 3 root root 4096 Jun 11 09:53 .
drwxr-xr-x. 3 root root 4096 Jun 11 09:58 ..
drwx------. 2 root root 16384 Jun 11 09:53 lost+found
```

​	mkdir 命令(参见第3章)在虚拟目录中创建了挂载点, mount 命令将新的硬盘分区添加到挂载点。 mount 命令的 -t 选项指明了要挂载的文件系统类型(ext4)。现在你可以在新分区中保存新文件和目录了!

***

**说明**	这种挂载文件系统的方法只能临时挂载文件系统。当重启Linux系统时,文件系统并不会自动挂载。要强制Linux在启动时自动挂载新的文件系统,可以将其添加到/etc/fstab文件。

***

​	现在文件系统已经被挂载了到虚拟目录中,可以投入日常使用了。遗憾的是,在日常使用过程中有可能会出现一些严重的问题,例如文件系统损坏。下一节将演示如何应对这种问题。

#### 8.2.3 文件系统的检查与修复

​	就算是现代文件系统,碰上突然断电或者某个不规矩的程序在访问文件时锁定了系统,也会出现错误。幸而有一些命令行工具可以帮你将文件系统恢复正常。

​	每个文件系统都有各自可以和文件系统交互的恢复命令。这可能会让局面变得不太舒服,随着Linux环境中可用的文件系统变多,你也不得不去掌握大量对应的命令。好在有个通用的前端程序,可以决定存储设备上的文件系统并根据要恢复的文件系统调用适合的文件系统恢复命令。

​	fsck 命令能够检查和修复大部分类型的Linux文件系统,包括本章早些时候讨论过的ext、ext2、ext3、ext4、ReiserFS、JFS和XFS。该命令的格式是:

​	`fsck options filesystem`

​	你可以在命令行上列出多个要检查的文件系统。文件系统可以通过设备名、在虚拟目录中的挂载点以及分配给文件系统的唯一UUID值来引用。

​	fsck 命令使用/etc/fstab文件来自动决定正常挂载到系统上的存储设备的文件系统。如果存储设备尚未挂载(比如你刚刚在新的存储设备上创建了个文件系统)，你需要用 -t 命令行选项来指定文件系统类型。表8-4列出了其他可用的命令行选项。

![表8-4.png]()

​	你可能注意到了,有些命令行选项是重复的。这是为多个命令实现通用的前端带来的部分问题。有些文件系统修复命令有一些额外的可用选项。如果要做更高级的错误检查,就需要查看这个文件系统修复工具的手册页面来确定是不是有该文件系统专用的扩展选项。

***

**窍门**	只能在未挂载的文件系统上运行 fsck 命令。对大多数文件系统来说,你只需卸载文件系统来进行检查,检查完成之后重新挂载就好了。但因为根文件系统含有所有核心的Linux命令和日志文件,所以你无法在处于运行状态的系统上卸载它。这正是亲手体验Linux LiveCD的好时机!只需用LiveCD启动系统即可,然后在根文件系统上运行 fsck 命令。

***

​	到目前为止,本章讲解了如何处理物理存储设备中的文件系统。Linux还有另一些方法可以为文件系统创建逻辑存储设备。下一节将告诉你如何使用逻辑存储设备。

### 8.3 逻辑卷管理

​	如果用标准分区在硬盘上创建了文件系统,为已有文件系统添加额外的空间多少是一种痛苦的体验。你只能在同一个物理硬盘的可用空间范围内调整分区大小。如果硬盘上没有地方了,你就必须弄一个更大的硬盘,然后手动将已有的文件系统移动到新的硬盘上。

​	这时候可以通过将另外一个硬盘上的分区加入已有文件系统,动态地添加存储空间。Linux逻辑卷管理器(logical volume manager,LVM)软件包正好可以用来做这个。它可以让你在无需重建整个文件系统的情况下,轻松地管理磁盘空间。

#### 8.3.1 逻辑卷管理布局

​	逻辑卷管理的核心在于如何处理安装在系统上的硬盘分区。在逻辑卷管理的世界里,硬盘称作物理卷(physical volume,PV)。每个物理卷都会映射到硬盘上特定的物理分区。

​	多个物理卷集中在一起可以形成一个卷组(volume group,VG)。逻辑卷管理系统将卷组视为一个物理硬盘,但事实上卷组可能是由分布在多个物理硬盘上的多个物理分区组成的。卷组提供了一个创建逻辑分区的平台,而这些逻辑分区则包含了文件系统。

​	整个结构中的最后一层是逻辑卷(logical volume,LV)。逻辑卷为Linux提供了创建文件系统的分区环境,作用类似于到目前为止我们一直在探讨的Linux中的物理硬盘分区。Linux系统将逻辑卷视为物理分区。

​	可以使用任意一种标准Linux文件系统来格式化逻辑卷,然后再将它加入Linux虚拟目录中的某个挂载点。

​	图8-1显示了典型Linux逻辑卷管理环境的基本布局。

![图8-1.png]()

​	图8-1中的卷组横跨了三个不同的物理硬盘,覆盖了五个独立的物理分区。在卷组内部有两个独立的逻辑卷。Linux系统将每个逻辑卷视为一个物理分区。每个逻辑卷可以被格式化成ext4文件系统,然后挂载到虚拟目录中某个特定位置。

​	注意,图8-1中,第三个物理硬盘有一个未使用的分区。通过逻辑卷管理,你随后可以轻松地将这个未使用分区分配到已有卷组:要么用它创建一个新的逻辑卷,要么在需要更多空间时用它来扩展已有的逻辑卷。

​	类似地,如果你给系统添加了一块硬盘,逻辑卷管理系统允许你将它添加到已有卷组,为某个已有的卷组创建更多空间,或是创建一个可用来挂载的新逻辑卷。这种扩展文件系统的方法要好用得多!

#### 8.3.2 Linux中的LVM

​	Linux LVM是由Heinz Mauelshagen开发的,于1998年发布到了Linux社区。它允许你在Linux上用简单的命令行命令管理一个完整的逻辑卷管理环境。

​	Linux LVM有两个可用的版本。

- LVM1: 最初的LVM包于1998年发布,只能用于Linux内核2.4版本。它仅提供了基本的逻辑卷管理功能。
- LVM2: LVM的更新版本,可用于Linux内核2.6版本。它在标准的LVM1功能外提供了额外的功能。

​	大部分采用2.6或更高内核版本的现代Linux发行版都提供对LVM2的支持。除了标准的逻辑卷管理功能外,LVM2还提供了另外一些好用的功能。

​	**1. 快照**

​	最初的Linux LVM允许你在逻辑卷在线的状态下将其复制到另一个设备。这个功能叫作快照。在备份由于高可靠性需求而无法锁定的重要数据时,快照功能非常给力。传统的备份方法在将文件复制到备份媒体上时通常要将文件锁定。快照允许你在复制的同时,保证运行关键任务的Web服务器或数据库服务器继续工作。遗憾的是,LVM1只允许你创建只读快照。一旦创建了快照,就不能再写入东西了。

​	LVM2允许你创建在线逻辑卷的可读写快照。有了可读写的快照,就可以删除原先的逻辑卷,然后将快照作为替代挂载上。这个功能对快速故障转移或涉及修改数据的程序试验(如果失败,需要恢复修改过的数据)非常有用。

​	**2. 条带化**

​	LVM2提供的另一个引人注目的功能是条带化(striping)。有了条带化,可跨多个物理硬盘创建逻辑卷。当Linux LVM将文件写入逻辑卷时,文件中的数据块会被分散到多个硬盘上。每个后继数据块会被写到下一个硬盘上。

​	条带化有助于提高硬盘的性能,因为Linux可以将一个文件的多个数据块同时写入多个硬盘,而无需等待单个硬盘移动读写磁头到多个不同位置。这个改进同样适用于读取顺序访问的文件,因为LVM可同时从多个硬盘读取数据。

​	**3. 镜像**

​	通过LVM安装文件系统并不意味着文件系统就不会再出问题。和物理分区一样,LVM逻辑卷也容易受到断电和磁盘故障的影响。一旦文件系统损坏,就有可能再也无法恢复。

​	LVM快照功能提供了一些安慰,你可以随时创建逻辑卷的备份副本,但对有些环境来说可能还不够。对于涉及大量数据变动的系统,比如数据库服务器,自上次快照之后可能要存储成百上千条记录。

​	这个问题的一个解决办法就是LVM镜像。镜像是一个实时更新的逻辑卷的完整副本。当你创建镜像逻辑卷时,LVM会将原始逻辑卷同步到镜像副本中。根据原始逻辑卷的大小,这可能需要一些时间才能完成。

​	一旦原始同步完成,LVM会为文件系统的每次写操作执行两次写入——一次写入到主逻辑卷,一次写入到镜像副本。可以想到,这个过程会降低系统的写入性能。就算原始逻辑卷因为某些原因损坏了,你手头也已经有了一个完整的最新副本!

#### 8.3.3 使用Linux LVM

​	现在你已经知道Linux LVM可以做什么了,本节将讨论如何创建LVM来帮助组织系统上的硬盘空间。 Linux LVM包只提供了命令行程序来创建和管理逻辑卷管理系统中所有组件。有些Linux发行版则包含了命令行命令对应的图形化前端,但为了完全控制你的LVM环境,最好习惯直接使用这些命令。

​	**1. 定义物理卷**

​	创建过程的第一步就是将硬盘上的物理分区转换成Linux LVM使用的物理卷区段。我们的朋友 fdisk 命令可以帮忙。在创建了基本的Linux分区之后,你需要通过 t 命令改变分区类型。

```shell
[...]
Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 8e
Changed system type of partition 1 to 8e (Linux LVM)

Command (m for help): p

Disk /dev/sdb: 5368 MB, 5368709120 bytes
255 heads, 63 sectors/track, 652 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xa8661341

Device Boot		Start	End		Blocks		Id		System
/dev/sdb1		1		262		2104483+	8e		Linux LVM

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
$
```

​	分区类型 8e 表示这个分区将会被用作Linux LVM系统的一部分,而不是一个直接的文件系统(就像你在前面看到的 83 类型的分区)。

***

**说明**	如果下一步中的 pvcreate 命令不能正常工作,很可能是因为LVM2软件包没有默认安装。可以使用软件包名lvm2,按照第9章中介绍的软件安装方法安装这个包。

****

​	下一步是用分区来创建实际的物理卷。这可以通过 pvcreate 命令来完成。 pvcreate 定义了用于物理卷的物理分区。它只是简单地将分区标记成Linux LVM系统中的分区而已。

```shell
$ sudo pvcreate /dev/sdb1
dev_is_mpath: failed to get device for 8:17
Physical volume "/dev/sdb1" successfully created
$
```

***

**说明**	别被吓人的消息 dev_is_mpath: failed to get device for 8:17 或类似的消息唬住了。只要看到了 successfully created 就没问题。 pvcreate 命令会检查分区是否为多路(multi-path,mpath)设备。如果不是的话,就会发出上面那段消息。

***

​	如果你想查看创建进度的话,可以使用 pvdisplay 命令来显示已创建的物理卷列表。

```shell
$ sudo pvdisplay /dev/sdb1
"/dev/sdb1" is a new physical volume of "2.01 GiB"
--- NEW Physical volume ---
PV Name			/dev/sdb1
VG Name
PV Size			2.01 GiB
Allocatable		NO
PE Size			0
Total PE		0
Free PE			0
Allocated PE	0
PV UUID			0FIuq2-LBod-IOWt-8VeN-tglm-Q2ik-rGU2w7
$
```

​	pvdisplay 命令显示出/dev/sdb1现在已经被标记为物理卷。注意,输出中的 VG Name 内容为空,因为物理卷还不属于某个卷组。

​	**2. 创建卷组**

​	下一步是从物理卷中创建一个或多个卷组。究竟要为系统创建多少卷组并没有既定的规则,你可以将所有的可用物理卷加到一个卷组,也可以结合不同的物理卷创建多个卷组。

​	要从命令行创建卷组,需要使用 vgcreate 命令。 vgcreate 命令需要一些命令行参数来定义卷组名以及你用来创建卷组的物理卷名。

```shell
$ sudo vgcreate Vol1 /dev/sdb1
Volume group "Vol1" successfully created
$
```

​	输出结果平淡无奇。如果你想看看新创建的卷组的细节,可用 vgdisplay 命令。

​	这个例子使用 /dev/sdb1 分区上创建的物理卷,创建了一个名为 Vol1 的卷组。

​	创建一个或多个卷组后,就可以创建逻辑卷了。

​	**3. 创建逻辑卷**

​	Linux系统使用逻辑卷来模拟物理分区,并在其中保存文件系统。Linux系统会像处理物理分区一样处理逻辑卷,允许你定义逻辑卷中的文件系统,然后将文件系统挂载到虚拟目录上。

​	要创建逻辑卷,使用 lvcreate 命令。虽然你通常不需要在其他Linux LVM命令中使用命令行选项,但 lvcreate 命令要求至少输入一些选项。表8-5显示了可用的命令行选项。

![表8-5]()

​	虽然命令行选项看起来可能有点吓人,但大多数情况下你用到的只是少数几个选项。

```shell
$ sudo lvcreate -l 100%FREE -n lvtest Vol1
Logical volume "lvtest" created
$
```

​	如果想查看你创建的逻辑卷的详细情况,可用 lvdisplay 命令。

```shell
$ sudo lvdisplay Vol1
--- Logical volume ---
LV Path						/dev/Vol1/lvtest
LV Name						lvtest
VG Name						Vol1
LV UUID						4W2369-pLXy-jWmb-lIFN-SMNX-xZnN-3KN208
LV Write Access				read/write
LV Creation host, time		... -0400
LV Status					available
# open						0
LV Size						2.00 GiB
Current LE					513
Segments					1
Allocation					inherit
Read ahead sectors			auto
- currently set to			256
Block device				253:2
$
```

​	现在可以看到你刚刚创建的逻辑卷了!注意,卷组名(Vol1)用来标识创建新逻辑卷时要使用的卷组。

​	-l 选项定义了要为逻辑卷指定多少可用的卷组空间。注意,你可以按照卷组空闲空间的百分比来指定这个值。本例中为新逻辑卷使用了所有的空闲空间。

​	你可以用 -l 选项来按可用空间的百分比来指定这个大小,或者用 -L 选项以字节、千字节(KB)、兆字节(MB)或吉字节(GB)为单位来指定实际的大小。 -n 选项允许你为逻辑卷指定一个名称(在本例中称作lvtest)。

​	**4. 创建文件系统**

​	运行完 lvcreate 命令之后,逻辑卷就已经产生了,但它还没有文件系统。你必须使用相应的命令行程序来创建所需要的文件系统。

```shell
$ sudo mkfs.ext4 /dev/Vol1/lvtest
mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
131376 inodes, 525312 blocks
26265 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=541065216
17 block groups
32768 blocks per group, 32768 fragments per group
7728 inodes per group
Superblock backups stored on blocks:
		32768, 98304, 163840, 229376, 294912
		
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 28 mounts or
180 days, whichever comes first.Use tune2fs -c or -i to override.
$
```

​	在创建了新的文件系统之后,可以用标准Linux mount 命令将这个卷挂载到虚拟目录中,就跟它是物理分区一样。唯一的不同是你需要用特殊的路径来标识逻辑卷。

```shell
$ sudo mount /dev/Vol1/lvtest /mnt/my_partition
$
$ mount
/dev/mapper/vg_server01-lv_root on / type ext4 (rw)
[...]
/dev/mapper/Vol1-lvtest on /mnt/my_partition type ext4 (rw)
$
$ cd /mnt/my_partition
$
$ ls -al
total 24
drwxr-xr-x. 3 root root 4096 Jun 12 10:22 .
drwxr-xr-x. 3 root root 4096 Jun 11 09:58 ..
drwx------. 2 root root 16384 Jun 12 10:22 lost+found
$
```

​	注意, mkfs.ext4 和 mount 命令中用到的路径都有点奇怪。路径中使用了卷组名和逻辑卷名,而不是物理分区路径。文件系统被挂载之后,就可以访问虚拟目录中的这块新区域了。

​	**5. 修改LVM**

​	Linux LVM的好处在于能够动态修改文件系统,因此最好有工具能够让你实现这些操作。在Linux有一些工具允许你修改现有的逻辑卷管理配置。

​	如果你无法通过一个很炫的图形化界面来管理你的Linux LVM环境,也不是什么都干不了。在本章中你已经看到了一些Linux LVM命令行程序的实际用法。还有一些其他的命令可以用来管理LVM的设置。表8-6列出了在Linux LVM包中的常见命令。

![表8-6]()

​	通过使用这些命令行程序,就能完全控制你的Linux LVM环境。

***

**窍门**	在手动增加或减小逻辑卷的大小时,要特别小心。逻辑卷中的文件系统需要手动修整来处理大小上的改变。大多数文件系统都包含了能够重新格式化文件系统的命令行程序,比如用于ext2、ext3和ext4文件系统的resize2fs程序。

***

