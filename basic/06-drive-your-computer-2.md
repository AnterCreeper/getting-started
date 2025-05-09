# 用好你的电脑 II

::: info 本文信息
作者：000lbh

状态：未审阅的草稿版本

许可：CC BY-NC-SA 3.0
:::

## 版本控制概览

你是否正在编写项目，希望反复尝试不同的代码的效果？你是否曾经为了调整功能删除过大段代码，后悔时却发现无法找回？你是否想和其他人合作开发项目，却发现代码难以同步？本章你将学习 Git 这一版本控制系统，可以解决以上问题。

版本控制系统（Version Control System, VCS）用来管理和追踪一个软件的源文件版本的系统，同时也可以提供协作、备份等功能。其可以分为中心化和去中心化两种工作方式。

### 中心化版本控制

中心化的工作方式必须有一个服务器，储存所有的版本记录，客户端只负责拉取某个版本，进行修改，并推送回去。代表作有 SVN。

### 去中心化的版本控制

而去中心化的工作方式中，每个人都有完整的版本记录，可以存在中心服务器用于交换各个客户端的提交，但是即使服务器下线或者不存在，两个人之间也可以通过互相交换信息来完成版本同步。代表作有 Git。请注意，Git 和 GitHub，GitLab 并不是同一个东西，前者是 VCS，后者是使用 Git 作为 VCS 的代码托管平台。

## Git 的基本理念

在介绍 Git 基本理念之前，我们先讲一点故事。2002 年以前，Linux 内核开发完全依赖于 Linus 一个人手工检查并合并全世界发来的补丁，这样工作量非常大。于是，Linus 的一个朋友介绍了 BitMover 公司开发的商业 VCS 软件 BitKeeper 免费授权给 Linux 开发团队使用。此举招致了 FSF 的 RMS 等人的批评，认为在自由软件开发中使用非自由软件是“道德上有污点”的行为，但是作为实用主义者的 Linus 并不在意这些事情，BitKeeper 作为去中心化的 VCS，满足了 Linus 的需求。然而好景不长，有 Linux 内核开发者逆向了 BitKeeper 的协议，致使 BitMover 公司在 2005 年决定收回其授权。Git 就是在这种条件下诞生的，据说第一版 Git 是 Linus 利用 1 周休假时间完成的。

Git 的设计出于这样一种基本抽象：一个项目的历史记录可以被看作是一个有向无环图（DAG），每个提交是一个节点，每个节点有一个或多个父节点，代表这个提交是由哪些提交衍生出来的。Git 的基本操作就是在这个图上进行操作，比如创建新的节点，删除节点，合并节点等等。或许同学们不熟悉有向无环图这个概念，我们举个例子：家谱就可以类比为一个有向无环图，每个人是一个节点，每个人有父母，父母又有父母，但是不可能有一个人的父母是他自己，也不会有一个人的父母是他的后代，这样就构成了一个有向无环图。

这样的抽象是自然的：人们写的代码大概率是基于之前写过的一份或多份代码，写完之后，又会有其他人基于这份代码继续开发，具有继承性。这样的抽象也是实用的，每一次提交（一个节点）都可以看作是一个快照，你可以随时回到这个快照，查看这个快照的内容，或者基于这个快照进行修改。你也能知道目前的状态是如何从过去的状态演变而来的，这样你就可以知道每一次修改的意图，也可以知道每一次修改的后果。

对于 Git 来说，有三个目录：工作区（Working Directory），暂存区（Staging Area）和版本库（Repository）。工作区就是你的项目目录，你可以随意改动，直到你决定记录你的修改。版本库是 Git 存储有向无环图的地方。暂存区可能不那么好理解，暂存区是一个缓冲区，你可以把你的修改放到暂存区，然后一次性提交到版本库，差不多就是这样：

<p align="center">
<img src="https://git-scm.com/book/en/v2/images/areas.png" alt="areas" width="400"/>
</p>

有点抽象，我们举个例子：

我们假设有一个 Git 仓库（Repository），里面有两个文件 A 和 B。仓库之前有提交记录。此时你开始基于之前的提交记录工作，你从历史中取出了文件 A 和 B 放到工作区（这其实是自动的）。此刻就像这样：

<p align="center">
<img src="/assets/basic/06-drive-your-computer-2/git-graph-no-modified.png" alt="areas" width="300"/>
</p>

