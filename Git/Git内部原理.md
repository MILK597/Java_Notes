# 目录

[TOC]





	**首先要弄明白一点，从根本上来讲 Git 是一个内容寻址（content-addressable）文件系统，并在此之上提供了一个版本控制系统的用户界面。** 

# .git 目录

​	当在一个新目录或已有目录执行 `git init` 时，Git 会创建一个 `.git` 目录。 这个目录包含了几乎所有 Git 存储和操作的东西。 如若想备份或复制一个版本库，只需把这个目录拷贝至另一处即可。

​	新初始化的 `.git` 目录的典型结构如下：

```console
$ ls -F1
config
description
HEAD
hooks/
info/
objects/
refs/
```

随着 Git 版本的不同，该目录下可能还会包含其他内容。 不过对于一个全新的 `git init` 版本库，这将是你看到的默认结构。

## `description` 文件

​	仅供 GitWeb 程序使用，我们无需关心



## `config` 文件

​	包含项目特有的配置选项



##  `info` 目录

​	包含一个全局性排除（global exclude）文件， 用以放置那些不希望被记录在 `.gitignore` 文件中的忽略模式（ignored patterns）



## `hooks` 目录

​	包含客户端或服务端的钩子脚本（hook scripts）



​	**`HEAD` 文件、（尚待创建的）`index` 文件，和 `objects` 目录、`refs` 目录。 它们都是 Git 的核心组成部分。** 



## `objects` 目录

​	存储所有数据内容



## `refs` 目录

​	存储指向数据（分支、远程仓库和标签等）的提交对象的指针



## `HEAD` 文件

指向目前被检出的分支



## `index` 文件

保存暂存区信息

------

# Git 对象

## 数据对象

Git 是一个内容寻址文件系统，听起来很酷。但这是什么意思呢？ 这意味着，Git 的核心部分是一个简单的键值对数据库（key-value data store）。 你可以向 Git 仓库中插入任意类型的内容，它会返回一个唯一的键，通过该键可以在任意时刻再次取回该内容。

可以通过底层命令 `git hash-object` 来演示上述效果——该命令可将任意数据保存于 `.git/objects` 目录（即 **对象数据库**），并返回指向该数据对象的唯一的键。

首先，我们需要初始化一个新的 Git 版本库，并确认 `objects` 目录为空：

```console
$ git init test
Initialized empty Git repository in /tmp/test/.git/
$ cd test
$ find .git/objects
.git/objects
.git/objects/info
.git/objects/pack
$ find .git/objects -type f
```

可以看到 Git 对 `objects` 目录进行了初始化，并创建了 `pack` 和 `info` 子目录，但均为空。 接着，我们用 `git hash-object` 创建一个新的数据对象并将它手动存入你的新 Git 数据库中：

```console
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
```

在这种最简单的形式中，`git hash-object` 会接受你传给它的东西，而它只会返回可以存储在 Git 仓库中的唯一键。 `-w` 选项会指示该命令不要只返回键，还要将该对象写入数据库中。 最后，`--stdin` 选项则指示该命令从标准输入读取内容；若不指定此选项，则须在命令尾部给出待存储文件的路径。

此命令输出一个长度为 40 个字符的校验和。 这是一个 SHA-1 哈希值——一个将待存储的数据外加一个头部信息（header）一起做 SHA-1 校验运算而得的校验和。后文会简要讨论该头部信息。 现在我们可以查看 Git 是如何存储数据的：

```console
$ find .git/objects -type f
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

如果你再次查看 `objects` 目录，那么可以在其中找到一个与新内容对应的文件。 这就是开始时 Git 存储内容的方式——一个文件对应一条内容， 以该内容加上特定头部信息一起的 SHA-1 校验和为文件命名。 校验和的前两个字符用于命名子目录，余下的 38 个字符则用作文件名。

一旦你将内容存储在了对象数据库中，那么可以通过 `cat-file` 命令从 Git 那里取回数据。 这个命令简直就是一把剖析 Git 对象的瑞士军刀。 为 `cat-file` 指定 `-p` 选项可指示该命令自动判断内容的类型，并为我们显示大致的内容：

```console
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
```

至此，你已经掌握了如何向 Git 中存入内容，以及如何将它们取出。 我们同样可以将这些操作应用于文件中的内容。 例如，可以对一个文件进行简单的版本控制。 首先，创建一个新文件并将其内容存入数据库：

```console
$ echo 'version 1' > test.txt
$ git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30
```

接着，向文件里写入新内容，并再次将其存入数据库：

```console
$ echo 'version 2' > test.txt
$ git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
```

对象数据库记录下了该文件的两个不同版本，当然之前我们存入的第一条内容也还在：

```console
$ find .git/objects -type f
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

现在可以在删掉 `test.txt` 的本地副本，然后用 Git 从对象数据库中取回它的第一个版本：

```console
$ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
$ cat test.txt
version 1
```

或者第二个版本：

```console
$ git cat-file -p 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a > test.txt
$ cat test.txt
version 2
```

然而，记住文件的每一个版本所对应的 SHA-1 值并不现实；另一个问题是，在这个（简单的版本控制）系统中，文件名并没有被保存——我们仅保存了文件的内容。 上述类型的对象我们称之为 **数据对象（blob object）**。 利用 `git cat-file -t` 命令，可以让 Git 告诉我们其内部存储的任何对象类型，只要给定该对象的 SHA-1 值：

```console
$ git cat-file -t 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
blob
```

------

## 树对象

​	Git 以一种类似于 UNIX 文件系统的方式存储内容，但作了些许简化。 所有内容均以树对象和数据对象的形式存储，其中树对象对应了 UNIX 中的目录项，数据对象则大致上对应了 inodes 或文件内容。 

### 树对象格式：

​	一个树对象包含了一条或多条树对象记录（tree entry），每条记录含有一个指向数据对象或者子树对象的 SHA-1 指针，以及相应的模式、类型、文件名信息。



 例如，某项目当前对应的最新树对象可能是这样的：

```console
$ git cat-file -p master^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859      README
100644 blob 8f94139338f9404f26296befa88755fc2598c289      Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0      lib
```

