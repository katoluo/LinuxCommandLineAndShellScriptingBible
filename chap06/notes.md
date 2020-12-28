## 第六章 使用Linux环境变量

​	Linux环境变量能帮你提升Linux shell体验。很多程序和脚本都通过环境变量来获取系统信息、存储临时数据和配置信息。在Linux系统上有很多地方可以设置环境变量,了解去哪里设置相应的环境变量很重要。

​	本章将带你逐步了解Linux环境变量:它们存储在哪里,怎样使用,以及怎样创建自己的环境变量。最后以数组变量的用法作结。

### 6.1 什么是环境变量

​	bash shell用一个叫作**环境变量**(environment variable)的特性来存储有关shell会话和工作环境的信息(这也是它们被称作环境变量的原因)。这项特性允许你在内存中存储数据,以便程序或shell中运行的脚本能够轻松访问到它们。这也是存储持久数据的一种简便方法。

​	在bash shell中,环境变量分为两类：**全局变量**和**局部变量**

#### 6.1.1 全局环境变量

​	全局环境变量对于shell会话和所有生成的子shell都是可见的。局部变量则只对创建它们的shell可见。这让全局环境变量对那些所创建的子shell需要获取父shell信息的程序来说非常有用。

​	Linux系统在你开始bash会话时就设置了一些全局环境变量(如想了解此时设置了哪些变量,请参见6.6节)。系统环境变量基本上都是使用全大写字母,以区别于普通用户的环境变量。

​	要查看全局变量,可以使用 `env` 或 `printenv` 命令。

​	系统为bash shell设置的全局环境变量数目众多，其中有很多是在登录过程中设置的,另外,你的登录方式也会影响到所设置的环境变量。

​	要显示个别环境变量的值,可以使用 `printenv` 命令,但是不要用 `env` 命令。

```shell
[kato@aragne ~]$ env HOME
env: “HOME”: 没有那个文件或目录
[kato@aragne ~]$ printenv HOME
/home/kato
```

​	也可以使用 echo 显示变量的值。在这种情况下引用某个环境变量的时候,必须在变量前面加上一个美元符( $ )。

```shell
[kato@aragne ~]$ echo $HOME
/home/kato
```

​	在 echo 命令中,在变量名前加上 $ 可不仅仅是要显示变量当前的值。它能够让变量作为命令行参数。

```shell
[kato@aragne ~]$ ls $HOME
Book  Desktop  Documents  Downloads ...
```

#### 6.1.2 局部环境变量

​	顾名思义,局部环境变量只能在定义它们的进程中可见。尽管它们是局部的,但是和全局环境变量一样重要。事实上,Linux系统也默认定义了标准的局部环境变量。不过你也可以定义自己的局部变量,如你所想,这些变量被称为用户定义局部变量。

​	查看局部环境变量的列表有点复杂。遗憾的是,在Linux系统并没有一个只显示局部环境变量的命令。 set 命令会显示为某个特定进程设置的所有环境变量,包括局部变量、全局变量以及用户定义变量。

### 6.2 设置用户定义变量

​	可以在bash shell中直接设置自己的变量。本节将介绍怎样在交互式shell或shell脚本程序中创建自己的变量并引用它们。

#### 6.2.1 设置局部用户变量

​	一旦启动了bash shell(或者执行一个shell脚本),就能创建在这个shell进程内可见的局部变量了。可以通过等号给环境变量赋值,值可以是数值或字符串。

```shell
[kato@aragne ~]$ echo $my_variable

[kato@aragne ~]$ my_variable=Hello
[kato@aragne ~]$ echo $my_variable
Hello
```

​	非常简单!现在每次引用 my_variable 环境变量的值,只要通过 $my_variable 引用即可。

​	如果要给变量赋一个含有空格的字符串值,必须用双引号来界定字符串的首和尾。

​	记住,变量名、等号和值之间没有空格,这一点非常重要。如果在赋值表达式中加上了空格，bash shell就会把值当成一个单独的命令：

```shell
[kato@aragne ~]$ my_variable = Hello
bash: my_variable：未找到命令
```

​	设置了局部环境变量后,就能在shell进程的任何地方使用它了。但是,如果生成了另外一个shell,它在子shell中就不可用。

#### 6.2.2 设置全局环境变量

