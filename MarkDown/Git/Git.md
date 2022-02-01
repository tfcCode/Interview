# 一、基础命令

```c
git init
```

初始化本地仓库



```c
git add 文件名
```

将文件添加到暂存区，此时还没有添加到本地仓库，可以添加多个文件到暂存区



```c
git commit -m '提交说明'
```

将暂存区的文件全部添加到当前分支



```c
git status
```

查看仓库中的文件的变化



``` c
git diff 文件名
```

查看文件具体改动的哪些内容



# 二、回退

每当你觉得文件修改到一定程度的时候，就可以“保存一个快照”，这个快照在 Git 中被称为`commit`。一旦你把文件改乱了，或者误删了文件，还可以从最近的一个`commit`恢复，然后继续工作，而不是把几个月的工作成果全部丢失

## 1、查看提交记录

```c
git log
```

查看历史提交记录，从近到远。

你看到的一大串类似`1094adb...`的是`commit id`（版本号），和SVN不一样，Git的`commit id`不是1，2，3……递增的数字，而是一个 SHA1 计算出来的一个非常大的数字，用十六进制表示，而且你看到的`commit id`和我的肯定不一样，以你自己的为准

为什么`commit id`需要用这么一大串数字表示呢？因为Git是分布式的版本控制系统，后面我们还要研究多人在同一个版本库里工作，如果大家都用1，2，3……作为版本号，那肯定就冲突了



```c
git reflog
```

这个命令可以记录你所有的版本回退，既可以向前回退，也可以向后回退



## 2、版本回退

```c
git reset --hard  HEAD^/版本号
```

用`HEAD`表示当前版本，也就是最新的提交`1094adb...`，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100` 

也可以直接写版本号，版本号没必要写全，前几位就可以了，Git会自动去找。当然也不能只写前一两位，因为Git可能会找到多个版本号，就无法确定是哪一个了



总结：版本回退方法：

1、git reflog 查看所有的回退记录

2、根据 commit 的提示信息回退到需要的版本，执行 git reset --hard 版本号



## 3、撤销修改

```c
git checkout -- <fileName>
```

丢弃工作区的修改（还没有执行 git add）

命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

1. 一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态

2. 一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态



```c
git restore --staged <fileName>
```

撤销 git add 操作，将提交到暂存区的内容撤销，回退到提交前的状态



## 4、删除文件

一般情况下，你通常直接在文件管理器中把没用的文件删了，或者用 `rm` 命令删了，Git 知道你删除了文件，因此，**工作区和版本库就不一致了**，`git status` 命令会立刻告诉你哪些文件被删除了

```c
$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	deleted:    test.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

现在你有两个选择，一是确实要从版本库中删除该文件，那就用命令 `git rm` 删掉，并且 `git commit`，之后，文件就从版本库中被删除了

小提示：先手动删除文件，然后使用 `git rm <file>` 和 `git add <file>` 效果是一样的

注意：从来没有被添加到版本库就被删除的文件，是无法恢复的！



# 三、远程仓库

## 1、将本机与远程服务器相关联

1）打开 Git Bash，执行下列命令

```c
ssh-keygen -t rsa -C "youremail@example.com"
```

你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可

2）在用户主目录里找到`.ssh`目录，里面有`id_rsa`和`id_rsa.pub`两个文件，这两个就是SSH Key的秘钥对，`id_rsa`是私钥，不能泄露出去，`id_rsa.pub`是公钥，可以放心地告诉任何人

登陆GitHub，打开“Account settings”，“SSH Keys”页面，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容即可

当然，GitHub 允许你添加多个 Key。假定你有若干电脑，你一会儿在公司提交，一会儿在家里提交，只要把每台电脑的 Key 都添加到 GitHub，就可以在每台电脑上往GitHub推送了



## 2、关联远程仓库

```c
git remote add origin git@github.com:michaelliao/learngit.git
```

其中，`origin` 是远程库的别名，最后一个长串参数是远程库地址