`master^{tree}` 语法表示 `master` 分支上最新的提交所指向的树对象。 请注意，`lib` 子目录（所对应的那条树对象记录）并不是一个数据对象，而是一个指针，其指向的是另一个树对象：

```console
$ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b      simplegit.rb
```



从概念上讲，Git 内部存储的数据有点像这样：

![简化版的 Git 数据模型。](img/data-model-1.png)

------

​	你可以轻松创建自己的树对象。 通常，Git 根据某一时刻暂存区（即 index 区域，下同）所表示的状态创建并记录一个对应的树对象， 如此重复便可依次记录（某个时间段内）一系列的树对象。 因此，为创建一个树对象，首先需要通过暂存一些文件来创建一个暂存区。

​	可以通过底层命令 `git update-index` 为一个单独文件——我们的 test.txt 文件的首个版本——创建一个暂存区。 利用该命令，可以把 `test.txt` 文件的首个版本人为地加入一个新的暂存区。 必须为上述命令指定 `--add` 选项，因为此前该文件并不在暂存区中（我们甚至都还没来得及创建一个暂存区呢）； 同样必需的还有 `--cacheinfo` 选项，因为将要添加的文件位于 Git 数据库中，而不是位于当前目录下。 同时，需要指定文件模式、SHA-1 与文件名：

```console
$ git update-index --add --cacheinfo 100644 \
  83baae61804e65cc73a7201a7252750c76066a30 test.txt
```

​	本例中，我们指定的文件模式为 `100644`，表明这是一个普通文件。 其他选择包括：`100755`，表示一个可执行文件；`120000`，表示一个符号链接。 这里的文件模式参考了常见的 UNIX 文件模式，但远没那么灵活——上述三种模式即是 Git 文件（即数据对象）的所有合法模式（当然，还有其他一些模式，但用于目录项和子模块）。

现在，可以通过 `git write-tree` 命令将暂存区内容写入一个树对象。 此处无需指定 `-w` 选项——如果某个树对象此前并不存在的话，当调用此命令时， 它会根据当前暂存区状态自动创建一个新的树对象：

```console
$ git write-tree
d8329fc1cc938780ffdd9f94e0d364e0ea74f579
$ git cat-file -p d8329fc1cc938780ffdd9f94e0d364e0ea74f579
100644 blob 83baae61804e65cc73a7201a7252750c76066a30      test.txt
```

不妨用之前见过的 `git cat-file` 命令验证一下它确实是一个树对象：

```console
$ git cat-file -t d8329fc1cc938780ffdd9f94e0d364e0ea74f579
tree
```

------

接着我们来创建一个新的树对象，它包括 `test.txt` 文件的第二个版本，以及一个新的文件：

```console
$ echo 'new file' > new.txt
$ git update-index --add --cacheinfo 100644 \
  1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
$ git update-index --add new.txt
```

暂存区现在包含了 `test.txt` 文件的新版本，和一个新文件：`new.txt`。 记录下这个目录树（将当前暂存区的状态记录为一个树对象），然后观察它的结构：

```console
$ git write-tree
0155eb4229851634a0f03eb265b69f5a2d56f341
$ git cat-file -p 0155eb4229851634a0f03eb265b69f5a2d56f341
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a      test.txt
```

我们注意到，新的树对象包含两条文件记录，同时 test.txt 的 SHA-1 值（`1f7a7a`）是先前值的“第二版”。 

你可以将第一个树对象加入第二个树对象，使其成为新的树对象的一个子目录。 通过调用 `git read-tree` 命令，可以把树对象读入暂存区。 本例中，可以通过对该命令指定 `--prefix` 选项，将一个已有的树对象作为子树读入暂存区：

```console
$ git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579
$ git write-tree
3c4e9cd789d88d8d89c1073707c3585e41b0e614
$ git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614
040000 tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579      bak
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a      test.txt
```

如果基于这个新的树对象创建一个工作目录，你会发现工作目录的根目录包含两个文件以及一个名为 `bak` 的子目录，该子目录包含 `test.txt` 文件的第一个版本。 可以认为 Git 内部存储着的用于表示上述结构的数据是这样的：

![当前 Git 的数据内容结构。](img/data-model-2.png)

------

df557656b1c731e276d96f1311a09e30d7ddbf6f

## 提交对象

​	如果你做完了以上所有操作，那么现在就有了三个树对象，分别代表我们想要跟踪的不同项目快照。 然而问题依旧：若想重用这些快照，你必须记住所有三个 SHA-1 哈希值。 并且，你也完全不知道是谁保存了这些快照，在什么时刻保存的，以及为什么保存这些快照。 而以上这些，正是提交对象（commit object）能为你保存的基本信息。

​	可以通过调用 `commit-tree` 命令创建一个提交对象，为此需要指定一个树对象的 SHA-1 值，以及该提交的父提交对象（如果有的话）。 我们从之前创建的第一个树对象开始：

```console
$ echo 'first commit' | git commit-tree d8329f
fdf4fc3344e67ab068f836878b6c4951e3b15f3d
```

​	由于创建时间和作者数据不同，你现在会得到一个不同的散列值。

​	请将后续内容中的提交和标签的散列值替换为你自己的校验和。

​	在可以通过 `git cat-file` 命令查看这个新提交对象：

```console
$ git cat-file -p fdf4fc3
tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
author Scott Chacon <schacon@gmail.com> 1243040974 -0700
committer Scott Chacon <schacon@gmail.com> 1243040974 -0700

first commit
```

### 	**提交对象的格式：**

​	它先指定一个顶层树对象，代表当前项目快照； 然后是可能存在的父提交（前面描述的提交对象并不存在任何父提交）； 之后是作者/提交者信息（依据你的 `user.name` 和 `user.email` 配置来设定，外加一个时间戳）； 留空一行，最后是提交注释。

------

​	接着，我们将创建另两个提交对象，它们分别引用各自的上一个提交（作为其父提交对象）：

```console
$ echo 'second commit' | git commit-tree 0155eb -p fdf4fc3
cac0cab538b970a37ea1e769cbbde608743bc96d
$ echo 'third commit'  | git commit-tree 3c4e9c -p cac0cab
1a410efbd13591db07496601ebc7a059dd55cfe9
```

