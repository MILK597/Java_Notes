# 目录

[TOC]





# 配置git

​	Git 使用一系列配置文件来保存你自定义的行为。 它首先会查找系统级的 **`/etc/gitconfig`** 文件，该文件含有系统里每位用户及他们所拥有的仓库的配置值。 如果你传递 **`--system`** 选项给 `git config`，它就会读写该文件。

​	接下来 Git 会查找每个用户的 **`~/.gitconfig`** 文件（或者 `~/.config/git/config` 文件）。 你可以传递 **`--global`** 选项让 Git 读写该文件。

​	最后 Git 会查找你正在操作的仓库所对应的 Git 目录下的配置文件（**`.git/config`**）。 这个文件中的值只对该仓库有效，它对应于向 `git config` 传递 **`--local`** 选项。

​	以上三个层次中每层的配置（系统、全局、本地）都会覆盖掉上一层次的配置，所以 `.git/config` 中的值会覆盖掉 `/etc/gitconfig` 中所对应的值。



Git 能够识别的配置项分为两大类：客户端和服务器端。

## 客户端基本配置

### `core.editor`

​	默认情况下，Git 会调用你通过环境变量 `$VISUAL` 或 `$EDITOR` 设置的文本编辑器， 如果没有设置，默认则会调用 `vi` 来创建和编辑你的提交以及标签信息。 你可以使用 `core.editor` 选项来修改默认的编辑器：

```console
$ git config --global core.editor emacs
```



### `commit.template`

​	如果把此项指定为你的系统上某个文件的路径，当你提交的时候， Git 会使用该文件的内容作为提交的默认初始化信息。 创建的自定义提交模版中的值可以用来提示自己或他人适当的提交格式和风格。

```
	$ git config --global commit.template ~/.gitmessage.txt
```



### `core.pager`

该配置项指定 Git 运行诸如 `log` 和 `diff` 等命令所使用的分页器。 你可以把它设置成用 `more` 或者任何你喜欢的分页器（默认用的是 `less`），当然也可以设置成空字符串，关闭该选项：

```console
$ git config --global core.pager ''
```

这样不管命令的输出量多少，Git 都会在一页显示所有内容。



### `user.signingkey`

如果你要创建经签署的含附注的标签， 那么把你的 GPG 签署密钥设置为配置项会更好。如下设置你的密钥 ID：

```console
$ git config --global user.signingkey <gpg-key-id>
```

现在，你每次运行 `git tag` 命令时，即可直接签署标签，而无需定义密钥：

```console
$ git tag -s <tag-name>
```



### `core.excludesfile`

​	你可以在你的项目的 `.gitignore` 文件里面规定无需纳入 Git 管理的文件的模板，这样它们既不会出现在未跟踪列表， 也不会在你运行 `git add` 后被暂存。

​	不过有些时候，你想要在你所有的版本库中忽略掉某一类文件。 如果你的操作系统是 macOS，很可能就是指 `.DS_Store`。 如果你把 Emacs 或 Vim 作为首选的编辑器，你肯定知道以 `~` 结尾的文件名。

这个配置允许你设置类似于全局生效的 `.gitignore` 文件。 如果你按照下面的内容创建一个 `~/.gitignore_global` 文件：

```ini
*~
.*.swp
.DS_Store
```

……然后运行 `git config --global core.excludesfile ~/.gitignore_global`，Git 将把那些文件永远地拒之门外。



### `help.autocorrect`

假如你打错了一条命令，会显示：

```console
$ git chekcout master
git：'chekcout' 不是一个 git 命令。参见 'git --help'。

您指的是这个么？
  checkout
```

Git 会尝试猜测你的意图，但是它不会越俎代庖。 如果你把 `help.autocorrect` 设置成 1，那么只要有一个命令被模糊匹配到了，Git 会自动运行该命令。

```console
$ git chekcout master
警告：您运行一个不存在的 Git 命令 'chekcout'。继续执行假定您要要运行的
是 'checkout'
在 0.1 秒钟后自动运行...
```