### 查看远程库

```c
git remote -v
```



### 删除远程库

```c
git remote rm 别名
```

此处的"删除"其实是**解除了本地和远程的绑定关系，并不是物理上删除了远程库**。远程库本身并没有任何改动。要真正删除远程库，需要登录到 GitHub，在后台页面找到删除按钮再删除



## 3、推送

```c
git push -u origin master
```

解释：向远程库 origin 推送本地的 master 分支

第一次推送`master`分支时，加上了`-u`参数，Git不但会把本地的`master`分支内容推送的远程新的`master`分支，还会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令

**注意：最新的远程默认分支是 main 分支，旧版本本地的默认分支是 master，需要先修改本地分支名** 



## 4、克隆远程库

假设我们从零开发，那么最好的方式是先创建远程库，然后，从远程库克隆

```c
git clone git@github.com:michaelliao/gitskills.git
```

注意把 Git 仓库地址换成你自己的

Git支持多种协议，默认的`git://`使用ssh，但也可以使用`https`等其他协议。

使用`https`除了速度慢以外，还有个最大的麻烦是每次推送都必须输入口令，但是在某些只开放http端口的公司内部就无法使用`ssh`协议而只能用`https` 



# 四、分支

> 分支在实际中有什么用呢？

假设你准备开发一个新功能，但是需要两周才能完成，第一周你写了50%的代码，如果立刻提交，由于代码还没写完，不完整的代码库会导致别人不能干活了。如果等代码全部写完再一次提交，又存在丢失每天进度的巨大风险

现在有了分支，就不用怕了。你创建了一个属于你自己的分支，别人看不到，还继续在原来的分支上正常工作，而你在自己的分支上干活，想提交就提交，**直到开发完毕后，再一次性合并到原来的分支上**，这样，既安全，又不影响别人工作

每次提交，Git 都把它们串成一条时间线，**这条时间线就是一个分支**。截止到目前，只有一条时间线，在Git里，这个分支叫主分支，即`master`分支。`HEAD`严格来说不是指向提交，而是指向`master`，`master`才是指向提交的，所以，`HEAD`指向的就是当前分支

每次提交，`master`分支都会向前移动一步，这样，随着你不断提交，`master`分支的线也越来越长



## 1、创建分支

```c
git checkout -b dev  / git switch -c dev
```