这三个提交对象分别指向之前创建的三个树对象快照中的一个。 现在，如果对最后一个提交的 SHA-1 值运行 `git log` 命令，会出乎意料的发现，你已有一个货真价实的、可由 `git log` 查看的 Git 提交历史了：

```console
$ git log --stat 1a410e
commit 1a410efbd13591db07496601ebc7a059dd55cfe9
Author: Scott Chacon <schacon@gmail.com>
Date:   Fri May 22 18:15:24 2009 -0700

	third commit

 bak/test.txt | 1 +
 1 file changed, 1 insertion(+)

commit cac0cab538b970a37ea1e769cbbde608743bc96d
Author: Scott Chacon <schacon@gmail.com>
Date:   Fri May 22 18:14:29 2009 -0700

	second commit

 new.txt  | 1 +
 test.txt | 2 +-
 2 files changed, 2 insertions(+), 1 deletion(-)

commit fdf4fc3344e67ab068f836878b6c4951e3b15f3d
Author: Scott Chacon <schacon@gmail.com>
Date:   Fri May 22 18:09:34 2009 -0700

    first commit

 test.txt | 1 +
 1 file changed, 1 insertion(+)
```



**每次我们运行 `git add` 和 `git commit` 命令时，Git 所做的工作实质就是将被改写的文件保存为数据对象， 更新暂存区，记录树对象，最后创建一个指明了顶层树对象和父提交的提交对象。** 

**这三种主要的 Git 对象——数据对象、树对象、提交对象——最初均以单独文件的形式保存在 `.git/objects` 目录下。**

下面列出了目前示例目录内的所有对象，辅以各自所保存内容的注释：

```console
$ find .git/objects -type f
.git/objects/01/55eb4229851634a0f03eb265b69f5a2d56f341 # tree 2
.git/objects/1a/410efbd13591db07496601ebc7a059dd55cfe9 # commit 3
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a # test.txt v2
.git/objects/3c/4e9cd789d88d8d89c1073707c3585e41b0e614 # tree 3
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30 # test.txt v1
.git/objects/ca/c0cab538b970a37ea1e769cbbde608743bc96d # commit 2
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4 # 'test content'
.git/objects/d8/329fc1cc938780ffdd9f94e0d364e0ea74f579 # tree 1
.git/objects/fa/49b077972391ad58037050f2a75f74e3671e92 # new.txt
.git/objects/fd/f4fc3344e67ab068f836878b6c4951e3b15f3d # commit 1
```

如果跟踪所有的内部指针，将得到一个类似下面的对象关系图：

![你的 Git 目录下所有可达的对象。](img/data-model-3.png)

------

## 对象存储

前文曾提及，你向 Git 仓库提交的所有对象都会有个头部信息一并被保存。 让我们略花些时间来看看 Git 是如何存储其对象的。 通过在 Ruby 脚本语言中交互式地演示，你将看到一个数据对象——本例中是字符串“what is up, doc?”——是如何被存储的。

可以通过 `irb` 命令启动 Ruby 的交互模式：

```console
$ irb
>> content = "what is up, doc?"
=> "what is up, doc?"
```

**1.Git 首先会以识别出的对象的类型作为开头来构造一个头部信息，本例中是一个“blob”字符串。 接着 Git 会在头部的第一部分添加一个空格，随后是数据内容的字节数，最后是一个空字节（null byte）**：

```console
>> header = "blob #{content.length}\0"
=> "blob 16\u0000"
```

**2.Git 会将上述头部信息和原始数据拼接起来，并计算出这条新内容的 SHA-1 校验和。** 

在 Ruby 中可以这样计算 SHA-1 值——先通过 `require` 命令导入 SHA-1 digest 库， 然后对目标字符串调用 `Digest::SHA1.hexdigest()`：

```console
>> store = header + content
=> "blob 16\u0000what is up, doc?"
>> require 'digest/sha1'
=> true
>> sha1 = Digest::SHA1.hexdigest(store)
=> "bd9dbf5aae1a3862dd1526723246b20206e5fc37"
```

我们来比较一下 `git hash-object` 的输出。 这里使用了 `echo -n` 以避免在输出中添加换行。

```console
$ echo -n "what is up, doc?" | git hash-object --stdin
bd9dbf5aae1a3862dd1526723246b20206e5fc37
```

**3.Git 会通过 zlib 压缩这条新内容。**

在 Ruby 中可以借助 zlib 库做到这一点。 先导入相应的库，然后对目标内容调用 `Zlib::Deflate.deflate()`：

```console
>> require 'zlib'
=> true
>> zlib_content = Zlib::Deflate.deflate(store)
=> "x\x9CK\xCA\xC9OR04c(\xCFH,Q\xC8,V(-\xD0QH\xC9O\xB6\a\x00_\x1C\a\x9D"
```

**4.最后，需要将这条经由 zlib 压缩的内容写入磁盘上的某个对象。 要先确定待写入对象的路径（SHA-1 值的前两个字符作为子目录名称，后 38 个字符则作为子目录内文件的名称）。** 

如果该子目录不存在，可以通过 Ruby 中的 `FileUtils.mkdir_p()` 函数来创建它。 接着，通过 `File.open()` 打开这个文件。最后，对上一步中得到的文件句柄调用 `write()` 函数，以向目标文件写入之前那条 zlib 压缩过的内容：

```console
>> path = '.git/objects/' + sha1[0,2] + '/' + sha1[2,38]
=> ".git/objects/bd/9dbf5aae1a3862dd1526723246b20206e5fc37"
>> require 'fileutils'
=> true
>> FileUtils.mkdir_p(File.dirname(path))
=> ".git/objects/bd"
>> File.open(path, 'w') { |f| f.write zlib_content }
=> 32
```

我们用 `git cat-file` 查看一下该对象的内容：

```console
---
$ git cat-file -p bd9dbf5aae1a3862dd1526723246b20206e5fc37
what is up, doc?
---
```

就是这样——你已创建了一个有效的 Git 数据对象。