然后你修改了文件 A，这对 Git 的状态没有任何影响：因为你没有告诉 Git 你修改了文件 A。这时候你可以把文件 A 放到暂存区，这样 Git 就知道你修改了文件 A。这时候 Git 的状态是这样的：

<p align="center">
<img src="/assets/basic/06-drive-your-computer-2/git-a-changed.png" alt="areas" width="300"/>
</p>

然后你修改了文件 B，并把 B 放到暂存区，这时候 Git 的状态是这样的：

<p align="center">
<img src="/assets/basic/06-drive-your-computer-2/git-b-changed.png" alt="areas" width="300"/>
</p>

你觉得差不多了，这时你打算永久保存工作区目前的状态，就把暂存区提交到版本库，这时候 Git 的状态是这样的：

<p align="center">
<img src="/assets/basic/06-drive-your-computer-2/git-commited.png" alt="areas" width="300"/>
</p>

你的暂存区已经被保存到了版本库，就是版本 Y 节点。这时候工作区和版本库最新节点一致，暂存区是空的。

## Git 的使用

下面我们分步介绍 Git 的使用方法：

### 初始化仓库

我们使用 init 子命令来初始化一个仓库。打开你的 shell，执行：

```shell
mkdir git-example
cd git-example
git init
```

你可能会看到以下内容：

```plain
已初始化空的 Git 仓库于 /path/to/example/git-example/.git/
```

这说明一个空的 git 仓库已经创建好了。

如果你看到如下内容，意思是你系统的 git 默认选择了 master 作为主分支的名字。目前我们推荐使用 main 作为主分支的名字，你可以根据它的建议进行配置：

```plain
提示： 使用 'master' 作为初始分支的名称。这个默认分支名称可能会更改。要在新仓库中
提示： 配置使用初始分支名，并消除这条警告，请执行：
提示：
提示：  git config --global init.defaultBranch <名称>
提示：
提示： 除了 'master' 之外，通常选定的名字有 'main'、'trunk' 和 'development'。
提示： 可以通过以下命令重命名刚创建的分支：
提示：
提示：  git branch -m <name>
```

### 配置 Git

我们可能需要对仓库进行一些配置，比如设置用户名和邮箱，设置代理等等。

配置 Git 只需要用到 config 子命令。如果需要修改全局设置，可以加上 `--global` 参数，如果需要打开配置文件进行编辑，可以加上 `--edit` 参数。现在我们修改一下全局参数，执行：

```shell
git config --global --edit
```

然后你的终端应该会打开一个文本编辑器（可能是 vim，在 Windows 上也可能是记事本之类的），然后在 `[user]` 模块下找到 `name = xxx` 和 `email = xxx@xxx`，将两者修改为自己的信息。如果这两行不存在，你可以在 `[user]` 后另起一行，加上这两行信息，如果 `[user]` 也不存在，你可以在文件末尾另起一行加上。很多代码托管平台，比如 GitHub，使用提交的邮箱判断提交的作者。

由于众所周知的原因，你可能需要使用代理。请在文件末尾另起一行，填写以下内容，其中链接需要填写你自己的链接：

```plain
[http]
    proxy = http://127.0.0.1:7890（请更改为你自己的链接）
[https]
    proxy = http://127.0.0.1:7890（同上）
```

有时候对于某些 repo，你想使用其他的名称或者邮箱进行提交，这时你可以在 repo 目录中执行：

```shell
git config --edit
```

用和全局配置类似的方法，配置你的用户名和邮箱。

### 暂存你的更改

使用 add 子命令可以暂存某一个文件的更改，以便后续提交。先试着在目录中创建一个文件，内容是 Hello, world!，然后暂存：

```shell
echo Hello, world! > example1.txt
git add .
```

使用`.`指示所有未被忽略的文件，你也可以写出具体的文件路径进行暂存。

执行：

```shell
git status
```

查看当前分支的状态，如果你前面操作全部正确，你应该看到如下内容：

```plain
位于分支 master

尚无提交

要提交的变更：# 这里就是暂存区，笔者注
  （使用 "git rm --cached <文件>..." 以取消暂存）
        新文件：   example1.txt
```

### 提交你的更改

使用 commit 子命令来提交你的更改。执行：

```shell
git commit
```

