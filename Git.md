# Git

# 初步认识

## 版本控制

版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的软件。所谓版本，就是某个时刻所有维护文件的内容，可以看作是维护对象的快照。通过版本控制，用户可以随意将某个文件甚至整个项目回退到历史某个时刻。

Git 是分布式版本控制系统，相对还有集中式，它的客户端不只是提取最新版本的文件快照，而是把代码仓库完整镜像下来，包括完整的历史记录。这样下来，服务器一旦发生故障，就可以用任何一个镜像的本地仓库恢复。因为每一次克隆远程仓库，实际上都是一次完整备份。Git 功能强大，且免费、开源，非常流行。

## 基本原理

### 基于快照的存储

大部分版本控制系统常以变更列表方式存储信息，这类系统把存储的信息看作一组基本文件加上每个文件随时间累计的修改。而 Git 更像是把数据看作对小型文件系统的一系列快照，每次提交更新或保存项目状态，基本都会对全部文件创建一个快照并保存其索引。为了效率，如果已有文件没有修改，它不会被重复保存，而是记录一个链接指向之前存储的文件。Git 对待数据更像是一个快照流。

### 本地执行的操作

因为 Git 客户端会把远程仓库几乎所有数据拷贝下来，可以看作是远程仓库的一比一备份。所以，用户对本地仓库执行的几乎所有操作，包括修改、提交、回退等等，基本都是本地执行。即便不联网，Git 依然能正常工作，但是无法及时获取其它用户的提交或者推送提交到服务端。

### 校验保证完整性

Git 所有的数据在存储前都会计算校验和，然后以校验和来引用，这意味着无法在 Git 不知情时修改任何文件内容或目录内容，传送过程中如果丢失信息或损坏文件，都会被 Git 发现。

用于计算校验和的机制叫做 SHA-1 散列（Hash），这是一个由 40 个十六进制字符组成的字符串，基于文件内容和目录结构计算，下例是一个校验和。

```
24b9da6552252987aa493b52f8696cd6d3b00373
```

Git 使用这种校验和的地方很多，比如数据库中保存的信息都是以文件内容的校验和来索引，而非文件名。

### 总是只添加数据

Git 操作几乎只往数据库添加数据，用户很难从 Git 数据库中删除数据，所以 Git 几乎不会执行任何可能导致文件不可恢复的操作。当然，没有保存的修改可能会丢失或损毁。

### 仓库的三种状态

Git 项目的文件有 3 种状态，每个文件必定处于其中一种：

* 已提交（Committed）：文件已被修改，但还没保存到数据库；
* 已修改（Modified）：标记修改文件，使之包含在下次提交的快照；
* 已暂存（Staged）：数据已经保存到本地数据库。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/git-54ee62ac.png" style="zoom: 80%;" />

工作区是项目的某个版本独立提取出来的内容，打开仓库目录就是工作区，其中的文件源于 Git 数据库，放在磁盘供用户使用或者修改；暂存区是一个文件，保存着下次将要提交的文件列表；Git 仓库是用来保存项目的元数据和对象数据库的地方，这是 Git 最重要的部分，从其它地方克隆仓库时，复制的就是这部分数据。

基本 Git 工作流程如下：

1. 工作区修改文件；
2. 把要保存的修改选择性暂存；
3. 提交更新，把暂存区的文件的快照永久性地存储到 Git 目录。

只要 Git 目录中保存着特定版本的文件，就表示这些文件已提交。如果文件已修改并被放入暂存区，就处于已暂存状态。如果文件被跟踪，做了修改但还未放到暂存区，就处于已修改状态。

## 初始配置

### 配置文件

Git 有一个 `git config` 工具，用于设置控制 Git 外观和行为的配置变量，这些变量被放在 3 个位置：