所有的 Git 对象均以这种方式存储，区别仅在于类型标识——另两种对象类型的头部信息以字符串“commit”或“tree”开头，而不是“blob”。 另外，虽然数据对象的内容几乎可以是任何东西，但提交对象和树对象的内容却有各自固定的格式。

------

# Git引用

​	如果你对仓库中从一个提交（比如 `1a410e`）开始往前的历史感兴趣，那么可以运行 `git log 1a410e` 这样的命令来显示历史，不过你需要记得 `1a410e` 是你查看历史的起点提交。 如果我们有一个文件来保存 SHA-1 值，而该文件有一个简单的名字， 然后用这个名字指针来替代原始的 SHA-1 值的话会更加简单。在 Git 中，这种简单的名字被称为**“引用（references，或简写为 refs）”**。

你可以在 `.git/refs` 目录下找到这类含有 SHA-1 值的文件。

若要创建一个新引用来帮助记忆最新提交所在的位置，从技术上讲我们只需简单地做如下操作：

```console
$ echo 1a410efbd13591db07496601ebc7a059dd55cfe9 > .git/refs/heads/master
```

现在，你就可以在 Git 命令中使用这个刚创建的新引用来代替 SHA-1 值了：

```console
$ git log --pretty=oneline master
1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
cac0cab538b970a37ea1e769cbbde608743bc96d second commit
fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit
```

我们不提倡直接编辑引用文件。 如果想更新某个引用，Git 提供了一个更加安全的命令 `update-ref` 来完成此事：

```console
$ git update-ref refs/heads/master 1a410efbd13591db07496601ebc7a059dd55cfe9
```

## Git 分支的本质：

​	一个指向某一系列提交之首的指针或引用。 



若想在第二个提交上创建一个分支，可以这么做：

```console
$ git update-ref refs/heads/test cac0ca
```

这个分支将只包含从第二个提交开始往前追溯的记录：

```console
$ git log --pretty=oneline test
cac0cab538b970a37ea1e769cbbde608743bc96d second commit
fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit
```

至此，我们的 Git 数据库从概念上看起来像这样：

![包含分支引用的 Git 目录对象。](img/data-model-4.png)

当运行类似于 `git branch <branch>` 这样的命令时，Git 实际上会运行 `update-ref` 命令， 取得当前所在分支最新提交对应的 SHA-1 值，并将其加入你想要创建的任何新引用中。



## HEAD 引用

​	现在的问题是，当你执行 `git branch <branch>` 时，Git 如何知道最新提交的 SHA-1 值呢？ 答案是 HEAD 文件。

​	HEAD 文件通常是一个符号引用（symbolic reference），指向目前所在的分支。 所谓符号引用，表示它是一个指向其他引用的指针。

​	然而在某些罕见的情况下，HEAD 文件可能会包含一个 git 对象的 SHA-1 值。 当你在检出一个标签、提交或远程分支，让你的仓库变成 “分离 HEAD”状态时，就会出现这种情况。

​	如果查看 HEAD 文件的内容，通常我们看到类似这样的内容：

```console
$ cat .git/HEAD
ref: refs/heads/master
```

如果执行 `git checkout test`，Git 会像这样更新 HEAD 文件：

```console
$ cat .git/HEAD
ref: refs/heads/test
```

当我们执行 `git commit` 时，该命令会创建一个提交对象，并用 HEAD 文件中那个引用所指向的 SHA-1 值设置其父提交字段。

你也可以手动编辑该文件，然而同样存在一个更安全的命令来完成此事：`git symbolic-ref`。 可以借助此命令来查看 HEAD 引用对应的值：

```console
$ git symbolic-ref HEAD
refs/heads/master
```

同样可以设置 HEAD 引用的值：

```console
$ git symbolic-ref HEAD refs/heads/test
$ cat .git/HEAD
ref: refs/heads/test
```

不能把符号引用设置为一个不符合引用规范的值：

```console
$ git symbolic-ref HEAD test
fatal: Refusing to point HEAD outside of refs/
```

------

## 标签引用

**标签对象（tag object）** 非常类似于一个提交对象——它包含一个标签创建者信息、一个日期、一段注释信息，以及一个指针。 

主要的区别在于，标签对象通常指向一个提交对象，而不是一个树对象。 它像是一个永不移动的分支引用——永远指向同一个提交对象，只不过给这个提交对象加上一个更友好的名字罢了。

正如前面所述，Git存在两种类型的标签：**附注标签**和**轻量标签**。 可以像这样创建一个**轻量标签**：

```console
$ git update-ref refs/tags/v1.0 cac0cab538b970a37ea1e769cbbde608743bc96d
```

这就是轻量标签的全部内容——一个固定的引用。



**附注标签**则更复杂一些。 若要创建一个附注标签，Git 会创建一个标签对象，并记录一个引用来指向该标签对象，而不是直接指向提交对象。 可以通过创建一个附注标签来验证这个过程（使用 `-a` 选项）：

```console
$ git tag -a v1.1 1a410efbd13591db07496601ebc7a059dd55cfe9 -m 'test tag'
```

下面是上述过程所建标签对象的 SHA-1 值：

```console
$ cat .git/refs/tags/v1.1
9585191f37f7b0fb9444f35a9bf50de191beadc2
```

现在对该 SHA-1 值运行 `git cat-file -p` 命令：

```console
$ git cat-file -p 9585191f37f7b0fb9444f35a9bf50de191beadc2
object 1a410efbd13591db07496601ebc7a059dd55cfe9
type commit
tag v1.1
tagger Scott Chacon <schacon@gmail.com> Sat May 23 16:48:58 2009 -0700

test tag
```

​	我们注意到，object 条目指向我们打了标签的那个提交对象的 SHA-1 值。 另外要注意的是，标签对象并非必须指向某个提交对象；你可以对任意类型的 Git 对象打标签。 例如，在 Git 源码中，项目维护者将他们的 GPG 公钥添加为一个数据对象，然后对这个对象打了一个标签。

## 远程引用

​	 如果你添加了一个远程版本库并对其执行过推送操作，Git 会记录下最近一次推送操作时每一个分支所对应的值，并保存在 `refs/remotes` 目录下。

