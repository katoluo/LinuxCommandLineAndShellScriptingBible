## 第12章 使用结构化命令

​	在第11章给出的那些shell脚本里, shell按照命令在脚本中出现的顺序依次进行处理。对顺序操作来说,这已经足够了,因为在这种操作环境下,你想要的就是所有的命令按照正确的顺序执行。然而,并非所有程序都如此操作。

​	许多程序要求对shell脚本中的命令施加一些逻辑流程控制。有一类命令会根据条件使脚本跳过某些命令。这样的命令通常称为结构化命令(structured command)。

​	结构化命令允许你改变程序执行的顺序。在bash shell中有不少结构化命令,我们会逐个研究。本章来看一下 if-then 和 case 语句。

### 12.1 使用if-then语句

​	最基本的结构化命令就是 if-then 语句。 if-then 语句有如下格式。

```shell
if command
then
	command
fi
```

​	如果你在用其他编程语言的 if-then 语句,这种形式可能会让你有点困惑。在其他编程语言中, if 语句之后的对象是一个等式,这个等式的求值结果为 TRUE 或 FALSE 。但bash shell的 if 语句并不是这么做的。

​	bash shell的 if 语句会运行 if 后面的那个命令。如果该命令的退出状态码(参见第11章)是 0(该命令成功运行),位于 then 部分的命令就会被执行。如果该命令的退出状态码是其他值, then部分的命令就不会被执行,bash shell会继续执行脚本中的下一个命令。 fi 语句用来表示 if-then语句到此结束。

​	这里有个简单的例子可解释这个概念。

```shell
[kato@aragne shell]$ cat test1.sh
#!/bin/bash
# testing the if statement
if pwd
then
  echo "It worked"
fi
```

​	这个脚本在 if 行采用了 pwd 命令。如果命令成功结束, echo 语句就会显示该文本字符串。在命令行运行该脚本时,会得到如下结果。

```shell
[kato@aragne shell]$ ./test1.sh
/home/kato/Workspace/shell
It worked
```

​	shell执行了 if 行中的 pwd 命令。由于退出状态码是 0 ,它就又执行了 then 部分的 echo 语句。下面是另外一个例子。

```shell
[kato@aragne shell]$ cat test2.sh
#!/bin/bash
# testing a bad command
if IamNotaCommand
then
  echo "It worked"
fi
echo "We are outside the if statement"
[kato@aragne shell]$ chmod 744 test2.sh
[kato@aragne shell]$ ./test2.sh
./test2.sh:行3: IamNotaCommand：未找到命令
We are outside the if statement
```

​	在这个例子中,我们在 if 语句行故意放了一个不能工作的命令。由于这是个错误的命令,所以它会产生一个非零的退出状态码,且bash shell会跳过 then 部分的 echo 语句。还要注意,运行if 语句中的那个错误命令所生成的错误消息依然会显示在脚本的输出中。有时你可能不想看到错误信息。第15章将会讨论如何避免这种情况。

***

**说明**	你可能在有些脚本中看到过 if-then 语句的另一种形式:

```shell
if command; then
	command
fi
```

通过把分号放在待求值的命令尾部,就可以将 then 语句放在同一行上了,这样看起来更像其他编程语言中的 if-then 语句。

***

​	在 then 部分,你可以使用不止一条命令。可以像在脚本中的其他地方一样在这里列出多条命令。bash shell会将这些命令当成一个块,如果 if 语句行的命令的退出状态值为 0 ,所有的命令都会被执行;如果 if 语句行的命令的退出状态不为 0 ,所有的命令都会被跳过。

```shell
[kato@aragne shell]$ cat test3.sh
#!/bin/bash
# testing multiple commands in the then section
#
testusr=kato
#
if grep $testusr /etc/passwd; then
  echo "This is my first command"
  echo "This is my second command"
  echo "I can even put in other command besides echo:"
  ls -a /home/$testusr/.b*
fi
```

​	if 语句行使用 grep 命令在/etc/passwd文件中查找某个用户名当前是否在系统上使用。如果有用户使用了那个登录名,脚本会显示一些文本信息并列出该用户HOME目录的bash文件。

```shell
[kato@aragne shell]$ chmod u+x test3.sh
[kato@aragne shell]$ ./test3.sh
kato:x:1000:1000:kato:/home/kato:/bin/bash
This is my first command
This is my second command
I can even put in other command besides echo:
/home/kato/.bash_history  /home/kato/.bash_profile
/home/kato/.bash_logout   /home/kato/.bashrc
```

​	但是,如果将 testuser 变量设置成一个系统上不存在的用户,则什么都不会显示。

