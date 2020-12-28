## 第七章 理解Linux文件权限

### 7.1 Linux 的安全性

​	Linux安全系统的核心是用户账户。每个能进入Linux系统的用户都会被分配唯一的用户账户。用户对系统中各种对象的访问权限取决于他们登录系统时用的账户。

​	用户权限是通过创建用户时分配的用户ID(User ID,通常缩写为`UID`)来跟踪的。`UID`是数值,每个用户都有唯一的`UID`,但在登录系统时用的不是`UID`,而是登录名。登录名是用户用来登录系统的最长八字符的字符串(字符可以是数字或字母),同时会关联一个对应的密码。

​	Linux系统使用特定的文件和工具来跟踪和管理系统上的用户账户。在我们讨论文件权限之前,先来看一下Linux是怎样处理用户账户的。本节会介绍管理用户账户需要的文件和工具,这样在处理文件权限问题时,你就知道如何使用它们了。

#### 7.1.1 /etc/passwd文件

​	Linux系统使用一个专门的文件来将用户的登录名匹配到对应的`UID`值。这个文件就是`/etc/passwd`文件,它包含了一些与用户有关的信息。

​	root用户账户是Linux系统的管理员,固定分配给它的`UID`是 0 。Linux系统会为各种各样的功能创建不同的用户账户,而这些账户并不是真的用户。这些账户叫作系统账户,是系统上运行的各种服务进程访问资源用的特殊账户。所有运行在后台的服务都需要用一个系统用户账户登录到Linux系统上。

​	在安全成为一个大问题之前,这些服务经常会用root账户登录。遗憾的是,如果有非授权的用户攻陷了这些服务中的一个,他立刻就能作为root用户进入系统。为了防止发生这种情况,现在运行在Linux服务器后台的几乎所有的服务都是用自己的账户登录。这样的话,即使有人攻入了某个服务,也无法访问整个系统。

​	Linux为系统账户预留了500以下的`UID`值。有些服务甚至要用特定的`UID`才能正常工作。为普通用户创建账户时,大多数Linux系统会从500开始,将第一个可用`UID`分配给这个账户(并非所有的Linux发行版都是这样)。

​	你可能已经注意到`/etc/passwd`文件中还有很多用户登录名和`UID`之外的信息。 `/etc/passwd`文件的字段包含了如下信息：

- 登录用户名
- 用户密码
- 用户账户的`UID`（数字形式）
- 用户账户的组ID（`GID`）
- 用户账户的文本描述（称为备注字段）
- 用户HOME目录的位置
- 用户的默认shell

​	`/etc/passwd`文件中的密码字段都被设置成了x,这并不是说所有的用户账户都用相同的密码。在 早 期 的 Linux 上 , /etc/passwd 文 件 里 有 加 密 后 的 用 户 密 码 。 但 鉴 于 很 多 程 序 都 需 要 访 问`/etc/passwd`文件获取用户信息,这就成了一个安全隐患。随着用来破解加密密码的工具的不断演进,用心不良的人开始忙于破解存储在`/etc/passwd`文件中的密码。Linux开发人员需要重新考虑这个策略。

​	现在,绝大多数Linux系统都将用户密码保存在另一个单独的文件中(叫作shadow文件,位置在/etc/shadow)。只有特定的程序(比如登录程序)才能访问这个文件。

​	`/etc/passwd`是一个标准的文本文件。你可以用任何文本编辑器在/etc/password文件里直接手动进行用户管理(比如添加、修改或删除用户账户)。但这样做极其危险。如果`/etc/passwd`文件出现损坏,系统就无法读取它的内容了,这样会导致用户无法正常登录(即便是root用户)。用标准的Linux用户管理工具去执行这些用户管理功能就会安全许多。

#### 7.1.2 /etc/shadow 文件

​	/etc/shadow文件对Linux系统密码管理提供了更多的控制。只有root用户才能访问/etc/shadow文件,这让它比起`/etc/passwd`安全许多。

​	/etc/shadow文件为系统上的每个用户账户都保存了一条记录。记录就像下面这样：

```shell
rich:$1$.FfcK0ns$f1UgiyHQ25wrB/hykCn020:11627:0:99999:7:::
```

​	在/etc/shadow文件的每条记录中都有9个字段:

