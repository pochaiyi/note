# Git

Git 是免费、开源的分布式版本控制工具，相对的，还有集中式版本控制工具。

版本控制：记录文件内容的变化，以方便将来查阅特定版本的修订情况 。

Git 客户端可以使用 Linux 命令。

# 安装配置

## Git 配置

Git 提供有 `git config` 工具，用于配置和读取工作环境变量。这些环境变量决定了 Git 的工作方式和行为。它们可以存放在三个地方：

* `/etc/gitconfig` 文件：适用于系统所有用户。使用 `git config` 时选择 `--system`，读写的就是该文件。
* `~/.gitconfig` 文件：只适用于单个用户。使用 `git config` 时选择 `--global`，读写的就是该文件。
* 本地仓库 `.git` 目录中的 `config` 文件：仅针对当前仓库有效。

每一级都会覆盖上一层相同的配置。

在 Windows 系统，Git 会寻找用户目录中的 `.gitconfig` 文件。此外，Git 还会尝试寻找 `/etc/gitconfig` 文件，以 Git 的安装目录作为根目录来定位。

## 查看配置

查看已有的配置信息：

```
git config [选项] --list

--system
--global
```

如果没有选项，则列出所有配置信息，这会返回名称相同而位置不同的环境变量，Git 采用最后的那个。

也可以查看特定的环境变量：

```
git config [选项] 变量名

--system
--global
```

编辑当前局部|全局的配置文件：

```
git config -e [--global]
```

## 用户信息

配置当前 Git 用户的签名：

```
git config --global user.name username	# 签名
git config --global user.email	# 邮箱
```

用户签名的作用是区分操作者身份，Git 必须明确每次操作都是谁执行的。

用户签名和 GitHub（或其他代码托管中心）的账号没有任何关系，仅代表一个 Git 客户端。

## 其它配置

* 设置文本编辑器，如 vi，vim 等：

  ```
  git config --global core.editor vim
  ```

* 设置差异分析工具，用于解决合并冲突：

  ```
  git config --global merge.tool vimdiff

# 版本控制

**工作流程：工作区	add->	暂存区	commit->	本地仓库	push->	远程仓库**

* 工作区：本地仓库当前的文件。

* 暂存区：存放在 `.git` 目录的 index 文件，所以也把暂存区叫做索引。
* 版本库：工作区的隐藏目录 `.git` 。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/git-e1ec9c82.png)

图中左侧为工作区，右侧为版本库，其中标记为 `index` 的是暂存区，标记为 `master` 的是 `master` 分支所代表的目录树。`objects` 标识的区域为 Git 的对象库，实际位于 `.git/objects` 目录，里面包含了创建的各种对象及内容。

`Head` 是指向当前分支当前版本的游标，此时它就指向 `master`。

对工作区修改的文件执行 `git add` 命令时，暂存区的目录树会被更新。同时工作区修改的文件内容被写入对象库的一个新对象中，该对象的 ID 被记录在暂存区的文件索引中。

执行提交操作 `git commit` 时，暂存区的目录树被写到版本库，`Head` 指向的分支会做相应的更新。即 `master` 指向的目录树会更新为暂存区的目录树。

## 版本回退 Git Reset

`git reset` 命令可以将当前的 `HEAD` 指向当前分支特定的版本。

`git reset` 命令有三种参数，默认选择 `--mixed`，它们的区别在于是否会影响工作区和暂存区。

```
git reset [--soft| --mixed| --hard] <Head版本>
```

* `--mixed`：默认。重置 `Head` 到特定版本，同时重置暂存区到相同版本，不影响工作区。

* `--soft`：重置 `Head` 到特定版本，不影响工作区和暂存区。

* `--hard`：重置 `Head` 到特定版本，同时重置工作区、暂存区到指定版本。

可以使用 `Head` 和 `^` 表示当前分支之前的版本：

* `Head`：当前版本
* `Head^`：上个版本
* `Head^^`：上上个版本

也可以用 `Head` 和数字表示：

* `Head~0`：当前版本
* `Head~1`：上个版本
* `Head~2：上上个版本

此外，还可以使用精确的版本号进行版本回退：

```
git reset fc3f635
```

版本号可以通过 `git reflog`，`git log` 在日志中查看。

所以，可以使用 `git reset HEAD` 清空暂存区的修改数据，使用 `git reset --hard HEAD` 清空工作区和暂存区的修改数据。

# 基本操作

* 查看本地仓库状态：

  ```
  git status [-s]
  ```

  默认展示当前分支，提交记录，暂存区文件等，`-s` 表示返回简短内容。