```shell
[kato@aragne shell]$ cat test3.sh
#!/bin/bash
# testing multiple commands in the then section
#
testusr=NoSuchUser
#
if grep $testusr /etc/passwd; then
  echo "This is my first command"
  echo "This is my second command"
  echo "I can even put in other command besides echo:"
  ls -a /home/$testusr/.b*
fi
$
$./test3.sh
$
```

​	看起来也没什么新鲜的。如果在这里显示的一些消息可说明这个用户名在系统中未找到,这样可能就会显得更友好。是的,可以用 if-then 语句的另外一个特性来做到这一点。

### 12.2 if-then-else语句

​	在 if-then 语句中,不管命令是否成功执行,你都只有一种选择。如果命令返回一个非零退出状态码,bash shell会继续执行脚本中的下一条命令。在这种情况下,如果能够执行另一组命令就好了。这正是 if-then-else 语句的作用。

​	if-then-else 语句在语句中提供了另外一组命令。

```shell
if command
then
	commands
else
	commands
fi
```

​	当 if 语句中的命令返回退出状态码 0 时, then 部分中的命令会被执行,这跟普通的 if-then语句一样。当 if 语句中的命令返回非零退出状态码时,bash shell会执行 else 部分中的命令。

​	现在可以复制并修改测试脚本来加入 else 部分。

```shell
$ cp test3.sh test4.sh
$
$ nano test4.sh
$
$ cat test4.sh
#!/bin/bash
# testing the else section
#
testuser=NoSuchUser
#
if grep $testuser /etc/passwd
then
	echo "The bash files for user $testuser are:"
	ls -a /home/$testuser/.b*
	echo
else
	echo "The user $testuser does not exist on this system."
	echo
fi
$
$ ./test4.sh
The user NoSuchUser does not exist on this system.
$
```

​	这样就更友好了。跟 then 部分一样, else 部分可以包含多条命令。 fi 语句说明 else 部分结束了。

### 12.3 嵌套if

​	有时你需要检查脚本代码中的多种条件。对此,可以使用嵌套的 if-then 语句。

​	要检查/etc/passwd文件中是否存在某个用户名以及该用户的目录是否尚在,可以使用嵌套的if-then 语句。嵌套的 if-then 语句位于主 if-then-else 语句的 else 代码块中。

```shell
$ ls -d /home/NoSuchUser/
/home/NoSuchUser/
$
$ cat test5.sh
#!/bin/bash
# Testing nested ifs
#
testuser=NoSuchUser
#
if grep $testuser /etc/passwd
then
	echo "The user $testuser exists on this system."
else
	echo "The user $testuser does not exist on this system."
	if ls -d /home/$testuser/
	then
		echo "However, $testuser has a directory."
	fi
fi
$
$ ./test5.sh
The user NoSuchUser does not exist on this system.
/home/NoSuchUser/
However, NoSuchUser has a directory.
$
```

​	这个脚本准确无误地发现,尽管登录名已经从 /etc/passwd 中删除了,但是该用户的目录仍然存在。在脚本中使用这种嵌套 if-then 语句的问题在于代码不易阅读,很难理清逻辑流程。

​	可以使用 else 部分的另一种形式: elif 。这样就不用再书写多个 if-then 语句了。 elif 使用另一个 if-then 语句延续 else 部分。

```shell
if command1
then
	commands
elif command2
then
	more commands
fi
```

​	elif 语句行提供了另一个要测试的命令,这类似于原始的 if 语句行。如果 elif 后命令的退出状态码是 0 ,则bash会执行第二个 then 语句部分的命令。使用这种嵌套方法,代码更清晰,逻辑更易懂。

```shell
$ cat test5.sh
#!/bin/bash
# Testing nested ifs - use elif
#
testuser=NoSuchUser
#
if grep $testuser /etc/passwd
then
	echo "The user $testuser exists on this system."
#
elif ls -d /home/$testuser
then
	echo "The user $testuser does not exist on this system."
	echo "However, $testuser has a directory."
#
fi
$
$ ./test5.sh
/home/NoSuchUser
The user NoSuchUser does not exist on this system.
However, NoSuchUser has a directory.
$
```

​	甚至可以更进一步,让脚本检查拥有目录的不存在用户以及没有拥有目录的不存在用户。这可以通过在嵌套 elif 中加入一个 else 语句来实现。