​	例如，你可以添加一个叫做 `origin` 的远程版本库，然后把 `master` 分支推送上去：

```console
$ git remote add origin git@github.com:schacon/simplegit-progit.git
$ git push origin master
Counting objects: 11, done.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (7/7), 716 bytes, done.
Total 7 (delta 2), reused 4 (delta 1)
To git@github.com:schacon/simplegit-progit.git
  a11bef0..ca82a6d  master -> master
```

​	此时，如果查看 `refs/remotes/origin/master` 文件，可以发现 `origin` 远程版本库的 `master` 分支所对应的 SHA-1 值，就是最近一次与服务器通信时本地 `master` 分支所对应的 SHA-1 值：

```console
$ cat .git/refs/remotes/origin/master
ca82a6dff817ec66f44342007202690a93763949
```

​	远程引用和分支（位于 `refs/heads` 目录下的引用）之间最主要的区别在于，远程引用是只读的。 虽然可以 `git checkout` 到某个远程引用，但是 Git 并不会将 HEAD 引用指向该远程引用。因此，你永远不能通过 `commit` 命令来更新远程引用。 Git 将这些远程引用作为记录远程服务器上各分支最后已知位置状态的书签来管理。

------

# 包文件

​	如果你跟着做完了上一节中的所有步骤，那么现在应该有了一个测试用的 Git 仓库， 其中包含 11 个对象：四个数据对象，三个树对象，三个提交对象和一个标签对象：

```console
$ find .git/objects -type f
.git/objects/01/55eb4229851634a0f03eb265b69f5a2d56f341 # tree 2
.git/objects/1a/410efbd13591db07496601ebc7a059dd55cfe9 # commit 3
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a # test.txt v2
.git/objects/3c/4e9cd789d88d8d89c1073707c3585e41b0e614 # tree 3
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30 # test.txt v1
.git/objects/95/85191f37f7b0fb9444f35a9bf50de191beadc2 # tag
.git/objects/ca/c0cab538b970a37ea1e769cbbde608743bc96d # commit 2
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4 # 'test content'
.git/objects/d8/329fc1cc938780ffdd9f94e0d364e0ea74f579 # tree 1
.git/objects/fa/49b077972391ad58037050f2a75f74e3671e92 # new.txt
.git/objects/fd/f4fc3344e67ab068f836878b6c4951e3b15f3d # commit 1
```

​	**Git 使用 zlib 压缩这些文件的内容**

​	 为了便于演示，我们要把之前在 Grit 库中用到过的 `repo.rb` 文件添加进来——这是一个大小约为 22K 的源代码文件：

```console
$ curl https://raw.githubusercontent.com/mojombo/grit/master/lib/grit/repo.rb > repo.rb
$ git checkout master
$ git add repo.rb
$ git commit -m 'added repo.rb'
[master 484a592] added repo.rb
 3 files changed, 709 insertions(+), 2 deletions(-)
 delete mode 100644 bak/test.txt
 create mode 100644 repo.rb
 rewrite test.txt (100%)
```

​	如果你查看生成的树对象，可以看到 `repo.rb` 文件对应的数据对象的 SHA-1 值：

```console
$ git cat-file -p master^{tree}
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob 033b4468fa6b2a9547a70d88d1bbe8bf3f9ed0d5      repo.rb
100644 blob e3f094f522629ae358806b17daf78246c27c007b      test.txt
```

​	接下来你可以使用 `git cat-file` 命令查看这个对象有多大：

```console
$ git cat-file -s 033b4468fa6b2a9547a70d88d1bbe8bf3f9ed0d5
22044
```

​	现在，稍微修改这个文件，然后看看会发生什么：

```console
$ echo '# testing' >> repo.rb
$ git commit -am 'modified repo.rb a bit'
[master 2431da6] modified repo.rb a bit
 1 file changed, 1 insertion(+)
```

​	查看这个最新的提交生成的树对象，你会看到一些有趣的东西：

```console
$ git cat-file -p master^{tree}
100644 blob fa49b077972391ad58037050f2a75f74e3671e92      new.txt
100644 blob b042a60ef7dff760008df33cee372b945b6e884e      repo.rb
100644 blob e3f094f522629ae358806b17daf78246c27c007b      test.txt
```

​	repo.rb 对应一个与之前完全不同的数据对象，这意味着，虽然你只是在一个 400 行的文件后面加入一行新内容，Git 也会用一个全新的对象来存储新的文件内容：

```console
$ git cat-file -s b042a60ef7dff760008df33cee372b945b6e884e
22054
```

​	你的磁盘上现在有两个几乎完全相同、大小均为 22K 的对象（每个都被压缩到大约 7K）。

​	 如果 Git 只完整保存其中一个，再保存另一个对象与之前版本的差异内容，岂不更好？

​	Git 最初向磁盘中存储对象时所使用的格式被称为“松散（loose）”对象格式。 但是，Git 会时不时地将多个这些对象打包成一个称为**“包文件（packfile）”**的二进制文件，以节省空间和提高效率。 当版本库中有太多的松散对象，或者你手动执行 `git gc` 命令，或者你向远程服务器执行推送时，Git 都会这样做。 

要看到打包过程，你可以手动执行 `git gc` 命令让 Git 对对象进行打包：

```console
$ git gc
Counting objects: 18, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (14/14), done.
Writing objects: 100% (18/18), done.
Total 18 (delta 3), reused 0 (delta 0)
```

​	这个时候再查看 `objects` 目录，你会发现大部分的对象都不见了，与此同时出现了一堆新文件：

```console
$ find .git/objects -type f
.git/objects/bd/9dbf5aae1a3862dd1526723246b20206e5fc37
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
.git/objects/info/packs
.git/objects/pack/pack-978e03944f5c581011e6998cd0e9e30000905586.idx
.git/objects/pack/pack-978e03944f5c581011e6998cd0e9e30000905586.pack
```

​	仍保留着的几个对象是未被任何提交记录引用的数据对象——在此例中是你之前创建的 “what is up, doc?” 和 “test content” 这两个示例数据对象。 因为你从没将它们添加至任何提交记录中，所以 Git 认为它们是**悬空（dangling）**的，不会将它们打包进新生成的包文件中。