注意提示信息中的“0.1 秒”。`help.autocorrect` 接受一个代表十分之一秒的整数。 所以如果你把它设置为 50, Git 将在自动执行命令前给你 5 秒的时间改变主意。



### git 中的着色

#### `color.ui`

Git 会自动着色大部分输出内容，但如果你不喜欢花花绿绿，也可以关掉。 要想关掉 Git 的终端颜色输出，试一下这个：

```console
$ git config --global color.ui false
```

这个设置的默认值是 `auto`，它会着色直接输出到终端的内容；而当内容被重定向到一个管道或文件时，则忽略着色功能。

你也可以设置成 `always`，来忽略掉管道和终端的不同，即在任何情况下着色输出。 你很少会这么设置，在大多数场合下，如果你想在被重定向的输出中插入颜色码，可以传递 `--color` 标志给 Git 命令来强制它这么做。 默认设置就已经能满足大多数情况下的需求了。

#### `color.*`

要想具体到哪些命令输出需要被着色以及怎样着色，你需要用到和具体命令有关的颜色配置选项。 它们都能被置为 `true`、`false` 或 `always`：

```
color.branch
color.diff
color.interactive
color.status
```

另外，以上每个配置项都有子选项，它们可以被用来覆盖其父设置，以达到为输出的各个部分着色的目的。 例如，为了让 `diff` 的输出信息以蓝色前景、黑色背景和粗体显示，你可以运行

```console
$ git config --global color.diff.meta "blue black bold"
```

你能设置的颜色有：`normal`、`black`、`red`、`green`、`yellow`、`blue`、`magenta`、`cyan` 或 `white`。 正如以上例子设置的粗体属性，想要设置字体属性的话，可以选择包括：`bold`、`dim`、`ul`（下划线）、`blink`、`reverse`（交换前景色和背景色）。



### 格式化与多余的空白字符

#### `core.autocrlf`

假如你正在 Windows 上写程序，而你的同伴用的是其他系统（或相反），你可能会遇到 CRLF 问题。 这是因为 Windows 使用回车（CR）和换行（LF）两个字符来结束一行，而 macOS 和 Linux 只使用换行（LF）一个字符。 虽然这是小问题，但它会极大地扰乱跨平台协作。许多 Windows 上的编辑器会悄悄把行尾的换行字符转换成回车和换行， 或在用户按下 Enter 键时，插入回车和换行两个字符。

Git 可以在你提交时自动地把回车和换行转换成换行，而在检出代码时把换行转换成回车和换行。 你可以用 `core.autocrlf` 来打开此项功能。 如果是在 Windows 系统上，把它设置成 `true`，这样在检出代码时，换行会被转换成回车和换行：

```console
$ git config --global core.autocrlf true
```

如果使用以换行作为行结束符的 Linux 或 macOS，你不需要 Git 在检出文件时进行自动的转换； 然而当一个以回车加换行作为行结束符的文件不小心被引入时，你肯定想让 Git 修正。 你可以把 `core.autocrlf` 设置成 input 来告诉 Git 在提交时把回车和换行转换成换行，检出时不转换：

```console
$ git config --global core.autocrlf input
```

这样在 Windows 上的检出文件中会保留回车和换行，而在 macOS 和 Linux 上，以及版本库中会保留换行。

如果你是 Windows 程序员，且正在开发仅运行在 Windows 上的项目，可以设置 `false` 取消此功能，把回车保留在版本库中：

```console
$ git config --global core.autocrlf false
```

**简言之：`autocrlf` 为 `true` ，那么git在提交时，自动把`\r\n`换成`\n`，检出时，恢复为`\r\n`。**

**如果为`false` ，则不会做任何改动。**

如果为 `input` ，提交时会把 `\r\n` 换成  `\n` ，检出时不做改动



#### `core.whitespace`

Git 预先设置了一些选项来探测和修正多余空白字符问题。 它提供了六种处理多余空白字符的主要选项 —— 其中三个默认开启，另外三个默认关闭，不过你可以自由地设置它们。