```shell
$ cat test5.sh
#!/bin/bash
# Testing nested ifs - use elif & else
#
testuser=NoSuchUser
#
if grep $testuser /etc/passwd
then
	echo "The user $testuser exists on this system."
#
elif ls -d /home/$testuser
then
	echo "The user $testuser does not exist on this system."
	echo "However, $testuser has a directory."
#
else
	echo "The user $testuser does not exist on this system."
	echo "And, $testuser does not have a directory."
fi
$
$ ./test5.sh
/home/NoSuchUser
The user NoSuchUser does not exist on this system.
However, NoSuchUser has a directory.
$
$ sudo rmdir /home/NoSuchUser
[sudo] password for Christine:
$
$ ./test5.sh
ls: cannot access /home/NoSuchUser: No such file or directory
The user NoSuchUser does not exist on this system.
And, NoSuchUser does not have a directory.
$
```

​	在/home/NoSuchUser目录被删除之前,这个测试脚本执行的是 elif 语句,返回零值的退出状态。因此 elif 的 then 代码块中的语句得以执行。删除了/home/NoSuchUser目录之后, elif 语句返回的是非零值的退出状态。这使得 elif 块中的 else 代码块得以执行。

​	可以继续将多个 elif 语句串起来,形成一个大的 if-then-elif 嵌套组合。

​	每块命令都会根据命令是否会返回退出状态码 0 来执行。记住, bash shell会依次执行 if 语句,只有第一个返回退出状态码 0 的语句中的 then 部分会被执行，即使elif中的命令的返回退出码为0也不会执行then部分。

​	尽管使用了 elif 语句的代码看起来更清晰,但是脚本的逻辑仍然会让人犯晕。在12.7节,你会看到如何使用 case 命令代替 if-then 语句的大量嵌套。

### 12.4 test命令

​	到目前为止,在 if 语句中看到的都是普通shell命令。你可能想问, if-then 语句是否能测试命令退出状态码之外的条件。

​	答案是不能。但在bash shell中有个好用的工具可以帮你通过 if-then 语句测试其他条件。

​	test 命令提供了在 if-then 语句中测试不同条件的途径。如果 test 命令中列出的条件成立,test 命令就会退出并返回退出状态码 0 。这样 if-then 语句就与其他编程语言中的 if-then 语句以类似的方式工作了。如果条件不成立, test 命令就会退出并返回非零的退出状态码,这使得if-then 语句不会再被执行。

​	test 命令的格式非常简单。

```shell
test condition
```

​	condition 是 test 命令要测试的一系列参数和值。当用在 if-then 语句中时, test 命令看起来是这样的。

```shell
if test condition
then
	commands
fi
```

​	如果不写 test 命令的 condition 部分,它会以非零的退出状态码退出,并执行 else 语句块。

```shell
$ cat test6.sh
#!/bin/bash
# Testing the test command
#
if test
then
	echo "No expression returns a True"
else
	echo "No expression returns a False"
fi
$
$ ./test6.sh
No expression returns a False
$
```

​	当你加入一个条件时, test 命令会测试该条件。例如,可以使用 test 命令确定变量中是否有内容。这只需要一个简单的条件表达式。

```bash
$ cat test6.sh
#!/bin/bash
# Testing the test command
#
my_variable="Full"
#
if test $my_variable
then
	echo "The $my_variable expression returns a True"
#
else
	echo "The $my_variable expression returns a False"
fi
$
$ ./test6.sh
The Full expression returns a True
$
```

​	变量 my_variable 中包含有内容( Full ),因此当 test 命令测试条件时,返回的退出状态为 0 。这使得 then 语句块中的语句得以执行。

​	如你所料,如果该变量中没有包含内容,就会出现相反的情况。

```shell
$ cat test6.sh
#!/bin/bash
# Testing the test command
#
my_variable=""
#
if test $my_variable
then
	echo "The $my_variable expression returns a True"
#
else
	echo "The $my_variable expression returns a False"
fi
$
$ ./test6.sh
The expression returns a False
$
```

​	bash shell提供了另一种条件测试方法,无需在 if-then 语句中声明 test 命令。

```bash
if [ condition ]
then
	commands
fi
```

​	方括号定义了测试条件。注意,第一个方括号之后和第二个方括号之前必须加上一个空格,否则就会报错。

​	test 命令可以判断三类条件:

- 数值比较
- 字符串比较
- 文件比较

​	后续章节将会介绍如何在 if-then 语句中使用这些条件测试。

#### 12.4.1 数值比较

​	使用 test 命令最常见的情形是对两个数值进行比较。表12-1列出了测试两个值时可用的条件参数。

![表12-1]()

​	数值条件测试可以用在数字和变量上。这里有个例子。

```bash
$ cat numeric_test.sh
#!/bin/bash
# Using numeric test evaluations
#
value1=10
value2=11
#
if [ $value1 -gt 5 ]
then
	echo "The test value $value1 is greater than 5"
fi
#
if [ $value1 -eq $value2 ]
then
	echo "The values are equal"
else
	echo "The values are different"
fi
#
$
```

