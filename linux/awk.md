# awk
Awk是一种便于使用且表达能力强的程序设计语言（样式扫描和处理语言），可应用于linux/unix下对文本各种计算和数据处理任务。数据可以来自标准输入、一个或多个文件，或其它命令的输出。

awk的处理文本和数据的方式是逐行扫描文件，从第一行到最后一行，寻找匹配的特定模式的行，并在这些行上进行你想要的操作。如果没有指定处理动作，则把匹配的行显示到标准输出(屏幕)，如果没有指定模式，则所有被操作所指定的行都被处理。

awk允许您创建简短的程序，这些程序读取输入文件、为数据排序、处理数据、对输入执行计算以及生成报表，还有无数其他的功能。默认以空格为分隔符将每行数据切片，切开的部分再进行各种分析处理，支持用户自定义函数和动态正则表达式等先进功能。

awk语法： 
```shell
awk [ -F fs ] [ -v var=value ] [ 'prog' | -f progfile ] [ file ...  ]
awk '{pattern + action}' {filenames}
```

`pattern`表示 AWK 在数据中查找的内容，而 action 是在找到匹配内容时所执行的一系列命令。花括号（{}）不需要在程序中始终出现，但它们用于根据特定的模式对一系列指令进行分组。 pattern就是要表示的正则表达式，用斜杠括起来。

awk语言的最基本功能是在文件或者字符串中基于指定规则浏览和抽取信息，awk抽取信息后，才能进行其他文本操作。完整的awk脚本通常用来格式化文本文件中的信息。

通常，awk是以文件的一行为处理单位的。awk每接收文件的一行，然后执行相应的命令，来处理文本。

选项：
- -F fs or --field-separator fs：指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，比如-F:
- -v var=value or --asign var=value：赋值一个用户定义变量
- -f scripfile or --file scriptfile：从脚本文件中读取awk命令

有三种方式调用awk：
- 命令行方式
    `awk [-F  field-separator]  'commands'  input-file(s)`，commands 是真正awk命令，[-F域分隔符]是可选的。 input-file(s) 是待处理的文件。
    
    在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。通常，在不指名-F域分隔符的情况下，默认的域分隔符是空格。
- shell脚本方式
    将所有的awk命令插入一个文件，并使awk程序可执行，然后awk命令解释器作为脚本的首行，一遍通过键入脚本名称来调用。相当于shell脚本首行的：`#!/bin/sh`，可以换成：`#!/bin/awk`
- 将所有的awk命令插入一个单独文件，然后调用
    `awk -f awk-script-file input-file(s)`，-f选项加载awk-script-file中的awk脚本，input-file(s)跟上面的是一样的。


## 记录
awk把每一个以换行符结束的行称为一个记录。

记录分隔符：默认的输入和输出的分隔符都是回车，保存在内建变量ORS和RS中。

$0变量：它指的是整条记录，将输出test文件中的所有记录，`awk '{print $0}' test`

变量NR：一个计数器，每处理完一条记录，NR的值就增加1。将输出test文件中所有记录，并在记录前显示记录号`awk '{print NR,$0}' test`

## 域（字段）
记录中每个单词称做“域”，默认情况下以空格或tab分隔。awk可跟踪域的个数，并在内建变量NF中保存该值。

域分隔符：内建变量FS保存输入域分隔符的值，默认是空格或tab。我们可以通过-F命令行选项修改FS的值。如：`awk -F: '{print $1,$5}' test`

可以同时使用多个域分隔符，这时应该把分隔符写成放到方括号中，如：`$awk -F'[:\t]' '{print $1,$3}' test`表示以空格、冒号和tab作为分隔符。

输出域的分隔符默认是一个空格，保存在OFS中。如：`awk -F: '{print $1,$5}' test`，$1和$5间的逗号就是OFS的值。

## 流程
awk的工作流程：
1. 执行BEGING
2. 然后读取文件，读入有`\n`换行符分割的一条记录，将记录按指定的域分隔符划分域，填充域，`$0`则表示所有域,`$1`表示第一个域,`$n`表示第n个域。默认域分隔符是"空白键" 或 "[tab]键"，可通过`-F`指定。随后开始执行模式所对应的动作action。接着开始读入第二条记录······直到所有的记录都读完，
3. 执行END操作

awk 支持'pattern + action'的模式，只有匹配了`pattern`的行才会执行`action`操作，如果没有指定`action`则默认输出当前行的内容。