`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令：

```c
git branch dev    // 创建分支
git checkout dev
```



## 2、查看分支

```c
git branch
```

`git branch`命令会列出所有分支，当前分支前面会标一个`*`号



## 3、切换分支

```c
git switch/checkout <branchName>
```

使用 switch 更容易理解



## 4、删除分支

```c
git branch -d <branchName>
```



## 5、修改分支名

修改当前项目分支名

```c
git branch -m oldName newName
```

修改默认分支名

```c
git config --global init.defaultBranch main
```





## 5、合并分支

1）首先切换到接收合并的分支

2）执行下列命令进行合并

```c
git merge 分支名
```

这里的分支名时被合并的分支名，比如将 dev 分支合并到 master 分支，先切换到 master 分支，然后执行
git merge dev 就可以将 dev 分支中的内容合并到 master 分支上



## 6、解决冲突

当不同的分支修改同一文件后进行合并时，会出现冲突，必须手动解决冲突，然后提交

解决冲突就是把 Git 合并失败的文件**手动编辑为我们希望的内容**，再提交



## 7、分支管理策略

通常，合并分支时，如果可能，Git会用`Fast forward`模式，但**这种模式下，删除分支后，会丢掉分支信息** 

如果要强制禁用 `Fast forward` 模式，Git 就会在 merge 时生成一个新的 commit，这样，从分支历史上就可以看出分支信息

```c
git merge --no-ff -m "merge with no-ff" <branchName>
```

请注意`--no-ff`参数，表示禁用`Fast forward` 

因为本次合并要创建一个新的commit，所以加上`-m`参数，把 commit 描述写进去

> 分支管理策略

1、首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活

2、干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把`dev`分支合并到`master`上，在`master`分支发布1.0版本；

3、你和你的小伙伴们每个人都在`dev`分支上干活，每个人都有自己的分支，时不时地往`dev`分支上合并就可以了。

所以，团队合作的分支看起来就像这样：

![](images/0.png)



## 8、Bug 分支

开发中，bug 就像家常便饭一样。有了 bug 就需要修复，在 Git 中，由于分支是如此的强大，所以，**每个bug都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除** 

当有 bug 出现时，使用下列命令保存工作现场，解决 bug 后恢复现场后继续工作

```c
git stash
```

现在，用`git status`查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复 bug

1、首先确定要在哪个分支上修复 bug，假定需要在`master`分支上修复，就从`master`创建临时分支

2、修复完成后，切换到`master`分支，并完成合并，最后删除`bug`分支，回到 dev 分支接着干活



工作区是干净的，刚才的工作现场存到哪去了？用`git stash list`命令看看：

```c
$ git stash list
stash@{0}: WIP on dev: ac97353 解决冲突
```

工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：

1、一是用`git stash apply`恢复，但是恢复后，stash 内容并不删除，你需要用`git stash drop`来删除

2、另一种方式是用`git stash pop`，恢复的同时把 stash 内容也删了



你可以多次stash，恢复的时候，先用`git stash list`查看，然后恢复指定的 stash，用命令：

```c
$ git stash apply stash@{0}
```



在 master 分支上修复了 bug 后，我们要想一想，dev 分支是早期从 master 分支分出来的，所以，这个bug其实在当前 dev 分支上也存在

**那怎么在 dev 分支上修复同样的 bug？重复操作一次，提交不就行了？有木有更简单的方法？有** 

同样的 bug，要在 dev 上修复，我们只需要把 bug 分支上所的提交所做的修改"复制"到 dev 分支即可

**注意：我们只想复制 bug 分支这个提交所做的修改，并不是把整个 master 分支 merge 过来** 

为了方便操作，Git专门提供了一个`cherry-pick`命令，让我们能复制一个特定的提交到当前分支：

```c
git cherry-pick <commitID>
```



# 五、多人协作

当你从远程仓库克隆时，实际上Git自动把本地的`master`分支和远程的`master`分支对应起来了，并且，远程仓库的默认名称是`origin` 

使用 git remote -v 查看详细信息

```c
$ git remote -v
origin  git@github.com:michaelliao/learngit.git (fetch)
origin  git@github.com:michaelliao/learngit.git (push)
```

上面显示了可以抓取和推送的`origin`的地址。如果没有推送权限，就看不到push的地址



## 1、推送分支

把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上

```c
git push origin <localBranchName>
```

把本地分支推送到远程分支，origin 默认关联的是远程的 master 分支



## 2、抓取分支

从远程库clone时，默认情况下，你只能看到本地的`master`分支。不信可以用`git branch`命令看看：

```c
$ git branch
* master
```

要在`dev`分支上开发，就必须创建远程`origin`的`dev`分支到本地，于是他用这个命令创建本地`dev`分支：

```c
$ git switch -c dev origin/dev
```

上述命令创建了一个本地 dev 分支，并且将远程的 dev 分支与本地的相关联

也可以手动进行关联：

1、创建一个本地的普通 dev 分支

2、使用如下命令与远程的分支相关联：

```c
git branch --set-upstream-to=<originBranchName> <localBranchName>
```



## 3、解决冲突

若两个人同时修改了同一文件，并且另一人先提交并推送到远程仓库，这时你再推送就会报错，解决如下：

1、先使用 `git pull` 将远程分支抓取到本地

2、手动解决冲突

3、提交、推送

git pull 之前需要先关联指定本地`dev`分支与远程`origin/dev`分支：

```c
git branch --set-upstream-to=<originBranchName> <localBranchName>
```

这次`git pull`成功，但是合并有冲突，需要手动解决，解决后，提交，再 push



# 六、标签管理

发布一个版本时，我们通常先在版本库中打一个标签（tag），这样，就**唯一确定了打标签时刻的版本**。将来无论什么时候，取某个标签的版本，就是把那个打标签的时刻的历史版本取出来。所以，标签也是版本库的一个快照

Git 的标签虽然是版本库的快照，但**其实它就是指向某个 commit 的指针**（跟分支很像对不对？但是分支可以移动，标签不能移动），所以，创建和删除标签都是瞬间完成的

> Git有commit，为什么还要引入tag？

“请把上周一的那个版本打包发布，commit号是6a5819e...”

“一串乱七八糟的数字不好找！”

如果换一个办法：

“请把上周一的那个版本打包发布，版本号是v1.2”

“好的，按照tag v1.2查找commit就行！”



**所以，tag就是一个让人容易记住的有意义的名字，它跟某个commit绑在一起** 



## 1、创建标签

1）首先切换到需要打标签的分支上

2）敲命令`git tag <name>`就可以打一个新标签：

```c
git tag v1.0
```

**默认标签是打在最新提交的commit上的**，可以用命令`git tag`查看所有标签：

有时候，如果忘了打标签，比如，现在已经是周五了，但应该在周一打的标签没有打，怎么办？

* **方法是找到历史提交的 commitId，然后打上就可以了** 

比方说要对`add merge`这次提交打标签，它对应的commiId是`f52c633`，敲入命令：

```c
$ git tag v0.9 f52c633
```

注意，标签不是按时间顺序列出，而是按字母排序的。可以用`git show <tagname>`查看标签信息：

```c
$ git show v0.9
commit f52c63349bc3c1593499807e5c8e972b82c8f286 (tag: v0.9)
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:56:54 2018 +0800

    add merge