​	第一个条件测试：`if [ $value1 -gt 5 ]`，测试变量value1的值是否大于5。

​	第二个条件测试：`if [ $value1 -eq $value2 ]`，测量变量value1的值是否和变量value2的值相等。两个数值条件测试的结果和预想一致。

```bash
$ ./numeric_test.sh
The test value 10 is greater than 5
The values are different
$
```

​	但是涉及浮点值时,数值条件测试会有一个限制。

```bash
$ cat floating_point_test.sh
#!/bin/bash
# Using floating point numbers in test evaluations
#
value1=5.555
#
echo "The test value is $value1"
#
if [ $value1 -gt 5 ]
then
	echo "The test value $value1 is greater than 5"
fi
#
$ ./floating_point_test.sh
The test value is 5.555
./floating_point_test.sh: line 8:
[: 5.555: integer expression expected
$
```

​	此例,变量 value1 中存储的是浮点值。接着,脚本对这个值进行了测试。显然这里出错了。记住,bash shell只能处理整数。如果你只是要通过 echo 语句来显示这个结果,那没问题。但是,在基于数字的函数中就不行了,例如我们的数值测试条件。最后一行就说明我们不能在test 命令中使用浮点值。

#### 12.4.2 字符串比较

​	条件测试还允许比较字符串值。比较字符串比较烦琐,你马上就会看到。表12-2列出了可用的字符串比较功能。

![表12-2]()

​	下面几节将会详细介绍不同的字符串比较功能。

​	**1. 字符串相等性**

​	字符串的相等和不等条件不言自明,很容易看出两个字符串值是否相同。

```shell
$ cat test7.sh
#!/bin/bash
# testing string equality
testuser=rich
#
if [ $USER = $testuser ]
then
	echo "Welcome $testuser"
fi
$
$ ./test7.sh
Welcome rich
$
```

​	字符串不等条件也可以判断两个字符串是否有相同的值。

```bash
$ cat test8.sh
#!/bin/bash
# testing string equality
testuser=baduser
#
if [ $USER != $testuser ]
then
	echo "This is not $testuser"
else
	echo "Welcome $testuser"
fi
$
$ ./test8.sh
This is not baduser
$
```

​	记住,在比较字符串的相等性时,比较测试会将所有的标点和大小写情况都考虑在内。

​	**2. 字符串顺序**

​	要测试一个字符串是否比另一个字符串大就是麻烦的开始。当要开始使用测试条件的大于或小于功能时,就会出现两个经常困扰shell程序员的问题:

- 大于号和小于号必须转义,否则shell会把它们当作重定向符号,把字符串值当作文件名;
- 大于和小于顺序和 sort 命令所采用的不同。

​	在编写脚本时,第一条可能会导致一个不易察觉的严重问题。下面的例子展示了shell脚本编程初学者时常碰到的问题。

```bash
$ cat badtest.sh
#!/bin/bash
# mis-using string comparisons
#
val1=baseball
val2=hockey
#
if [ $val1 > $val2 ]
then
	echo "$val1 is greater than $val2"
else
	echo "$val1 is less than $val2"
fi
$
$ ./badtest.sh
baseball is greater than hockey
$ ls -l hockey
-rw-r--r--	1 rich	rich	0 Sep 30 19:08 hockey
$
```

​	这个脚本中只用了大于号,没有出现错误,但结果是错的。脚本把大于号解释成了输出重定向(参见第15章)。因此,它创建了一个名为hockey的文件。由于重定向的顺利完成, test 命令返回了退出状态码 0 , if 语句便以为所有命令都成功结束了。

​	要解决这个问题,就需要正确转义大于号。

```bash
$ cat test9.sh
#!/bin/bash
# mis-using string comparisons
#
val1=baseball
val2=hockey
#
if [ $val1 \> $val2 ]
then
	echo "$val1 is greater than $val2"
else
	echo "$val1 is less than $val2"
fi
$
$ ./test9.sh
baseball is less than hockey
$
```

​	现在的答案已经符合预期的了。

​	第二个问题更细微,除非你经常处理大小写字母,否则几乎遇不到。 sort 命令处理大写字母的方法刚好跟 test 命令相反。让我们在脚本中测试一下这个特性。

```bash
$ cat test9b.sh
#!/bin/bash
# testing string sort order
val1=Testing
val2=testing
#
if [ $val1 \> $val2 ]
then
	echo "$val1 is greater than $val2"
else
	echo "$val1 is less than $val2"
fi
$
$ ./test9b.sh
Testing is less than testing
$
$ sort testfile
testing
Testing
$
```

​	在比较测试中,大写字母被认为是小于小写字母的。但 sort 命令恰好相反。当你将同样的字符串放进文件中并用 sort 命令排序时,小写字母会先出现。这是由各个命令使用的排序技术不同造成的。