模式和操作都是可选的，如果没有模式，则action应用到全部记录，如果没有action，则输出匹配全部记录。

默认情况下，每一个输入行都是一条记录，但用户可通过RS变量指定不同的分隔符进行分隔。

## 模式
- /正则表达式/：使用通配符的扩展集。
- 关系表达式：可以用下面运算符表中的关系运算符进行操作，可以是字符串或数字的比较，如$2>%1选择第二个字段比第一个字段长的行。
- 模式匹配表达式：用运算符~(匹配)和~!(不匹配)。
- 模式：指定一个行的范围。该语法不能包括BEGIN和END模式。
- BEGIN：让用户指定在第一条输入记录被处理之前所发生的动作，通常可在这里设置全局变量。
- END：让用户在最后一条输入记录被读取之后发生的动作。

范围模板匹配从：第一个模板的第一次出现到第二个模板的第一次出现之间所有行。如果有一个模板没出现，则匹配到开头或末尾。

如：`awk '/root/,/mysql/' test` 显示root第一次出现到mysql第一次出现之间的所有行。

## 操作
操作由一个或多个命令、函数、表达式组成，之间由换行符或分号隔开，并位于大括号内。主要有四部份：
- 变量或数组赋值
- 输出命令
- 内置函数
- 控制流命令

## 变量
awk有许多内置环境变量用来设置环境信息，这些变量可以被改变：
- $n：当前记录的第n个字段，字段间由FS分隔
- $0：完整的输入记录
- ARGC：命令行参数个数
- ARGV：包含命令行参数的数组
- ARGIND：命令行中当前文件的位置(从0开始算)
- CONVFMT：数字转换格式(默认值为%.6g)
- ENVIRON：环境变量关联数组
- ERRNO：最后一个系统错误的描述
- FIELDWIDTHS：字段宽度列表(用空格键分隔)
- FILENAME：awk浏览的当前文件名
- FNR：浏览文件的记录数，同NR，但相对于当前文件
- FS：设置输入域（字段）分隔符，(默认是任何空格)，等价于命令行`-F`选项
- IGNORECASE：如果为真，则进行忽略大小写的匹配
- NF：当前浏览记录的域（字段）的个数，当前行的字段个数
- NR：当前读取的记录数（行号）
- OFMT：数字的输出格式(默认值是%.6g)
- OFS：输出字段分隔符(默认值是一个空格)
- ORS：输出记录分隔符(默认值是一个换行符)
- RLENGTH：由match函数所匹配的字符串的长度
- RS：记录分隔符(默认是一个换行符)
- RSTART：由match函数所匹配的字符串的第一个位置
- SUBSEP：数组下标分隔符(默认值是\034)

在awk中，变量不需要定义就可以直接使用，变量类型可以是数字或字符串。

awk可以在命令行中给变量赋值，然后将这个变量传输给awk脚本。如`awk -F: -f awkscript month=4 year=2004 test`month和year都是自定义变量，分别被赋值为4和2004。在awk脚本中，这些变量使用起来就象是在脚本中建立的一样。注意，如果参数前面出现test，那么在BEGIN语句中的变量就不能被使用。

域变量也可被赋值和修改，如`awk '{$2 = 100 + $1; print }' test`，如果第二个域不存在，awk将计算表达式100加$1的值，并将其赋值给$2，如果第二个域存在，则用表达式的值覆盖$2原来的值。再例如`awk '$1 == "root"{$1 ="test";print}' test`如果第一个域的值是“root”，则把它赋值为“test”，注意，字符串一定要用双引号。

把IGNORECASE设为1代表忽略大小写，打印第一个域是mary的记录数、第一个域、第二个域和最后一个域`awk -F: '{IGNORECASE=1; $1 == "MARY"{print NR,$1,$2,$NF}'test`

## 运算符
- 赋值：=、+=、-=、*=、/=、%=、^=、**=
- C条件表达式：?:
- 逻辑或：
- 逻辑与：&&
- 匹配正则表达式和不匹配正则表达式：~、~!
- 关系运算符：<、<=、>、>=、!=、==
- 连接：空格
- 计算：+、-、*、/、&
- 一元加、减和逻辑非：+ - !
- 求幂：^ ***
- 增加或减少，作为前缀或后缀：++ --
- 字段引用：$
- 数组成员：in