默认被打开的三个选项是：`blank-at-eol`，查找行尾的空格；`blank-at-eof`，盯住文件底部的空行； `space-before-tab`，警惕行头 tab 前面的空格。

默认被关闭的三个选项是：`indent-with-non-tab`，揪出以空格而非 tab 开头的行（你可以用 `tabwidth` 选项控制它）；`tab-in-indent`，监视在行头表示缩进的 tab；`cr-at-eol`，告诉 Git 忽略行尾的回车。

通过设置 `core.whitespace`，你可以让 Git 按照你的意图来打开或关闭以逗号分割的选项。 要想关闭某个选项，你可以在输入设置选项时不指定它或在它前面加个 `-`。 例如，如果你想要打开除`space-before-tab` 之外的所有选项，那么可以这样 （ `trailing-space` 涵盖了 `blank-at-eol` 和 `blank-at-eof` ）：

```console
$ git config --global core.whitespace \
    trailing-space,-space-before-tab,indent-with-non-tab,tab-in-indent,cr-at-eol
```

你也可以只指定自定义的部分：

```console
$ git config --global core.whitespace \
    -space-before-tab,indent-with-non-tab,tab-in-indent,cr-at-eol
```

当你运行 `git diff` 命令并尝试给输出着色时，Git 将探测到这些问题，因此你在提交前就能修复它们。 用 `git apply` 打补丁时你也会从中受益。 如果正准备应用的补丁存有特定的空白问题，你可以让 Git 在应用补丁时发出警告：

```console
$ git apply --whitespace=warn <patch>
```

或者让 Git 在打上补丁前自动修正此问题：

```console
$ git apply --whitespace=fix <patch>
```

这些选项也能运用于 `git rebase`。 如果提交了有空白问题的文件，但还没推送到上游，你可以运行 `git rebase --whitespace=fix` 来让 Git 在重写补丁时自动修正它们。

------



## 服务器端配置

### `receive.fsckObjects`

Git 能够确认每个对象的有效性以及 SHA-1 检验和是否保持一致。 但 Git 不会在每次推送时都这么做。这个操作很耗时间，很有可能会拖慢提交的过程，特别是当库或推送的文件很大的情况下。 如果想在每次推送时都要求 Git 检查一致性，设置 `receive.fsckObjects` 为 true 来强迫它这么做：

```console
$ git config --system receive.fsckObjects true
```

现在 Git 会在每次推送生效前检查库的完整性，确保没有被有问题的客户端引入破坏性数据。



### `receive.denyNonFastForwards`

如果你变基已经被推送的提交，继而再推送，又或者推送一个提交到远程分支，而这个远程分支当前指向的提交不在该提交的历史中，这样的推送会被拒绝。 这通常是个很好的策略，但有时在变基的过程中，你确信自己需要更新远程分支，可以在 push 命令后加 `-f` 标志来强制更新（force-update）。

要禁用这样的强制更新推送（force-pushes），可以设置 `receive.denyNonFastForwards`：

```console
$ git config --system receive.denyNonFastForwards true
```

稍后我们会提到，用服务器端的接收钩子也能达到同样的目的。 那种方法可以做到更细致的控制，例如禁止某一类用户做非快进（non-fast-forwards）推送。



### `receive.denyDeletes`

有一些方法可以绕过 `denyNonFastForwards` 策略。其中一种是先删除某个分支，再连同新的引用一起推送回该分支。 把 `receive.denyDeletes` 设置为 true 可以把这个漏洞补上：

```console
$ git config --system receive.denyDeletes true
```

这样会禁止通过推送删除分支和标签 — 没有用户可以这么做。 要删除远程分支，必须从服务器手动删除引用文件。 

------



## git属性

https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E5%B1%9E%E6%80%A7

------

## Git Hooks

​	和其它版本控制系统一样，Git 能在特定的重要动作发生时触发自定义脚本。 有两组这样的钩子：客户端的和服务器端的。 **客户端钩子**由诸如提交和合并这样的操作所调用，而**服务器端钩子**作用于诸如接收被推送的提交这样的联网操作。