## 创建仓库

* 初始化仓库，当前目录或指定目录，生成 `.git` 文件夹：

  ```
  git init
  ```

* 克隆远程仓库，默认会创建与远程仓库同名的目录，也可以指定目录名称：

  ```
  git clone <repo> [directory]
  ```

## 提交与修改

* 从工作区添加全部或指定文件到暂存区：

  ```
  git add [.] [file1] [file2]
  ```

* 将文件从暂存区和工作区删除：

  ```
  git rm <file>

* 将文件从暂存区删除：

  ```
  git rm --cached [file1] [file2]

* 提交暂存区全部或部分文件到本地仓库：

  ```
  git commit [file1] [file2] -m [message]
  ```

* 设置修改文件后，不需要执行 `git add`，直接提交：

  ```
  git commit -a
  ```

* 移动或重命名一个文件、目录或软连接：

  ```
  git mv [file] [newfile]

## 查看差异

* 查看暂存区和工作区的差异：

  ```
  git diff [file]
  ```

* 查看暂存区和上次 `commit` 的差异：

  ```
  git diff --cached [file]
  ```

* 查看分支之间的差异：

  ```
  git diff [first-branch]...[second-branch] # ...是固定格式，不是省略号
  ```

## 查看日志

* 查看历史提交记录：

  ```
  git log
  ```

* 查看历史记录的简洁的版本：

  ```
  git log --oneline
  ```

  ```
  git reflog