## BEGIN
BEGIN模块后紧跟着动作块，这个动作块在awk处理任何输入文件之前执行。所以它可以在没有任何输入的情况下进行测试。它==通常用来改变内建变量的值==，如OFS,RS和FS等，以及打印标题。如`awk 'BEGIN{FS=":"; OFS="\t"; ORS="\n\n"}{print $1,$2,$3} test`，在处理输入文件以前，域分隔符(FS)被设为冒号，输出文件分隔符(OFS)被设置为制表符，输出记录分隔符(ORS)被设置为两个换行符。`awk 'BEGIN{print "TITLE TEST"}`只打印标题。

## END
END不匹配任何的输入文件，但是执行动作块中的所有动作，在整个输入文件处理完成后被执行。如`awk 'END{print "The number of records is" NR}' test`打印所有被处理的记录数。

## 重定向和管道

awk可使用shell的重定向符进行重定向输出，如：`awk '$1 = 100 {print $1 > "output_file" }' test`表示如果第一个域的值等于100，则把它输出到output_file中。也可以用>>来重定向输出。

输入重定向需用到getline函数。getline从标准输入、管道或者当前正在处理的文件之外的其他输入文件获得输入。它负责从输入获得下一行的内容，并给NF、NR和FNR等内建变量赋值。如果得到一条记录，getline函数返回1，如果到达文件的末尾就返回0，如果出现错误，例如打开文件失败，就返回-1。如：`awk 'BEGIN{ "date" | getline d; print d}' test`执行linux的date命令，并通过管道输出给getline，然后再把输出赋值给自定义变量d，并打印它。

执行shell的date命令，并通过管道输出给getline，然后getline从管道中读取并将输入赋值给d，split函数把变量d转化成数组mon，然后打印数组mon的第二个元素`awk 'BEGIN{"date" | getline d; split(d,mon); print mon[2]}' test`

命令ls的输出传递给geline作为输入，循环使getline从ls的输出中读取一行，并把它打印到屏幕。这里没有输入文件，因为BEGIN块在打开输入文件前执行，所以可以忽略输入文件`awk 'BEGIN{while( "ls" | getline) print}'`

在屏幕上打印”What is your name?"，并等待用户应答。当一行输入完毕后，getline函数从终端接收该行输入，并把它储存在自定义变量name中。如果第一个域匹配变量name的值，print函数就被执行，END块打印See you和name的值`awk 'BEGIN{printf "What is your name?"; getline name < "/dev/tty" } $1 ~name {print "Found" name on line ", NR "."} END{print "See you," name "."} test`

`awk 'BEGIN{while (getline < "/etc/passwd" > 0) lc++; print lc}'`awk将逐行读取文件/etc/passwd的内容，在到达文件末尾前，计数器lc一直增加，当到末尾时，打印lc的值。注意，如果文件不存在，getline返回-1，如果到达文件的末尾就返回0，如果读到一行，就返回1，所以命令 while (getline < "/etc/passwd")在文件不存在的情况下将陷入无限循环，因为返回-1表示逻辑真。

可以在awk中打开一个管道，且同一时刻只能有一个管道存在。通过close()可关闭管道。如：`awk '{print $1, $2 | "sort" }' test END {close("sort")}`awk把print语句的输出通过管道作为linux命令sort的输入,END块执行关闭管道操作。

system函数可以在awk中执行linux的命令。如：`awk 'BEGIN{system("clear")'`

fflush函数用以刷新输出缓冲区，如果没有参数，就刷新标准输出的缓冲区，如果以空字符串为参数，如fflush(""),则刷新所有文件和管道的输出缓冲区。

## 流程控制与循环
awk中的条件语句是从C语言中借鉴来的，if,else,else if 均可使用。

awk中的循环语句同样借鉴于C语言，支持while、do/while、for、break、continue，这些关键字的语义和C语言中的语义完全相同。

## 数组
因为awk中数组的下标可以是数字和字母，数组的下标通常被称为关键字(key)。值和关键字都存储在内部的一张针对key/value应用hash的表格里。由于hash不是顺序存储，因此在显示数组内容时会发现，它们并不是按照你预料的顺序显示出来的。数组和变量一样，都是在使用时自动创建的，awk也同样会自动判断其存储的是数字还是字符串。一般而言，awk中的数组用来从记录中收集信息，可以用于计算总和、统计单词以及跟踪模板被匹配的次数等等。


## 内建函数
### 字符串函数
#### sub函数
匹配记录中最大、最靠左边的子字符串的正则表达式，并用替换字符串替换这些字符串。如果没有指定目标字符串就默认使用整个记录。替换只发生在第一次匹配的时候。格式如下：
``` shell
# sub (regular expression, substitution string):
# sub (regular expression, substitution string, target string)

