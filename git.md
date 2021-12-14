- [Version Control : Git](#version-control--git)
  - [Git Command-line interface](#git-command-line-interface)
    - [Basics](#basics)
    - [Branching and merging](#branching-and-merging)
    - [Remotes](#remotes)
    - [Undo](#undo)
    - [Advanced Git](#advanced-git)
  - [Git's data model](#gits-data-model)
    - [Snapshots](#snapshots)
    - [Modeling history: realating snapshots](#modeling-history-realating-snapshots)
    - [Data model](#data-model)
    - [Objects and content-addressing](#objects-and-content-addressing)
    - [References](#references)
# Version Control : Git

## Git Command-line interface 
### Basics
- git help \<command> : get help for a git command
- git init:creates a new git repo , with data stored in .git directory
- git status : important ! 
- git add \<filename> : adds files to staging area
- git commit : creates a new commit 
  - [write good commit messages!](https://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
  - [another resource](https://cbea.ms/git-commit/)
- git log
- git log --all --graph --decorate : visualizes history as a DAG
- git diff \<filename> : show changes you made relative to the staging area
- git diff \<revision> \<filename> : show differences in a file between snapshots
- git checkout <revision> : updata HEAD and current branch

### Branching and merging
- git branch : show branches
- git branch \<name> : creates a branch
- git checkout -b \<name> : equal to git branch \<name> ; git checkout \<name>
- git merge \<revision> : merges into the current branch
- git mergetool : use a fancy tool to help resolve merge conflicts

### Remotes

- git remote : list remotes
- git remote add \<name> \<url> : add a remote 
- git push \<remote> \<local branch> : \<remote branch> :send objects to remote , and update remote reference
- git branch --set-upstream-to=\<local>/\<remote branch> : set up correspondence between between local and remote branch
- git fetch : retrive objects / references from a remote
- git pull : same as git fetch : git merge
- git clone

### Undo
- git commit --amend : edit a commit's contents/message
- git reset HEAD \<file> : unstage a file
- git checkout -- \<file>:discard changes

### Advanced Git 
Advanced Git
- git config: Git is highly customizable

- git clone --depth=1: shallow clone, without entire version history
- git add -p: interactive staging
- git rebase -i: interactive rebasing
- git blame: show who last edited which line
- git stash: temporarily remove modifications to working directory
- git bisect: binary search history (e.g. for regressions)
- .gitignore: specify intentionally untracked files to ignore


## Git's data model

### Snapshots

Git把一系列的文件和文件夹模型化为snapshot.文件称为**blob**(只是字节串)，文件夹称为**tree**(映射字符串名字到blob | tree).

    <root> (tree)
    |
    +- foo (tree)
    |  |
    |  + bar.txt (blob, contents = "hello world")
    |
    +- baz.txt (blob, contents = "git is wonderful")

### Modeling history: realating snapshots

Git使用有向图(DAG)来连接snapshots而不是linear history.
例如，一个commit history may look like this:

    o <-- o <-- o <-- o
                ^
                \
                --- o <-- o

上图的每一个O代表一个snapshot,箭头指向这个commit's parent.在某个时间节点，我们可能会把两条分支合并起来，产生一个新的snapshot(commit)

    o <-- o <-- o <-- o <---- o
                ^            /
                 \          v
                  --- o <-- o

### Data model

    // a file is a bunch of bytes
    type blob = array<byte>

    // a directory contains named files and directories
    type tree = map<string, tree | blob>

    // a commit has parents, metadata, and the top-level tree
    type commit = struct {
        parents: array<commit>
        author: string
        message: string
        snapshot: tree
    }

### Objects and content-addressing

一个对象是blob | tree | commit

        type object = blob | tree | commit

在git中所有对象都用Sha-1函数来进行内容索引

    objects = map<string, object>

    def store(object):
        id = sha1(object)
        objects[id] = object

    def load(id):
        return objects[id]

可以使用下面的指令来查看一个SHA-1值具体代表什么
>git cat-file -p

### References 

现在所有的snapshots都可以被认为是SHA-1的哈希，但是它是一个40十六进制长度的字符串，不能理解，Git提供了一种references的方法。引用是指向commit的指针，例如master引用经常指向编程的主分支的最新一个commit.

    references = map<string, string>

    def update_reference(name, id):
        references[name] = id

    def read_reference(name):
        return references[name]

    def load_reference(name_or_id):
        if name_or_id in references:
            return load(references[name_or_id])
        else:
            return load(name_or_id)

在Git中，**HEAD**指针通常指向*where we currently are*