​	剩下的文件是新创建的**包文件**和一个**索引**。 

​	**包文件**包含了刚才从文件系统中移除的所有对象的内容。

​	**索引文件**包含了包文件的偏移信息，我们通过索引文件就可以快速定位任意一个指定对象。

​	 有意思的是运行 `gc` 命令前磁盘上的对象大小约为 15K，而这个新生成的包文件大小仅有 7K。 通过打包对象减少了一半的磁盘占用空间。

​	**Git 打包对象时，会查找命名及大小相近的文件，并只保存文件不同版本之间的差异内容。** 

​	你可以查看包文件，观察它是如何节省空间的。 `git verify-pack` 这个底层命令可以让你查看已打包的内容：

```console
$ git verify-pack -v .git/objects/pack/pack-978e03944f5c581011e6998cd0e9e30000905586.idx
2431da676938450a4d72e260db3bf7b0f587bbc1 commit 223 155 12
69bcdaff5328278ab1c0812ce0e07fa7d26a96d7 commit 214 152 167
80d02664cb23ed55b226516648c7ad5d0a3deb90 commit 214 145 319
43168a18b7613d1281e5560855a83eb8fde3d687 commit 213 146 464
092917823486a802e94d727c820a9024e14a1fc2 commit 214 146 610
702470739ce72005e2edff522fde85d52a65df9b commit 165 118 756
d368d0ac0678cbe6cce505be58126d3526706e54 tag    130 122 874
fe879577cb8cffcdf25441725141e310dd7d239b tree   136 136 996
d8329fc1cc938780ffdd9f94e0d364e0ea74f579 tree   36 46 1132
deef2e1b793907545e50a2ea2ddb5ba6c58c4506 tree   136 136 1178
d982c7cb2c2a972ee391a85da481fc1f9127a01d tree   6 17 1314 1 \
  deef2e1b793907545e50a2ea2ddb5ba6c58c4506
3c4e9cd789d88d8d89c1073707c3585e41b0e614 tree   8 19 1331 1 \
  deef2e1b793907545e50a2ea2ddb5ba6c58c4506
0155eb4229851634a0f03eb265b69f5a2d56f341 tree   71 76 1350
83baae61804e65cc73a7201a7252750c76066a30 blob   10 19 1426
fa49b077972391ad58037050f2a75f74e3671e92 blob   9 18 1445
#
b042a60ef7dff760008df33cee372b945b6e884e blob   22054 5799 1463
#
033b4468fa6b2a9547a70d88d1bbe8bf3f9ed0d5 blob   9 20 7262 1 \
b042a60ef7dff760008df33cee372b945b6e884e
#这是同一行
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a blob   10 19 7282
non delta: 15 objects
chain length = 1: 3 objects
.git/objects/pack/pack-978e03944f5c581011e6998cd0e9e30000905586.pack: ok
```

​	此处，`033b4` 这个数据对象（即 `repo.rb` 文件的第一个版本，如果你还记得的话）引用了数据对象 `b042a`，即该文件的第二个版本。 命令输出内容的第三列显示的是各个对象在包文件中的大小，可以看到 `b042a` 占用了 22K 空间，而 `033b4` 仅占用 9 字节。 同样有趣的地方在于，第二个版本完整保存了文件内容，而原始的版本反而是以差异方式保存的——这是因为大部分情况下需要快速访问文件的最新版本。

最妙之处是你可以随时重新打包。 Git 时常会自动对仓库进行重新打包以节省空间。当然你也可以随时手动执行 `git gc` 命令来这么做。

------

# 引用规范

https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E5%BC%95%E7%94%A8%E8%A7%84%E8%8C%83

# 传输协议

https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE

# 维护

## git gc

​	Git 会不定时地自动运行一个叫做 “auto gc” 的命令。 大多数时候，这个命令并不会产生效果。 然而，如果有太多松散对象（不在包文件中的对象）或者太多包文件，Git 会运行一个完整的 `git gc` 命令。 “gc” 代表垃圾回收，这个命令会做以下事情：收集所有松散对象并将它们放置到包文件中， 将多个包文件合并为一个大的包文件，移除与任何提交都不相关的陈旧对象。

​	可以像下面一样手动执行自动垃圾回收：

```console
$ git gc --auto
```

​	就像上面提到的，这个命令通常并不会产生效果。 大约需要 7000 个以上的松散对象或超过 50 个的包文件才能让 Git 启动一次真正的 gc 命令。 你可以通过修改 `gc.auto` 与 `gc.autopacklimit` 的设置来改动这些数值。



​	`gc` 将会做的另一件事是打包你的引用到一个单独的文件。 假设你的仓库包含以下分支与标签：

```console
$ find .git/refs -type f
.git/refs/heads/experiment
.git/refs/heads/master
.git/refs/tags/v1.0
.git/refs/tags/v1.1
```

​	如果你执行了 `git gc` 命令，`refs` 目录中将不会再有这些文件。 为了保证效率 Git 会将它们移动到名为 `.git/packed-refs` 的文件中，就像这样：

```console
$ cat .git/packed-refs
# pack-refs with: peeled fully-peeled
cac0cab538b970a37ea1e769cbbde608743bc96d refs/heads/experiment
ab1afef80fac8e34258ff41fc1b867c702daa24b refs/heads/master
cac0cab538b970a37ea1e769cbbde608743bc96d refs/tags/v1.0
9585191f37f7b0fb9444f35a9bf50de191beadc2 refs/tags/v1.1
^1a410efbd13591db07496601ebc7a059dd55cfe9
```

​	如果你更新了引用，Git 并不会修改这个文件，而是向 `refs/heads` 创建一个新的文件。 为了获得指定引用的正确 SHA-1 值，Git 会首先在 `refs` 目录中查找指定的引用，然后再到 `packed-refs` 文件中查找。 所以，如果你在 `refs` 目录中找不到一个引用，那么它或许在 `packed-refs` 文件中。

注意这个文件的最后一行，它会以 `^` 开头。 这个符号表示它上一行的标签是附注标签，`^` 所在的那一行是附注标签指向的那个提交。

