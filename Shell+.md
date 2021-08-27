# Shell Tools and Scripting

### Shell Scripting 
shell脚本语言与其他脚本语言不同的点在于，他会针对shell相关任务进行优化

##### 对变量赋值
e.g.

    foo=bar
    //foo = bar 是无效赋值，空格在shell中非常重要，在这里会被解释为给foo 程序传入了两个参数 = 和 bar
##### 变量访问

    echo $foo //打印bar
##### " " 与‘ ’
    foo=bar
    echo "$foo" #print bar
    echo '$foo' #print $foo

##### 控制流
bash支持控制流如：if,case,while,for.同时bash也可以运行有参数函数
e.g

    mcd.sh:
    mcd(){
        mkdir -p "$1"
        cd "$1"
    }                                               
这个函数被定义在mcd.sh中，然后使用source mcd.sh将这个函数加载到shell中
之后就可以直接使用mcd test创建一个test文件夹并进入，就像ls,cd等程序一样

shell脚本中有一些特殊变量，指向参数，错误代码，以及其他一些相关变量，例如
- $0 脚本名字
- \$1 to \$9 脚本的参数，可以表示9个参数 
- $@ 表示所有参数
- $# 表示参数的个数
- $? 表示上一个命令的返回,例如错误代码
- $$ 进程代码 process identification number for the current script
- !! 表示上一个命令的全部,如果一条命令输入没有权限，之后可以用sudo !!
- $_ 表示上一个命令的最后一个参数

命令总会使用STDOUT来返回输出，使用STDERR和Return code 来报告错误

还可以使用 **&&** 和 **||** 来执行命令，和C语言一样他们都是short-circuiting operator
同时可以使用 **;** 来分割两条命令

##### Command Substitution
- $(CMD) 它会执行CMD,获得CMD的输出并把\$(CMD)替换成输出.例如 

        for file in $(ls),可以遍历当前文件夹

- <(CMD) 它会执行CMD然后把输出放到一个temporary file中，之后把<(CMD)替换成那个暂时文件的名字
  这称为process substitute,对于要求输入是一个文件的指令非常有用，例如
        
        diff <(ls foo) <(ls bar) #这个命令会显示foo和bar文件夹中文件的不同
一个例子

    #!/bin/bash
    echo "Starting program at $(date)"
    echo "Running program $0 with $# arguments with pid $$"

    for file in "$@";do
        grep foobar "$file" > /dev/null 2> /dev/null
        if[[ $? -ne 0]];then
            echo "File $file does not have any foobar,adding one"
            echo "# foobar" >> "$file"
        fi
    done;

在进行比较的时候，尽量使用[[]],在这里我们比较返回是不是0,代表有没有在一个文件中找到foobar

##### 通配符(globbing)
- wildcards
  - **?/\***
  - **[ ]** e.g. [A-Za-z]
- Curly braces{},可以拓展为一系列的子序列
    
        convert image.{png,jpg}
        #will expand to
        convert image.png image.jpg
        
        cp /path/to/project/{foo,bar,baz}.sh /newpath
        
        mv *{.sh,.py} folder

        touch {foo,bar}/{a..h} #create foo/a foo/b `````` foo.h

shellcheck 可以debug sh/bash 脚本

### Shell Tools

##### Finding files
find会迭代地寻找所有符合一定要求的文件
    
    #Find all directories named src
    find . -name src -type d
    #Find all python files that have a folder named test in their path
    find . -path '*/test/*.py' -type f
    #Find all files modifies in the last day
    find . -mtime -1
    #Find all zip files with size in range 500k to 10M
    find . -size +500k -size -10M -name '*.tar.gz'
    
find也可以帮助我们对找到的文件进行操作e.g.

    #Delete all files with .jpg
    find . -name '*.jpg' -exec rm {}\; 
    #Find all PNG files and convert them to JPG
    find . -name '*.png' -exec convert {} {}.jpg \;

关于为什么在find exec 里面有{} \; 贴一段Ubantu社区的回答:
If you run find with exec, {} expands to the filename of each file or directory found with find (so that ls in your example gets every found filename as an argument - note that it calls ls or whatever other command you specify once for each file found).

Semicolon ; ends the command executed by exec. It needs to be escaped with \ so that the shell you run find inside does not treat it as its own special character, but rather passes it to find.