- 与`/etc/passwd`文件中的登录名字段对应的登录名
- 加密后的密码
- 自上次修改密码后过去的天数密码(自1970年1月1日开始计算)
- 多少天后才能更改密码
- 多少天后必须更改密码
- 密码过期前提前多少天提醒用户更改密码
- 密码过期后多少天禁用用户账户
- 用户账户被禁用的日期(用自1970年1月1日到当天的天数表示)
- 预留字段给将来使用

​	使用shadow密码系统后,Linux系统可以更好地控制用户密码。它可以控制用户多久更改一次密码,以及什么时候禁用该用户账户,如果密码未更新的话。

#### 7.1.3 添加新用户

​	用来向Linux系统添加新用户的主要工具是 `useradd `。这个命令简单快捷,可以一次性创建新用户账户及设置用户HOME目录结构。 `useradd` 命令使用系统的默认值以及命令行参数来设置用户账户。系统默认值被设置在`/etc/default/useradd`文件中。可以使用加入了 -D 选项的 `useradd`命令查看所用Linux系统中的这些默认值。

```shell
[kato@aragne ~]$ /usr/bin/useradd -D
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=no
```

***

**说明**	一些Linux发行版会把Linux用户和组工具放在`/usr/sbin`目录下,这个目录可能不在 PATH 环境变量里。如果你的Linux系统是这样的话,可以将这个目录添加进 PATH 环境变量,或者用绝对文件路径名来使用这些工具。

***

​	在创建新用户时,如果你不在命令行中指定具体的值, `useradd `命令就会使用 -D 选项所显示的那些默认值。这个例子列出的默认值如下:

- 新用户会被添加到`GID`为 100 的公共组;
- 新用户的HOME目录将会位于`/home/loginname`;
- 新用户账户密码在过期后不会被禁用;
- 新用户账户未被设置过期日期;
- 新用户账户将bash shell作为默认shell;
- 系统会将`/etc/skel`目录下的内容复制到用户的HOME目录下;
- 系统为该用户账户在mail目录下创建一个用于接收邮件的文件。

​	倒数第二个值很有意思。 `useradd` 命令允许管理员创建一份默认的HOME目录配置,然后把它作为创建新用户HOME目录的模板。这样就能自动在每个新用户的HOME目录里放置默认的系统文件。在`Manjaro Linux`系统上，`/etc/skel`目录有下列文件：

```shell
[kato@aragne ~]$ ls -la /etc/skel/
总用量 60
drwxr-xr-x   4 root root  4096 11月 22 10:19 .
drwxr-xr-x 106 root root 12288 12月 27 17:12 ..
-rw-r--r--   1 root root    21  7月 11 00:57 .bash_logout
-rw-r--r--   1 root root    57  7月 11 00:57 .bash_profile
-rw-r--r--   1 root root  3838  7月 11 00:57 .bashrc
drwxr-xr-x   8 root root  4096 11月 22 10:19 .config
-rw-r--r--   1 root root  4855 10月 30  2017 .dir_colors
drwxr-xr-x   3 root root  4096 10月  1 20:59 .local
-rwxr-xr-x   1 root root   100 10月 30  2017 .Xclients
-rwxr-xr-x   1 root root  1521 11月 22 04:09 .xinitrc
-rw-r--r--   1 root root   256  9月 20 02:58 .zshrc
```

​	根据第6章的内容,你应该能知道这些文件是做什么的。它们是bash shell环境的标准启动文件。系统会自动将这些默认文件复制到你创建的每个用户的HOME目录。

​	可以用默认系统参数创建一个新用户账户,然后检查一下新用户的HOME目录。

```shell
[kato@aragne ~]$ sudo useradd -m test
[sudo] kato 的密码：
[kato@aragne ~]$ sudo ls -la /home/test/
总用量 48
drwx------ 4 test test 4096 12月 27 17:59 .
drwxr-xr-x 4 root root 4096 12月 27 17:59 ..
-rw-r--r-- 1 test test   21  7月 11 00:57 .bash_logout
-rw-r--r-- 1 test test   57  7月 11 00:57 .bash_profile
-rw-r--r-- 1 test test 3838  7月 11 00:57 .bashrc
drwxr-xr-x 8 test test 4096 11月 22 10:19 .config
-rw-r--r-- 1 test test 4855 10月 30  2017 .dir_colors
drwxr-xr-x 3 test test 4096 10月  1 20:59 .local
-rwxr-xr-x 1 test test  100 10月 30  2017 .Xclients
-rwxr-xr-x 1 test test 1521 11月 22 04:09 .xinitrc
-rw-r--r-- 1 test test  256  9月 20 02:58 .zshrc
```