​	比较测试中使用的是标准的ASCII顺序,根据每个字符的ASCII数值来决定排序结果。 sort命令使用的是系统的本地化语言设置中定义的排序顺序。对于英语,本地化设置指定了在排序顺序中小写字母出现在大写字母前。

​	**3. 字符串大小**

​	-n 和 -z 可以检查一个变量是否含有数据。

```bash
$ cat test10.sh
#!/bin/bash
# testing string length
val1=testing
val2=''
#
if [ -n $val1 ]
then
	echo "The string '$val1' is not empty"
else
	echo "The string '$val1' is empty"
fi
#
if [ -z $val2 ]
then
	echo "The string '$val2' is empty"
else
	echo "The string '$val2' is not empty"
fi
#
if [ -z $val3 ]
then
	echo "The string '$val3' is empty"
else
	echo "The string '$val3' is not empty"
fi
$
$ ./test10.sh
The string 'testing' is not
The string '' is empty
The string '' is empty
$
```

​	这个例子创建了两个字符串变量。 val1 变量包含了一个字符串, val2 变量包含的是一个空字符串。后续的比较如下:

```bash
if [ -n $val1 ] # 判断 val1 变量是否长度非0,而它的长度正好非0,所以 then 部分被执行了。
if [ -z $val2 ] # 判断 val2 变量是否长度为0,而它正好长度为0,所以 then 部分被执行了。
if [ -z $val3 ] # 判断 val3 变量是否长度为0。这个变量并未在shell脚本中定义过,所以它的字符串长度仍然为0,尽管它未被定义过。
```

***

**窍门**	空的和未初始化的变量会对shell脚本测试造成灾难性的影响。如果不是很确定一个变量的内容,最好在将其用于数值或字符串比较之前先通过 -n 或 -z 来测试一下变量是否含有值。

***

#### 12.4.3 文件比较

​	最后一类比较测试很有可能是shell编程中最为强大、也是用得最多的比较形式。它允许你测试Linux文件系统上文件和目录的状态。表12-3列出了这些比较。

![表12-3]()

​	这些测试条件使你能够在shell脚本中检查文件系统中的文件。它们经常出现在需要进行文件访问的脚本中。鉴于其使用广泛,我们来逐个看看。

​	**1. 检查目录**

​	-d 测试会检查指定的目录是否存在于系统中。如果你打算将文件写入目录或是准备切换到某个目录中,先进行测试总是件好事情。

```bash
$ cat test11.sh
#!/bin/bash
# Look before you leap
#
jump_directory=/home/arthur
#
if [ -d $jump_directory ]
then
	echo "The $jump_directory directory exists"
	cd $jump_directory
	ls
else
	echo "The $jump_directory directory does not exist"
fi
#
$
$ ./test11.sh
The /home/arthur directory does not exist
$
```

​	示例代码中使用了 -d 测试条件来检查 jump_directory 变量中的目录是否存在:若存在,就使用 cd 命令切换到该目录并列出目录中的内容;若不存在,脚本就输出一条警告信息,然后退出。

​	**2. 检查对象是否存在**

​	-e 比较允许你的脚本代码在使用文件或目录前先检查它们是否存在。

```bash
$ cat test12.sh
#!/bin/bash
# Check if either a directory or file exists
#
location=$HOME
file_name="sentinel"
#
if [ -e $location ]
then #Directory does exist
	echo "OK on the $location directory."
	echo "Now checking on the file, $file_name."
	#
	if [ -e $location/$file_name ]
	then #File does exist
		echo "OK on the filename"
		echo "Updating Current Date..."
		date >> $location/$file_name
	#
	else #File does not exist
		echo "File does not exist"
		echo "Nothing to update"
	fi
#
else
#Directory does not exist
	echo "The $location directory does not exist."
	echo "Nothing to update"
fi
#
$
$ ./test12.sh
OK on the /home/Christine directory.
Now checking on the file, sentinel.
File does not exist
Nothing to update
$
$ touch sentinel
$
$ ./test12.sh
OK on the /home/Christine directory.
Now checking on the file, sentinel.
OK on the filename
Updating Current Date...
$
```

​	第一次检查用 -e 比较来判断用户是否有$HOME目录。

​	如果有,接下来的 -e 比较会检查sentinel文件是否存在于$HOME目录中。如果不存在,shell脚本就会提示该文件不存在,不需要进行更新。

​	为确保更新操作能够正常进行,我们创建了sentinel文件,然后重新运行这个shell脚本。这一次在进行条件测试时,$HOME和sentinel文件都存在,因此当前日期和时间就被追加到了文件中。

​	**3. 检查文件**

