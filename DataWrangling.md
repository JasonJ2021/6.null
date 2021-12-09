[toc]
##Data Wrangling
数据格式转换
####举个例子 : **|**
journalctl列出所有system log entries,
        
        journalctl | grep -i intel #把log entris 转换成更有用的条目

首先我们来看看谁在尝试登陆服务器
    
        ssh myserver journalctl | grep sshd
仍然有许多数据，进行进一步的数据转换
        
        ssh myserver 'journalctl | grep ssh | grep "Disconnected from"' | less
        
        #加additional quoting的原因：文件加载我们自己的电脑上可能很慢，可以在远程服务器上完成这个操作
        
        #less给了我们一个pager，允许在长输出中进行翻页
        
        #我们还可以把输出存放到一个文件中方便之后的debug
        ssh myserver 'journalctl | grep ssh | grep "Disconnected from"' > ssh.log
        less ssh.log

#### sed
sed is a "stream editor" ，可以给出短指令去命令修改文件.
e.g.

        ssh myserver journalctl\
        | grep sshd\
        | grep "Disconnected from"
        | sed 's/.*Disconnecter from //'

s 指令以如下格式书写： s/REGEX/SUBSTITUTION,REGEX是匹配搜索正则表达式，SUBSTITUTION是想要替换成的文本
### Regular expressions
正则表达式经常被/包围
特殊字符清单
- .代表任何一个单个字符
- *匹配任意多个前面的字符
- +匹配一个或多个前面的字符
- \[abc]匹配abc其中的一个字符
- (RX1|RX2)匹配RX1或RX2
- ^一行的开始
- $一行的结束
  
sed的正则表达式需要在特殊字符前面加\，或者-E

        Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
对上面的数据进行data wrangling会输出

        46.97.239.16 port 55920 [preauth]
因为*和+默认都是贪婪的，会匹配尽量多的字符串(可以使用*?来使得*变成非贪婪的)

         | sed -E 's/.*Disconnected from (invalid | authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//'
         #这个可以把这一行都替换成空白，
[^ ]匹配任意非空字符

#####capture groups
上面的代码并不能完成只显示用户名称的目的,我们可以使用capture groups
所有括号括起来的匹配项都是一个capture group被存储起来,e.g.
\1(第一个capture group)

        | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
### Back TO DataWrangling
        ssh myserver journalctl
        | grep sshd
        | grep "Disconnected from"
        | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
在完成了上述操作之后，取出了所有的用户名，但是并不是很有意义
        
        |sort 
        |uniq -c
        |sort -nk1,1
        |tail -n10 #查看最后10个数据

#####sort
可以把stream从小到大排序 #sort -r 可以从大到小排序
#####uniq
uniq可以把重复的行去除，-c输出出现的次数 ,如 10 , Jason
#####sort
-n代表以数字大小排序 k1,1代表以第一个被空格分隔的列排序
,n代表搜索到第n个区域
还有最后两个命令

        | awk '{print $2}' | paste -sd,
##### paste
-s允许combine lines,-d是一个过滤器选项在这里以，为delimiter

### awk Another editor
        | awk '{print $2}'
awk是一种programming language
awk接受可选的匹配模式（默认为所有行）,\$0代表每行的所有内容，\$1代表第一列的内容
每列由空格(default)或可选字符分隔（-F修改）

我们还可以做一些更有趣的事情，比如

        |awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ {print $2}'|wc -l
awk中使用regular expression需要~ /***/
        
        |wc -l 统计行数
awk还可以用来编程，例如

        |awk 'BEGIN {row = 0}
        $1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
        END {print row}'

##Exercise
###[1.Take this short interactive regex tutorial.](https://regexone.com/)
        Lesson Notes
        abc…	Letters
        123…	Digits
        \d	Any Digit
        \D	Any Non-digit character
        .	Any Character
        \.	Period
        [abc]	Only a, b, or c
        [^abc]	Not a, b, nor c
        [a-z]	Characters a to z
        [0-9]	Numbers 0 to 9
        \w	Any Alphanumeric character
        \W	Any Non-alphanumeric character
        {m}	m Repetitions
        {m,n}	m to n Repetitions
        *	Zero or more repetitions
        +	One or more repetitions
        ?	Optional character
        \s	Any Whitespace
        \S	Any Non-whitespace character
        ^…$	Starts and ends
        (…)	Capture Group
        (a(bc))	Capture Sub-group
        (.*)	Capture all
        (abc|def)	Matches abc or def
###2.
        Find the number of words (in /usr/share/dict/words) that contain at least three as and don’t have a 's ending. What are the three most common last two letters of those words? sed’s y command, or the tr program, may help you with case insensitivity. How many of those two-letter combinations are there? And for a challenge: which combinations do not occur?

Answer:
1:

        cat words | sed 'y/A/a/' | awk '$1 ~ /^.*a.*a.*a.*$/ {print $1}' | grep -v ".*'s" | wc -l
2:

        cat words | sed 'y/A/a/' | awk '$1 ~ /^.*a.*a.*a.*$/ {print $1}' | grep -v ".*'s" | sed -E 's/.*(..)/\1/' | sort | uniq -c| sort -nk1,1 | tail -n3
3:

        cat words | sed 'y/A/a/' | awk '$1 ~ /^.*a.*a.*a.*$/ {print $1}' | grep -v ".*'s" | sed -E 's/.*(..)/\1/'| sort | uniq | wc -l
        #只有sort之后才能用uniq

4: 
[参考](https://shadekcse.wordpress.com/2020/12/01/data-wrangling-exercise-using-seds-y-command-grep-comm-and-crunch/)

        crunch 2 2 -o all.txt #生成所有2个字符组成的单词
        cat words | sed 'y/A/a/' | awk '$1 ~ /^.*a.*a.*a.*$/ {print $1}' | grep -v ".*'s" | sed -E 's/.*(..)/\1/'| sort | uniq > ~/Desktop/compare.txt
        comm -23 all.txt combinations.txt | wc -l

### 3
To do in-place substitution it is quite tempting to do something like 
sed s/REGEX/SUBSTITUTION/ input.txt > input.txt. However this is a bad idea, why? Is this particular to sed? Use man sed to find out how to accomplish this.

        On Linux, sed -i is the way to go. sed isn't actually designed for in-place editing, though; historically, it's a filter, a program which edits a stream of data in a pipeline, and for this usage you would need to write to a temporary file and then rename it.

        The reason you get an empty file is that the shell opens (and truncates) the file before running the command.

### 4
Find your average, median, and max system boot time over the last ten boots. 
Use journalctl on Linux and log show on macOS, 
and look for log timestamps near the beginning and end of each boot.

### 5
        Look for boot messages that are not shared between your past three reboots (see journalctl’s -b flag). Break this task down into multiple steps. First, find a way to get just the logs from the past three boots. There may be an applicable flag on the tool you use to extract the boot logs, or you can use sed '0,/STRING/d' to remove all lines previous to one that matches STRING. Next, remove any parts of the line that always varies (like the timestamp). Then, de-duplicate the input lines and keep a count of each one (uniq is your friend). And finally, eliminate any line whose count is 3 (since it was shared among all the boots).

        cat answer5.txt | grep -v '.*Logs begin at.*' |\sed -E 's/(.*[0-9]+:[0-9]+:[0-9]+) (.*)/\2/' | sort | uniq -c | grep -v '.*3.*'
