## 第五章 理解 Shell

### 5.1 shell 的类型

​	系统启动什么样的shell程序取决于你个人的用户ID配置。在/etc/passwd文件中，在用户ID记录的第7个字段中列出了默认的shell程序。

​	在下面的例子中，用户christine使用GNU bash shell作为自己的默认shell程序：

```shell
$ cat /etc/passwd
[...]
Christine:x:501:501:Christine B:/home/Christine:/bin/bash
$
```

​	bash shell程序位于/bin目录内。从长列表中可以看出/bin/bash (bash shell)是一个可执行程序:

```shell
$ ls -lF /bin/bash
-rwxr-xr-x. 1 root root 938832 Jul 18	2013 /bin/bash*
$
```

​	默认的交互shell会在用户登录某个虚拟控制台终端或在GUI中运行终端仿真器时启动。不过还有另外一个默认shell是/bin/sh,它作为默认的系统shell,用于那些需要在启动时使用的系统shell脚本。

​	你经常会看到某些发行版使用软链接将默认的系统shell设置成bash shell：

```shell
$ ls -l /bin/sh
lrwxrwxrwx. 1 root root 4 Mar 18 15:05 /bin/sh -> bash
$
```

​	但要注意的是在有些发行版上,默认的系统shell和默认的交互shell并不相同。

***

**窍门**	对bash shell脚本来说,这两种不同的shell(默认的交互shell和默认的系统shell)会造成问题。一定要阅读第11章中有关bash shell脚本首行的语法要求,以避免这些麻烦。

***

### 5.2 shell 的父子关系

​	用于登录某个虚拟控制器终端或在GUI中运行终端仿真器时所启动的默认的交互shell，是一个父shell。本书到目前为止都是父shell提供CLI提示符,然后等待命令输入。

​	使用`ps -f`、`ps --forest`来查看进程之间的关系。详细情况自己操作。主要看PID和PPID之间的关系。

#### 5.2.1 进程列表

​	你可以在一行中指定要依次运行的一系列命令。这可以通过命令列表来实现,只需要在命令之间加入分号(;)即可。

```shell
$ pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls
/home/Christine
Desktop	 Downloads	Music
...
$
```

​	在上面的例子中,所有的命令依次执行,不存在任何问题。不过这并不是进程列表。命令列表要想成为进程列表，这些命令必须包含在括号里。

```shell
$ (pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls)
```

​	尽管多出来的括号看起来没有什么太大的不同，但起到的效果确是非同寻常。括号的加入使命令列表变成了进程列表，生成了一个子shell来执行对应的命令。

***

**进程**	进程列表是一种命令分组(command grouping)。另一种命令分组是将命令放入花括号中，并在命令列表尾部加上分号(;)。语法为 { command; } 。使用花括号进行命令分组并不会像进程列表那样创建出子shell。

***

​	要想知道是否生成了子shell，得借助一个使用了环境变量的命令。(环境变量会在第6章中详述。)这个命令就是 echo $BASH_SUBSHELL 。如果该命令返回 0 ，就表明没有子shell。如果返回1 或者其他更大的数字，就表明存在子shell。

```shell
$ pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL
0
$ (pwd ; ls ; cd /etc ; pwd ; cd ; pwd ; ls ; echo $BASH_SUBSHELL)
1
$ ( pwd ; (echo $BASH_SUBSHELL))
2
```

​	在shell脚本中，经常使用子shell进行多进程处理。但是采用子shell的成本不菲，会明显拖慢处理速度。在交互式的CLI shell会话中，子shell同样存在问题。它并非真正的多进程处理，因为终端控制着子shell的I/O。

#### 5.2.2 别出心裁的子 shell 用法

​	在交互式的shell CLI中，还有很多更富有成效的子shell用法。进程列表、协程和管道(第11章会讲到)都利用了子shell。它们都可以有效地在交互式shell中使用。

​	在交互式shell中,一个高效的子shell用法就是使用后台模式。在讨论如果将后台模式与子shell搭配使用之前，你得先搞明白什么是后台模式。

​	**1. 探索后台模式**

​	在后台模式中运行命令可以在处理命令的同时让出CLI，以供他用。演示后台模式的一个经典命令就是 sleep 。

​	sleep 命令接受一个参数,该参数是你希望进程等待(睡眠)的秒数。这个命令在脚本中常用于引入一段时间的暂停。命令 sleep 10 会将会话暂停10秒钟,然后返回shell CLI提示符。

​	要想将命令置入后台模式,可以在命令末尾加上字符 & 。把 sleep 命令置入后台模式可以让我们利用 ps 命令来小窥一番。