会弹出文本编辑器，请在第一行写你的提交信息，比如 `My first commit`，然后退出编辑器。或者你也可以执行：

```shell
git commit -m "My first commit
```

达到同样的效果。此时你应该能看到如下信息：

```plain
[master（根提交） 7a6ab77] My first commit
 1 file changed, 1 insertion(+)
 create mode 100644 example1.txt
```

这个时候我们再执行一遍：

```shell
git status
```

你会发现目前处于“干净的工作区”

```plain
位于分支 master
无文件要提交，干净的工作区
```

总结一下，git 整体的工作流程就是修改-暂存-提交-下一轮修改-……这样一直进行。

如果你觉得暂存操作比较麻烦，可以加上 `-a` 参数，此参数会在提交前自动暂存修改过和删除的文件，但是新的文件不会被包括进来。

有时候上一个提交还没有完成，你可以使用 `--amend` 参数修订上一个提交。

### 查看并回退到指定历史版本

在开始讲解之前，我们再建立一个提交，方便后续讲解。执行：

```shell
echo Hello, Git! > example1.txt
git commit -am "My second commit"
```

如果你之前都是按照教程完成的，你应该可以看到：

```plain
[master 37f7d83] My second commit
 1 file changed, 1 insertion(+), 1 deletion(-)
```

然后我们使用 log 子命令，执行：

```shell
git log
```

你应该可以看到类似以下内容：

```plain
commit 37f7d83baa4f765071daad0a316b8ec380fcedb3 (HEAD -> master)
Author: 000lbh <73009215+000lbh@users.noreply.github.com>
Date:   Wed Aug 28 14:41:19 2024 +0800

    My second commit

commit 7a6ab774caa62ba9d0a091a2c1dc3e96af04ffa7
Author: 000lbh <73009215+000lbh@users.noreply.github.com>
Date:   Wed Aug 28 14:28:30 2024 +0800

    My first commit
```

其中
commit 后面跟着的编号（实际上是散列值）、日期会不同，作者和邮箱信息应该是你刚刚设置的。

此时我们想检查第一个提交，这个时候我们可以使用多种方式来完成，我们先使用 checkout 子命令：

```shell
git checkout 7a6ab774caa62ba9d0a091a2c1dc3e96af04ffa7
```

此时该提交被检出，当前工作区应该回到上一个提交的状态，显示：

```plain
注意：正在切换到 '7a6ab774caa62ba9d0a091a2c1dc3e96af04ffa7'。

您正处于分离头指针状态。您可以查看、做试验性的修改及提交，并且您可以在切换
回一个分支时，丢弃在此状态下所做的提交而不对分支造成影响。

如果您想要通过创建分支来保留在此状态下所做的提交，您可以通过在 switch 命令
中添加参数 -c 来实现（现在或稍后）。例如：

  git switch -c <新分支名>

或者撤销此操作：

  git switch -

通过将配置变量 advice.detachedHead 设置为 false 来关闭此建议

HEAD 目前位于 7a6ab77 My first commit

```

你可以使用

```shell
cat example1.txt
```

检查文件内容。

事实上，使用散列值指定提交时，若无歧义，写前 5 个字符即可。

如果你想回到最新的提交，执行：

```shell
git checkout master
```

即可。

如果你想回退到当前提交，可以使用

```shell
git reset --hard 7a6ab
```

此命令将签出并将头指针指向指定提交。后续提交除非你知道提交的散列值，否则你无法找回提交。可以使用垃圾回收（gc 子命令）清除未被引用的提交。

::: danger 警告
请谨慎使用 hard reset
:::

### 排除掉特定的文件

有时候一些文件不应该被版本管理系统追踪，如编译生成的目标文件，可执行文件，一些敏感配置等等。我们可以使用 `.gitignore` 文件来排除指定文件和文件夹。执行以下内容：

```shell
mkdir confidential
echo Password is not a good password > confidential/password.txt
echo This is pretend to be a object file > main.o
git status
```

应该可以看到以下内容：

```plain
位于分支 master
未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）
        confidential/
        main.o

提交为空，但是存在尚未跟踪的文件（使用 "git add" 建立跟踪）
```

我们再执行：

```shell
echo confidential\n\*.o > .gitignore
git status
```

应该可以看到