# 在整个记录中匹配，替换只发生在第一次匹配发生的时候。如要在整个文件中进行匹配需要用到gsub
awk '{ sub(/test/, "mytest"); print }' testfile
# 在整个记录的第一个域中进行匹配，替换只发生在第一次匹配发生的时候
awk '{ sub(/test/, "mytest"); $1}; print }' testfile
```

#### gsub函数
作用如sub，但它在整个文档中进行匹配。
```shell
# gsub (regular expression, substitution string)  
# gsub (regular expression, substitution string, target string)  

# 在整个文档中匹配test，匹配的都被替换成mytest
awk '{ gsub(/test/, "mytest"); print }' testfile  
# 在整个文档的第一个域中匹配，所有匹配的都被替换成mytest
awk '{ gsub(/test/, "mytest"), $1 }; print }' testfile
```

#### index函数
返回子字符串第一次被匹配的位置，偏移量从位置1开始。
```shell
index(string, substring)  

# 实例返回test在mytest的位置，结果应该是3。
awk '{ print index("test", "mytest") }' testfile  
```

#### length函数
返回记录的字符数。
```shell
# length( string )
# length
# 返回test字符串的长度
awk '{ print length( "test" ) }'   
# 返回testfile文件中第条记录的字符数
awk '{ print length }' testfile  
```

#### substr函数
返回从位置1开始的子字符串，如果指定长度超过实际长度，就返回整个字符串。

```shell
substr( string, starting position )  
substr( string, starting position, length of string )  

# 截取了world子字符串
awk '{ print substr( "hello world", 7,11 ) }'   
```

#### match函数

返回在字符串中正则表达式位置的索引，如果找不到指定的正则表达式则返回0。match函数会设置内建变量RSTART为字符串中子字符串的开始位置，RLENGTH为到子字符串末尾的字符个数。substr可利用这些变量来截取字符串。

```shell
# match( string, regular expression )  

# 打印以连续小写字符结尾的开始位置，这里是11
awk '{start=match("this is a test",/[a-z]+$/); print start}'  

# 还打印RSTART和RLENGTH变量，这里是11(start)，11(RSTART)，4(RLENGTH)
awk '{start=match("this is a test",/[a-z]+$/); print start, RSTART, RLENGTH }'  
```

#### toupper、tolower

可用于字符串大小间的转换，该功能只在gawk中有效。
```shell
# toupper( string )  
# tolower( string )  
awk '{ print toupper("test"), tolower("TEST") }'  
```

#### split函数
可按给定的分隔符把字符串分割为一个数组。如果分隔符没提供，则按当前FS值进行分割。
```shell
# split( string, array, field separator )  
# split( string, array )  

# 把时间按冒号分割到time数组内，并显示第二个数组元素18
awk '{ split( "20:18:00", time, ":" ); print time[2] }'  
```

### 时间函数
#### systime函数
返回从1970年1月1日开始到当前时间(不计闰年)的整秒数。
```shell
# systime()  

$ awk '{ now = systime(); print now }'  
```
#### strftime函数
使用C库中的strftime函数格式化时间。
```shell
systime( [format specification][,timestamp] )
```

### 内建数学函数
- atan2(x,y)：y,x范围内的余切
- cos(x)：余弦函数
- exp(x)：求幂
- int(x)：取整
- log(x)：自然对数
- rand()：随机数
- sin(x)：正弦
- sqrt(x)：平方根
- srand(x)：x是rand()函数的种子
- int(x)：取整，过程没有舍入
- rand()：产生一个大于等于0而小于1的随机数

### 自定义函数
在awk中还可自定义函数
```shell
function name ( parameter, parameter, parameter, ... ) {  
    statements  
    return expression # the return statement and expression are optional  
}
```

## 示例
```shell
# 查看Nginx日志最后访问的十个IP：
tail -10 /usr/local/nginx/logs/access.log | awk '{print $1}'

#显示/etc/passwd的账户，-F指定域分隔符为`:`：
cat /etc/passwd |awk -F ':' '{print $1}'

# 显示/etc/passwd的账户和账户对应的shell，而账户与shell之间以tab键分割
cat /etc/passwd | awk -F ':' '{print $1"\t"$7}'