### 	安装一个钩子

钩子都被存储在 Git 目录下的 `hooks` 子目录中。 也即绝大部分项目中的 `.git/hooks` 。

把一个正确命名（不带扩展名）且可执行的文件放入 `.git` 目录下的 `hooks` 子目录中，即可激活该钩子脚本。 这样一来，它就能被 Git 调用。

当你用 `git init` 初始化一个新版本库时，Git 默认会在这个目录中放置一些示例脚本。 这些脚本除了本身可以被调用外，它们还透露了被触发时所传入的参数。 所有的示例都是 shell 脚本，其中一些还混杂了 Perl 代码，不过，任何正确命名的可执行脚本都可以正常使用 —— 你可以用 Ruby 或 Python，或任何你熟悉的语言编写它们。 这些示例的名字都是以 `.sample` 结尾，如果你想启用它们，得先移除这个后缀。



### 客户端钩子

#### 提交工作流钩子

##### 1.`pre-commit` 钩子

​	`pre-commit` 钩子在键入提交信息前运行。 它用于检查即将提交的快照，例如，检查是否有所遗漏，确保测试运行，以及核查代码。 如果该钩子以非零值退出，Git 将放弃此次提交，不过你可以用 `git commit --no-verify` 来绕过这个环节。 



##### 2.`prepare-commit-msg` 钩子

`prepare-commit-msg` 钩子在启动提交信息编辑器之前，默认信息被创建之后运行。 它允许你编辑提交者所看到的默认信息。 该钩子接收一些选项：存有当前提交信息的文件的路径、提交类型和修补提交的提交的 SHA-1 校验。 它对一般的提交来说并没有什么用；然而对那些会自动产生默认信息的提交，如提交信息模板、合并提交、压缩提交和修订提交等非常实用。 你可以结合提交模板来使用它，动态地插入信息。



##### 3.`commit-msg` 钩子

`commit-msg` 钩子接收一个参数，此参数即上文提到的，存有当前提交信息的临时文件的路径。 如果该钩子脚本以非零值退出，Git 将放弃提交，因此，可以用来在提交通过前验证项目状态或提交信息。



##### 4.`post-commit` 钩子

`post-commit` 钩子在整个提交过程完成后运行。 它不接收任何参数，但你可以很容易地通过运行 `git log -1 HEAD` 来获得最后一次的提交信息。 该钩子一般用于通知之类的事情。

#### 

#### 电子邮件工作流钩子

你可以给电子邮件工作流设置三个客户端钩子。 它们都是由 `git am` 命令调用的，因此如果你没有在你的工作流中用到这个命令，可以跳到下一节。 如果你需要通过电子邮件接收由 `git format-patch` 产生的补丁，这些钩子也许用得上。

第一个运行的钩子是 `applypatch-msg` 。 它接收单个参数：包含请求合并信息的临时文件的名字。 如果脚本返回非零值，Git 将放弃该补丁。 你可以用该脚本来确保提交信息符合格式，或直接用脚本修正格式错误。

下一个在 `git am` 运行期间被调用的是 `pre-applypatch` 。 有些难以理解的是，它正好运行于应用补丁 *之后*，产生提交之前，所以你可以用它在提交前检查快照。 你可以用这个脚本运行测试或检查工作区。 如果有什么遗漏，或测试未能通过，脚本会以非零值退出，中断 `git am` 的运行，这样补丁就不会被提交。

`post-applypatch` 运行于提交产生之后，是在 `git am` 运行期间最后被调用的钩子。 你可以用它把结果通知给一个小组或所拉取的补丁的作者。 但你没办法用它停止打补丁的过程。



#### 其它客户端钩子

##### 1.`pre-rebase` 钩子

​	运行于变基之前，以非零值退出可以中止变基的过程。 你可以使用这个钩子来禁止对已经推送的提交变基。 Git 自带的 `pre-rebase` 钩子示例就是这么做的，不过它所做的一些假设可能与你的工作流程不匹配。