* 以列表形式查看指定文件的历史修改记录：

  ```
  git blame <file>

# 分支管理

## 本地分支

使用分支意味着可以从开发主线上分离开来，在不影响主线的同时继续工作。

Git 仓库总有一个主分支，名为 `master`。

* 查看分支：

  ```
  git branch [-选项]
  
  -v 查看当前分支的版本号和最后一次提交注释
  -a 查看远程分支
  ```

  在当前分支的前面会标注 `*`。

* 创建分支：

  ```
  git branch [选项] <branchname>
  
  -b 创建后切换到新分支
  ```

  新分支的初始内容与当前分支此刻的内容相同。

* 切换分支：

  ```
  git checkout <branchname>
  ```

  切换分支时，Git 会用该分支最后提交的快照替换工作目录的内容， 所以多个分支不需要多个目录。

* 合并分支：

  ```
  git merge
  ```

* 删除分支：

  ```
  git branch -d <branchname>
  ```

* 将指定分支合并到当前分支：

  ```
  git merge <branchname>
  ```

  合并时如果某文件发生合并冲突，查看该文件，可以发现其内容已经被 Git 修改，你能够轻易地看到冲突的地方，以及当前分支与合并分支的内容。根据需要，删减它们。

  最后 `git add` 发生冲突的文件，以告诉 Git 冲突已解决。

## 远程跟踪分支

本地仓库有两种分支：本地分支和远程跟踪分支，通常我们谈到的和使用的分支是本地分支。

远程跟踪分支是远程分支状态的引用。它们是你无法移动的本地引用。一旦你进行了网络通信， Git 就会为你移动它们以精确反映远程仓库的状态。请将它们看做书签， 这样可以提醒你该分支在远程仓库中的位置就是你最后一次连接到它们的位置。

它们以 `<remote>/<branch>` 的形式命名。 例如，如果你想要看你最后一次与远程仓库 `origin` 通信时 `master` 分支的状态，你可以查看 `origin/master` 分支。 你与同事合作解决一个问题并且他们推送了一个 `iss53` 分支，你可能有自己的本地 `iss53` 分支， 然而在服务器上的分支会以 `origin/iss53` 来表示。

这可能有一点儿难以理解，让我们来看一个例子。 假设你的网络里有一个在 `git.ourcompany.com` 的 Git 服务器。 如果你从这里克隆，Git 的 `clone` 命令会为你自动将其命名为 `origin`，拉取它的所有数据， 创建一个指向它的 `master` 分支的指针，并且在本地将其命名为 `origin/master`。 Git 也会给你一个与 `origin` 的 `master` 分支指向同一个地方的本地 `master` 分支，这样你就有工作的基础。

# 远程操作

## 操作远程仓库 Git Remote

* 查看当前配置的远程仓库：

  ```
  git remote
  
  origin
  ```

  **origin** 为远程主机的默认别名，加上参数 `-v`，还可以查看每个别名的实际链接地址：

  ```
  git remote -v
  
  origin  https://gitee.com/pochaiyi/harmp.git (fetch)
  origin  https://gitee.com/pochaiyi/harmp.git (push)
  ```

* 查看指定远程仓库的信息：

  ```
  git remote show [url]
  ```

* 添加远程仓库：

  ```
  git remote add [shortname] [url] # shortname是本地版本库
  ```

* 删除远程仓库：

  ```
  git remote rm name
  ```

* 修改仓库名：

  ```
  git remote rename old_name new_name
  ```

## 更新远程跟踪分支 Git Fetch

* 使用远程分支更新远程跟踪分支：

  ```
  git fetch [alias]
  ```

* 将远程跟踪分支合并到当前分支：

  ```
  git merge [alias]/[branch]
  
  eg：git merge origin/master
  ```

## 拉取远程仓库 Git Pull

`git pull` 是 `git fetch` 和 `git merge FETCH_HEAD` 的简写，`FETCH_HEAD` 是刚刚从远程仓库更新内容的游标。

```
git pull <远程主机名> <远程分支名>:<本地分支名>
```

如果合并的分支是当前分支，则冒号后面部分可省略：

```
git pull <远程主机名> <远程分支名>
```

## 推送合并 Git Push

将本地分支推送到远程分支：

```
git push <远程主机名> <本地分支名>:<远程分支名>
```

如果本地分支名与远程分支名相同，则冒号后面部分可省略：

```
git push <远程主机名> <本地分支名>
```

如果本地版本与远程版本有差异，但又要强制推送可以使用 `--force` 参数：

```
git push --force <远程主机名> <本地分支名>
```

删除远程仓库的分支可以使用 `--delete` 参数，以下命令表示删除 `origin` 主机的 `master` 分支：

```
git push origin --delete master
```

如果当前分支与远程分支存在追踪关系（即分支名相同），则可以省略分支名称：

```
git push origin
```

如果当前分支只有一个追踪分支，那么远程主机名都可以省略：

```
git push
```

如果当前分支与多个主机存在追踪关系，则可以使用 `-u` 参数指定默认主机，后面的推送就可以不加任何参数：

```
git push -u origin master
```

# 远程仓库

Git 是分布式的版本管理工具，如果想将本地仓库的内容与人分享，需要将本地的内容推送到一台公共的服务器。比如 GitHub 等。

## SSH 免密登录

1. 添加远程仓库，仓库地址必须是 SSH url：

   ```
   git remote add [shortname] [url] # shortname是本地版本库
   ```

2. 由于本地 Git 仓库和远程仓库之间的传输是通过 SSH 加密的，所以需要配置验证信息：

   ```
   ssh-keygen -t rsa -C "2837141937@qq.com"
   ```

   邮箱地址为远程仓库创建者的注册邮箱

   后面设置全部默认回车即可。成功后会在  `~/` 目录生成 `.ssh` 文件夹，其中 `id_rsa.pub` 文件内容就是 `key`。

3. 回到远程仓库中心，进入用户设置，选择 SSH 公钥，添加前面生成的公钥即可。

4. 回到终端，输入：

   ```
   ssh -T git@gitee.com
   ```

   首次使用需要确认并添加主机到本机 SSH 可信列表。若返回 `Hi XXX! You've successfully authenticated, but Gitee.com does not provide shell access.` 内容，则证明添加成功。

5. 现在，便可以不输入密码直接 `push` 本地分支到远程分支。

## Windows 凭据管理器

1. 添加远程仓库

   ```
   git remote add [shortname] [url] # shortname是本地版本库
   ```

2. 首次 `push` 会要求身份验证，win 系统凭据管理器会存储远程仓库中心（GitHub）的用户信息。

   ```
   GitHub 仓库成员管理：settings -> manage access

# ignore 配置

创建并修改 *.gitignore* 文件，文件名任意，但后缀固定。语法：

```
# 注释
/ 开头表示目录
* 匹配多个任意字符
? 匹配单个任意字符
[] 匹配单个列表中的字符
! 不能忽略匹配的文件或目录
```

示例：

```
/.idea
/target
*.iml
*.jar
```

## 全局设置

全局配置文件设置 *ignore* 策略：

```
excludesfile = ignore文件路径
```

Git 命令全局设置 *ignore* 策略：

```
git config --global core.excludesfile ignore文件路径
```

## 局部设置

* 在本地仓库创建 *.gitignore* 文件。

* 设置本地仓库的 *ignore* 策略，两种方式：
  * 在本地仓库的配置文件 */.git/config* 内添加 `excludesfile = ignore文件路径`，根目录是本地仓库目录。
  * Git 命令：`git config core.excludesfile .gitignore`。