​	-e 比较可用于文件和目录。要确定指定对象为文件,必须用 -f 比较。

```bash
$ cat test13.sh
#!/bin/bash
# Check if either a directory or file exists
#
item_name=$HOME
echo
echo "The item being checked: $item_name"
echo
#
if [ -e $item_name ]
then #Item does exist
	echo "The item, $item_name, does exist."
	echo "But is it a file?"
	echo
	#
	if [ -f $item_name ]
	then #Item is a file
		echo "Yes, $item_name is a file."
	#
	else #Item is not a file
		echo "No, $item_name is not a file."
	fi
#
else
#Item does not exist
	echo "The item, $item_name, does not exist."
	echo "Nothing to update"
fi
#
$ ./test13.sh
The item being checked: /home/Christine
The item, /home/Christine, does exist.
But is it a file?
No, /home/Christine is not a file.
$
```

​	这一小段脚本进行了大量的检查!它首先使用 -e 比较测试$HOME是否存在。如果存在,继续用 -f 来测试它是不是一个文件。如果它不是文件(当然不会是了),就会显示一条消息,表明这不是一个文件。

​	我们对变量 item_name 作了一个小小的修改,将目录$HOME替换成文件HOME/sentinel,结果就不一样了。

```bash
$ nano test13.sh
$
$ cat test13.sh
#!/bin/bash
# Check if either a directory or file exists
#
item_name=$HOME/sentinel
[...]
$
$ ./test13.sh
The item being checked: /home/Christine/sentinel
The item, /home/Christine/sentinel, does exist.
But is it a file?
Yes, /home/Christine/sentinel is a file.
$
```

​	这里只列出了脚本test13.sh的部分代码,因为只改变了脚本变量 item_name 的值。当运行这个脚本时,对$HOME/sentinel进行的 -f 测试所返回的退出状态码为 0 , then 语句得以执行,然后输出消息: Yes, /home/Christine/sentinel is a file。

​	**4. 检查是否可读**

​	在尝试从文件中读取数据之前,最好先测试一下文件是否可读。可以使用 -r 比较测试。

```bash
$ cat test14.sh
#!/bin/bash
# testing if you can read a file
pwfile=/etc/shadow
#
# first, test if the file exists, and is a file
if [ -f $pwfile ]
then
	# now test if you can read it
	if [ -r $pwfile ]
	then
		tail $pwfile
	else
		echo "Sorry, I am unable to read the $pwfile file"
	fi
else
	echo "Sorry, the file $file does not exist"
fi
$
$ ./test14.sh
Sorry, I am unable to read the /etc/shadow file
$
```

​	/etc/shadow文件含有系统用户加密后的密码,所以它对系统上的普通用户来说是不可读的。-r 比较确定该文件不允许进行读取,因此测试失败,bash shell执行了 if-then 语句的 else 部分。

​	**5. 检查空文件**

​	应该用 -s 比较来检查文件是否为空,尤其是在不想删除非空文件的时候。要留心的是,当-s 比较成功时,说明文件中有数据。

```bash
$ cat test15.sh
#!/bin/bash
#testing if a file is empty
#
file_name=$HOME/sentinel
#
if [ -f $file_name ]
then
  if [ -s $file_name ]
  then
    echo "The $file_name file exists and has data in it."
    echo "Will not remove this file."
  #
  else
    echo "The $file_name file exists, but is empty."
    echo "Deleting empty file..."
    rm $file_name
  fi
else
  echo "File, $file_name, does not exists."
fi
$ ls -l $HOME/sentinel
-rw-rw-r--. 1 Christine Christine 29 Jun 25 05:32 /home/Christine/sentinel
$
$ ./test15.sh
The /home/Christine/sentinel file exists and has data in it.
Will not remove this file.
$
```

​	-f 比较测试首先测试文件是否存在。如果存在,由 -s 比较来判断该文件是否为空。空文件会被删除。可以从 ls –l 的输出中看出sentinel并不是空文件,因此脚本并不会删除它。

​	**6. 检查是否可写**

​	-w 比较会判断你对文件是否有可写权限。脚本test16.sh只是脚本test13.sh的修改版。现在不单检查 item_name 是否存在、是否为文件,还会检查该文件是否有写入权限。