##### 2.`post-rewrite` 钩子

​	它被那些会替换提交记录的命令调用，比如 `git commit --amend` 和 `git rebase`（不过不包括 `git filter-branch`）。 它唯一的参数是触发重写的命令名，同时从标准输入中接受一系列重写的提交记录。 这个钩子的用途很大程度上跟 `post-checkout` 和 `post-merge` 差不多。



##### 3.`post-checkout` 钩子

​	在 `git checkout` 成功运行后，`post-checkout` 钩子会被调用。你可以根据你的项目环境用它调整你的工作目录。 其中包括放入大的二进制文件、自动生成文档或进行其他类似这样的操作。



##### 4.`post-merge` 钩子

在 `git merge` 成功运行后，`post-merge` 钩子会被调用。 你可以用它恢复 Git 无法跟踪的工作区数据，比如权限数据。 这个钩子也可以用来验证某些在 Git 控制之外的文件是否存在，这样你就能在工作区改变时，把这些文件复制进来。



##### 5.`pre-push` 钩子

`pre-push` 钩子会在 `git push` 运行期间， 更新了远程引用但尚未传送对象时被调用。 它接受远程分支的名字和位置作为参数，同时从标准输入中读取一系列待更新的引用。 你可以在推送开始之前，用它验证对引用的更新操作（一个非零的退出码将终止推送过程）。



##### 6.`pre-auto-gc` 钩子

Git 的一些日常操作在运行时，偶尔会调用 `git gc --auto` 进行垃圾回收。 `pre-auto-gc` 钩子会在垃圾回收开始之前被调用，可以用它来提醒你现在要回收垃圾了，或者依情形判断是否要中断回收。



### 服务器端钩子

除了客户端钩子，作为系统管理员，你还可以使用若干服务器端的钩子对项目强制执行各种类型的策略。 这些钩子脚本在推送到服务器之前和之后运行。 推送到服务器前运行的钩子可以在任何时候以非零值退出，拒绝推送并给客户端返回错误消息，还可以依你所想设置足够复杂的推送策略。

#### `pre-receive`

处理来自客户端的推送操作时，最先被调用的脚本是 `pre-receive`。 它从标准输入获取一系列被推送的引用。如果它以非零值退出，所有的推送内容都不会被接受。 你可以用这个钩子阻止对引用进行非快进（non-fast-forward）的更新，或者对该推送所修改的所有引用和文件进行访问控制。

#### `update`

`update` 脚本和 `pre-receive` 脚本十分类似，不同之处在于它会为每一个准备更新的分支各运行一次。 假如推送者同时向多个分支推送内容，`pre-receive` 只运行一次，相比之下 `update` 则会为每一个被推送的分支各运行一次。 它不会从标准输入读取内容，而是接受三个参数：引用的名字（分支），推送前的引用指向的内容的 SHA-1 值，以及用户准备推送的内容的 SHA-1 值。 如果 update 脚本以非零值退出，只有相应的那一个引用会被拒绝；其余的依然会被更新。

#### `post-receive`

`post-receive` 挂钩在整个过程完结以后运行，可以用来更新其他系统服务或者通知用户。 它接受与 `pre-receive` 相同的标准输入数据。 它的用途包括给某个邮件列表发信，通知持续集成（continous integration）的服务器， 或者更新问题追踪系统（ticket-tracking system） —— 甚至可以通过分析提交信息来决定某个问题（ticket）是否应该被开启，修改或者关闭。 该脚本无法终止推送进程，不过客户端在它结束运行之前将保持连接状态， 所以如果你想做其他操作需谨慎使用它，因为它将耗费你很长的一段时间。

### 使用强制策略的一个例子

https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-%E4%BD%BF%E7%94%A8%E5%BC%BA%E5%88%B6%E7%AD%96%E7%95%A5%E7%9A%84%E4%B8%80%E4%B8%AA%E4%BE%8B%E5%AD%90

------

