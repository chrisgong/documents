## git config

* `git config -e` : 打开config文件
* `git config -e --global` : 打开全局配置文件
* `git config -e --system` : 打开系统配置文件
> git的三个配置级别优先级: 版本库级别文件 > 全局配置文件 > 系统级配置文件

## git add

* `git add -u` : 快速标记改动, 将本地有改动(修改和删除)的文件标记到暂存区.
* `git add -i` : 交互式添加

```
  $ git add -i
  *** Commands ***
    1: status	  2: update	  3: revert	  4: add untracked
    5: patch	  6: diff	  7: quit	  8: help
  What now> 4
  Add untracked>> 1
  * 1: add.txt
  Add untracked>>
  added one path

  *** Commands ***
    1: status	  2: update	  3: revert	  4: add untracked
    5: patch	  6: diff	  7: quit	  8: help
  What now> 1
             staged     unstaged path
    1:        +0/-0      nothing add.txt

  *** Commands ***
    1: status	  2: update	  3: revert	  4: add untracked
    5: patch	  6: diff	  7: quit	  8: help
  What now> 7
  Bye.
```

## git commit

* `git commit --amend --allow-empty --reset-author` : 修改提交message
  * `--amend` : 对刚刚的提交进行修补.
  * `--allow-empty` : 允许空白提交
  * `--reset-author` : 含义是将Author(作者)的ID同步修改, 否则只会影响提交者(Commit)的ID.
  * `-a -m` : 偷懒式提交, 不用预先执行 `git add`

## git revert

运行该命令相当于将HEAD提交反向再提交一次.

## git status

精简状态输出

```
$ git status -s
MM README.md
```

* 在执行git add命令之前, 这个M位于第二列(第一列是空格), 在执行完git add之后, 字符M位于第一列(第二列是空格)
* 位于第一列的字母M的含义是: 版本库中的文件与处于中间状态-----提交任务(提交暂存区, stage)中的文件相比有改动.
* 位于第二列的字母M的含义是: 工作区当前的文件与处于中间状态----提交任务(提交暂存区, stage)中的文件相比有改动

```
//-b参数, 能够同时显示出当前工作分支的名称
$ git status -s -b
## master...origin/master [ahead 3, behind 3]
MM README.md
```

## git diff

* `git diff` : 显示工作区的最新改动, 即工作区与提交任务(提交缓存区, stage)中相比的差异
* `git diff B A` : 比较里程碑B和里程碑A
* `git diff A` : 比较工作区和里程碑A
* `git diff --cached A` : 比较暂存区和版本库A
* `git diff` : 比较工作区和暂存区
* `git diff --cached` : 比较暂存区和HEAD
* `git diff HEAD` : 比较工作区和HEAD
* `git diff --word-diff` : 逐词比较, 默认是逐行比较

## git log

* `git log --pretty=raw --graph` : 输出提交日志, 显示跟踪曲线(`--graph`)
* `git log --graph --oneline` : 输出简短提交ID

## git blame

* `git blame <filename> -L 5,+10` : 查看文件某几行

## git checkout

* `git checkout [-q] [<commit>] [--] <paths>`
  * <commit>是可选项, 如果省略则相当于从暂存区(index)进行检出. 和重置命令区别: 重置的默认值是HEAD, 而检出的默认值是暂存区. 因此重置一般用于重置暂存区(除非使用--hard参数, 否则不重置工作区), 而检出命令主要是覆盖工作区(如果<commit>不省略, 也会替换暂存区中相应的文件.)
  * 不会改变HEAD头指针, 主要是用于置顶版本的文件覆盖工作区中对应的文件. 如果省略<commit>, 则会用暂存区的文件覆盖工作区的文件, 否则用指定提交中的文件覆盖暂缓区和工作区中对应的文件.
* `git checkout [branch]`
  * 会改变HEAD头指针. 之所以后面的参数写作<branch>, 是因为只有HEAD切换到一个分支才可以对提交进行跟踪, 否则仍然会进入"分离头指针"的状态. 在"分离头指针"状态下的提交不能被引用关联到, 从而可能丢失. 所以用法二最主要的作用就是切换到分支. 如果省略<branch>则相当于对工作区进行状态检查.
