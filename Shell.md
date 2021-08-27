# The shell
### Using the shell

    missing:~$

missing 代表现在的机器名称  
:后代表现在的工作文件夹/位置
$代表现在不是root user

##### date指令
    jasonj@laptopofjason:~/Desktop/6.null$ date
    2021年 08月 22日 星期日 22:24:26 CST
##### echo
    jasonj@laptopofjason:~/Desktop/6.null$ echo hello
    hello
这里我们告诉shell执行echo程序,它只是打印参数到屏幕上.
每个参数由空格分离，如果要输入包含空格的参数可以“My photo” | My\ Photo
echo的执行逻辑：
shell是一个编程环境，和python , ruby差不多,所以有变量，控制，函数等.
如果输入了一个非关键字，它就会去全局变量$PATH寻找程序
**可以使用which echo来查询echo程序所在的位置**

### Navigating in the Shell
在Linux或者MacOs中,/地址代表文件根系统，而windows系统每个硬盘分区都有一个root
**.** 代表当前文件夹, **..** 代表母文件夹
##### pwd
输出当前的位置

    jasonj@laptopofjason:~/Desktop/6.null$ pwd
    /home/jasonj/Desktop/6.null
##### ls
在运行一个程序的时候，默认操作在当前文件夹

    jasonj@laptopofjason:~/Desktop/6.null$ ls ../
    6.null  CS106L  CSAPP  hello  Hello.c  log  save  test

大部分指令接受flags 和 options(flags 有值)，他们以-开头来改变指令的行为
例如-h or --help 可以打印出一些帮助信息

    jasonj@laptopofjason:~/Desktop/6.null$ ls -l
    total 4
    -rw-rw-r-- 1 jasonj jasonj 1452 8月  22 22:44 Shell.md

这个指令告诉了我们文件的更详细信息
开头d:代表是一个文件夹
后面跟着三组rws,每组分别代表所有者权限(jasonj[owner]) , owning group权(users) , 所有人权限
-代表相应没有权限
如果要进入一个文件夹，user必须有搜索权限(x)
##### mv
如果要重命名或者把文件移动位置，使用mv命令

    jasonj@laptopofjason:~/Desktop/6.null$ mv ../Shell.md ./
##### cp
复制文件夹

    jasonj@laptopofjason:~/Desktop/6.null$ cp Shell.md ../shell.md
##### mkdir
创建文件夹

### Connecting programs
在shell中，程序有两个初始流，input stream 和output stream一般都是终端，但是我们可以重定向他们。
##### 使用cat查看和 <>重定向文件
    jasonj@laptopofjason:~/Desktop/6.null$ cat hello.txt > hello2.txt
    jasonj@laptopofjason:~/Desktop/6.null$ cat hello2.txt
    hello
##### 使用>>来追加一个文件

##### |
管道提供了一种可能，把一个程序的output连接到另外一个程序的input，形成一个程序链

    jasonj@laptopofjason:~/Desktop/6.null$ ls -l / | tail -n1
    drwxr-xr-x  14 root root       4096 2月  10  2021 var
tail默认查看最后10条信息，-n1指定查看一条信息

### A versatile and powerful tool
sudo指令可以让我们使用root权限

    $ echo 3 | sudo tee brightness
tee是一个把input写入output并且输出到屏幕的程序，这里使用root权限

##### xdf-open指令
xdf-open shell.md可以打开shell.md文件
这样就不用鼠标啦！

<!-- - date指令
  ls --help
xdg-open -->