​	在设定全局环境变量的进程所创建的子进程中,该变量都是可见的。创建全局环境变量的方法是先创建一个局部环境变量,然后再把它导出到全局环境中。

​	这个过程通过 export 命令来完成,变量名前面不需要加 $ 。

```shell
[kato@aragne ~]$ global_variable=Hello
[kato@aragne ~]$ export global_variable
[kato@aragne ~]$ echo $global_variable
Hello
[kato@aragne ~]$ zsh
kato@aragne Linux 5.8.18-1-MANJARO x86_64 20.2 Nibia
~ >>> echo $global_variable
Hello
~ >>> exit
[kato@aragne ~]$ printenv global_variable
Hello
```

​	修改子shell中全局环境变量并不会影响到父shell中该变量的值。

​	在定义并导出变量 my_variable 后, bash 命令启动了一个子shell。在这个子shell中能够正确显示出全局环境变量 my_variable 的值。子shell随后改变了这个变量的值。但是这种改变仅在子shell中有效,并不会被反映到父shell中。

​	子shell甚至无法使用 export 命令改变父shell中全局环境变量的值。

​	尽管子shell重新定义并导出了变量 my_variable ,但父shell中的 my_variable 变量依然保留着原先的值。

### 6.3 删除环境变量

​	当然,既然可以创建新的环境变量,自然也能删除已经存在的环境变量。可以用 unset 命令完成这个操作，`unset my_variable`。在 unset 命令中引用环境变量时,记住不要使用 $ 。

​	在处理全局环境变量时,事情就有点棘手了。如果你是在子进程中删除了一个全局环境变量,这只对子进程有效。该全局环境变量在父进程中依然可用。

​	和修改变量一样,在子shell中删除全局变量后,你无法将效果反映到父shell中。

### 6.4 默认的shell环境变量

​	默认情况下,bash shell会用一些特定的环境变量来定义系统环境。这些变量在你的Linux系统上都已经设置好了,只管放心使用。bash shell源自当初的Unix Bourne shell,因此也保留了Unix Bourne shell里定义的那些环境变量。

### 6.5 设置 PATH 环境变量

​	当你在shell命令行界面中输入一个外部命令时(参见第5章),shell必须搜索系统来找到对应的程序。 PATH 环境变量定义了用于进行命令和程序查找的目录。

```shell
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:
/sbin:/bin:/usr/games:/usr/local/games
$
```

​	输出中显示了有8个可供shell用来查找命令和程序。 PATH 中的目录使用冒号分隔。

​	如果命令或者程序的位置没有包括在 PATH 变量中,那么如果不使用绝对路径的话, shell是没法找到的。如果shell找不到指定的命令或程序,它会产生一个错误信息：

```
$ myprog
-bash: myprog: command not found
$
```

​	问题是,应用程序放置可执行文件的目录常常不在 PATH 环境变量所包含的目录中。解决的办法是保证 PATH 环境变量包含了所有存放应用程序的目录。

​	可以把新的搜索目录添加到现有的 PATH 环境变量中,无需从头定义。 PATH 中各个目录之间是用冒号分隔的。你只需引用原来的 PATH 值,然后再给这个字符串添加新目录就行了。可以参考下面的例子。

```shell
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:
/sbin:/bin:/usr/games:/usr/local/games
$
$ PATH=$PATH:/home/christine/Scripts
$
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/home/christine/Scripts
```

​	将目录加到 PATH 环境变量之后,你现在就可以在虚拟目录结构中的任何位置执行程序。

​	如果希望子shell也能找到你的程序的位置,一定要记得把修改后的 PATH 环境变量导出。

​	程序员通常的办法是将单点符也加入 PATH 环境变量。该单点符代表当前目录。

​	对 PATH 变量的修改只能持续到退出或重启系统。这种效果并不能一直持续。在下一节中,你会学到如何永久保持环境变量的修改效果。

### 6.6 定位系统环境变量

​	环境变量在Linux系统中的用途很多。你现在已经知道如何修改系统环境变量,也知道了如何创建自己的环境变量。接下来的问题是怎样让环境变量的作用持久化。

​	在你登入Linux系统启动一个bash shell时,默认情况下bash会在几个文件中查找命令。这些文件叫作启动文件或环境文件。bash检查的启动文件取决于你启动bash shell的方式。启动bash shell有3种方式：