​	默认情况下, `useradd` 命令不会创建HOME目录,但是 -m 命令行选项会使其创建HOME目录。你能在此例中看到, `useradd` 命令创建了新HOME目录,并将`/etc/skel`目录中的文件复制了过来。

​	要想在创建用户时改变默认值或默认行为,可以使用命令行参数。具体查看man手册。

​	你会发现,在创建新用户账户时使用命令行参数可以更改系统指定的默认值。但如果总需要修改某个值的话,最好还是修改一下系统的默认值。

​	可以在 -D 选项后跟上一个指定的值来修改系统默认的新用户设置。

#### 7.1.4 删除用户

​	如果你想从系统中删除用户, `userdel `可以满足这个需求。默认情况下, `userdel `命令会只删除`/etc/passwd`文件中的用户信息,而不会删除系统中属于该账户的任何文件。

​	如果加上 -r 参数, `userdel `会删除用户的HOME目录以及邮件目录。然而,系统上仍可能存有已删除用户的其他文件。这在有些环境中会造成问题。

​	加了 -r 参数后,用户先前的那个/home/test目录已经不存在了。

#### 7.1.5 修改用户

​	Linux提供了一些不同的工具来修改已有用户账户的信息。表7-3列出了这些工具。

| 命令command |                      描述description                       |
| :---------: | :--------------------------------------------------------: |
|  `usermod`  | 修改用户账户的字段，还可以指定主要组成以及附加组的所属关系 |
|  `passwd`   |                     修改已有用户的密码                     |
| `chpasswd`  |            从文件中读取登录名密码对，并更新密码            |
|   `chage`   |                     修改密码的过期密码                     |
|   `chfn`    |                   修改用户账户的备注信息                   |
|   `chsh`    |                修改用户账户的默认登录shell                 |

​	每种工具都提供了特定的功能来修改用户账户信息。下面的几节将具体介绍这些工具。

​	**1. `usermod`**

​	`usermod` 命令是用户账户修改工具中最强大的一个。它能用来修改`/etc/passwd`文件中的大部分字段,只需用与想修改的字段对应的命令行参数就可以了。参数大部分跟 `useradd` 命令的参数一样(比如, -c 修改备注字段, -e 修改过期日期, -g 修改默认的登录组)。除此之外,还有另外一些可能派上用场的选项。

- -l 修改用户账户的登录名。
- -L 锁定账户,使用户无法登录。
- -p 修改账户的密码。
- -U 解除锁定,使用户能够登录。

​	-L 选项尤其实用。它可以将账户锁定,使用户无法登录,同时无需删除账户和用户的数据。要让账户恢复正常,只要用 -U 选项就行了。

​	**2. `passwd` 和 `chpasswd`**

​	改变用户密码的一个简便方法就是用 `passwd` 命令。

```shell
# passwd test
Changing password for user test.
New UNIX password:
Retype new UNIX password:
passwd: all authentication tokens updated successfully.
#
```

​	如果只用 `passwd` 命令,它会改你自己的密码。系统上的任何用户都能改自己的密码,但只有root用户才有权限改别人的密码。

​	-e 选项能强制用户下次登录时修改密码。你可以先给用户设置一个简单的密码,之后再强制在下次登录时改成他们能记住的更复杂的密码。

​	如果需要为系统中的大量用户修改密码, `chpasswd `命令可以事半功倍。 `chpasswd` 命令能从标准输入自动读取登录名和密码对(由冒号分割)列表,给密码加密,然后为用户账户设置。你也可以用重定向命令来将含有 `userid:passwd `对的文件重定向给该命令。

```shell
# chpasswd < users.txt
#
```

​	**3. `chsh、chfn、chage`**