```plain
位于分支 master
未跟踪的文件:
  （使用 "git add <文件>..." 以包含要提交的内容）
        .gitignore

提交为空，但是存在尚未跟踪的文件（使用 "git add" 建立跟踪）

```

可以发现 `credential` 目录和所有的 `.o` 文件都被忽略了。

最后我们执行

```shell
git add .
git commit -m "add .gitignore"
```

将这一修改纳入 VCS 进行管理，完成这一节。

### 分支管理

有时候我们会想同时开发新功能，并且调优以前的代码，这样可能就需要两条线进行开发，此时分支相关的功能就会很有帮助。分支的英文是 branch，其实就像树枝一样，你可以在树干上开出一个新的树枝，然后在这个树枝上进行开发，多条树枝之间不会影响，同时树枝也可以合并到树干上（这个有点不符合自然常理，不过你可以做一些想象）。

接下来的例子，我们将演示如何创建分支、变基分支、合并分支以及冲突解决。

首先我们执行：

```shell
git log
```

查看目前分支的记录，参考结果如下：

```plain
commit 714d500c7f5f15445bfa59a0d04d01e177602db5 (HEAD -> master)
Author: 000lbh <73009215+000lbh@users.noreply.github.com>
Date:   Wed Aug 28 15:18:36 2024 +0800

    add .gitignore

commit 37f7d83baa4f765071daad0a316b8ec380fcedb3
Author: 000lbh <73009215+000lbh@users.noreply.github.com>
Date:   Wed Aug 28 14:41:19 2024 +0800

    My second commit

commit 7a6ab774caa62ba9d0a091a2c1dc3e96af04ffa7
Author: 000lbh <73009215+000lbh@users.noreply.github.com>
Date:   Wed Aug 28 14:28:30 2024 +0800

    My first commit

```

#### 创建分支

我们想以第二个提交为根节点，向上延伸分支，我们可以执行：

```shell
git checkout -b update-example 37f7d
```

以上等价于执行

```shell
git branch update-example 37f7d
git checkout update-example
```

然后我们将文件 `example1.txt` 改为 `Hello, Git2!`，执行：

```shell
git commit -am "Branch!"
```

提交更改。

#### 变基分支

接着我们将刚刚创建的提交变到主线上，如下图所示：

```plain
A-----B-----C (master)
       \                  A-----B-----C-----D  (update_example)
        \            ===>         (master)
         D    (update-example)
```

只需执行

```plain
git rebase master
git checkout master
```

分支 update-example 将重新以 master 的最新提交为根基。
请注意，rebase 会使得移动的全部提交的散列值被重新计算！因为 git 提交的散列值与上一个提交的散列值有关。

#### 合并分支与冲突解决

合并分支是一种比较有意思的操作，因为其会产生一种叫做合并提交的提交。
合并提交本身的特别性在于，其具有多于一个的父提交，因此可以将两个分支合并到一起。

我们将 master 的 HEAD 设置到刚刚 rebase 后的分支的顶部，然后我们新建一个分支：

```shell
git checkout -b merge-example
echo Lorem ipsum > example2.txt
echo Hello, Git6! > example1.txt
git add example2.txt
git commit -m "Prepare to merge"
```

然后我们检出 master，然后执行：

```plain
git merge merge-example
```

由于合并的两个分支涉及同一行的修改，git 没有办法决定如何应用这些修改，因此需要手动介入解决。
变基操作也会出现冲突，感兴趣的同学可以尝试一下如何解决。
如果不出意外，你应该看到：

```plain
自动合并 example1.txt
冲突（内容）：合并冲突于 example1.txt
自动合并失败，修正冲突然后提交修正的结果。
```

我们打开 `example1.txt` 查看内容：

```plain
<<<<<<< HEAD
Hello, Git2!
=======
Hello, Git6!
>>>>>>> merge-example

```

如果你使用 VSCode 等 IDE，应该已经自动显示修正冲突的选项。我们在这里把结果修正为：

```plain
Hello, Git8!
```

然后我们运行：

```shell
git add .
git merge --continue
```

应该会弹出一个文本编辑器，编辑合并提交的消息，然后退出即可。

### Git 服务器与多人合作

到这里你已经完成了 Git 大部分基础功能的学习！下面我们看看如何用 Git 进行多人合作：

#### 克隆仓库

克隆就是把别人的代码仓库复制一份过来。一般来说，执行：