* `git checkout [-m] [[-b|-orphan] <new_branch>] [<start_point>]`
  * 创建和切换到新的分支(<new_branch>), 新的分支从<start_point>指定的提交开始创建.

![](http://7xs2xr.com1.z0.glb.clouddn.com/git-checkout.png)

## 工作区/版本库/缓存区

![](http://7xs2xr.com1.z0.glb.clouddn.com/git%E5%8E%9F%E7%90%86.jpg)

* 图中左侧为工作区, 右侧为版本库. 在版本库中标记为index的区域是暂存区, 标记为master的是master分支所代表的目录树.
* 图中可以看出, 此时HEAD实际是指向master分支的一个"游标", 所以图示的命令中出现HEAD的地方可以用master来替换.
* 图中的objects标识的区域为Git的对象库, 实际位于.git/objects目录下.
* 当对工作区修改(或新增)的文件执行 `git add` 命令时, 暂存区的目录树被更新, 同时工作区修改(或新增)的文件内容会被写入到对象库中的一个新的对象中, 而该对象的ID被记录在暂存区的文件索引上.
* 当执行提交操作( `git commit` )时, 暂存区的目录树会写到版本库(对象库)中, master分支会做相应的更新, 即master最新指向的目录树就是提交时原暂存区的目录树.
* 当执行 `git reset HEAD` 命令时, 暂存区的目录树会被重写, 会被master分支指向的目录树所替换, 但是工作区不受影响.
* 当执行 `git rm --cached` 命令时, 会直接从暂存区删除文件, 工作区则不做出改变.
* 当执行 `git checkout .` 或 `git checkout --`命令时, 会用暂存区全部的文件或指定的文件替换工作区的文件. 这个操作很危险, 会清除工作区中未添加到暂存区的改动.
* 当执行 `git checkout HEAD.` 或 `git checkout HEAD`命令时, 会用HEAD指向的master分支中的全部或部分文件替换暂存和工作区中的文件. 这个命令也是极具危险性的, 因为不但会清除工作区中未提交的改动, 也会清除暂存区中未提交的改动.

## 重置

### git reset

两种用法, 其中<commit>都是可选项, 可以使用引用或提交ID, 如果省略<commit>则相当于使用了HEAD指向作为提交ID.

* `git reset [-q] [commit] [--] <paths>...` : 不会重置引用, 更不会改变工作区, 而是用指定提交状态(<commit>)下的文件(<paths>)替换掉缓存区中的文件. 例如命令 `git reset HEAD <paths>` 相当于取消之前执行的 `git add <paths>` 命令时改变的暂存区.
* `git reset [--soft | --mixed | --hard | --merge | --keep] [q] [<commit>]` : 会重置引用. 根据不同的选项, 可以对暂存区或工作区进行重置.
![](http://7xs2xr.com1.z0.glb.clouddn.com/git-reset.png)
  * `--hard` : 会执行上图中的全部动作①, ②, ③, 即:
    1. 替换引用的指向. 引用指向新的提交ID.
    2. 替换暂存区. 替换后, 暂存区的内容和引用指向的目录树一致.
    3. 替换工作区. 替换后, 工作区的内容变得和暂存区一致, 也和HEAD所指向的目录树内容相同.
  * `--soft` : 会执行上图中的操作①. 即只更改引用的指向, 不改变暂存区和工作区.
  * `--mixed` : 或不适用参数(默认为 `--mixed`) : 会执行上图操作①, ②. 即更改引用的指向及重置暂存区, 但是不改变工作区.

> 示例:

* git reset
仅用HEAD指向的目录树重置暂存区, 工作区不会受到影响, 相当于将之前用 `git add` 命令更新到暂存区的内容撤出暂存区. 引用也未改变, 因为引用重置到HEAD相当于没有重置.
* git reset HEAD
同上.
* git reset -- filename
仅将文件filename的改动撤出暂存区, 暂存区中其他文件不改变. 相当于对命令 `git add filename` 的反向操作.
* git reset HEAD filename
同上.
* git reset --soft HEAD^
工作区和暂存区不改变, 但是引用向前回退一次. 当对新提交的提交说明或提交的更改不满意时, 撤销最新的提交以便重新提交.
> TIP:  `git commit --amend` : 用于对最新的提交进行重新提交以修补错误的提交说明或错误的提交文件. 修补提示命令实际上相当于执行了一下两条命令:
```
 $ git reset --soft HEAD^
 $ git commit -e -F .git/COMMIT_EDITMSG
```
* git reset HEAD^
工作区不变化, 但是暂存区会回退到上一次提交之前, 引用也会回退一次.
* git reset --mixed HEAD^
同上.
* git reset --hard HEAD^
彻底撤销最近的提交. 引用回退到前一次, 而且工作区和暂存区都会回退到上一次提交的状态. 自上一次依赖的提交全部丢失.

### git reflog

通过.git/logs目录下的日志文件记录了分支的变更.

```
//检查配置是否打开
$ git config core.logallrefupdates
true
```

通过 `git reflog` 查看记录分支变更

```
$ git reflog develop
e29f924 develop@{0}: reset: moving to HEAD^
1c3608c develop@{1}: reset: moving to HEAD^
5f48d64 develop@{2}: commit: change files
1c3608c develop@{3}: branch: Created from master
```

查看 `git reflog` 的输出和直接查看日志文件最大的不同在于显示顺序不同, 即最新改变放在了最前面显示, 而且只显示每次改变的最终的SHA1哈希值. 还有个重要的区别在与 `git reflog` 命令的输出中还提供了一个方便易记的表达式: `<refname>@{<n>}`. 这个表达式的含义是引用 `<refname>` 之前第 `<n>` 次改变时的SHA1哈希值.

* 重置develop为两次改变之前的值

```
 $ git reset --hard develop@{2}
 HEAD is now at e29f924 does develop follow this new commit?
```

### git rebase

* `git rebase --onto <newbase> <since> <till>`
  1. 首先会执行 `git checkout` 切换到 `<till>` , 如果操作在detached HEAD(分离指针头)状态进行, 当rebase结束后, 还要对当前分支执行重置以实现rebase结果在分支中生效.
  2. 将`<since>..<till>`所表示的提交范围写到一个临时文件中(包括 `<till>` 的所有历史提交排除 `<since>` 及 `<since>` 的历史提交后形成的版本范围.)
  3. 将当前分支强制重置( `git reset --hard` )到 `<newbase>` .
  4. 从保存的临时文件中的提交列表中, 将提交逐一按顺序重新提交到重置之后的分支上.
  5. 如果遇到提交已经在分支中包含, 则跳过该提交
  6. 如果在提交过程遇到冲突, 则rebase过程暂停, 用户解决冲突后, 执行 `git rebase --continue` 继续rebase操作. 或者执行 `git rebase --skip` 跳过此提交. 或者执行 `git rebase -- abort` 就此终止rebase操作切换到rebase前的分支上.

* `git rebase -i` : 交互式rebase操作
  1. `reword` : 简写为r. 在rebase时会应用此提交, 但是在提交时候允许用户修改提交说明.
  2. `edit` : 简写为e. 在rebase时应用此提交, 当时会在应用后暂停rebase, 提示用户使用 `git commit --amend` 执行提交, 以便对提交进行修补. 当用户执行commit完成提交后, 还需要执行 `git rebase --continue` 继续rebase操作. 用户在rebase暂停状态下可以执行多次提交, 从而实现把一个提交分解为多个提交.
  3. `squash` : 简写为s. 该提交会与前面的提交压缩为一个.
  4. `fixup` : 简写为f. 类似动作squash, 但是此提交的提交说明被丢弃.
  5. `exec` : 简写为x. 可以通过修改编辑任务文件中的各个提交的先后循序, 进而改变最终rebase后提交的先后顺序.
  6. `drop` : 简写为d. 可以修改rebase任务文件, 删除包含相应提交的行, 这样该提交就不会被应用, 进而在rebase后的提交中被删除.

## 删除

* `git rm` : 删除并且更新暂存区.

## 恢复删除

* `git cat-file -p HEAD~1:<filename> > <filename>`
* `git show HEAD~1:<filename> > <filename>`
* `git checkout HEAD~1 -- <filename>`

## 移动文件

* `git mv <paths> <paths>` : 改名

## 查看历史版本文件列表

* `git ls-files --with-tree=HEAD^`

## 查看历史版本中文件内容

* `git cat-file -p HEAD^:**`

## 克隆

* `git clone <repository> <directory>`
  > `git clone demo demo-step-1` : 克隆一份demo命名为demo-step-1
* `git clone --bare <repository> <directory.git>`
* `git clone --mirror <repository> <directory.git>`

## 版本范围标识

* `git rev-list HEAD | wc -l` : 历史提交次数
* `git rev-list --oneline A` : 版本A的所有历史提交
* `git rev-list --oneline A B` : 两个或多个版本

## 保存恢复工作进度

* `git stash` : 保存工作进度
* `git stash list` : 查看保存工作进度列表
* `git stash pop[--index][<stash>]` : 恢复到最近保存进度
  * 如果不使用任何参数, 会恢复最新保存工作进度, 并将恢复的工作进度从存储的工作进度列表中清除.
  * 如果提供<stash>参数(来自git stash list显示的列表), 则从该<stash>中恢复. 恢复完毕也将从进度列表中删除<stash>.
  * 选项--index除了恢复工作区的文件外, 还尝试恢复暂存区.
* `git stash[save[--patch][-k|--[no-]keep-index][-q|--quiet][<message>]]`
  * 这条命令是git stash命令的完整版版. 即如果需要在保存工作进度时候使用指定的说明, 格式如下: `git stash save 'message...'`
  * 使用参数--patch会显示工作区和HEAD的差异, 通过对差异文件的编译决定在进度中最终要保存的工作区的内容, 通过编辑差异文件可以在进度中排除无关内容
  * 使用-k或--keep-index参数, 在保存进度后不会将暂存区重试. 默认会将暂存区和工作区强制重置.
* `git stash apply[--index][<stash>]`
  * 除了不删除恢复的进度之外, 其余和git stash pop命令一样.
* `git stash drop[<stash>]` : 删除一个存储的进度. 默认删除最新的进度.
* `git stash clear` : 删除所有存储的进度;
* `git stash branch<branchname><stash>` : 基于进度创建分支

## 清除本地多余目录文件

查看那些文件和目录会被删除: `git clean -nd`

```
$ git clean -nd                                                   
Would remove detached-commit.txt
```
强制删除多余的目录和文件: `git clean -fd`

```
$ git clean -fd                                                   
Removing detached-commit.txt
```

## 丢弃历史

用A^{tree}语法访问里程碑A对应的目录树

```
$ git cat-file -p A^{tree}

```


## 文件归档

* `git archive`

基于最新提交建立归档文件:

```
$ git archive -o laster.zip HEAD
```

只讲目录src和doc简历到归档partial.tar中

```
$ git archive -o partial.tar HEAD src doc
```

基于tag v1.0建立归档, 并且为归档中的文件添加目录前缀1.0

```
$ git archive --format=tar --prefix=1.0/ v1.0 | gzip > foo-1.0.tar.gz
```

## 忽略文件语法

* 忽略文件中的空行或以井号(#)开始的行会被忽略.
* 可以使用通配符, 参见Linux手册: glob(7). 例如: \*代表任意多字符, ?代表一个字符, 方括号([abc])代表可选字符范围等.
* 如果名称的最前面是一个路径分隔符(/), 表明要忽略的文件在此目录下, 而非子目录的文件.
* 如果名称的最后面是一个路径分隔符(/), 表明要忽略的是整个目录, 同名文件不忽略, 苟泽同名的文件和目录都忽略.
* 通过在名称的最前面加一个感叹号(!), 代表不忽略.