​	`chsh` 、 `chfn` 和 `chage` 工具专门用来修改特定的账户信息。 `chsh` 命令用来快速修改默认的用户登录shell。使用时必须用shell的全路径名作为参数,不能只用shell名。

```shell
# chsh -s /bin/csh test
Changing shell for test.
Shell changed.
#
```

​	`chfn` 命令提供了在`/etc/passwd`文件的备注字段中存储信息的标准方法。 `chfn `命令会将用于Unix的 finger 命令的信息存进备注字段,而不是简单地存入一些随机文本(比如名字或昵称之类的)，或是将备注字段留空。 finger 命令可以非常方便地查看Linux系统上的用户信息。

```shell
# finger rich
Login: rich				Name: Rich Blum
Directory: /home/rich	Shell: /bin/bash
On since Thu Sep 20 18:03 (EDT) on pts/0 from 192.168.1.2
No mail.
No Plan.
#
```

​	出于安全性考虑,很多Linux系统管理员会在系统上禁用 finger 命令,不少Linux发行版甚至都没有默认安装该命令。

​	如果在使用 `chfn` 命令时没有参数,它会向你询问要将哪些适合的内容加进备注字段。

```shell
# chfn test
Changing finger information for test.
Name []: Ima Test
Office []: Director of Technology
Office Phone []: (123)555-1234
Home Phone []: (123)555-9876
Finger information changed.

# finger test
Login: test							Name: Ima Test
Directory: /home/test				Shell: /bin/csh
Office: Director of Technology		Office Phone: (123)555-1234
Home Phone: (123)555-9876
Never logged in.
No mail.
No Plan.
#
# grep test /etc/passwd
test:x:504:504:Ima Test,Director of Technology,(123)555-
1234,(123)555-9876:/home/test:/bin/csh
#
```

​	所有的指纹信息现在都存在`/etc/passwd`文件中了。

### 7.2 使用Linux组

​	用户账户在控制单个用户安全性方面很好用,但涉及在共享资源的一组用户时就捉襟见肘了。为了解决这个问题,Linux系统采用了另外一个安全概念——组(group)。

​	组权限允许多个用户对系统中的对象(比如文件、目录或设备等)共享一组共用的权限。(更多内容会在7.3节中细述。)

​	Linux发行版在处理默认组的成员关系时略有差异。有些Linux发行版会创建一个组,把所有用户都当作这个组的成员。遇到这种情况要特别小心,因为文件很有可能对其他用户也是可读的。有些发行版会为每个用户创建单独的一个组,这样可以更安全一些。

​	每个组都有唯一的`GID`——跟`UID`类似,在系统上这是个唯一的数值。除了`GID`,每个组还有唯一的组名。Linux系统上有一些组工具可以创建和管理你自己的组。本节将细述组信息是如何保存的,以及如何用组工具创建新组和修改已有的组。

#### 7.2.1 /etc/group文件

​	与用户账户类似,组信息也保存在系统的一个文件中。/etc/group文件包含系统上用到的每个组的信息。下面是一些来自Linux系统上/etc/group文件中的典型例子。

```shell
root:x:0:root
bin:x:1:root,bin,daemon
daemon:x:2:root,bin,daemon
sys:x:3:root,bin,adm
adm:x:4:root,adm,daemon
rich:x:500:
mama:x:501:
katie:x:502:
jessica:x:503:
mysql:x:27:
test:x:504:
```

​	和`UID`一样,`GID`在分配时也采用了特定的格式。系统账户用的组通常会分配低于500的`GID`值,而用户组的`GID`则会从500开始分配。/etc/group文件有4个字段：

- 组名
- 组密码
- `GID`
- 属于该组的用户列表

​	组密码允许非组内成员通过它临时成为该组成员。这个功能并不很普遍,但确实存在。

​	千万不能通过直接修改/etc/group文件来添加用户到一个组,要用 `usermod` 命令(在7.1节中介绍过)。在添加用户到不同的组之前,首先得创建组。

​	用户账户列表某种意义上有些误导人。你会发现,在列表中,有些组并没有列出用户。这并不是说这些组没有成员。当一个用户在`/etc/passwd`文件中指定某个组作为默认组时,用户账户不会作为该组成员再出现在/etc/group文件中。

#### 7.2.2 创建新组

​	`groupadd` 命令可在系统上创建新组。