* */etc/gitconfig*：含有系统每一个用户以及它们仓库的通用配置，执行 `git config` 时带上 *--system* 就会读写该文件中的配置变量；对于 Windows，它是 *MSys\etc\gitconfig*，*MSys* 是 Git 安装目录。
* *~/.gitconfig* 或 *~/.config/git/config*：仅针对当前用户，带上 *--global* 选项就会读写该文件，这会对系统上所有仓库生效；对于 Windows，它是 *C:\Users\\$USER\\.gitconfig*。
* Git 仓库目录中的 *config* 文件：仅针对当前仓库，带上 *--local* 选项就会读写该文件，优先级最高，修改该处变量需要先进入项目目录。

每一个级别都会覆盖上一个级别的配置，所以 *.git/config* 将会覆盖 */etc/gitconfig* 中的配置变量，下例列出所有配置变量以及它们所在的文件。

```
git config --list --show-origin
```

### 用户信息

初次安装 Git 之后，要做的第一件事就是设置用户名和邮件地址，这很重要，因为每一个 Git 提交都要使用这两个基本信息，标识本次提交由谁发起，它们的值可随意设置。

```
$ git config --global user.name "John Doe"
$ git config --global user.email "johndoe@example.com"
```

### 查看配置

使用 `git config` 命令查看当前的配置变量，通过名字来指定。

```
git config <key>
```

选项 `--list` 表示列出所有变量，可能会看到多个同名变量，它们源于不同的文件，选项 `--show-origin` 表示显示变量所在文件。

```
git config --list --show-origin
```

## 帮助信息

获得指定 Git 命令的帮助信息，这会打开一个浏览器页面。

```
git help <command>
```

获得简洁的帮助信息，直接显示在当前 shell 窗口。

```
git <command> -h
```

# 基础操作

## 获取仓库

通常有两种获取 Git 仓库的方式，它们最终都会在本地得到一个 Git 项目目录。

### 初始本地目录

现有一个尚未进行版本控制的普通目录，要把这个目录转换为 Git 仓库，进入目录执行以下命令：

```
git init
```

这个命令将会创建一个 *.git* 子目录，这就是 Git 仓库，其中包含所有必须文件，这只是一个初始操作，项目的文件都还没有被跟踪，若要管理这些文件，还要使用 `add` 等命令。

### 克隆已有仓库

使用以下命令从远程服务器克隆一个已有 Git 仓库，它会复制远程仓库几乎所有的数据。

```
git clone <url> [newName]
```

该命令会在本地创建一个与远程仓库同名的目录，然后初始化 *.git* 子目录，从远程仓库拉取的所有数据都放在这个子目录内，最后使用最新版本的数据初始化工作目录。默认情况，该命令会拷贝远程仓库的所有版本，所以当远程仓库损毁时，可以使用本地仓库进行恢复。如果想自定义本地仓库的名字，填充 newName 即可。

Git 支持多种数据传输协议，包括 HTTP、GIT 以及 SSH 传输协议，比如 *user@server:path/to/repo.git*。

## 记录修改

### 文件状态

工作区的文件不外乎两种状态：已跟踪、未跟踪，已跟踪文件指已被 Git 纳入版本管理的文件，它们出现在上一次快照当中。工作一段时间之后，已跟踪文件的状态可能是未修改、已修改或已暂存，修改已跟踪文件，这些文件会被标记为已修改，用户可以选择性地把这些文件的当前版本加入暂存区等待提交。

至于未跟踪文件，它们是新添加到工作区的文件，上一次快照中没有它们，使用 `git add` 命令对其进行跟踪并把当前版本加入暂存区，从这来看，`git add` 命令至少有跟踪和暂存两个作用。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/git-0ab88182.png" style="zoom: 80%;" />

### 检查状态

执行 `git status` 命令查看工作区各文件的状态，第一行指出当前分支，以及是否偏离远程分支。若有未跟踪或已修改、或未暂存的文件，这里都会显示出来。

以下返回表示当前工作区很干净，所有文件都是已跟踪未修改。

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
nothing to commit, working directory clean
```

添加一个新文件 *README* 到工作区，查看状态，该文件被标记为未跟踪。

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Untracked files:
  (use "git add <file>..." to include in what will be committed)

    README

nothing added to commit but untracked files present (use "git add" to track)
```