# 显示/etc/passwd的账户和账户对应的shell，而账户与shell之间以逗号分割，第一行添加列名name,shell，在最后一行添加"blue,/bin/nosh"
cat /etc/passwd | awk -F ':' 'BEGIN {print "name,shell"} {print $1","$7} END{print "blue,/bin/nosh"}'

# 搜索/etc/passwd有root关键字的所有行
awk '/root/' /etc/passwd

# root开头的所有行
awk '/^root/' /etc/passwd

# 搜索/etc/passwd有root关键字的所有行，并显示对应的shell
awk -F: '/root/ {print $7}' /etc/passwd

# 统计/etc/passwd:文件名，每行的行号，每行的列数，对应的完整行内容:
awk -F: '{print "filename:" FILENAME "line:" NR "columns:" NF, "all:" $0}' /etc/passwd
awk -F: '{printf ("filename:%s,line:s%,columns:%s,all:%s\n",FILENAME,NR,NF,$0)}' /etc/passwd

# 统计/etc/passwd的账户人数
awk '{count++;print $0} END {print "count is:", count}' /etc/passwd
# count是自定义变量，action{}可以有多个语句，以;号隔开，这里没有初始化count，默认是0
awk 'BEGIN {count=0;print "start count:", count} {count=count+1;print $0} END {print "end count:", count}' /etc/passwd

# 统计某个文件夹下的文件占用的字节数
ls -l | awk 'BEGIN {size=0;print "start size:",size}{size=size+$5} END {print "end size:",size}'
# 格式化为MB
ls -l | awk 'BEGIN {size=0;print "start size:",size} {size=size+$5} END {print "end size:",size/1024/1024,"M"}'

# 统计某个文件夹下的文件占用的字节数,过滤4096大小的文件
ls -l | awk 'BEGIN {size=0;print "start size:",size} {if($5!=4096){size=size+$5}} END {print "end size:",size/1024/1024,"M"}'

# 显示/etc/passwd的账户
awk -F: 'BEGIN {count=0;} {name[count]=$1;count++} END {for(i=0;i<NR;i++) {print i,name[i]}}' /etc/passwd

# 打印所有以模式no或so开头的行
wk '/^(no|so)/' test

# 如果记录以n或s开头，就打印这个记录
awk '/^[ns]/{print $1}' test

# 如果第一个域以两个数字结束就打印这个记录
awk '$1 ~/[0-9][0-9]$/(print $1}' test

# 如果第一个或等于100或者第二个域小于50，则打印该行
awk '$1 == 100 || $2 < 50' test

# 如果第一个域不等于10就打印该行
awk '$1 != 10' test

# 如果记录包含正则表达式test，则第一个域加10并打印出来
awk '/test/{print $1 + 10}' test

# 如果第一个域大于5则打印问号后面的表达式值，否则打印冒号后面的表达式值
awk '{print ($1 > 5 ? "ok "$1: "error"$1)}' test

# 打印以正则表达式root开头的记录到以正则表达式mysql开头的记录范围内的所有记录。如果找到一个新的正则表达式root开头的记录，则继续打印直到下一个以正则表达式mysql开头的记录为止，或到文件末尾
awk '/^root/,/^mysql/' test

# length($0) > 72 Print lines longer than 72 characters.

cat /etc/passwd | awk -F: '\ 
    NF != 7{\ 
        printf("line %d,does not have 7 fields:%s\n",NR,$0)}\ 
    $1 !~ /[A-Za-z0-9]/{printf("line %d,non alpha and numeric user id:%d: %s\n,NR,$0)}\ 
    $2 == "*" {printf("line %d, no password: %s\n",NR,$0)}'
# cat把结果输出给awk，awk把域之间的分隔符设为冒号。
# 如果域的数量(NF)不等于7，就执行下面的程序。
# printf打印字符串"line ?? does not have 7 fields"，并显示该条记录。
# 如果第一个域没有包含任何字母和数字，printf打印“no alpha and numeric user id" ，并显示记录数和记录。
# 如果第二个域是一个星号，就打印字符串“no passwd”，紧跟着显示记录数和记录本身。

# awk先扫描第一个域，一旦test匹配，就把第二个域的值加上第三个域的值，并把结果赋值给变量count，最后打印出来。
awk '$1 ~/test/{count = $2 + $3; print count}' test 

```

参考：
- https://www.jianshu.com/p/8c6a0d0d4f0d
- https://awk.readthedocs.io/en/latest/chapter-one.html

未参考：
- https://coolshell.cn/articles/9070.html