```shell
# /usr/sbin/groupadd shared
# tail /etc/group
haldaemon:x:68:
xfs:x:43:
gdm:x:42:
rich:x:500:
mama:x:501:
katie:x:502:
jessica:x:503:
mysql:x:27:
test:x:504:
shared:x:505:
#
```

​	在创建新组时,默认没有用户被分配到该组。 `groupadd` 命令没有提供将用户添加到组中的选项,但可以用 `usermod` 命令来弥补这一点。

```shell
# /usr/sbin/usermod -G shared rich
# /usr/sbin/usermod -G shared test
# tail /etc/group
haldaemon:x:68:
xfs:x:43:
gdm:x:42:
rich:x:500:
mama:x:501:
katie:x:502:
jessica:x:503:
mysql:x:27:
test:x:504:
shared:x:505:rich, test
#
```

​	shared组现在有两个成员: test 和 rich 。 `usermd` 命令的 -G 选项会把这个新组添加到该用户账户的组列表里。

​	如果更改了已登录系统账户所属的用户组,该用户必须登出系统后再登录,组关系的更改才能生效。

​	为用户账户分配组时要格外小心。如果加了 -g 选项,指定的组名会替换掉该账户的默认组。 -G 选项则将该组添加到用户的属组的列表里,不会影响默认组。

#### 7.2.3 修改组

​	在/etc/group文件中可以看到,需要修改的组信息并不多。 `groupmod` 命令可以修改已有组的`GID`(加 -g 选项)或组名(加 -n 选项)。

```shell
# /usr/sbin/groupmod -n sharing shared
# tail /etc/group
haldaemon:x:68:
xfs:x:43:
gdm:x:42:
rich:x:500:
mama:x:501:
katie:x:502:
jessica:x:503:
mysql:x:27:
test:x:504:
sharing:x:505:test,rich
#
```

​	修改组名时,`GID`和组成员不会变,只有组名改变。由于所有的安全权限都是基于`GID`的,你可以随意改变组名而不会影响文件的安全性。

### 7.3 理解文件权限

​	现在你已经了解了用户和组,是时候解读 ls 命令输出时所出现的谜一般的文件权限了。本节将会介绍如何对权限进行分析以及它们的来历。

#### 7.3.1 使用文件权限符

​	如果你还记得第3章,那应该知道 ls 命令可以用来查看Linux系统上的文件、目录和设备的权限。

```shell
[kato@aragne ~]$ ls -lF
总用量 60
drwxr-xr-x  4 kato kato 4096 12月 24 13:29 Book/
drwxr-xr-x  2 kato kato 4096 11月 20 11:43 Desktop/
drwxr-xr-x  5 kato kato 4096 12月 22 14:54 Documents/
drwxr-xr-x  5 kato kato 4096 12月 27 00:11 Downloads/
drwxr-xr-x  3 kato kato 4096 11月 27 21:52 Git/
drwxr-xr-x  8 kato kato 4096 12月 12 01:15 Gitclone/
drwxr-xr-x  3 kato kato 4096 11月 21 15:58 Music/
drwxr-xr-x  4 kato kato 4096 11月 24 15:28 Net/
drwxr-xr-x  4 kato kato 4096 12月 10 00:02 Pictures/
drwxr-xr-x  2 kato kato 4096 11月 20 11:43 Public/
drwxr-xr-x  6 kato kato 4096 12月  8 00:37 Qt/
drwxr-xr-x 11 kato kato 4096 12月 10 21:42 QtProjects/
drwxr-xr-x  2 kato kato 4096 11月 20 11:45 Templates/
drwxr-xr-x  2 kato kato 4096 11月 20 11:43 Videos/
drwxr-xr-x  8 kato kato 4096 12月 25 23:33 Workspace/
```

​	输出结果的第一个字段就是描述文件和目录权限的编码。这个字段的第一个字符代表了对象的类型：

- -代表文件
- d 代表目录
- l 代表链接
- c 代表字符型设备
- b 代表块设备
- n代表网络设备

​	之后有3组三字符的编码。每一组定义了3种访问权限:

- r 代表对象是可读的
- w 代表对象是可写的
- x 代表对象是可执行的

​	若没有某种权限,在该权限位会出现单破折线。这3组权限分别对应对象的3个安全级别:

- 对象的属主
- 对象的属组
- 系统其他用户

​	讨论这个问题的最简单的办法就是找个例子,然后逐个分析文件权限。

```shell
-rwxrwxr-x 1 rich rich 4882 2010-09-18 13:58 myprog
```

​	文件`myprog`有下面3组权限。

- `rwx` :文件的属主(设为登录名rich)
- `rwx` :文件的属组(设为组名rich)
- `r-x` :系统上其他人。

​	这些权限说明登录名为rich的用户可以读取、写入以及执行这个文件(可以看作有全部权限)。类似地,rich组的成员也可以读取、写入和执行这个文件。然而不属于rich组的其他用户只能读取和执行这个文件: w 被单破折线取代了,说明这个安全级别没有写入权限。

#### 7.3.2 默认文件权限

​	你可能会问这些文件权限从何而来,答案是 `umask` 。 `umask` 命令用来设置所创建文件和目录的默认权限。

```shell
$ touch newfile
$ ls -al newfile
-rw-r--r--	1 rich	rich	0 Sep 20 19:16 newfile
$
```

​	touch 命令用分配给我的用户账户的默认权限创建了这个文件。 `umask` 命令可以显示和设置这个默认权限。

```shell
$ umask
0022
$
```

​	遗憾的是, `umask` 命令设置没那么简单明了,想弄明白其工作原理就更混乱了。第一位代表了一项特别的安全特性,叫作粘着位(sticky bit)。这部分内容会在7.5节详述。

​	后面的3位表示文件或目录对应的`umask` 八进制值。要理解 `umask` 是怎么工作的,得先理解八进制模式的安全性设置。

​	八进制模式的安全性设置先获取这3个 `rwx` 权限的值,然后将其转换成3位二进制值,用一个八进制值来表示。在这个二进制表示中,每个位置代表一个二进制位。因此,如果读权限是唯一置位的权限,权限值就是 r-- ,转换成二进制值就是 100 ,代表的八进制值是 4 。表7-5列出了可能会遇到的组合。

| 权限  | 二进制值 | 八进制值 |        描述        |
| :---: | :------: | :------: | :----------------: |
| `---` |   000    |    0     |    没有任何权限    |
| `--x` |   001    |    1     |    只有执行权限    |
| `-w-` |   010    |    2     |    只有写入权限    |
| `-wx` |   011    |    3     |  有写入和执行权限  |
| `r--` |   100    |    4     |    只有读取权限    |
| `r-x` |   101    |    5     | 只有读取和执行权限 |
| `rw-` |   110    |    6     |  有读取和写入权限  |
| `rwx` |   111    |    7     |     有全部权限     |

​	八进制模式先取得权限的八进制值,然后再把这三组安全级别(属主、属组和其他用户)的八进制值顺序列出。因此,八进制模式的值 664 代表属主和属组成员都有读取和写入的权限,而其他用户都只有读取权限。

​	了解八进制模式权限是怎么工作的之后, `umask` 值反而更叫人困惑了。我的Linux系统上默认的八进制的 `umask` 值是 0022 ,而我所创建的文件的八进制权限却是 644 ,这是如何得来的呢?

​	`umask` 值只是个掩码。它会屏蔽掉不想授予该安全级别的权限。接下来我们还得再多进行一些八进制运算才能搞明白来龙去脉。

​	要把 `umask` 值从对象的全权限值中减掉。对文件来说,全权限的值是 666 (所有用户都有读和写的权限)；而对目录来说,则是 777 (所有用户都有读、写、执行权限)。

​	所以在上例中,文件一开始的权限是 666 ,减去 `umask` 值 022 之后,剩下的文件权限就成了 644 。

​	在大多数Linux发行版中, `umask` 值通常会设置在/etc/profile启动文件中(参见第6章),不过有一些是设置在`/etc/login.defs`文件中的(如`Ubuntu`)。可以用 `umask `命令为默认 `umask` 设置指定一个新值。

```shell
$ umask 026
$ touch newfile2
$ ls -l newfile2
-rw-r-----	1 rich	rich	0 Sep 20 19:46 newfile2
```

​	在把 `umask` 值设成 026 后,默认的文件权限变成了 640 ,因此新文件现在对组成员来说是只读的,而系统里的其他成员则没有任何权限。