`add` 新文件之后，查看状态，其被标记为等待提交，说明它已被跟踪并暂存。

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)

    new file:   README
```

修改 *README* 文件，查看状态，显示该文件虽被跟踪，但当前版本还没被暂存，这个时候应该使用 `git add` 命令暂存该文件当前版本，使其纳入下一次提交。

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   README
```

工作区的文件可能既是未被暂存又是等待提交，这说明该文件在上一次暂存后又发生了修改，导致工作区版本与暂存区版本不同。

### 忽略文件

某些工作区的文件不需进行版本管理，又不想它们总是出现在未跟踪列表，通常都是一些自动生成的文件，比如编译文件。这种情况，可在工作区创建一个 *.gitignore* 文件，写下想要忽略的文件的模式，格式规范如下：

* 空行或以 `#` 开头的行会被忽略；
* 使用标准 glob 模式匹配，默认会递归地应用到当前目录及子目录；
* 匹配模式可以以 `/` 开头防止递归；
* 匹配模式可以以 `/` 结尾指定目录；
* 要忽略指定模式以外的文件或目录，可在模式前加上叹号 `!` 取反。

通常，我们只需在工作区根目录创建一个 *.gitignore* 文件，它会递归地应用到整个仓库。但是，子目录下还可以额外创建 *.gitignore* 文件，它的规则只作用于当前目录，且优先级更高。

以下是一些示例：

```
# 忽略所有 .class 文件
*.class

# 跟踪 Main.class，即使前面设置忽略 .class 文件
!Main.class

# 忽略当前目录下的 TODO 文件
/TODO

# 忽略所有目录下的 build 目录
build/

# 忽略 doc 目录下的 .txt 文件
doc/*.txt

# 忽略 doc/ 目录及其子目录下的 .pdf 文件
doc/**/*.pdf
```

### 查看修改

`git status` 返回信息可能还是太简单，若要了解文件具体修改了什么内容，可以使用 `git diff` 命令，它会以文件补丁的格式具体显示出哪些行发什么了什么变化。

如果不加任何参数，`git diff` 比较的是工作区与暂存区的差异。

```
git diff <file>
```

添加 `--staged` 参数，`git diff` 比较的是暂存区与最后一次提交的差异。

```
git diff --staged <file>
```

### 提交更新

文件暂存之后，就要进行提交，执行 `git commit` 提交暂存区所有文件到 Git 仓库，这会打开一个文本编辑器以供用户输入提交说明，至于启动哪个编辑器，由 shell 环境变量 EDITOR 决定，通常是 vim 或 emacs，可以使用命令 `git config --global core.editor` 自定义。默认会把最后一次 `git status` 返回内容放到注释行供用户查看，退出时 Git 会丢弃所有注释行，用输入的说明生成一次提交。

更常见的方式是使用 `-m` 选项，把提交说明和命令放在同一行。返回信息包括当前分支、本次提交的完整 SHA-1 校验和，以及有多少文件修订，多少行添加和删改。

```
$ git commit -m "Story 182: Fix benchmarks for speed"
[master 463dc4f] Story 182: Fix benchmarks for speed
 2 files changed, 2 insertions(+)
 create mode 100644 README
```

### 删除文件

把文件从 Git 版本管理移除，首先需从暂存区删除该文件，然后提交。命令 `git rm` 会把文件移出版本管理，同时删除工作区的文件。如果只在工作区删除文件，查看状态会显示这个删除操作尚未暂存。

```
git rm <file>
```

如果文件修改过或已放入暂存区，则删除需要带上 `-f` 强制删除选项，这只是一种安全特性，因为这些数据还没有存入 Git 仓库，删除后就再也无法恢复。

```
git rm -f <file>
```

如果使用 `--cached` 选项，文件将从 Git 仓库和暂存区删除，但在工作区的文件依然保留。

```
git rm --cached <file>
```

除了确切的文件名，还能用 glob 模式选择要删除的文件，下例删除所有 *.class* 文件。

```
git rm .class
```

 ### 移动文件

使用 `git mv` 移动或重命名文件，它与 `mv` 没啥区别，但是会连带更新 Git 仓库。