------

# 数据恢复

​	在你使用 Git 的时候，你可能会意外丢失一次提交。 通常这是因为你强制删除了正在工作的分支，但是最后却发现你还需要这个分支， 亦或者硬重置了一个分支，放弃了你想要的提交。 如果这些事情已经发生，该如何找回你的提交呢？

​	下面的例子将硬重置你的测试仓库中的 `master` 分支到一个旧的提交，以此来恢复丢失的提交。 首先，让我们看看你的仓库现在在什么地方：

```console
$ git log --pretty=oneline
ab1afef80fac8e34258ff41fc1b867c702daa24b modified repo a bit
484a59275031909e19aadb7c92262719cfcdf19a added repo.rb
1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
cac0cab538b970a37ea1e769cbbde608743bc96d second commit
fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit
```

​	现在，我们将 `master` 分支硬重置到第三次提交：

```console
$ git reset --hard 1a410efbd13591db07496601ebc7a059dd55cfe9
HEAD is now at 1a410ef third commit
$ git log --pretty=oneline
1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
cac0cab538b970a37ea1e769cbbde608743bc96d second commit
fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit
```

​	现在顶部的两个提交已经丢失了——没有分支指向这些提交。 你需要找出最后一次提交的 SHA-1 然后增加一个指向它的分支。 窍门就是找到最后一次的提交的 SHA-1 ——但是估计你记不起来了，对吗？

## `git reflog`



​	当你正在工作时，Git 会默默地记录每一次你改变 HEAD 时它的值。 每一次你提交或改变分支，引用日志都会被更新。 引用日志（reflog）也可以通过 `git update-ref` 命令更新。

​	你可以在任何时候通过执行 `git reflog` 命令来了解你曾经做过什么：

```console
$ git reflog
1a410ef HEAD@{0}: reset: moving to 1a410ef
ab1afef HEAD@{1}: commit: modified repo.rb a bit
484a592 HEAD@{2}: commit: added repo.rb
```

​	这里可以看到我们已经检出的两次提交，然而并没有足够多的信息。 



## `git log -g`

​	为了使显示的信息更加有用，我们可以执行 `git log -g`，这个命令会以标准日志的格式输出引用日志。

```console
$ git log -g
commit 1a410efbd13591db07496601ebc7a059dd55cfe9
Reflog: HEAD@{0} (Scott Chacon <schacon@gmail.com>)
Reflog message: updating HEAD
Author: Scott Chacon <schacon@gmail.com>
Date:   Fri May 22 18:22:37 2009 -0700

		third commit

commit ab1afef80fac8e34258ff41fc1b867c702daa24b
Reflog: HEAD@{1} (Scott Chacon <schacon@gmail.com>)
Reflog message: updating HEAD
Author: Scott Chacon <schacon@gmail.com>
Date:   Fri May 22 18:15:24 2009 -0700

       modified repo.rb a bit
```

## 	

## 恢复丢失的提交

看起来下面的那个就是你丢失的提交，你可以通过创建一个新的分支指向这个提交来恢复它。 例如，你可以创建一个名为 `recover-branch` 的分支指向这个提交（ab1afef）：

```console
$ git branch recover-branch ab1afef
$ git log --pretty=oneline recover-branch
ab1afef80fac8e34258ff41fc1b867c702daa24b modified repo a bit
484a59275031909e19aadb7c92262719cfcdf19a added repo.rb
1a410efbd13591db07496601ebc7a059dd55cfe9 third commit
cac0cab538b970a37ea1e769cbbde608743bc96d second commit
fdf4fc3344e67ab068f836878b6c4951e3b15f3d first commit
```



接下来，假设你丢失的提交因为某些原因不在引用日志中，这时该如何恢复那次提交？ 

## `gi fsck`

一种方式是使用 `git fsck` 实用工具，将会检查数据库的完整性。 如果使用一个 `--full` 选项运行它，它会向你显示出所有没有被其他对象指向的对象：

```console
$ git fsck --full
Checking object directories: 100% (256/256), done.
Checking objects: 100% (18/18), done.
dangling blob d670460b4b4aece5915caf5c68d12f560a9fe3e4
dangling commit ab1afef80fac8e34258ff41fc1b867c702daa24b
dangling tree aea790b9a58f6cf6f2804eeac9f0abbe9631e4c9
dangling blob 7108f7ecb345ee9d0084193f147cdad4d2998293
```

在这个例子中，你可以在 “dangling commit” 后看到你丢失的提交。 现在你可以用和之前相同的方法恢复这个提交，也就是添加一个指向这个提交的分支。

------

# 移除对象

Git 有很多很棒的功能，但是其中一个特性会导致问题，`git clone` 会下载整个项目的历史，包括每一个文件的每一个版本。 如果所有的东西都是源代码那么这很好，因为 Git 被高度优化来有效地存储这种数据。 然而，如果某个人在之前向项目添加了一个大小特别大的文件，即使你将这个文件从项目中移除了，每次克隆还是都要强制的下载这个大文件。 之所以会产生这个问题，是因为这个文件在历史中是存在的，它会永远在那里。

当你迁移 Subversion 或 Perforce 仓库到 Git 的时候，这会是一个严重的问题。 因为这些版本控制系统并不下载所有的历史文件，所以这种文件所带来的问题比较少。 如果你从其他的版本控制系统迁移到 Git 时发现仓库比预期的大得多，那么你就需要找到并移除这些大文件。

**警告：这个操作对提交历史的修改是破坏性的。**它会从你必须修改或移除一个大文件引用最早的树对象开始重写每一次提交。 如果你在导入仓库后，在任何人开始基于这些提交工作前执行这个操作，那么将不会有任何问题——否则， 你必须通知所有的贡献者他们需要将他们的成果变基到你的新提交上。



为了演示，我们将添加一个大文件到测试仓库中，并在下一次提交中删除它，现在我们需要找到它，并将它从仓库中永久删除。 首先，添加一个大文件到仓库中：