```shell
[kato@aragne ~]$ sleep 20&
[1] 12413
[kato@aragne ~]$ ps -f
UID          PID    PPID  C STIME TTY          TIME CMD
kato        8514    8506  0 10:08 pts/1    00:00:00 /bin/bash
kato       12413    8514  0 10:33 pts/1    00:00:00 sleep 20
kato       12423    8514  0 10:33 pts/1    00:00:00 ps -f
```

​	sleep 命令会在后台( & )睡眠20秒(50分钟)。当它被置入后台,在shell CLI提示符返回之前,会出现两条信息。第一条信息是显示在方括号中的后台作业(background job)号( 1 )。第二条是后台作业的进程ID( 12413) 。

​	ps 命令用来显示各种进程。我们可以注意到命令 sleep 20 已经被列出来了。在第二列显示的进程ID(PID)和命令进入后台时所显示的PID是一样的,都是 12413。

​	除了 ps 命令,你也可以使用 jobs 命令来显示后台作业信息。 jobs 命令可以显示出当前运行在后台模式中的所有用户的进程(作业)。

```shell
$ jobs
[1]+ Running	sleep 3000 &
$
```

​	jobs 命令在方括号中显示出作业号( 1 )。它还显示了作业的当前状态( running )以及对应的命令( sleep 3000 & )。

​	利用 jobs 命令的 -l (字母L的小写形式)选项,你还能够看到更多的相关信息。除了默认信息之外, -l 选项还能够显示出命令的PID。一旦后台作业完成,就会显示出结束状态。

​	后台模式非常方便,它可以让我们在CLI中创建出有实用价值的子shell。

​	**2. 将进程列表置入后台**

​	之前说过,进程列表是运行在子shell中的一条或多条命令。使用包含了 sleep 命令的进程列表,并显示出变量 BASH_SUBSHELL ,结果和期望的一样。

```shell
[kato@aragne ~]$ (sleep 2 ; echo $BASH_SUBSHELL ; sleep 2)
1
[kato@aragne ~]$
```

​	在上面的例子中,有一个2秒钟的暂停,显示出的数字 1 表明只有一个子shell,在返回提示符之前又经历了另一个2秒钟的暂停。没什么大事。

​	将相同的进程列表置入后台模式会在命令输出上表现出些许不同。

```shell
[kato@aragne ~]$ (sleep 2 ; echo $BASH_SUBSHELL ; sleep 2)&
[1] 15700
[kato@aragne ~]$ 1

[1]+  已完成               ( sleep 2; echo $BASH_SUBSHELL; sleep 2 )
[kato@aragne ~]$
```

​	把进程列表置入后台会产生一个作业号和进程ID,然后返回到提示符。不过奇怪的是表明单一级子shell的数字 1 显示在了提示符的旁边!不要不知所措,只需要按一下回车键,就会得到另一个提示符。

​	在CLI中运用子shell的创造性方法之一就是将进程列表置入后台模式。你既可以在子shell中进行繁重的处理工作,同时也不会让子shell的I/O受制于终端。

​	当然了, sleep 和 echo 命令的进程列表只是作为一个示例而已。使用 tar (参见第4章)创建备份文件是有效利用后台进程列表的一个更实用的例子。

```shell
$ (tar -cf Rich.tar /home/rich ; tar -cf My.tar /home/christine)&
[3] 2423
$
```

​	将进程列表置入后台模式并不是子shell在CLI中仅有的创造性用法。协程就是另一种方法。

​	**3. 协程**

​	协程可以同时做两件事。它在后台生成一个子shell,并在这个子shell中执行命令。要进行协程处理,得使用 coproc 命令,还有要在子shell中执行的命令。

```shell
[kato@aragne ~]$ coproc sleep 10
[1] 39996
```

​	除了会创建子shell之外,协程基本上就是将命令置入后台模式。当输入 coproc 命令及其参数之后,你会发现启用了一个后台作业。屏幕上会显示出后台作业号( 1 )以及进程ID( 2544 )。

​	jobs 命令能够显示出协程的处理状态。

```shell
[kato@aragne ~]$ jobs
[1]+  运行中               coproc COPROC sleep 10 &
```

​	在上面的例子中可以看到在子shell中执行的后台命令是 coproc COPROC sleep 10 。 COPROC是 coproc 命令给进程起的名字。你可以使用命令的扩展语法自己设置这个名字。

```shell
$ coproc My_Job { sleep 10; }
[1] 2570
$
$ jobs
[1]+ Running	coproc My_Job { sleep 10; } &
$
```

​	通过使用扩展语法,协程的名字被设置成 My_Job 。这里要注意的是,扩展语法写起来有点麻烦。必须确保在第一个花括号( { )和命令名之间有一个空格。还必须保证命令以分号(;)结尾。另外,分号和闭花括号( } )之间也得有一个空格。