- 登录时作为默认登录shell
- 作为非登录shell的交互式shell
- 作为运行脚本的非交互式shell

#### 6.6.1 登录 shell

​	当你登录Linux系统时,bash shell会作为登录shell启动。登录shell会从5个不同的启动文件里读取命令：

- `/etc/profile`
- `$HOME/.bash_profile`
- `$HOME/.bashrc`
- `$HOME/.bash_login`
- `$HOME/.profile`

​	/etc/profile文件是系统上默认的bash shell的主启动文件。系统上的每个用户登录时都会执行这个启动文件。

***

**说明**	要 留 意 的 是 有 些 Linux 发 行 版 使 用 了 可 拆 卸 式 认 证 模 块 ( Pluggable Authentication Modules ,PAM)。在这种情况下,PAM文件会在bash shell启动之前处理,这些文件中可能会包含环境变量。PAM文件包括 /etc/environment 文件和 `$HOME/.pam_environment` 文件。PAM更多的相关信息可以在http://linux-pam.org中找到。

***

​	另外4个启动文件是针对用户的,可根据个人需求定制。我们来仔细看一下各个文件。

​	**1. /etc/profile**

​	/etc/profile文件是bash shell默认的的主启动文件。只要你登录了Linux系统,bash就会执行/etc/profile 启动文件中的命令。不同的Linux发行版在这个文件里放了不同的命令。

​	这个文件中的大部分命令和语法都会在第12章以及后续章节中具体讲到。每个发行版的/etc/profile文件都有不同的设置和命令。

​	/etc/profile文件都用到了同一个特性: for 语句。它用来迭代`/etc/profile.d`目录下的所有文件。(该语句会在第13章中详述。)这为Linux系统提供了一个放置特定应用程序启动文件的地方,当用户登录时,shell会执行这些文件。

​	不难发现,有些文件与系统中的特定应用有关。大部分应用都会创建两个启动文件:一个供bash shell使用(使用.sh扩展名),一个供c shell使用(使用`.csh`扩展名)。

​	`lang.csh`和`lang.sh`文件会尝试去判定系统上所采用的默认语言字符集,然后设置对应的 LANG

环境变量。

​	**2. $HOME目录下的启动文件**

​	剩下的启动文件都起着同一个作用:提供一个用户专属的启动文件来定义该用户所用到的环境变量。大多数Linux发行版只用这四个启动文件中的一到两个：

- `$HOME/.bash_profile`
- `$HOME/.bashrc`
- `$HOME/.bash_login`
- `$HOME/.profile`

​	注意,这四个文件都以点号开头,这说明它们是隐藏文件(不会在通常的 ls 命令输出列表中出现)。它们位于用户的HOME目录下,所以每个用户都可以编辑这些文件并添加自己的环境变量,这些环境变量会在每次启动bash shell会话时生效。

​	shell会按照按照下列顺序,运行第一个被找到的文件,余下的则被忽略：

- `$HOME/.bash_profile`
- `$HOME/.bash_login`
- `$HOME/.profile`

​	注意,这个列表中并没有`$HOME/.bashrc`文件。这是因为该文件通常通过其他文件运行的。

​	.bash_profile启动文件会先去检查HOME目录中是不是还有一个叫`.bashrc`的启动文件。如果有的话,会先执行启动文件里面的命令。

#### 6.6.2 交互式 shell 进程

​	如果你的bash shell不是登录系统时启动的(比如是在命令行提示符下敲入 bash 时启动),那么你启动的shell叫作交互式shell。交互式shell不会像登录shell一样运行,但它依然提供了命令行提示符来输入命令。

​	如果bash是作为交互式shell启动的,它就不会访问/etc/profile文件,只会检查用户HOME目录中的`.bashrc`文件。

​	`.bashrc`文件有两个作用:一是查看/etc目录下通用的`bashrc`文件,二是为用户提供一个定制自己的命令别名(参见第5章)和私有脚本函数(将在第17章中讲到)的地方。

#### 6.6.3 非交互式 shell

​	最后一种shell是非交互式shell。系统执行shell脚本时用的就是这种shell。不同的地方在于它没有命令行提示符。但是当你在系统上运行脚本时,也许希望能够运行一些特定启动的命令。

***