```
$ git mv README.md README
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    renamed:    README.md -> README
```

使用 `git mv` 重命名，相当于执行以下 3 条命令，两种方式的结果都一样，Git 能够识别出重命名操作。

```
$ mv README.md README
$ git rm README.md
$ git add README
```

## 提交历史

### 常规使用

执行 `git log` 命令查看 Git 项目的提交历史，默认会按时间先后顺序列出所有提交，较近的提交放在上面。如下所示，它会显示提交的校验和、作者名字、邮件地址、提交时间，以及提交说明。

```
$ git log
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number
```

选项 `-n` 指定只显示最近几条提交，`n` 是一个整数，选项 `-p` 显示每次提交所引入的差异。

```
$ git log -p -1
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

diff --git a/Rakefile b/Rakefile
index a874b73..8f94139 100644
...
```

选项 `--stat` 指示增加一些总结性信息，比如有多少个文件修订，添加多少行，删除多少行。

```
$ git log --stat
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number

 Rakefile | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

选项 `--pretty` 指定使用其它格式显示提交信息，比如 `oneline` 会把一个提交放在一行。

```
$ git log --pretty=oneline
ca82a6dff817ec66f44342007202690a93763949 changed the version number
085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 removed unnecessary test
a11bef06a3f659402fe7563abf99ad00de2209e6 first commit
```

把 `oneline` 与 `--graph` 结合十分有用，它会用一些 ASCII 字符串来形象地展示分支、合并历史。

```
$ git log --pretty=format:"%h %s" --graph
* 2d3acf9 ignore errors from SIGCHLD on trap
*  5e3ee11 Merge branch 'master' of git://github.com/dustin/grit
|\
| * 420eac9 Added a method for getting the current branch.
* | 30e367c timeout code and tests
* | 5a09431 add timeout protection to grit
* | e1193f8 support for heads with slashes in them
|/
* d6016bc require time for xmlschema
*  11d191e Merge branch 'defunkt' into local
```

### 筛选输出

除了 `-n`，`git log` 还有许多选项用于限制日志输出，比如 `--after` 和 `--before` 通过时间来限制，下例列出最近两周的所有提交。

```
git log --after=2.weeks
```

其它用于过滤的选项：比如 `--author` 显示指定作者的提交，`--committer` 显示指定提交者的提交，如果想忽略合并提交，使用 `--no-merges` 选项。

## 撤销操作

### 重新提交

很多时候，我们的提交可能会遗漏信息，或者写错说明，这时可以使用 `--amend` 选项重新提交，这个新的提交会替换上一次提交，从效果上来说，上一次提交好像从未存在过一样，它不会出现在提交历史中。

```
git commit --amend -m 'replace commit'
```

### 取消暂存

如果此次提交想要跳过暂存区某个文件，可以使用 `reset` 命令把这个文件从暂存区删除。关于 `reset`，更后面再深入学习。

```
git reset HEAD <file>
```

### 撤销修改

如果连文件在工作区的修改也不想保留，把它恢复成最近一个版本的样子，考虑使用 `checkout` 命令。这个命令非常危险，它会用最近的版本覆盖当前文件，无法恢复，因为这些修改还没有保存。

```
git checkout -- <file>
```

## 远程仓库

### 查看远程仓库

列出配置的远程仓库，默认只显示仓库的名字，选项 `-v` 显示仓库地址。可以配置多个远程仓库，至于用户在这些远程仓库的读写权限，得看远程仓库怎么设置。

```
$ git remote -v
origin    git@github.com:mojombo/grit.git (fetch)
origin    git@github.com:mojombo/grit.git (push)
koke      git://github.com/koke/grit.git (fetch)
koke      git://github.com/koke/grit.git (push)
```

显示某个远程仓库的详细信息，这个命令会列出远程仓库的地址和跟踪分支的信息，比如运行 `push`、`pull` 将对哪个远程仓库进行推送和拉取，哪些分支本地没有，哪些分支远程仓库没有。

```
$ git remote show origin
* remote origin
  URL: https://github.com/my-org/complex-project
  Fetch URL: https://github.com/my-org/complex-project
  Push  URL: https://github.com/my-org/complex-project
  HEAD branch: master
  Remote branches:
    master                           tracked
    dev-branch                       tracked
    markdown-strip                   tracked
    issue-43                         new (next fetch will store in remotes/origin)
    issue-45                         new (next fetch will store in remotes/origin)
    refs/remotes/origin/issue-11     stale (use 'git remote prune' to remove)
  Local branches configured for 'git pull':
    dev-branch merges with remote dev-branch
    master     merges with remote master
  Local refs configured for 'git push':
    dev-branch                     pushes to dev-branch                     (up to date)
    markdown-strip                 pushes to markdown-strip                 (up to date)
    master                         pushes to master                         (up to date) 