​	你可以发挥才智,将协程与进程列表结合起来产生嵌套的子shell。只需要输入进程列表,然后把命令 coproc 放在前面就行了。

```shell
[kato@aragne ~]$ coproc (sleep 10;sleep 2)
[1] 41643
[kato@aragne ~]$ ps --forest
    PID TTY          TIME CMD
   8514 pts/1    00:00:00 bash
  41643 pts/1    00:00:00  \_ bash
  41666 pts/1    00:00:00  |   \_ sleep
  41668 pts/1    00:00:00  \_ ps
```

​	记住,生成子shell的成本不低,而且速度还慢。创建嵌套子shell更是火上浇油!

​	在命令行中使用子shell能够获得灵活性和便利。要想获得这些优势,重要的是理解子shell的行为方式。对于命令也是如此。在下一节中,我们将研究内建命令与外部命令之间的行为差异。

### 5.3 理解 Shell 的内建命令

#### 5.3.1 外部命令

​	外部命令,有时候也被称为文件系统命令,是存在于bash shell之外的程序。它们并不是shell程序的一部分。外部命令程序通常位于/bin、/usr/bin、/sbin或/usr/sbin中。

​	ps 就是一个外部命令。你可以使用 which 和 type 命令找到它。

```shell
[kato@aragne ~]$ which ps
/usr/bin/ps
[kato@aragne ~]$ type -a ps
ps 是 /usr/bin/ps
```

​	当外部命令执行时,会创建出一个子进程。这种操作被称为衍生(forking)。外部命令 ps 很方便显示出它的父进程以及自己所对应的衍生子进程。

​	当进程必须执行衍生操作时,它需要花费时间和精力来设置新子进程的环境。所以说,外部命令多少还是有代价的。

#### 5.3.2 内建命令

​	内建命令和外部命令的区别在于前者不需要使用子进程来执行。它们已经和shell编译成了一体,作为shell工具的组成部分存在。不需要借助外部程序文件来运行。

​	cd 和 exit 命令都内建于bash shell。可以利用 type 命令来了解某个命令是否是内建的。

```shell
[kato@aragne ~]$ type cd
cd 是 shell 内建
[kato@aragne ~]$ type exit
exit 是 shell 内建
[kato@aragne ~]$
[kato@aragne ~]$ type -a pwd
pwd 是 shell 内建
pwd 是 /usr/bin/pwd
```

​	因为既不需要通过衍生出子进程来执行,也不需要打开程序文件,内建命令的执行速度要更快,效率也更高。

​	要注意,有些命令有多种实现。例如 echo 和 pwd 既有内建命令也有外部命令。两种实现略有不同。要查看命令的不同实现,使用 type 命令的 -a 选项。注意， which 命令只显示出了外部命令文件。

​	**1. 使用history命令**

​	一个有用的内建命令是 history 命令。bash shell会跟踪你用过的命令。你可以唤回这些命令并重新使用。

​	要查看最近用过的命令列表,可以输入不带选项的 history 命令。通常历史记录中会保存最近的1000条命令。这个数量可是不少的！

​	你可以设置保存在bash历史记录中的命令数。要想实现这一点,你需要修改名为 HISTSIZE的环境变量(参见第6章)。

​	你可以唤回并重用历史列表中最近的命令。这样能够节省时间和击键量。输入 !! ,然后按回车键就能够唤出刚刚用过的那条命令来使用。

​	当输入 !! 时,bash首先会显示出从shell的历史记录中唤回的命令。然后执行该命令。

​	命令历史记录被保存在隐藏文件.bash_history中,它位于用户的主目录中。这里要注意的是,bash命令的历史记录是先存放在内存中,当shell退出时才被写入到历史文件中。

​	可以在退出shell会话之前强制将命令历史记录写入.bash_history文件。要实现强制写入,需要使用 history 命令的 -a选 项。

​	你可以唤回历史列表中任意一条命令。只需输入惊叹号和命令在历史列表中的编号即可。

​	**2. 命名别名**

​	alias 命令是另一个shell的内建命令。命令别名允许你为常用的命令(及其参数)创建另一个名称,从而将输入量减少到最低。

​	你所使用的Linux发行版很有可能已经为你设置好了一些常用命令的别名。要查看当前可用的别名,使用 alias 命令以及选项 -p 。

​	在定义好别名之后,你随时都可以在shell中使用它,就算在shell脚本中也没问题。要注意,因为命令别名属于内部命令,一个别名仅在它所被定义的shell进程中才有效。

​	不过好在有办法能够让别名在不同的子shell中都奏效。下一章中就会讲到具体的做法,另外还会介绍环境变量。