```shell
git clone url://path/to/be/cloned
```

就可以了。在当前工作目录下会新建克隆项目名字的文件夹，一般 http(s)和 ssh 协议是常见的 clone 协议。对于后者，你可能需要本地生成 ssh 密钥对，并将公钥上传到服务器。

#### 拉取代码

有时候远端代码库已经更新，你需要更新本地代码，这时候用 pull 子命令。

```shell
git pull
```

事实上，pull 子命令同时执行了 fetch 然后将当前分支的 HEAD 指针指向远端对应分支的 HEAD 指针。

如果本地有远端不存在的提交，则拉取代码不能以默认的 “fast-forward” 方式进行，因此需要指定 `--no-ff` 参数进行合并拉取或者指定 `--rebase` 进行变基拉取。在特别有必要时，也可以直接 hard reset 到远端 HEAD 处，丢弃本地未上传的提交。

由于本地的提交可以很方便的进行变基，不用担心散列值重新计算带来的合作上的冲突，建议出现拉取冲突时使用变基的方式解决，而合并会引入不必要的非线性历史，可能会让历史记录不太简洁。

::: danger 警告
再次提醒，请谨慎使用 hard reset。如果你已经弄丢了一些提交，可以通过提交 hash 找回你的提交。
:::

#### 推送代码

在工作完成，提交完成之后，可以用这个子命令将修改推送至远端。若有远端有本地没有的提交，需要先进行拉取，才能推送，或者 `--force` 强制推送，此时不一致的提交会被本地提交代替。

::: danger 警告
使用 `--force` 参数前请三思，仔细检查你将要提交的内容！远端可能配置了自动 gc，在这种情况下，你的提交再也无法找回！
:::

#### 分叉与合并请求/拉取请求

如果你没有一个远程仓库的写权限而又想贡献代码，可以考虑通过代码托管平台提供的分叉(fork)功能，分叉一份代码进行开发，分叉的仓库与原来的仓库独立，一般来说，如果原仓库被删除，分叉的仓库会失去与原仓库的联系，但是并不会被删除。一般情况下，在分叉的仓库中进行开发时也不要使用默认分支（一般叫 master 或者 main），部分代码托管平台拒绝分叉中的默认分支的合并请求被自动合并。

开发完成之后，你可以发起合并请求(Merge Request, MR)/拉取请求(Pull Request, PR)，向原仓库提交你的修改。原仓库的作者可能会经过一系列审查之后，选择合并你的代码，提出修改意见或者拒绝并关闭你的请求，不过一般直接关闭一个合并请求并不是很礼貌，所以原仓库的作者更可能提出拒绝的意见，此时你可以说服他或者自行关闭请求。

除了分叉可以发起合并请求/拉取请求，项目仓库内存在的分支也可以发起向其他分支的合并请求。这一般适用于对仓库有写权限的人，为了审查讨论代码修改，避免意外，不直接将修改放入默认分支而是先使用其他分支进行开发，通过合并请求/拉取请求再将代码放入默认分支。

如果你是仓库的所有者，你可能会收到合并请求/拉取请求，一般的代码托管平台提供三种合并方式：

- 合并(merge)：创建一个基于提出请求的分支和目标分支的合并提交（见前面合并分支部分的说明）
- 变基(rebase)：将提出请求的分支中的相关提交的修改内容依次应用到目标分支上
- 压缩(squash)：将提出请求的分支中的相关提交的修改内容作为一个提交应用到目标分支上

请注意，变基的合并方式不会把提出请求的分支中的提交原样纳入到目标分支上，因为变基需要重新计算每个提交的散列值（部分代码托管平台对于可以快进的情况变基不会重新计算散列值），因此基于散列值的内容，比如提交签名，可能会失效。

如果需要线性历史，建议不使用合并的请求合并方式。

### 图形化工具的使用

1. VSCode

   VSCode 自带 Git 管理功能，可以使用该功能进行可视化编辑和提交

2. gitg

   Gnome 桌面的 git 管理软件

3. kommit

   KDE 桌面的 git 管理软件

## 结语

Git 部分的介绍就暂告一段段落了。Git 本身有上百个子命令，本教程肯定无法完全覆盖，很多高级用法自然也没有办法介绍。更多信息可以访问[Git Book](https://git-scm.com/book/en/v2)学习，也可以参考 man 手册。