diff --git a/readme.txt b/readme.txt
...
```



还可以创建带有说明的标签，用`-a`指定标签名，`-m`指定说明文字：

```c
$ git tag -a v0.1 -m "version 0.1 released" 1094adb
```

注意：标签总是和某个commit挂钩。如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签



## 2、操作标签

如果标签打错了，也可以删除：

```c
$ git tag -d v0.1
```

因为创建的标签都只存储在本地，不会自动推送到远程。所以，打错的标签可以在本地安全删除

如果要推送某个标签到远程，使用命令`git push origin <tagname>`：

```c
git push origin v1.0
```

或者，一次性推送全部尚未推送到远程的本地标签：

```c
git push origin --tags
```



如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：

```c
$ git tag -d v0.9
```

然后，从远程删除。删除命令也是push，但是格式如下：

```c
git push origin :refs/tags/v0.9
```

要看看是否真的从远程库删除了标签，可以登陆GitHub查看



# 七、.gitignore

有些时候，你必须把某些文件放到Git工作目录中，但又不能提交它们，比如保存了数据库密码的配置文件等等，每次`git status`都会显示`Untracked files ...`，有强迫症的人心里肯定不爽

解决办法：在Git工作区的根目录下创建一个特殊的`.gitignore`文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件



不需要从头写`.gitignore`文件，GitHub 已经为我们准备了各种配置文件，只需要组合一下就可以使用了。所有配置文件可以直接在线浏览：https://github.com/github/gitignore



有些时候，你想添加一个文件到Git，但发现添加不了，原因是这个文件被`.gitignore`忽略了，如果你确实想添加该文件，可以用`-f`强制添加到Git：

```c
git add -f App.class
```



如果`.gitignore`写得有问题，需要找出来到底哪个规则写错了，可以用`git check-ignore`命令检查：

```c
$ git check-ignore -v App.class
.gitignore:3:*.class	App.class
```

Git会告诉我们，`.gitignore`的第3行规则忽略了该文件，于是我们就可以知道应该修订哪个规则



把指定文件排除在`.gitignore`规则外的写法就是`!`+文件名，所以，只需把例外文件添加进去即可

