​	`umask` 值同样会作用在创建目录上。

```shell
$ mkdir newdir
$ ls -l
drwxr-x--x	2 rich	rich	4096 Sep 20 20:11 newdir/
$
```

​	由于目录的默认权限是 777 ,` umask` 作用后生成的目录权限不同于生成的文件权限。 `umask`值 026 会从 777 中减去,留下来 751 作为目录权限设置。

### 7.4 改变安全性设置

​	如果你已经创建了一个目录或文件,需要改变它的安全性设置,在Linux系统上有一些工具能够完成这项任务。本节将告诉你如何更改文件和目录的已有权限、默认文件属主以及默认属组。

#### 7.4.1 改变权限

​	`chmod` 命令用来改变文件和目录的安全性设置。该命令的格式：

​	`chmod options mode file`

​	mode 参数可以使用八进制模式或符号模式进行安全性设置。八进制模式设置非常直观,直接用期望赋予文件的标准3位八进制权限码即可。

```shell
$ chmod 760 newfile
$ ls -l newfile
-rwxrw----	1 rich	rich	0 Sep 20 19:16 newfile
```

​	八进制文件权限会自动应用到指定的文件上。符号模式的权限就没这么简单了。

​	与通常用到的3组三字符权限字符不同, `chmod` 命令采用了另一种方法。下面是在符号模式下指定权限的格式。

​	`[ugoa...][[+-=][rwxXstugo...]`

​	非常有意义,不是吗?第一组字符定义了权限作用的对象:

- u代表用户
- g代表组
- o代表其他
- a代表上述所有

​	下一步,后面跟着的符号表示你是想在现有权限基础上增加权限(+),还是在现有权限基础上移除权限(-),或是将权限设置成后面的值(=)。

​	最后,第三个符号代表作用到设置上的权限。你会发现,这个值要比通常的 `rwx` 多。额外的设置有以下几项。

- X :如果对象是目录或者它已有执行权限,赋予执行权限。
- s :运行时重新设置`UID`或`GID`。
- t :保留文件或目录。
- u :将权限设置为跟属主一样。
- g :将权限设置为跟属组一样。
- o :将权限设置为跟其他用户一样。

​	像这样使用这些权限。

```shell
$ chmod o+r newfile
$ ls -lF newfile
-rwxrw-r--	1 rich	rich	0 Sep 20 19:16 newfile*
```

​	不管其他用户在这一安全级别之前都有什么权限, o+r 都给这一级别添加读取权限。

```shell
$ chmod u-x newfile
$ ls -lF newfile
-rw-rw-r--	1 rich	rich	0 Sep 20 19:16 newfile
```

​	u-x 移除了属主已有的执行权限。注意 ls 命令的 -F 选项,它能够在具有执行权限的文件名后加一个星号。

​	options 为 `chmod` 命令提供了另外一些功能。 -R 选项可以让权限的改变递归地作用到文件和子目录。你可以使用通配符指定多个文件,然后利用一条命令将权限更改应用到这些文件上。

#### 7.4.2	改变所属关系

​	有时你需要改变文件的属主,比如有人离职或开发人员创建了一个在产品环境中需要归属在系统账户下的应用。Linux提供了两个命令来实现这个功能: `chown` 命令用来改变文件的属主,`chgrp` 命令用来改变文件的默认属组。

​	`chown` 命令的格式如下。

​	`chown options owner[.group] file`

​	可用登录名或`UID`来指定文件的新属主。

```shell
# chown dan newfile
# ls -l newfile
-rw-rw-r--	1 dan	rich	0 Sep 20 19:16 newfile
```

​	非常简单。 `chown` 命令也支持同时改变文件的属主和属组。

```shell
# chown dan.shared newfile
# ls -l newfile
-rw-rw-r--	1 dan	shared	0 Sep 20 19:16 newfile
```

​	如果你不嫌麻烦,可以只改变一个目录的默认属组。

```shell
# chown .rich newfile
# ls -l newfile
-rw-rw-r--	1 dan	rich	0 Sep 20 19:16 newfile
```

​		最后,如果你的Linux系统采用和用户登录名匹配的组名,可以只用一个条目就改变二者。