```console
$ curl https://www.kernel.org/pub/software/scm/git/git-2.1.0.tar.gz > git.tgz
$ git add git.tgz
$ git commit -m 'add git tarball'
[master 7b30847] add git tarball
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 git.tgz
```

哎呀——其实这个项目并不需要这个巨大的压缩文件。 现在我们将它移除：

```console
$ git rm git.tgz
rm 'git.tgz'
$ git commit -m 'oops - removed large tarball'
[master dadf725] oops - removed large tarball
 1 file changed, 0 insertions(+), 0 deletions(-)
 delete mode 100644 git.tgz
```



现在，我们执行 `gc` 来查看数据库占用了多少空间：

```console
$ git gc
Counting objects: 17, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (13/13), done.
Writing objects: 100% (17/17), done.
Total 17 (delta 1), reused 10 (delta 0)
```

你也可以执行 `count-objects` 命令来快速的查看占用空间大小：

```console
$ git count-objects -v
count: 7
size: 32
in-pack: 17
packs: 1
size-pack: 4868
prune-packable: 0
garbage: 0
size-garbage: 0
```

`size-pack` 的数值指的是你的包文件以 KB 为单位计算的大小，所以你大约占用了 5MB 的空间。 在最后一次提交前，使用了不到 2KB ——显然，从之前的提交中移除文件并不能从历史中移除它。 每一次有人克隆这个仓库时，他们将必须克隆所有的 5MB 来获得这个微型项目，只因为你意外地添加了一个大文件。 现在来让我们彻底的移除这个文件。

首先你必须找到它。 在本例中，你已经知道是哪个文件了。 但是假设你不知道；该如何找出哪个文件或哪些文件占用了如此多的空间？ 如果你执行 `git gc` 命令，所有的对象将被放入一个包文件中，你可以通过运行 `git verify-pack` 命令， 然后对输出内容的第三列（即文件大小）进行排序，从而找出这个大文件。 你也可以将这个命令的执行结果通过管道传送给 `tail` 命令，因为你只需要找到列在最后的几个大对象。

```console
$ git verify-pack -v .git/objects/pack/pack-29…69.idx \
  | sort -k 3 -n \
  | tail -3
dadf7258d699da2c8d89b09ef6670edb7d5f91b4 commit 229 159 12
033b4468fa6b2a9547a70d88d1bbe8bf3f9ed0d5 blob   22044 5792 4977696
82c99a3e86bb1267b236a4b6eff7868d97489af1 blob   4975916 4976258 1438
```

你可以看到这个大对象出现在返回结果的最底部：占用 5MB 空间。 为了找出具体是哪个文件，可以使用 `rev-list` 命令，我们在 指定特殊的提交信息格式 中曾提到过。 如果你传递 `--objects` 参数给 `rev-list` 命令，它就会列出所有提交的 SHA-1、数据对象的 SHA-1 和与它们相关联的文件路径。 可以使用以下命令来找出你的数据对象的名字：

```console
$ git rev-list --objects --all | grep 82c99a3
82c99a3e86bb1267b236a4b6eff7868d97489af1 git.tgz
```

现在，你只需要从过去所有的树中移除这个文件。 使用以下命令可以轻松地查看哪些提交对这个文件产生改动：

```console
$ git log --oneline --branches -- git.tgz
dadf725 oops - removed large tarball
7b30847 add git tarball
```

现在，你必须重写 `7b30847` 提交之后的所有提交来从 Git 历史中完全移除这个文件。 为了执行这个操作，我们要使用 `filter-branch` 命令，这个命令在 重写历史 中也使用过：

```console
$ git filter-branch --index-filter \
  'git rm --ignore-unmatch --cached git.tgz' -- 7b30847^..
Rewrite 7b30847d080183a1ab7d18fb202473b3096e9f34 (1/2)rm 'git.tgz'
Rewrite dadf7258d699da2c8d89b09ef6670edb7d5f91b4 (2/2)
Ref 'refs/heads/master' was rewritten
```

`--index-filter` 选项类似于在 重写历史 中提到的的 `--tree-filter` 选项， 不过这个选项并不会让命令将修改在硬盘上检出的文件，而只是修改在暂存区或索引中的文件。

你必须使用 `git rm --cached` 命令来移除文件，而不是通过类似 `rm file` 的命令——因为你需要从索引中移除它，而不是磁盘中。 还有一个原因是速度—— Git 在运行过滤器时，并不会检出每个修订版本到磁盘中，所以这个过程会非常快。 如果愿意的话，你也可以通过 `--tree-filter` 选项来完成同样的任务。 `git rm` 命令的 `--ignore-unmatch` 选项告诉命令：如果尝试删除的模式不存在时，不提示错误。 最后，使用 `filter-branch` 选项来重写自 `7b30847` 提交以来的历史，也就是这个问题产生的地方。 否则，这个命令会从最旧的提交开始，这将会花费许多不必要的时间。

你的历史中将不再包含对那个文件的引用。 不过，你的引用日志和你在 `.git/refs/original` 通过 `filter-branch` 选项添加的新引用中还存有对这个文件的引用，所以你必须移除它们然后重新打包数据库。 在重新打包前需要移除任何包含指向那些旧提交的指针的文件：

```console
$ rm -Rf .git/refs/original
$ rm -Rf .git/logs/
$ git gc
Counting objects: 15, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (11/11), done.
Writing objects: 100% (15/15), done.
Total 15 (delta 1), reused 12 (delta 0)
```

让我们看看你省了多少空间。

```console
$ git count-objects -v
count: 11
size: 4904
in-pack: 15
packs: 1
size-pack: 8
prune-packable: 0
garbage: 0
size-garbage: 0
```

打包的仓库大小下降到了 8K，比 5MB 好很多。 可以从 size 的值看出，这个大文件还在你的松散对象中，并没有消失；但是它不会在推送或接下来的克隆中出现，这才是最重要的。 如果真的想要删除它，可以通过有 `--expire` 选项的 `git prune` 命令来完全地移除那个对象：

```console
$ git prune --expire now
$ git count-objects -v
count: 0
size: 0
in-pack: 15
packs: 1
size-pack: 8
prune-packable: 0
garbage: 0
size-garbage: 0
```

# 环境变量

https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F