```bash
$ cat test16.sh
#!/bin/bash
# Check if a file is writable.
#
item_name=$HOME/sentinel
echo
echo "The item being checked: $item_name"
echo
[...]
	echo "Yes, $item_name is a file."
	echo "But is it writable?"
	echo
	#
	if [ -w $item_name ]
	then #Item is writable
		echo "Writing current time to $item_name"
		date +%H%M >> $item_name
	else #Item is not writable
		echo "Unable to write to $item_name"
	fi
# else #Item is not a file
	echo "No, $item_name is not a file."
fi
$
$ ls -l sentinel
-rw-rw-r--. 1 Christine Christine 0 Jun 27 05:38 sentinel
$
$ ./test16.sh
The item being checked: /home/Christine/sentinel
The item, /home/Christine/sentinel, does exist.
But is it a file?
Yes, /home/Christine/sentinel is a file.
But is it writable?
Writing current time to /home/Christine/sentinel
$
$ cat sentinel
0543
$
```

​	变量 item_name 被设置成$HOME/sentinel,该文件允许用户进行写入(有关文件权限的更多信息,请参见第7章)。因此当脚本运行时, -w 测试表达式会返回非零退出状态,然后执行 then代码块,将时间戳写入文件sentinel中。

​	如果使用 chmod 关闭文件sentinel的用户 写入权限, -w 测试表达式会返回非零的退出状态码,时间戳不会被写入文件。

```bash
$ chmod u-w sentinel
$
$ ls -l sentinel
-r--rw-r--. 1 Christine Christine 5 Jun 27 05:43 sentinel
$
$ ./test16.sh
The item being checked: /home/Christine/sentinel
The item, /home/Christine/sentinel, does exist.
But is it a file?
Yes, /home/Christine/sentinel is a file.
But is it writable?
Unable to write to /home/Christine/sentinel
$
```

​	chmod 命令可用来为读者再次回授写入权限。这会使得写入测试表达式返回退出状态码 0 ,并允许一次针对文件的写入尝试。

​	**7. 检查文件是否可运行**

​	-x 比较是判断特定文件是否有执行权限的一个简单方法。虽然可能大多数命令用不到它,但如果你要在shell脚本中运行大量脚本,它就能发挥作用。

```bash
$ cat test17.sh
#!/bin/bash
# testing file execution
#
if [ -x test16.sh ]
then
	echo "You can run the script: "
	./test16.sh
else
	echo "Sorry, you are unable to execute the script"
fi
$
$ ./test17.sh
You can run the script:
[...]
$
$ chmod u-x test16.sh
$
$ ./test17.sh
Sorry, you are unable to execute the script
$
```

​	这段示例shell脚本用 -x 比较来测试是否有权限执行 test16.sh 脚本。如果有权限,它会运行这个脚本。在首次成功运行 test16.sh 脚本后,更改文件的权限。这次, -x 比较失败了,因为你已经没有 test16.sh 脚本的执行权限了。

​	**8. 检查所属关系**

​	-O 比较可以测试出你是否是文件的属主。

```bash
#!/bin/bash
# check file ownership
#
if [ -O /etc/passwd ]
then
	echo "You are the owner of the /etc/passwd file"
else
	echo "Sorry, you are not the owner of the /etc/passwd file"
fi
$
$ ./test18.sh
Sorry, you are not the owner of the /etc/passwd file
$
```

​	这段脚本用 -O 比较来测试运行该脚本的用户是否是/etc/passwd文件的属主。这个脚本是运行在普通用户账户下的,所以测试失败了。

​	**9. 检查默认属组关系**

​	-G 比较会检查文件的默认组,如果它匹配了用户的默认组,则测试成功。由于 -G 比较只会检查默认组而非用户所属的所有组,这会叫人有点困惑。这里有个例子。

```bash
$ cat test19.sh
#!/bin/bash
# check file group test
#
if [ -G $HOME/testing ]
then
	echo "You are in the same group as the file"
else
	echo "The file is not owned by your group"
fi
$
$ ls -l $HOME/testing
-rw-rw-r-- 1 rich rich 58 2014-07-30 15:51 /home/rich/testing
$
$ ./test19.sh
You are in the same group as the file
$
$ chgrp sharing $HOME/testing
$
$ ./test19
The file is not owned by your group
$
```

​	第一次运行脚本时,$HOME/testing文件属于 rich 组,所以通过了 -G 比较。接下来,组被改成了 sharing 组,用户也是其中的一员。但是, -G 比较失败了,因为它只比较默认组,不会去比较其他的组。

### 12.5 复合条件测试

​	if-then 语句允许你使用布尔逻辑来组合测试。有两种布尔运算符可用:

- [ condition1 ] && [ condition2 ]
- [ condition1 ] || [ condition2 ]

​	第一种布尔运算使用 AND 布尔运算符来组合两个条件。要让 then 部分的命令执行,两个条件都必须满足。

​	第二种布尔运算使用 OR 布尔运算符来组合两个条件。如果任意条件为 TRUE , then 部分的命令就会执行。

​	下例展示了 AND 布尔运算符的使用。