```

### 添加远程仓库

克隆的远程仓库默认会把克隆仓库设为远程仓库，并命名为 origin，还能用以下命令添加其它远程仓库。

```
git remote add <shortname> <url>
```

### 拉取远程数据

从指定的远程仓库拉取本地没有的数据，接着本地就会有这个仓库所有分支的引用，用户可以随时查看或合并这些分支。注意，`fetch` 只是拉取数据，没有其它什么操作，如有需要，用户应该手动合并。

```
git fetch <remote>
```

如果当前分支设置了跟踪远程分支，运行 `pull` 抓取数据的同时还会合并上游分支到本地分支。默认情况，克隆得到的仓库会自动设置本地 master 分支跟踪克隆仓库 master 分支。

### 推送远程仓库

以下命令表示推送本地 branch 分支到 remote 远程仓库，仅当用户拥有远程仓库的读写权限，且前面没有其它人推送时这条命令才能生效。否则产生冲突，推送被拒绝，需要先合并别人的修改才能推送自己的修改。

```
git push <remote> <branch>
```

### 修改远程仓库

修改远程仓库的名字，同时连带修改该远程仓库的远程跟踪分支的名字。

```
git remote rename <remote> <newname>
```

移除一个远程仓库，所有与该仓库相关的远程跟踪分支以及配置都会被删除。

```
git remote rm <remote>
```

## 提交标签

Git 支持给仓库历史中的某一个提交打上标签，以示重要，比如使用这个功能标记发布结点。

### 查看标签

按字母顺序列出所有标签，选项 `-l` 可省。

```
git tag [-l]
```

通过名字匹配来查找标签，此时 `-l` 必须存在，下例查找 1.8.5 系列标签。

```
git tag -l "v1.8.5*"
```

### 创建标签

Git 标签有两种类型：轻量标签（Lightweight）、附注标签（Annotated）。

**轻量标签**

轻量标签本质是把提交校验和存入一个文件，这和分支的定义好像一样，创建这种标签只需提供名字。

```
git tag v1.4-lw
```

查看轻量标签，其只包含提交信息。

```
$ git show v1.4-lw
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number
```

**附注标签**

附注标签是 Git 仓库中的完整对象，具有校验和，其中包含打标签者的名字、邮件地址、日期时间，另外还有标签说明。与轻量标签相比，它的优势是附带额外信息。

使用 `-a` 选项创建附注标签，`-m` 设置说明，没有 `-m` 将会打开文本编辑器，这点与 `commit` 相同。

```
git tag -a v1.4 -m "my version 1.4"
```

查看附注标签，可以看到它的标签信息和对应的提交信息。

```
$ git show v1.4
tag v1.4
Tagger: Ben Straub <ben@straub.cc>
Date:   Sat May 3 20:19:12 2014 -0700

my version 1.4

commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number
```

### 补充标签

创建标签时默认标记最近一次提交，使用 `-a` 选项根据校验和指定标记某个提交。

```
git tag -a <tagname> <hashcode>
```

### 删除标签

通过名字指定删除哪个标签

```
git tag -d <tagname>
```

前面命令只是删除本地标签，要想把这个删除推送到远程仓库，需要使用以下命令。

```
git push <remote> --delete <tagname>
```

### 推送标签

标签不会随着分支一起推送，需要使用 `push` 命令单独推送。

```
git push <remote> <tagname>
```

使用 `--tags` 选项把远程仓库没有的标签一次性全都推送上去，包括轻量标签和附注标签。

```
git push <remote> --tags
```

### 检出标签

直接检出标签所指版本会使仓库处于分离头指针（Detached HEAD）状态，处于这种状态若用户做了修改并提交它们，标签不会变化，但新提交将不属于任何分支且无法访问，除非通过确切的提交校验和。

```
git checkout <tagname>
```

所以，如果非要回到某个标签的版本，基于标签创建一个新分支更好，往后的提交会从这个新分支延申下去。

```
git checkout -b <branchname> <tagname>
```

# 版本分支

## 分支原理

### 提交对象

已知 Git 保存数据的方式是快照，现有一个工作目录，包含 3 个文件等待暂存和提交。执行暂存操作，会为每个文件计算校验和，然后把当前版本的文件快照存入 Git 仓库，这里 3 个文件要用 3 个 blob 对象，最后把校验和加到暂存区等待提交。执行提交操作，Git 会为每个子目录计算校验和，然后在仓库中把它们存为树对象，树对象记录目录结构和 blob 对象索引。最后创建提交对象，包含作者、邮箱、提交说明，以及父对象和树对象的指针。

现在，Git 仓库就有 5 个对象，3 个 blob 对象、1 个树对象、1 个提交对象，它们的关系如下所示。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/git-5364df99.png" style="zoom:50%;" />

现在做些修改并提交，这次产生的提交对象会包含一个指向上次提交对象（父对象）的指针。首次提交产生的提交对象没有父对象，普通提交都有一个父对象，分支合并产生的提交对象有多个父对象。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/git-77fb5dc2.png" style="zoom:50%;" />

### 初识分支

Git 分支是保存着某个提交对象检验和的文件，以此成为指向提交对象的指针，好像标签呀！所以分支的创建和删除都很快，又因为每个提交对象会记录父对象，所以寻找恰当的合并基础也同样简单高效。

Git 仓库初始化时会自动创建一个分支，名为 master，这个分支与普通分支没有区别。每次提交，分支都会自动向前移动，保证总是指向最新的提交对象。

Git 有一个 HEAD 指针，指向当前所在的本地分支，Git 通过 HEAD 知道当前处于哪个分支，每次提交它也会随着分支自动向前移动。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/git-fe424144.png" style="zoom:50%;" />

## 基本操作

### 创建分支

基于指定分支创建一个新分支，默认使用当前分支，该命令不会自动切换到新分支。

```
git branch <newbranch> [branch]
```

与上相同，但创建分支同时会切换过去，等同 `git branch` + `git checkout`。

```
git checkout -b <newbranch> [branch]
```

### 切换分支

切换到指定分支，其实就是使 HEAD 指向这个分支。

```
git checkout <branchname>
```

切换分支前应尽量保证工作区、暂存区干净，避免污染其它分支或丢失修改。因为切换时 Git 会用新分支的数据恢复工作区，没有暂存的数据会丢失，没有跟踪的文件或未提交的暂存文件，则保留原样或产生冲突，如果有冲突会拒绝此次切换操作。

### 合并分支

把指定分支合并到当前分支，这不会影响合并分支，只会改变当前分支的内容。

```
git merge <branchname>
```

#### 快进合并

如果顺着一个分支可以直接到达另一个分支，那么合并它们的时候直接移动 HEAD 指针即可，这种情况的合并没有需要解决的分歧，称作"快进"（fast-forward）。

#### 三方合并

如果两个分支分叉，它们的合并则需一些额外工作，首先使用这两分支的末端快照以及它们的公共祖先做一个三方合并，再把合并的结果做成一个新快照并提交，这种提交叫做合并提交，它有不止一个父对象。

如下所示，C6 就是 C4 和 C5 合并提交的结果。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/git-73650f13.png" style="zoom:50%;" />

#### 合并矛盾

如果合并有矛盾，比如两个分支都在同一个文件的同一个位置做了修改，Git 就无法干净地合并它们。这时合并会停下来，发生冲突后使用 `git status` 查看 unmerged 状态文件，这些就是产生冲突的文件。要么手动修改它们要么使用 `git mergetool` 工具解决冲突，然后使用 `git add` 将其标记为已解决并提交。

如下所示，<<< 至 === 是 HEAD 所指分支的内容，=== 至 >>> 是 iss53 分支的内容，现在需要手动选择保留哪些内容，同时删除 <、=、> 这 3 行。这是内容冲突，还有删除、新增等冲突。

```
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
 please contact us at support@github.com