**窍门**	脚本能以不同的方式执行。只有其中的某一些方式能够启动子shell。你会在第11章中学习到shell不同的执行方式。

***

​	为了处理这种情况,bash shell提供了 `BASH_ENV` 环境变量。当shell启动一个非交互式shell进程时,它会检查这个环境变量来查看要执行的启动文件。如果有指定的文件,shell会执行该文件里的命令,这通常包括shell脚本变量设置。

​	如果 `BASH_ENV` 变量没有设置,shell脚本到哪里去获得它们的环境变量呢?别忘了有些shell脚本是通过启动一个子shell来执行的(参见第5章)。子shell可以继承父shell导出过的变量。

​	举例来说,如果父shell是登录shell,在`/etc/profile`、`/etc/profile.d/ * .sh`和`$HOME/.bashrc`文件中设置并导出了变量,用于执行脚本的子shell就能够继承这些变量。

​	要记住,由父shell设置但并未导出的变量都是局部变量。子shell无法继承局部变量。

​	对 于 那 些 不 启 动 子 shell 的 脚 本 , 变 量 已 经 存 在 于 当 前 shell 中 了 。 所 以 就 算 没 有 设 置`BASH_ENV `,也可以使用当前shell的局部变量和全局变量。

#### 6.6.4 环境变量持久化

​	现在你已经了解了各种shell进程以及对应的环境文件,找出永久性环境变量就容易多了。也可以利用这些文件创建自己的永久性全局变量或局部变量。

​	对全局环境变量来说(Linux系统中所有用户都需要使用的变量)可能更倾向于将新的或修改过的变量设置放在/etc/profile文件中,但这可不是什么好主意。如果你升级了所用的发行版,这个文件也会跟着更新,那你所有定制过的变量设置可就都没有了。

​	最好是在`/etc/profile.d`目录中创建一个以.sh结尾的文件。把所有新的或修改过的全局环境变量设置放在这个文件中。

​	在大多数发行版中,存储个人用户永久性bash shell变量的地方是`$HOME/.bashrc`文件。这一点适用于所有类型的shell进程。但如果设置了 `BASH_ENV` 变量,那么记住,除非它指向的是`$HOME/.bashrc`,否则你应该将非交互式shell的用户变量放在别的地方。

​	想想第5章中讲过的 alias 命令设置就是不能持久的。你可以把自己的 alias 设置放在`$HOME/.bashrc`启动文件中,使其效果永久化。

### 6.7 数组变量

​	环境变量有一个很酷的特性就是,它们可作为数组使用。数组是能够存储多个值的变量。这些值可以单独引用,也可以作为整个数组来引用。

​	要给某个环境变量设置多个值,可以把值放在括号里,值与值之间用空格分隔。

​	只有数组的第一个值显示出来了。要引用一个单独的数组元素,就必须用代表它在数组中位置的数值索引值。索引值要用方括号括起来。

​	要显示整个数组变量,可用星号作为通配符放在索引值的位置。

​	也可以改变某个索引值位置的值。

```shell
$ mytest=(one two three four five)
$
$ echo $mytest
one
$
$ echo ${mytest[2]}
three
$
$ echo ${mytest[*]}
one two three four five
$
$ mytest[2]=seven
$
$ echo ${mytest[*]}
one two seven four five
$
```

​	甚至能用 unset 命令删除数组中的某个值,但是要小心,这可能会有点复杂。看下面的例子。

```shell
$ unset mytest[2]
$
$ echo ${mytest[*]}
one two four five
$
$ echo ${mytest[2]}
$ echo ${mytest[3]}
four
$
```

​	这个例子用 unset 命令删除在索引值为 2 的位置上的值。显示整个数组时,看起来像是索引里面已经没这个索引了。但当专门显示索引值为 2 的位置上的值时,就能看到这个位置是空的。

​	最后,可以在 unset 命令后跟上数组名来删除整个数组。

```shell
$ unset mytest
$
$ echo ${mytest[*]}

$
```

​	有时数组变量会让事情很麻烦,所以在shell脚本编程时并不常用。对其他shell而言,数组变量的可移植性并不好,如果需要在不同的shell环境下从事大量的脚本编写工作,这会带来很多不便。有些bash系统环境变量使用了数组(比如 `BASH_VERSINFO` ),但总体上不会太频繁用到。