```bash
$ cat test22.sh
#!/bin/bash
# testing compound comparisons
#
if [ -d $HOME ] && [ -w $HOME/testing ]
then
	echo "The file exists and you can write to it"
else
	echo "I cannot write to the file"
fi
$
$ ./test22.sh
I cannot write to the file
$
$ touch $HOME/testing
$
$ ./test22.sh
The file exists and you can write to it
$
```

​	使用 AND 布尔运算符时,两个比较都必须满足。第一个比较会检查用户的HOME目录是否存在。第二个比较会检查在用户的$HOME目录是否有个叫testing的文件,以及用户是否有该文件的写入权限。如果两个比较中的一个失败了, if 语句就会失败,shell就会执行 else 部分的命令。如果两个比较都通过了,则 if 语句通过,shell会执行 then 部分的命令。

### 12.6 if-then 高级特性

​	bash shell提供了两项可在 if-then 语句中使用的高级特性:

- 用于数学表达式的双括号
- 用于高级字符串处理功能的双方括号

#### 12.6.1 使用双括号

​	双括号命令允许你在比较过程中使用高级数学表达式。 test 命令只能在比较中使用简单的算术操作。双括号命令提供了更多的数学符号,这些符号对于用过其他编程语言的程序员而言并不陌生。双括号命令的格式如: `(( expression ))`

​	expression 可以是任意的数学赋值或比较表达式。除了 test 命令使用的标准数学运算符,表12-4列出了双括号命令中会用到的其他运算符。

![表12-4]()

​	可以在 if 语句中用双括号命令,也可以在脚本中的普通命令里使用来赋值。

```bash
$ cat test23.sh
#!/bin/bash
# using double parenthesis
#
val1=10
#
if (( $val1 ** 2 > 90 ))
then
	(( val2 = $val1 ** 2 ))
	echo "The square of $val1 is $val2"
fi
$
$ ./test23.sh
The square of 10 is 100
$
```

​	注意,不需要将双括号中表达式里的大于号转义。这是双括号命令提供的另一个高级特性。

#### 12.6.2 使用双方括号

​	双方括号命令提供了针对字符串比较的高级特性。双方括号命令的格式如下:`[[expression]]`

​	双方括号里的 expression 使用了 test 命令中采用的标准字符串比较。但它提供了 test 命令未提供的另一个特性——模式匹配(pattern matching)。

​	在模式匹配中,可以定义一个正则表达式(将在第20章中详细讨论)来匹配字符串值。

```bash
$ cat test24.sh
#!/bin/bash
# using pattern matching
#
if [[ $USER == r* ]]
then
	echo "Hello $USER"
else
	echo "Sorry, I do not know you"
fi
$
$ ./test24.sh
Hello rich
$
```

### 12.7 case 命令

​	你会经常发现自己在尝试计算一个变量的值,在一组可能的值中寻找特定值。在这种情形下,你不得不写出很长的 if-then-else 语句,就像下面这样。

```bash
$ cat test25.sh
#!/bin/bash
# looking for a possible value
#
if [ $USER = "rich" ]
then
	echo "Welcome $USER"
	echo "Please enjoy your visit"
elif [ $USER = "barbara" ]
then
	echo "Welcome $USER"
	echo "Please enjoy your visit"
elif [ $USER = "testing" ]
then
	echo "Special testing account"
elif [ $USER = "jessica" ]
then
	echo "Do not forget to logout when you're done"
else
	echo "Sorry, you are not allowed here"
fi
$
$ ./test25.sh
Welcome rich
Please enjoy your visit
$
```

​	elif 语句继续 if-then 检查,为比较变量寻找特定的值。

​	有了 case 命令,就不需要再写出所有的 elif 语句来不停地检查同一个变量的值了。 case 命令会采用列表格式来检查单个变量的多个值。

```bash
case variable in
pattern1 | pattern2) commands1;;
pattern3) commands2;;
*) default commands;;
esac
```

​	case 命令会将指定的变量与不同模式进行比较。如果变量和模式是匹配的,那么shell会执行为该模式指定的命令。可以通过竖线操作符在一行中分隔出多个模式模式。星号会捕获所有与已知模式不匹配的值。这里有个将 if-then-else 程序转换成用 case 命令的例子。

```bash
$ cat test26.sh
#!/bin/bash
# using the case command
#
case $USER in
rich | barbara)
	echo "Welcome, $USER"
	echo "Please enjoy your visit";;
testing)
	echo "Special testing account";;
jessica)
	echo "Do not forget to log off when you're done";;
*)
	echo "Sorry, you are not allowed here";;
esac
$
$ ./test26.sh
Welcome, rich
Please enjoy your visit
$
```

​	case 命令提供了一个更清晰的方法来为变量每个可能的值指定不同的选项。