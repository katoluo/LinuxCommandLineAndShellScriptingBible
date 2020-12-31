## 第11章 构建基本脚本

## 11.1 使用多个命令

​	到目前为止,你已经了解了如何使用shell的命令行界面提示符来输入命令和查看命令的结果。shell脚本的关键在于输入多个命令并处理每个命令的结果,甚至需要将一个命令的结果传给另一个命令。shell可以让你将多个命令串起来,一次执行完成。如果要两个命令一起运行,可以把它们放在同一行中,彼此间用分号隔开。

```shell
[kato@aragne ~]$ date;who
2020年 12月 30日 星期三 09:54:08 CST
kato     :1           2020-12-30 09:38 (:1)
```

## 11.2 创建shell脚本文件

​	在创建shell脚本文件时,必须在文件的第一行指定要使用的shell。其格式为:

```shell
#!/bin/bash
```

​	在通常的shell脚本中,井号( # )用作注释行。shell并不会处理shell脚本中的注释行。然而,shell脚本文件的第一行是个例外, # 后面的惊叹号会告诉shell用哪个shell来运行脚本(是的,你可以使用bash shell,同时还可以使用另一个shell来运行你的脚本)。

​	在指定了shell之后,就可以在文件的每一行中输入命令,然后加一个回车符。之前提到过,注释可用 # 添加。例如:

```shell
#!/bin/bash
# This script displays the date and who's logged on
date
who
```

​	这就是脚本的所有内容了。可以根据需要,使用分号将两个命令放在一行上,但在shell脚本中,你可以在独立的行中书写命令。shell会按根据命令在文件中出现的顺序进行处理。

​	将这个脚本保存在名为test1的文件中,基本就好了。在运行新脚本前,还要做其他一些事。

​	现在运行脚本,结果可能会叫你有点失望。

```shell
$ test1
bash: test1: command not found
$
```

​	你要跨过的第一个障碍是让bash shell能找到你的脚本文件。如第6章所述,shell会通过 PATH环境变量来查找命令。快速查看一下 PATH 环境变量就可以弄清问题所在。

```shell
$ echo $PATH
/usr/kerberos/sbin:/usr/kerberos/bin:/usr/local/bin:/usr/bin
:/bin:/usr/local/sbin:/usr/sbin:/sbin:/home/user/bin 
$
```

​	PATH 环境变量被设置成只在一组目录中查找命令。要让shell找到test1脚本,只需采取以下两种作法之一:

- 将shell脚本文件所处的目录添加到 PATH 环境变量中;
- 在提示符中用绝对或相对文件路径来引用shell脚本文件。

​	在这个例子中,我们将用第二种方式将脚本文件的确切位置告诉shell。记住,为了引用当前目录下的文件,可以在shell中使用单点操作符。

```shell
$ ./test1
bash: ./test1: Permission denied
$
```

​	现在shell找到了脚本文件,但还有一个问题。shell指明了你还没有执行文件的权限。快速查看一下文件权限就能找到问题所在。

```shell
$ ls -l test1
-rw-rw-r--	1 user	user	73 Sep 24 19:56 test1
$
```

​	在创建test1文件时, umask 的值决定了新文件的默认权限设置。由于 umask 变量在Ubuntu中被设成了 022 (参见第7章)，所以系统创建的文件只有文件属主和属组才有读/写权限。

​	下一步是通过 chmod 命令(参见第7章)赋予文件属主执行文件的权限。

```shell
$ chmod u+x test1
$ ./test1
2020年 12月 30日 星期三 09:54:08 CST
kato     :1           2020-12-30 09:38 (:1)
```

​	成功了!现在万事俱备,只待执行新的shell脚本文件了。

### 11.3 显示消息

​	大多数shell命令都会产生自己的输出,这些输出会显示在脚本所运行的控制台显示器上。很多时候,你可能想要添加自己的文本消息来告诉脚本用户脚本正在做什么。可以通过 echo 命令来实现这一点。如果在 echo 命令后面加上了一个字符串,该命令就能显示出这个文本字符串。

```shell
$ echo This is a test
This is a test
$
```

​	注意,默认情况下,不需要使用引号将要显示的文本字符串划定出来。但有时在字符串中出现引号的话就比较麻烦了。

```shell
$ echo Let's see if this'll work
Lets see if thisll work
$
```

​	echo 命令可用单引号或双引号来划定文本字符串。如果在字符串中用到了它们,你需要在文本中使用其中一种引号,而用另外一种来将字符串划定起来。

```shell
$ echo "This is a test to see if you're paying attention"
This is a test to see if you're paying attention
$ echo 'Rich says "scripting is easy".'
Rich says "scripting is easy".
$
```

​	所有的引号都可以正常输出了。

​	可以将 echo 语句添加到shell脚本中任何需要显示额外信息的地方。

```shell
$ cat test1
#!/bin/bash
# This script displays the date and who's logged on
echo The time and date are:
date
echo "Let's see who's logged into the system:"
who
$
```

​	很好,但如果想把文本字符串和命令输出显示在同一行中,该怎么办呢?可以用 echo 语句的 -n 参数。只要将第一个 echo 语句改成这样就行:

```shell
echo -n "The time and date are: "
```

​	你需要在字符串的两侧使用引号,保证要显示的字符串尾部有一个空格。命令输出将会在紧接着字符串结束的地方出现。现在的输出会是这样:

```shell
$ ./test1
The time and date are: Mon Feb 21 15:42:23 EST 2014
Let's see who's logged into the system:
Christine tty2		2014-02-21 15:26
```

​	完美! echo 命令是shell脚本中与用户交互的重要工具。你会发现在很多地方都能用到它,尤其是需要显示脚本中变量的值的时候。我们下面继续了解这个。

### 11.4 使用变量

​	运行shell脚本中的单个命令自然有用,但这有其自身的限制。通常你会需要在shell命令使用其他数据来处理信息。这可以通过变量来实现。变量允许你临时性地将信息存储在shell脚本中,以便和脚本中的其他命令一起使用。本节将介绍如何在shell脚本中使用变量。

#### 11.4.1 环境变量

​	你已经看到过Linux的一种变量在实际中的应用。第6章介绍了Linux系统的环境变量。也可以在脚本中访问这些值。

​	shell维护着一组环境变量,用来记录特定的系统信息。比如系统的名称、登录到系统上的用户名、用户的系统ID(也称为UID)、用户的默认主目录以及shell查找程序的搜索路径。可以用set 命令来显示一份完整的当前环境变量列表。

​	在脚本中,你可以在环境变量名称之前加上美元符( $ )来使用这些环境变量。下面的脚本演示了这种用法。

```shell
$ cat test.sh
#!/bin/bash
# display user information from the system.
echo "User info for userid: $USER"
echo UID: $UID
echo HOME: $HOME
$
```

​	USER 、 UID 和 HOME 环境变量用来显示已登录用户的有关信息。脚本输出如下:

```shell
$chmod 744 test
$chmod u+x test
$./test
[kato@aragne shell]$ ./test
User info for userid: kato
UID: 1000
HOME: /home/kato
```

​	注意, echo 命令中的环境变量会在脚本运行时替换成当前值。另外,在第一个字符串中可以将 $USER 系统变量放置到双引号中,而shell依然能够知道我们的意图。但采用这种方法也有一个问题。看看下面这个例子会怎么样。

```shell
$ echo "The cost of the item is $15"
The cost of the item is 5
```

​	显然这不是我们想要的。只要脚本在引号中出现美元符,它就会以为你在引用一个变量。在这个例子中,脚本会尝试显示变量 $1 (但并未定义),再显示数字5。要显示美元符,你必须在它前面放置一个反斜线。

```shell
$ echo "The cost of the item is \$15"
The cost of the item is $15
```

​	看起来好多了。反斜线允许shell脚本将美元符解读为实际的美元符,而不是变量。下一节将会介绍如何在脚本中创建自己的变量。

#### 11.4.2 用户变量

​	除了环境变量,shell脚本还允许在脚本中定义和使用自己的变量。定义变量允许临时存储数据并在整个脚本中使用,从而使shell脚本看起来更像一个真正的计算机程序。

​	用户变量可以是任何由字母、数字或下划线组成的文本字符串,长度不超过20个。用户变量区分大小写,所以变量 Var1 和变量 var1 是不同的。这个小规矩经常让脚本编程初学者感到头疼。

​	使用等号将值赋给用户变量。在变量、等号和值之间不能出现空格(另一个困扰初学者的用法)。这里有一些给用户变量赋值的例子。

```shell
var1=10
var2=-57
var3=testing
var4="still more testing"
```

​	shell脚本会自动决定变量值的数据类型。在脚本的整个生命周期里,shell脚本中定义的变量会一直保持着它们的值,但在shell脚本结束时会被删除掉。

​	与系统变量类似,用户变量可通过美元符引用。

```shell
$ cat test3
#!/bin/bash
# testing variables
days=10
guest="Katie"
echo "$guest checked in $days days ago"
days=5
guest="Jessica"
echo "$guest checked in $days days ago"
$
```

​	运行脚本会有如下输出。

```shell
$ chmod u+x test3
$ ./test3
Katie checked in 10 days ago
Jessica checked in 5 days ago
$
```

​	变量每次被引用时,都会输出当前赋给它的值。重要的是要记住,引用一个变量值时需要使用美元符,而引用变量来对其进行赋值时则不要使用美元符。通过一个例子你就能明白我的意思。

```shell
$ cat test4
#!/bin/bash
# assigning a variable value to another variable
value1=10
value2=$value1
echo The resulting value is $value2
$
```

​	在赋值语句中使用 value1 变量的值时,仍然必须用美元符。这段代码产生如下输出。

```shell
$ chmod u+x test4
$ ./test4
The resulting value is 10
$
```

​	要是忘了用美元符,使得 value2 的赋值行变成了这样:

```shell
value2=value1
```

​	那你会得到如下输出:

```shell
$ ./test4
The resulting value is value1
$
```

​	没有美元符,shell会将变量名解释成普通的文本字符串,通常这并不是你想要的结果。

#### 11.4.3 命令替换

​	shell脚本中最有用的特性之一就是可以从命令输出中提取信息,并将其赋给变量。把输出赋给变量之后,就可以随意在脚本中使用了。这个特性在处理脚本数据时尤为方便。

​	有两种方法可以将命令输出赋给变量:

- 反引号字符( ` )
- $() 格式

​	要注意反引号字符,这可不是用于字符串的那个普通的单引号字符。由于在shell脚本之外很少用到,你可能甚至都不知道在键盘什么地方能找到这个字符。但你必须慢慢熟悉它,因为这是许多shell脚本中的重要组件。提示:在美式键盘上,它通常和波浪线(~)位于同一键位。

​	命令替换允许你将shell命令的输出赋给变量。尽管这看起来并不那么重要,但它却是脚本编程中的一个主要组成部分。

```shell
testing=`date`
```

​	要么使用 $() 格式:

```shell
testing=$(date)
```

​	shell会运行命令替换符号中的命令,并将其输出赋给变量 testing 。注意,赋值等号和命令替换字符之间没有空格。这里有个使用普通的shell命令输出创建变量的例子。

```shell
[kato@aragne shell]$ cat test5
#!/bin/bash
testing=$(date)
echo "The date and time are: " $testing
```

​	变量 testing 获得了 date 命令的输出,然后使用 echo 语句显示出它的值。运行这个shell脚本生成如下输出。

```shell
[kato@aragne shell]$ chmod 744 test5
[kato@aragne shell]$ ./test5
The date and time are:  2020年 12月 30日 星期三 13:29:14 CST
```

​	这个例子毫无吸引人的地方(也可以干脆将该命令放在 echo 语句中),但只要将命令的输出放到了变量里,你就可以想干什么就干什么了。

​	下面这个例子很常见,它在脚本中通过命令替换获得当前日期并用它来生成唯一文件名。

```shell
#!/bin/bash
# copy the /usr/bin directory listing to a log file
today=$(date +%y%m%d)
ls /usr/bin -al > log.$today
```

​	today 变量是被赋予格式化后的 date 命令的输出。这是提取日期信息来生成日志文件名常用的一种技术。 +%y%m%d 格式告诉 date 命令将日期显示为两位数的年月日的组合。

```shell
[kato@aragne shell]$ date +%y%m%d
201230
```

​	这个脚本将日期值赋给一个变量,之后再将其作为文件名的一部分。文件自身含有目录列表的重定向输出(将在11.5节详细讨论)。运行该脚本之后,应该能在目录中看到一个新文件。

```shell
-rw-r--r-- 1 kato kato 223922 12月 30 13:35 log.201230
```

​	目录中出现的日志文件采用 $today 变量的值作为文件名的一部分。日志文件的内容是/usr/bin目录内容的列表输出。如果脚本在明天运行,日志文件名会是log.140201,就这样为新的一天创建一个新文件。

***

**警告**	命令替换会创建一个子shell来运行对应的命令。子shell ( subshell)是由运行该脚本的shell所创建出来的一个独立的子shell(child shell)。正因如此,由该子shell所执行命令是无法使用脚本中所创建的变量的。

在命令行提示符下使用路径 ./ 运行命令的话,也会创建出子shell;要是运行命令的时候不加入路径,就不会创建子shell。如果你使用的是内建的shell命令,并不会涉及子shell。在命令行提示符下运行脚本时一定要留心!

***

### 11.5 重定向输入和输出

​	有些时候你想要保存某个命令的输出而不仅仅只是让它显示在显示器上。 bash shell提供了几个操作符,可以将命令的输出重定向到另一个位置(比如文件)。重定向可以用于输入,也可以用于输出,可以将文件重定向到命令输入。本节介绍了如何在shell脚本中使用重定向。

#### 11.5.1 输出重定向

​	最基本的重定向将命令的输出发送到一个文件中。bash shell用大于号(>)来完成这项功能:

```shell
command > outputfile
```

​	之前显示器上出现的命令输出会被保存到指定的输出文件中。

```shell
$ date > test6
$ cat test6
Thu Feb 10 17:56:58 EDT 2014
$
```

​	重定向操作符创建了一个文件test6(通过默认的 umask 设置),并将 date 命令的输出重定向到该文件中。如果输出文件已经存在了,重定向操作符会用新的文件数据覆盖已有文件。

```shell
$ who > test6
$ cat test6
user	pts/0	Feb 10 17:55
```

​	现在test6文件的内容就是 who 命令的输出。

​	有时,你可能并不想覆盖文件原有内容,而是想要将命令的输出追加到已有文件中,比如你正在创建一个记录系统上某个操作的日志文件。在这种情况下,可以用双大于号(>>)来追加数据。

```shell
$ date >> test6
$ cat test6
user	pts/0	Feb 10 17:55
Thu Feb 10 18:02:14 EDT 2014
```

​	test6文件仍然包含早些时候 who 命令的数据,现在又加上了来自 date 命令的输出。

#### 11.5.2 输入重定向

​	输入重定向和输出重定向正好相反。输入重定向将文件的内容重定向到命令,而非将命令的输出重定向到文件。

​	输入重定向符号是小于号(<):

```shell
command < inputfile
```

​	一个简单的记忆方法就是:在命令行上,命令总是在左侧,而重定向符号“指向”数据流动的方向。小于号说明数据正在从输入文件流向命令。

​	这里有个和 wc 命令一起使用输入重定向的例子。

```shell
$ wc < test6
	2	11	60
```

​	wc 命令可以对对数据中的文本进行计数。默认情况下,它会输出3个值:

- 文本的行数
- 文本的词数
- 文本的字节数

​	通过将文本文件重定向到 wc 命令,你立刻就可以得到文件中的行、词和字节的计数。这个例子说明test6文件有2行、11个单词以及60字节。

​	还有另外一种输入重定向的方法,称为内联输入重定向(inline input redirection)。这种方法无需使用文件进行重定向,只需要在命令行中指定用于输入重定向的数据就可以了。乍看一眼,这可能有点奇怪,但有些应用会用到这种方式(参见11.7节)。

​	内联输入重定向符号是远小于号(<<)。除了这个符号,你必须指定一个文本标记来划分输入数据的开始和结尾。任何字符串都可作为文本标记,但在数据的开始和结尾文本标记必须一致。

```shell
command << marker
data
marker
```

​	在命令行上使用内联输入重定向时,shell会用 PS2 环境变量中定义的次提示符(参见第6章)来提示输入数据。下面是它的使用情况。

```shell
[kato@aragne shell]$ wc << EOF
> Hello world
> i am a handsome boy
> EOF
 2  7 32
```

​	次提示符会持续提示,以获取更多的输入数据,直到你输入了作为文本标记的那个字符串。wc 命令会对内联输入重定向提供的数据进行行、词和字节计数。

### 11.6 管道

​	有时需要将一个命令的输出作为另一个命令的输入。这可以用重定向来实现,只是有些笨拙。

```shell
$ rpm -qa > rpm.list
$ sort < rpm.list
abrt-1.1.14-1.fc14.i686
abrt-addon-ccpp-1.1.14-1.fc14.i686
abrt-addon-kerneloops-1.1.14-1.fc14.i686
abrt-addon-python-1.1.14-1.fc14.i686
abrt-desktop-1.1.14-1.fc14.i686
abrt-gui-1.1.14-1.fc14.i686
abrt-libs-1.1.14-1.fc14.i686
abrt-plugin-bugzilla-1.1.14-1.fc14.i686
abrt-plugin-logger-1.1.14-1.fc14.i686
abrt-plugin-runapp-1.1.14-1.fc14.i686
acl-2.2.49-8.fc14.i686
[...]
```

​	rpm 命令通过Red Hat包管理系统(RPM)对系统(比如上例中的Fedora系统)上安装的软件包进行管理。配合 -qa 选项使用时,它会生成已安装包的列表,但这个列表并不会遵循某种特定的顺序。如果你在查找某个或某组特定的包,想在 rpm 命令的输出中找到就比较困难了。

​	通过标准输出重定向, rpm 命令的输出被重定向到了文件 rpm.list。命令完成后,rpm.list保存着系统中所有已安装的软件包列表。接下来,输入重定向将rpm.list文件的内容发送给 sort 命令,该命令按字母顺序对软件包名称进行排序。

​	这种方法的确管用,但仍然是一种比较繁琐的信息生成方式。我们用不着将命令输出重定向到文件中,可以将其直接重定向到另一个命令。这个过程叫作管道连接(piping)。

​	和命令替换所用的反引号(`)一样,管道符号在shell编程之外也很少用到。该符号由两个竖线构成,一个在另一个上面。然而管道符号的印刷体通常看起来更像是单个竖线(|)。在美式键盘上,它通常和反斜线(\)位于同一个键。管道被放在命令之间,将一个命令的输出重定向到另一个命令中:

```shell
command1 | command2
```

​		不要以为由管道串起的两个命令会依次执行。Linux系统实际上会同时运行这两个命令,在系统内部将它们连接起来。在第一个命令产生输出的同时,输出会被立即送给第二个命令。数据传输不会用到任何中间文件或缓冲区。

​	现在,可以利用管道将 rpm 命令的输出送入 sort 命令来产生结果。

```shell
$ rpm -qa | sort
abrt-1.1.14-1.fc14.i686
abrt-addon-ccpp-1.1.14-1.fc14.i686
abrt-addon-kerneloops-1.1.14-1.fc14.i686
abrt-addon-python-1.1.14-1.fc14.i686
abrt-desktop-1.1.14-1.fc14.i686
abrt-gui-1.1.14-1.fc14.i686
abrt-libs-1.1.14-1.fc14.i686
abrt-plugin-bugzilla-1.1.14-1.fc14.i686
abrt-plugin-logger-1.1.14-1.fc14.i686
abrt-plugin-runapp-1.1.14-1.fc14.i686
acl-2.2.49-8.fc14.i686
```

​	除非你的眼神特别好,否则可能根本来不及看清楚命令的输出。由于管道操作是实时运行的,所以只要 rpm 命令一输出数据, sort 命令就会立即对其进行排序。等到 rpm 命令输出完数据, sort命令就已经将数据排好序并显示了在显示器上。

​	可以在一条命令中使用任意多条管道。可以持续地将命令的输出通过管道传给其他命令来细化操作。

​	在这个例子中, sort 命令的输出会一闪而过,所以可以用一条文本分页命令(例如 less 或more )来强行将输出按屏显示

```shell
$ rpm -qa | sort | more
```

​	这行命令序列会先执行 rpm 命令,将它的输出通过管道传给 sort 命令,然后再将 sort 的输出通过管道传给 more 命令来显示,在显示完一屏信息后停下来。这样你就可以在继续处理前停下来阅读显示器上显示的信息。

​	如果想要更别致点,也可以搭配使用重定向和管道来将输出保存到文件中。

```shell
$ rpm -qa | sort > rpm.list
$ more rpm.list
abrt-1.1.14-1.fc14.i686
abrt-addon-ccpp-1.1.14-1.fc14.i686
abrt-addon-kerneloops-1.1.14-1.fc14.i686
abrt-addon-python-1.1.14-1.fc14.i686
abrt-desktop-1.1.14-1.fc14.i686
abrt-gui-1.1.14-1.fc14.i686
abrt-libs-1.1.14-1.fc14.i686
abrt-plugin-bugzilla-1.1.14-1.fc14.i686
abrt-plugin-logger-1.1.14-1.fc14.i686
abrt-plugin-runapp-1.1.14-1.fc14.i686
acl-2.2.49-8.fc14.i686
```

​	不出所料,rpm.list文件中的数据现在已经排好序了。

​	到目前为止,管道最流行的用法之一是将命令产生的大量输出通过管道传送给 more 命令。这对 ls 命令来说尤为常见。

### 11.7 执行数学运算

​	另一个对任何编程语言都很重要的特性是操作数字的能力。遗憾的是,对shell脚本来说,这个处理过程会比较麻烦。在shell脚本中有两种途径来进行数学运算。

#### 11.7.1 expr 命令

​	最开始,Bourne shell提供了一个特别的命令用来处理数学表达式。 expr 命令允许在命令行上处理数学表达式,但是特别笨拙。

```shell
$ expr 1 + 5
6
```

​	expr 命令能够识别少数的数学和字符串操作符,见表11-1。

![表11-1](https://github.com/katoluo/LinuxCommandLineAndShellScriptingBible/raw/master/chap11/images/%E8%A1%A811-1.png)

​	尽管标准操作符在 expr 命令中工作得很好,但在脚本或命令行上使用它们时仍有问题出现。许多 expr 命令操作符在shell中另有含义(比如星号)。当它们出现在在 expr 命令中时,会得到一些诡异的结果。

```shell
$ expr 5 * 2
expr: syntax error
$
```

​	要解决这个问题,对于那些容易被shell错误解释的字符,在它们传入 expr 命令之前,需要使用shell的转义字符(反斜线)将其标出来。

```shell
$ expr 5 \* 2
10
$
```

​	现在,麻烦才刚刚开始!在shell脚本中使用 expr 命令也同样复杂:

```shell
$ cat test6
#!/bin/bash
# An example of using the expr command
var1=10
var2=20
var3=$(expr $var2 / $var1)
echo The result is $var3
```

​	要将一个数学算式的结果赋给一个变量,需要使用命令替换来获取 expr 命令的输出:

```shell
$ chmod u+x test6
$ ./test6
The result is 2
$
```

​	幸好bash shell有一个针对处理数学运算符的改进,你将会在下一节中看到。

#### 11.7.2 使用方括号

​	bash shell为了保持跟Bourne shell的兼容而包含了 expr 命令,但它同样也提供了一种更简单的方法来执行数学表达式。在bash中,在将一个数学运算结果赋给某个变量时,可以用美元符和方括号( $[ operation ] )将数学表达式围起来。

```shell
$ var1=$[1 + 5]
$ echo $var1
6
$ var2=$[$var1 * 2]
$ echo $var2
12
$
```

​	用方括号执行shell数学运算比用 expr 命令方便很多。这种技术也适用于shell脚本。

```shell
$ cat test7
#!/bin/bash
var1=100
var2=50
var3=45
var4=$[$var1 * ($var2 - $var3)]
echo The final result is $var4
$
```

​	运行这个脚本会得到如下输出。

```shell
$ chmod u+x test7
$ ./test7
The final result is 500
$
```

​	同样,注意在使用方括号来计算公式时,不用担心shell会误解乘号或其他符号。shell知道它不是通配符,因为它在方括号内。

​	在bash shell脚本中进行算术运算会有一个主要的限制。请看下例:

```shell
$ cat test8
#!/bin/bash
var1=100
var2=45
var3=$[$var1 / $var2]
echo The final result is $var3
$
```

​	现在,运行一下,看看会发生什么:

```shell
$ chmod u+x test8
$ ./test8
The final result is 2
$
```

​	bash shell数学运算符只支持整数运算。若要进行任何实际的数学计算,这是一个巨大的限制。

***

**说明**	z shell(zsh)提供了完整的浮点数算术操作。如果需要在shell脚本中进行浮点数运算,可以考虑看看z shell(将在第23章中讨论)。

***

#### 11.7.3 浮点解决方案

​	有几种解决方案能够克服bash中数学运算的整数限制。最常见的方案是用内建的bash计算器,叫作 bc 。

​	**1. bc的基本用法**

​	bash计算器实际上是一种编程语言,它允许在命令行中输入浮点表达式,然后解释并计算该	表达式,最后返回结果。bash计算器能够识别:

- 数字(整数和浮点数)
- 变量(简单变量和数组)
- 注释(以#或C语言中的 /* */ 开始的行)
- 表达式
- 编程语句(例如 if-then 语句)
- 函数

​	可以在shell提示符下通过 bc 命令访问bash计算器:

```shell
[kato@aragne shell]$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
12 * 5.4
64.8
3.156 * (3 + 5)
25.248
quit
```

​	这个例子一开始输入了表达式 12 * 5.4 。bash计算器返回了计算结果。随后每个输入到计算器的表达式都会被求值并显示出结果。要退出bash计算器,你必须输入 quit 。

​	浮点运算是由内建变量 scale 控制的。必须将这个值设置为你希望在计算结果中保留的小数位数,否则无法得到期望的结果。

```shell
[kato@aragne shell]$ bc -q
3.3 / 4
0
scale = 3
3.3 / 4
.825
quit
```

​	scale 变量的默认值是 0 。在 scale 值被设置前,bash计算器的计算结果不包含小数位。在将其值设置成 4 后,bash计算器显示的结果包含四位小数。 -q 命令行选项可以不显示bash计算器冗长的欢迎信息。

​	除了普通数字,bash计算器还能支持变量。

```shell
$ bc -q
var1=10
var1 * 4
40
var2 = var1 / 5
print var2
2
quit
$
```

​	变量一旦被定义,你就可以在整个bash计算器会话中使用该变量了。 print 语句允许你打印变量和数字。

​	**2. 在脚本中使用bc**

​	现在你可能想问bash计算器是如何在shell脚本中帮助处理浮点运算的。还记得命令替换吗?是的,可以用命令替换运行 bc 命令,并将输出赋给一个变量。基本格式如下:

```shell
variable=$(echo "options; expression" | bc)
```

​	第一部分 options 允许你设置变量。如果你需要不止一个变量,可以用分号将其分开。expression 参数定义了通过 bc 执行的数学表达式。这里有个在脚本中这么做的例子。

```shell
$ cat test9
#!/bin/bash
var1=$(echo "scale=4; 3.44 / 5" | bc)
echo The answer is $var1
$
```

​	这个例子将 scale 变量设置成了四位小数,并在 expression 部分指定了特定的运算。运行这个脚本会产生如下输出。

```shell
$ chmod u+x test9
$ ./test9
The answer is .6880
$
```

​	太好了!现在你不会再只能用数字作为表达式值了。也可以用shell脚本中定义好的变量。

```shell
#!/bin/bash
var1=100
var2=45
var3=$(echo "scale=4; $var1 / $var2" | bc)
echo The answer for this is $var3
$
```

​	脚本定义了两个变量,它们都可以用在 expression 部分,然后发送给 bc 命令。别忘了用美元符表示的是变量的值而不是变量自身。这个脚本的输出如下。

```shell
$ ./test10
The answer for this is 2.2222
$
```

​	当然,一旦变量被赋值,那个变量也可以用于其他运算。

```shell
$ cat test11
#!/bin/bash
var1=20
var2=3.14159
var3=$(echo "scale=4; $var1 * $var1" | bc)
var4=$(echo "scale=4; $var3 * $var2" | bc)
echo The final result is $var4
$
```

​	这个方法适用于较短的运算,但有时你会涉及更多的数字。如果需要进行大量运算,在一个命令行中列出多个表达式就会有点麻烦。

​	有一个方法可以解决这个问题。 bc 命令能识别输入重定向,允许你将一个文件重定向到 bc命令来处理。但这同样会叫人头疼,因为你还得将表达式存放到文件中。

​	最好的办法是使用内联输入重定向,它允许你直接在命令行中重定向数据。在shell脚本中,你可以将输出赋给一个变量。

```shell
variable=$(bc << EOF
options
statements
expressions
EOF
)
```

​	EOF 文本字符串标识了内联重定向数据的起止。记住,仍然需要命令替换符号将 bc 命令的输出赋给变量。

​	现在可以将所有bash计算器涉及的部分都放到同一个脚本文件的不同行。下面是在脚本中使用这种技术的例子。

```shell
$ cat test12
#!/bin/bash
var1=10.46
var2=43.67
var3=33.2
var4=71
var5=$(bc << EOF
scale = 4
a1 = ( $var1 * $var2)
b1 = ($var3 * $var4)
a1 + b1
EOF
)
echo The final answer for this mess is $var5
$
```

​	将选项和表达式放在脚本的不同行中可以让处理过程变得更清晰,提高易读性。 EOF 字符串标识了重定向给 bc 命令的数据的起止。当然,必须用命令替换符号标识出用来给变量赋值的命令。

​	你还会注意到,在这个例子中,你可以在bash计算器中赋值给变量。这一点很重要:在bash计算器中创建的变量只在bash计算器中有效,不能在shell脚本中使用。

### 11.8 退出脚本

​	迄今为止所有的示例脚本中,我们都是突然停下来的。运行完最后一条命令时,脚本就结束了。其实还有另外一种更优雅的方法可以为脚本划上一个句号。

​	shell中运行的每个命令都使用退出状态码(exit status)告诉shell它已经运行完毕。退出状态码是一个0~255的整数值,在命令结束运行时由命令传给shell。可以捕获这个值并在脚本中使用。

#### 11.8.1 查看退出状态码

​	Linux提供了一个专门的变量 $? 来保存上个已执行命令的退出状态码。

​	对于需要进行检查的命令,必须在其运行完毕后立刻查看或使用 $? 变量。它的值会变成由shell所执行的最后一条命令的退出状态码。

```shell
$ date
Sat Jan 15 10:01:30 EDT 2014
$ echo $?
0
$
```

​	按照惯例,一个成功结束的命令的退出状态码是 0 。如果一个命令结束时有错误,退出状态码就是一个正数值。

```shell
$ asdfg
-bash: asdfg: command not found
$ echo $?
127
$
```

​	无效命令会返回一个退出状态码 127 。 Linux错误退出状态码没有什么标准可循,但有一些可用的参考,如表11-2所示。

![表11-2](https://github.com/katoluo/LinuxCommandLineAndShellScriptingBible/raw/master/chap11/images/%E8%A1%A811-2.png)

​	退出状态码 126 表明用户没有执行命令的正确权限。

```shell
$ ./myprog.c
-bash: ./myprog.c: Permission denied
$ echo $?
126
$
```

​	另一个会碰到的常见错误是给某个命令提供了无效参数。

```shell
$ date %t
date: invalid date '%t'
$ echo $?
1
$
```

​	这会产生一般性的退出状态码 1 ,表明在命令中发生了未知错误。

#### 11.8.2 exit 命令

​	默认情况下,shell脚本会以脚本中的最后一个命令的退出状态码退出。

​	你可以改变这种默认行为,返回自己的退出状态码。 exit 命令允许你在脚本结束时指定一个退出状态码。

```shell
$ cat test13
#!/bin/bash
# testing the exit status
var1=10
var2=30
var3=$[$var1 + $var2]
echo The answer is $var3
exit 5
$
```

​	当查看脚本的退出码时,你会得到作为参数传给 exit 命令的值。

```shell
$ chmod u+x test13
$ ./test13
The answer is 40
$ echo $?
5
$
```

​	也可以在 exit 命令的参数中使用变量。

```shell
$ cat test14
#!/bin/bash
# testing the exit status
var1=10
var2=30
var3=$[$var1 + $var2]
exit $var3
$
```

​	当你运行这个命令时,它会产生如下退出状态。

```shell
$ chmod u+x test14
$ ./test14
$ echo $?
40
$
```

​	你要注意这个功能,因为退出状态码最大只能是 255 。看下面例子中会怎样。

```shell
$ cat test14b
#!/bin/bash
# testing the exit status
var1=10
var2=30
var3=$[$var1 * $var2]
echo The value is $var3
exit $var3
$
```

​	现在运行它的话,会得到如下输出。

```shell
$ ./test14b
The value is 300
$ echo $?
44
$
```

​	退出状态码被缩减到了0~255的区间。shell通过模运算得到这个结果。一个值的模就是被除后的余数。最终的结果是指定的数值除以256后得到的余数。在这个例子中,指定的值是 300 (返回值),余数是44,因此这个余数就成了最后的状态退出码。

​	在第12章中,你会了解到如何用 if-then 语句来检查某个命令返回的错误状态,以便知道命令是否成功。