```shel
# chown test. newfile
# ls -l newfile
-rw-rw-r--	1 test	test	0 Sep 20 19:16 newfile
```

​	`chown` 命令采用一些不同的选项参数。 -R 选项配合通配符可以递归地改变子目录和文件的所属关系。 -h 选项可以改变该文件的所有符号链接文件的所属关系。

​	只有root用户能够改变文件的属主。任何属主都可以改变文件的属组,但前提是属主必须是原属组和目标属组的成员。

​	`chgrp` 命令可以更改文件或目录的默认属组。

```shell
$ chgrp shared newfile
$ ls -l newfile
-rw-rw-r--	1 rich	shared	0 Sep 20 19:16 newfile
```

​	用户账户必须是这个文件的属主,除了能够更换属组之外,还得是新组的成员。现在shared组的任意一个成员都可以写这个文件了。这是Linux系统共享文件的一个途径。然而,在系统中给一组用户共享文件也会变得很复杂。下一节会介绍如何实现。

### 7.5 共享文件

​	能你已经猜到了,Linux系统上共享文件的方法是创建组。但在一个完整的共享文件的环境中,事情会复杂得多。

​	在7.3节中你已经看到,创建新文件时,Linux会用你默认的`UID`和`GID`给文件分配权限。想让其他人也能访问文件,要么改变其他用户所在安全组的访问权限,要么就给文件分配一个包含其他用户的新默认属组。

​	如果你想在大范围环境中创建文档并将文档与人共享,这会很烦琐。幸好有一种简单的方法可以解决这个问题。

​	Linux还为每个文件和目录存储了3个额外的信息位。

- 设置用户ID(`SUID`):当文件被用户使用时,程序会以文件属主的权限运行。
- 设置组ID(`SGID`):对文件来说,程序会以文件属组的权限运行;对目录来说,目录中创建的新文件会以目录的默认属组作为默认属组。
- 粘着位:进程结束后文件还驻留(粘着)在内存中。

​	`SGID`位对文件共享非常重要。启用`SGID`位后,你可以强制在一个共享目录下创建的新文件都属于该目录的属组,这个组也就成为了每个用户的属组。

​	`SGID`可通过` chmod` 命令设置。它会加到标准3位八进制值之前(组成4位八进制值),或者在符号模式下用符号 s 。

​	如果你用的是八进制模式,你需要知道这些位的位置,如表7-6所示。

| 二进制值 | 八进制值 |           描述           |
| :------: | :------: | :----------------------: |
|   000    |    0     |       所有位都清零       |
|   001    |    1     |        粘着位置位        |
|   010    |    2     |       `SGID`位置位       |
|   011    |    3     |  `SGID`位和粘着位都置位  |
|   100    |    4     |       `SUID`位置位       |
|   101    |    5     |  `SUID`位和粘着位都置位  |
|   110    |    6     | `SUID`位和`SGID`位都置位 |
|   111    |    7     |       所有位都置位       |

​	因此,要创建一个共享目录,使目录里的新文件都能沿用目录的属组,只需将该目录的`SGID`位置位。

```shell
$ mkdir testdir
$ ls -l
drwxrwxr-x	2 rich	rich	4096 Sep 20 23:12 testdir/
$ chgrp shared testdir
$ chmod g+s testdir
$ ls -l
drwxrwsr-x	2 rich	shared	4096 Sep 20 23:12 testdir/
$ umask 002
$ cd testdir
$ touch testfile
$ ls -l
total 0
-rw-rw-r--	1 rich	shared	0 Sep 20 23:13 testfile
```

​	首先,用 `mkdir` 命令来创建希望共享的目录。然后通过 `chgrp`命令将目录的默认属组改为包含所有需要共享文件的用户的组(你必须是该组的成员)。最后,将目录的`SGID`位置位,以保证目录中新建文件都用shared作为默认属组。

​	为了让这个环境能正常工作,所有组成员都需把他们的 `umask` 值设置成文件对属组成员可写。在前面的例子中, `umask` 改成了 002 ,所以文件对属组是可写的。

​	写。在前面的例子中, `umask` 改成了 002 ,所以文件对属组是可写的。属组,而不是用户的默认属组。现在shared组的所有用户都能访问这个文件了。