</div>
>>>>>>> iss53:index.html
```

### 删除分支

使用选项 `-d` 删除分支

```
git branch -d <branchname>
```

### 分叉历史

查看提交历史，使用选项显示各个分支的指向以及项目的分支分叉情况。

```
git log --oneline --decorate --graph --all
```

### 查看分支

命令 `git branch` 如果不加任何参数，将会列出当前所有分支，* 号标记当前所在分支。

```
$ git branch
  iss53
* master
  testing
```

选项 `--merged`、`--no-merged` 表示滤出与指定分支已经合并或尚未合并的分支，默认与当前分支比较。

```
git branch --merged [branchname]
```

## 远程分支

### 初步认识

**远程引用**是对远程仓库的引用（指针），包括分支、标签等等。运行 `git ls-remote <remote>` 命令将列出远程引用的完整列表，运行 `git remote show <remote>` 显示远程分支的更多信息。

**远程跟踪分支**是远程分支状态的引用，存放在本地，但用户无法移动，它唯一的作用是标记该分支处于远程仓库什么位置。远程跟踪分支反映的是最后一次连接远程仓库时的状态，直到下一次通信之前，远程跟踪分支都不会发生变化，所以这种分支不具有实时性。远程跟踪分支命名格式：`<remote>/<branch>`，比如 origin/master。

克隆一个仓库 Git 会自动为其设置远程仓库，且命名为 origin，同时创建一个远程跟踪分支 origin/master，另外还会创建本地分支 master，启点就是 origin/master。如下所示。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/git-28ae3ced.png" style="zoom:50%;" />

与 master 相同，origin 只是一个默认名字，克隆时可以使用 `-o` 选项为远程仓库设置其它名字，它的远程跟踪分支名字也随之改变。

```
git clone -o <othername>
```

本地仓库 master 分支提交了一些工作，假如有个用户推送提交到远程仓库更新了 master 分支，这使本地提交与远程仓库走向不同方向。即便如此，只要不与 origin 连接，本地 origin/master 就不会移动。

手动更新远程跟踪分支，运行 `git fetch` 或 `git pull` 命令，后者会进行自动合并。

### 推送修改

推送本地分支到远程仓库，默认远程分支与本地分支同名。

```
git push <remote> <branch>
```

如果远程分支与本地分支不同名，使用以下命令。

```
git push <remote> <localbranch>:<remotebranch>
```

使用 `--delete` 选项删除远程分支

```
git push <remote> --delete <branch>
```

### 合并分支

运行 `git fetch` 拉取数据可能会生成新的远程跟踪分支，如果想要使用这些分支，需要执行以下命令手动合并远程分支到本地分支。

```
git merge <remote>/<branch>
```

或者，基于这个远程跟踪分支创建一个本地分支。

```
git checkout -b <branch> <remote>/<branch>
```

### 跟踪分支

基于一个远程跟踪分支创建一个本地分支会自动创建"跟踪分支"，被跟踪的分支叫做上游分支，跟踪分支是与远程分支有直接关系的本地分支。处于跟踪分支时运行 `git pull`，Git 会自动识别去哪个远程仓库抓取数据并自动合并到当前分支。

前面提到，克隆仓库时会自动基于 origin/master 创建一个 master 分支，这是一个跟踪分支。需要的话，可以设置其它跟踪分支，或一个在其它远程仓库上的跟踪分支，又或不跟踪 master 分支。使用 `checkout -b` 基于远程跟踪分支创建的本地分支，都是跟踪分支。

使用 `git branch` 的 `-vv` 选项，可以查看设置的所有跟踪分支，这会列出所有本地分支并且包含更多信息，比如哪个分支正在跟踪哪个远程分支，本地分支领先或落后远程分支几个提交。

```
$ git branch -vv
  iss53     7e424c3 [origin/iss53: ahead 2] forgot the brackets
  master    1ae2a45 [origin/master] deploying index fix
* serverfix f8674d9 [teamone/server-fix-good: ahead 3, behind 1] this should do it
  testing   5ea463a trying something new
```

需要注意，以上信息都源于最后一次连接远程仓库时抓取的数据，最好提前执行以下命令，抓取所有远程仓库。

```
git fetch --all;
```

# 内容重置

## 认识 3 棵树

学习 `reset` 和 `checkout` 工具之前，需先了解 Git 使用 3 棵树管理数据的思想，树在这指文件的集合，而非某种数据结构：

* HEAD：当前分支的引用，指向上一次提交的快照，也是下一次提交的父结点；

* Index：索引，就是暂存区，存放预期的下一次提交；
* Working Directory：工作区，内容是快照解包的结果，以供用户修改。

这 3 棵树只是一个概念，它们的关系和交互方式前面的章节反复提到，不必多说。假如有一个 Git 仓库，包含一个文件 *file.txt*，该文件已被修改 3 次并提交，Git 仓库和 3 棵树如下所示。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/git-cd585383.png" style="zoom: 50%;" />

## 回滚 Reset

### 撤销提交

运行 `reset` 时，它要做的第一件事是移动 HEAD 引用，注意，移动的是 HEAD 引用的分支而非 HEAD 本身。

```
git reset --soft <commit>
```

对任何带提交参数的调用，`reset` 第一步都是做以上动作，使用 `--soft` 选项，命令执行到这就会停下来，所以 `reset --soft` 等于撤销上一次提交，现在重新提交，等同于做了 `git commit --amend`。

> 参数 `commit` 指定提交快照，通常是校验和（前 6 位）或 HEAD 加 ~ 号，每一个 ~ 表示向前一个版本。

### 取消暂存

更进一步，如果使用 `--mixed` 选项，`reset` 将使用 HEAD 当前指向的快照更新索引的内容，即把暂存区恢复成上一次提交的样子，这也是默认行为。所以，`reset --mixed` 等于撤销上一次提交并取消所有暂存。

```
git reset --mixed <commit>
```

### 恢复修改

再进一步，如果使用 `--hard` 选项，Git 将用 HEAD 当前所指快照更新工作区，它会强制覆盖修改了的文件，非常危险。

```
git reset --hard <commit>
```

### 路径重置

除了 `commit`，其实还能提供一个作用路径，这时 `reset` 将会跳过第一步，不移动 HEAD 引用，并且将它的作用范围限定为指定的文件或文件集合。最后结果是暂存区和工作区被恢复成指定版本的样子。

```
git reset --mixed <commit> <filepath>
```

## 检出 Checkout

命令 `checkout` 也会操纵 3 棵树，但又有一些不同，这取决于传给它的参数。

### 带分支

传入一个分支，`checkout` 会同时更新 3 棵树，类似 `git reset --hard [branch]`，但有两点不同，`checkout` 对工作区安全，它会通过检查确保不把已修改的文件弄丢，甚至还会先尝试一下简单合并；另外，`checkout` 移动 HEAD 自身而非 HEAD 引用的分支。

```
git checkout <branch>
```

这样来看，命令 `checkout` 更实用，因为通常只想恢复数据，而不把两个分支混在一起，如下所示。

<img src="https://images-1305875271.cos.ap-chengdu.myqcloud.com/git-2ac0d2f5.png" style="zoom:50%;" />

### 带路径

如果传入一个路径，它与 `reset` 一样不移动 HEAD，使用指定的快照更新暂存区和工作区的文件，工作区的文件可能会被覆盖，不安全。

```
git checkout <commit> <filepath>
```
