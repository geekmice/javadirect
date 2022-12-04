

项目开发中git常用指令汇总

## 1、git分支操作

1、初始化项目，初始化git仓库，做一次提交操作。
	git init  #  当前项目初始化为git仓库
	git add . # 当前目录所有文件上传到暂存区域
	git commit  -m 'init project' # 将暂存区域文件上传到本地仓库

2、创建分支
	git branch 分支名字 # 创建制定名称的分支
	git branch dev
	
3、查看分支
	git branch # 查看所有分支，* 表示当前工作分支
	git branch -v # 查看分支详情，包括了分支指向commitId以及提交信息
	例子说明：
	  dev       997291d 提交第一次用来说明master分支

	 * master    997291d 提交第一次用来说明master分支
	  readme.md 997291d 提交第一次用来说明master分支

4、切换分支
	git checkout 分支名字 # 切换到指定分支
	git checkout -b 分支名字 # 创建并且切换分支
	例子说明：切换到dev分支，然后添加dev.txt文件并进行提交
	git checkout dev
	vim dev.txt
	git add .
	git commit -m 'dev分支创建dev.txt提交'
	

	例子说明：在dev分支上创建并切换到test分支
	git checkout dev # 切换分支
	git checkout -b test # 现在处于test分支

5、删除分支
	git branch -d 分支名字 # 删除一个干净的分支(即相对当前分支而言该分支没有新的提交记录)
	git branch -D 分支名字 # 强制删除一个分支，该分支有没有合并到当前分支的提交记录
	注意：删除分支前都需要先切换到其他分支才能进行删除操作

	例子说明：
		在dev分支上删除test分支，test分支相对于dev来说是干净的分支，所以可以直接删除，
	然后切换到master分支，删除dev分支，此时需要进行强制删除。
	
	git checkout dev # 切换到分支dev
	git branch -d test # 在dev分支删除分支test
	git checkout master # 切换到分支master
	git branch -d dev # 在master分支删除dev
		出现错误error: The branch 'dev' is not fully merged.?
		使用以下指令
	git branch -D dev #

6、分支恢复
	思路说明：对于已经有提交记录的分支删除后，实际上只是删除指针，commit记录还保留，
	如果想恢复，需要使用git reflog查找该分支指向的commitId，然后根据commitId创建新的分支
	git branch <branch_name> <hash_val> #根据指定commit创建新分支.
	例题说明：恢复已经删除的dev分支，该分支新增了dev.txt文件
	bbfc160 HEAD@{1}: checkout: moving from test to dev
	git reflog
	git branch dev commitId
	git checkout dev 
	ls 查看
	
7、重命名分支
	git branch -m 分支名 新的分支名
	例子说明：
	git branch -m dev Dev1
	git branch -v # 查看所有分支详情
	
8、分支合并
	git merge <branch_name> #将指定分支合并到当前分支
	如果两个分支没有产生分叉情况，那么会进行快速合并，即fast-forward方式，它并不会产生新的commitId，
	只是改变了指针的指向，产生分叉合并可能会有冲突情况 。
	

	示例：我们在pro分支上新增一次提交，然后合并到master分支上，
	git log查看最新一次的提交记录，显示的正是我们合并分支时的记录

- [1.1 “快进”(无冲突)]
- [1.2 非“快进”，修改不同文件。(无冲突)]
- [1.3 非“快进”，修改相同文件。(有冲突)]

https://blog.csdn.net/qq_42780289/article/details/97945300?utm_medium=distribute.pc_relevant.none-task-blog-searchFromBaidu-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-searchFromBaidu-1.control

git fetch：这将更新git remote 中所有的远程仓库所包含分支的最新commit-id, 
将其记录到.git/FETCH_HEAD文件中

> git fetch更新版本库步骤
> 	git fetch origin master:tmp 
> 	//在本地新建一个temp分支，并将远程origin仓库的master分支代码下载到本地temp分支
> 	git diff tmp 
> 	//来比较本地代码与刚刚从远程下载下来的代码的区别
> 	git merge tmp
> 	//合并temp分支到本地的master分支
> 	git branch -d temp
> 	//如果不想保留temp分支 可以用这步删除
>
> （1）如果直接使用git fetch，则步骤如下：
> 		创建并更新本地远程分支。即创建并更新origin/xxx 分支，拉取代码到origin/xxx分支上。
> 	在FETCH_HEAD中设定当前分支-origin/当前分支对应，如直接到时候git merge就可以将origin/abc合并到abc分支上。
>
> （2）git fetch origin
> 	只是手动指定了要fetch的remote。在不指定分支时通常默认为master
>
> （3）git fetch origin dev
> 	指定远程remote和FETCH_HEAD，并且只拉取该分支的提交。



9、分支合并细节
	git merge --no-ff -m "msg" <branch_name> #合并分支时禁用Fast forward模式
	我们知道如果使用fast-forward方式进行分支合并，只是简单改变了分支指针，而不会产生新的commit记录。
	为了保证合并数据的完整性，我们也可以在合并时指定不使用fast-forward方式，使用 --no-ff 选项。这样，在merge时就会生成一个新的commit，从日志上就可以看到分支合并记录了。
	

	示例：我们在pro分支上新增一次提交，然后合并到master分支上，git log查看最新一次的提交记录，显示的正是我们合并分支时的记录
	
	vim pro3.txt
	git add pro3.txt
	git commit -m 'add pro3.txt file'
	git checkout master
	git merge --no-ff -m 'merge pro branch ' pro
	git log --pretty=oneline -1

10、分支暂存
	git stash #将工作暂存
	git stash list #列出所有的暂存状态
	从暂存区之中进行恢复，有两种处理方式：
	1.先恢复，而后再删除暂存
	git stash apply
	git stash drop
	2.恢复的同时也将stash内容删除
	git stash pop
	当我们在分支上进行代码开发时，有可能会接到突发需求，而当前的代码尚未完成，所以还不能直接提交。
	为了解决这样的问题，git就提供了分支暂存的机制，可以将开发一半的分支进行保存，在适当的时候进行代码恢复。
	

	示例：在pro分支上新建文件，然后添加到暂存区表示尚未完成的任务，对当前分支进行暂存，git status显示工作空间是干净的。
	此时应该切换到master分支上创建新的分支完成突发需求(这里就不做演示了)，然后进行分支恢复。
	
	vim pro1.txt
	git add pro1.txt
	git stash
	git stash list
	git status
	git stash pop

11、分支冲突解决
当对分叉分支进行合并时，如果两个分支都对同一文件进行了修改，那么合并时就有可能会产生冲突情况。
如果两个分支对同一文件的修改是有规律的，比如对不同地方的修改，那么git工具可以实现自动合并，如果无法自动合并，则需要对冲突文件进行手动修改，修改完成后使用git add表示冲突已经解决，然后使用git commit进行提交

示例：在master分支上对两个文件进行修改提交，分别在dev文件的第一行和readme文件的最后一行添加内容。
cat dev.txt
cat readme.txt
git commit -a -m 'master commit'
然后切换到pro分支上对两个文件进行修改提交，分别在dev文件的最后一行和readme文件的最后一行添加内容。

git checkout pro	
cat dev.txt
cat readme.txt
git commit -a -m 'pro commit'

此时进行合并分支操作，提示说readme文件产生了合并冲突(merge conflict)，dev文件由于修改的是不同地方，所以自动合并。
我们查看readme文件的内容，==上面和下面的内容分别代表了不同分支的修改内容，将冲突标记去掉，然后内容根据需求进行恰当的修改，然后进行一次提交即完成了冲突的解决。

git merge pro
cat readme.txt
cat  readme.txt
git add.
git commit -m 'solve conflict'



## 2、问题汇总

### 1、warning: adding embedded git repository？添加嵌入式git仓库

当前目录下面有.git文件夹------默认是隐藏的，直接将.git文件夹掉，再重新git add .

则不再有报警提示，按正常的上传步骤上传代码即可。



### 2、Git解决nothing to commit,working tree clean

第一种就是属于正常情况的这种显示。

第二种是由于git设置为忽略大小写导致这种显示。可以修改当前项目的设置

git config core.ignorecase false  从而不忽略大小写



### 3、failed to push some refs to 'https://gitee.com/initialize_object/test-idea.git？

就是说明本地与远程产生冲突，或是有其他协作者提交了代码，或是你之前在远程上直接做了处理。这部分有两个处理方法，

本地和远程的文件应该合并后才能上传本地的新文件



一是直接强覆盖，二是先把远程的变化拉取下来，解决冲突后，再一并提交。
强覆盖
  git push -f origin master

拉取再提交
  git pull  --rebase 远程连接名 master
  git push origin master

 推荐第二种

### 4、克隆项目时候出现， destination path 'test-idea' already exists and is not an empty directory？？？

​	修改原来文件名
​	要么删除.git文件夹
​	

### 5、warning: LF will be replaced by CRLF in README.md.

The file will have its original line endings in your working directory？

	设置 core.autocrlf=true 后：检出时，git 会把文本文件的换行符转化为 CRLF（只转化纯 LF 的文件）提交时，
	把暂存区的内容（也就是我们对工作区做的改动）转化为 LF 然后放入版本库。转化暂存区的内容时，
	如果发现里面存在 LF 换行符，LF 会被转化成 CRLF，并给出题主提到的那条警告：
	”LF will be replaced by CRLF”这句警告的下面其实还有一句很重要的话：
	warning: LF will be replaced by CRLF in . 
	The file will have its original line endings in your working directory. 
	（翻译下就是：“在工作区里，这个文件会保持它原本的换行符。”） 
	
	简单来说，设置 core.autocrlf=true 后，我们工作区的文件都应该用CRLF 来换行。
	如果改动文件时引入了 LF，或者设置 core.autocrlf 之前，工作区已经有 LF 换行符。
	提交改动时，git 会警告你哪些文件不是纯 CRLF 文件，但 git 不会擅自修改工作区的那些文件，
	而是对暂存区（我们对工作区的改动）进行修改。


### 6、fatal: Not a valid object name: 'master'.

​	创建本地分支：git branch dev
​	报错：fatal: Not a valid object name: 'master'.
​	原因：
​	

	 问题描述-一个非法的master,原因：本地还没有创建master，你可以执行以下git branch，会发现没有看到本地分支列表
	
	解决方案：
	  如果本地没有文件，添加一个文件
	  git add .
	  gi commit -m '依赖文件'
	  git branch 
	  最后创建本地分支
	  git branch dev


### 7、删除远程分支，本地分支？

删除远程：git push origin --delete bug_f1 # 说明 origin:远程连接名字 bug_f1 远程分支名字

删除本地：git branch -d bug_f2 # 说明 bug_f2 本地分支名字



### 8、添加远程分支，本地分支？

添加远程：git remote add origin f1:f1 # 说明：origin:远程连接名字 第一个f1:表示远程分支名字 第二个f1：表示本地分支

添加本地：git branch f3 # f3 本地分支



### 9、查看本地仓库文件还没有添加内容？

查看已存放：（这个最有用）
git ls-files
查看还没添加的文件：
git status



### 10、删除本地仓库？

rm -rf .git 



### 11、git各个分支内容为什么一模一样？

你在test分支下新建了文件，要先add 、commit后再切换回master，master分支就不显示这个文件了



### 12、git remote -v 查询所有远程连接 什么都不显示？

​    需要首先进行连接

```
git remote add distant003 https://gitee.com/initialize_object/springboot-utools.git
```



### 13、执行git push出现"Everything up-to-date"？

在github上git clone一个项目，在里面创建一个目录，然后git push的时候，出现报错"Everything up-to-date"

原因：
1）没有git add .
2）没有git commit -m "提交信息"
如果上面两个步骤都成功执行，还出现这个错误是因为创建的目录下是空的，目录下必须有文件才能git push上传成功。

在github上创建文件的时候，在新文件名后加/符号就是文件夹，但是这种方式只支持英文名目录，中文名目录不支持。

### 14、git push -u 作用

git push -u origin master:master 

下次提交仓库直接使用git push  平时练习可以使用，但是出现多个分支，需要指定分支，说明清楚哪个远程分支名字



### 15、rejected]        dev        -> dev  (non-fast-forward)

 提示! [rejected] dev -> dev (non-fast-forward)， pull了远程代码重新提交，还是同样的提示，最终尝试另外的方式才得解决:

```
git fetch origin dev //获取远程dev分支的修改
git merge origin dev // 合并远程dev分支
git pull origin dev // 更新本地的代码
```

### 16、Git :fatal: refusing to merge unrelated histories

```
上网查到原因是两个分支是两个不同的版本，具有不同的提交历史

git pull origin master --allow-unrelated-histories
也可以是使用
git merge 分支名字 --allow-unrelated-histories
```

### 17、更新远程版本库推荐操作

```bash
多人合作：
	git fetch origin master:tmp 
	//在本地新建一个temp分支，并将远程origin仓库的master分支代码下载到本地temp分支
	git diff tmp 
	//来比较本地代码与刚刚从远程下载下来的代码的区别
	git merge tmp
	//合并temp分支到本地的master分支
	git branch -d temp
	//如果不想保留temp分支 可以用这步删除
单人练习：
  git pull  --rebase 远程连接名 master
```

### **18、git fetch失效** 

```bash
fatal:Refusing to fetch into current branch refs/heads/dev001 of non-bare repository
```

这种时候，先切换到master分支，然后再从master分支fetch分支dev001上的代码，就能够成功了。

```bash
Thinkpad@DESKTOP-4MND4UG MINGW64 /g/Desktop/项目组前期准备/hap架构/test-git (dev001)
$ git checkout master
Switched to branch 'master'

Thinkpad@DESKTOP-4MND4UG MINGW64 /g/Desktop/项目组前期准备/hap架构/test-git (master)
$ git fetch origin001 dev:dev001
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 1 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), 205 bytes | 0 bytes/s, done.
From https://gitee.com/initialize_object/test-idea001
 ! [rejected]        dev        -> dev001  (non-fast-forward)
 * [new branch]      dev        -> origin001/dev

```

究其原因，就和fatal的提示一样，在非bare的git仓库中，如果你要同步的本地跟踪分支是当前分支，就会出现拒绝fetch的情况。也就是说不可以在非bare的git仓库中通过fetch快进你的当前分支与远程同步。

```bash
Git裸仓库创建 使用命令行：git init –bare 使用TortoiseGit：右键菜单git creat repo here，选择Make it Bare
裸仓库可以直接作为服务器仓库供各开发者push、pull数据，实现数据共享和同步，不保存文件，只保存历史提交的版本信息
向非裸仓库push文件会报错，需要在.git 文件夹的config文件后加一句
[receive]
denyCurrentBranch = ignore
才能提交数据，非裸仓库使用git reset --hard命令可以看到提交文件
```



### **19、git pull, git push 快速推送和快速拉取**

```bash
There is no tracking information for the current branch.
Please specify which branch you want to merge with.
See git-pull(1) for details.

    git pull <remote> <branch>

If you wish to set tracking information for this branch you can do so with:

    git branch --set-upstream-to=<remote>/<branch> master
是因为本地分支和远程分支没有建立联系  (使用git branch -vv  可以查看本地分支和远程分支的关联关系)  .根据命令行提示只需要执行以下命令即可

git branch --set-upstream-to=origin/远程分支的名字 本地分支的名字   
```

### **20、git status 三种状态**

1、初始化git仓库

```
$ git init
状态如下-----untracked files
$ git status
On branch master

No commits yet
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        demo001.md
        m1.md
        m3241.md
        mtest001.md
nothing added to commit but untracked files present (use "git add" to track)
```

2、提交暂存区域

```bash
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   m1.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        demo001.md
        m3241.md
        mtest001.md
        sd.md

```

3、提交本地仓库

```
On branch master
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        demo001.md
        m3241.md
        mtest001.md
        sd.md

nothing added to commit but untracked files present (use "git add" to track)

```

​	以看到，除了之前的“Changes to be committed”状态，现在又多了一条“Changes not staged for commit”状态，表明文件已经修改，但是还没有放入暂存区域，也就是没生成快照。如果现在进行commit操作，只是将修改之前的文件快照提交到了git目录，一定记住：只有暂存区域的文件（即：文件状态为“Changes to be committed”）才会被提交。正如提示，通过“git add README.txt”命令将已修改文件更新到暂存区域中，如果想撤销修改，可以使用“git checkout -- README.txt”命令。

### 21、git拉取代码出现问题合并

[git中Please enter a commit message to explain why this merge is necessary.](https://www.cnblogs.com/wei325/p/5278922.html)

**Please enter a commit message to explain why this merge is necessary.**

git 在pull或者合并分支的时候有时会遇到这个界面。可以不管(直接下面3,4步)，如果要输入解释的话就需要:

1.按键盘字母 i 进入insert模式

2.修改最上面那行黄色合并信息,可以不修改

3.按键盘左上角"Esc"

4.输入":wq",注意是冒号+wq,按回车键即可

### 22、 The following untracked working tree files would be overwritten by merge:

删除本地相关的文件再次提交



## 3、git操作流程详细讲解

   **本地master-远程master分支提交**

```bash
 mkdir test-idea                                                         # 创建文件夹test-idea
 cd test-idea/ 													  # 切换到文件夹test-idea
 git init 												        # 将此文件夹初始化为git仓库
 touch readme.md 											 # 创建文件readme.md
 git add readme.md  								       # 将当前目录readme.md文件提交到暂存区域
 git commit -m "first commit" 						     # 将暂存区域内容提交本地仓库
 git remote add origin https://gitee.com/initialize_object/test-idea.git # 将本地版本库和远程进行连接
 git push origin master 							  # 将本地仓库推送到远程
```

**本地dev分支提交master分支**

```bash
一、dev分支执行以下命令
	git add . # 暂存区域所有更改
	git commit  -m '提交说明'
	git pull origin001 dev # 拉取代码到本地dev分支
	git push origin001 dev # 推送本地dev版本库到远程dev
	git checkout master # 切换分支master
二、master分支执行以下命令
	git merge dev # dev分支合并到master分支
	git branch -D dev # 强制删除dev分支
	git push origin001 master # 推送本地master分支到远程master
三、可能需要命令
	git log   #可以看到近期的相关提交日志（提交时候的备注等）
	git status # // 可以看到当前的文件状态 （如xx文件被修改，但未提交等)
```

**本地dev分支提交到远程dev分支**

```bash
1、git checkout -b dev # 切换并且创建dev分支
2、touch readme.md       # 创建文件
3、vim readme.md           # 编辑文件
4、git add readme.md         # 提交暂存区域
5、git commit -m '说明情况'    # 提交本地仓库
6、git pull origin001 dev       # 拉取远程相对应分支情况 origin001 远程名
7、git push origin001 dev         # 推送本地dev分支到远程版本库
```



## 4、多个git账户团队合作（重点）

克隆 Git 资源作为工作目录。

在克隆的资源上添加或修改文件。

如果其他人修改了，你可以更新资源。

在提交前查看修改。

提交修改。

在修改完成后，如果发现错误，可以撤回提交并再次修改并提交。

## 5、项目中常用git指令

**下载&初始化**

```bash
git clone A //从远程仓库下载文件 A 表示克隆地址
git init    //在需要上传的文件下初始化仓库 
```

**对文件进行操作**

```java
git add <filename> //将文件夹下的所有文件上传到工作区 , *表示上传所有
git commit -m '提交说明' //将添加的文件提交到仓库
git status //查看当前工作树的状态
```

**远程连接**

```java
git remote //查看所有的远程连接
git remove -v //查看链接的	详细信息
git remote add <远程名 例如：origin或其他> <远程地址> //添加远程连接
git pull <远程名> <远程分支名>:<本地分支名> //拉取远程上某个分支的文件，与本地分支文件合并
git push <远程名> <本地分支名>：<远程分支名> //将合并后的文件推送到远程仓库上    
```

**分支操作**

```sh
git branch //查看当前仓库所有的分支
git branch -a //查看本地和远程所有的分支
git branch -r //查看被远程跟踪的分支
git branch dev2 //新建一个分支
git checkout <branchName> //切换到指定分支
git branch -m <原分支名> <新分支名> //修改分支名称
git branch -d <branchName> //删除分支
git branch merge <被合并的分支名> //合并分支
```

**分享和更新项目**

```sh
git fetch：下载远程仓库 origin 到本地
git fetch remoterepository：下载指定远程仓库到本地
git fetch remoterepository branchname：下载指定远程仓库指定分支到本地
git pull remoterepository branchname[:localbranch]：拉取指定远程仓库指定分支到本地仓库指定分支（默认是当前分支）
git push remoterepository localbranch[:remotebranch] [--tags]：推送本地仓库指定分支到远程仓库指定分支（默认是与本地分支同名的远程分支），默认是不推送标签到远程仓库的，加上 –tags 就会推送标签
git remote：查看所有与本地仓库关联的远程仓库
git remote -v：查看所有与本地仓库关联的远程仓库，并显示 url
git remote add remote-name remote-url：添加与本地仓库关联的远程仓库
git remote rename oldname newname：为远程仓库重命名
git remote remove remote-name：移除远程仓库
```

**比较**

```sh
git show [-times]：显示最近 times 次（默认是一次）提交的所有对象信息
git log：查看提交记录
git log --all：查看所有提交记录
git log --oneline：查看提交记录，以 oneline 形式显示，只显示一行，显示的内容时提交hash的前7位与提交消息
git log -p -times：表示查看最近 times 次提交改变的内容
git log -stat [-times]：查看最近 times 次（默认是所有）提交记录，并显示文件的差异分析
git diff：查看工作目录与暂存区的差异
git diff --cached [<commit>]：查看暂存区与指定提交（默认是HEAD）的差异
git diff <commit>：查看工作目录与指定提交的差异
git diff <commit>：查看工作目录与指定提交的差异
git diff <commit> <commit>：查看两次指定提交的差异
git diff branchname：查看工作目录与指定分支的差异
git diff branchname branchname：查看两个指定分支间的差异
```

**调试**

```sh
显示修改和作者最后修改的文件的每一行，这就是一个“问责”的命令，如果哪里有问题，我们可以很快地找到该问题是谁导致的。 - git blame filename：查看指定文件所有的操作者，看看是谁错误地修改了该文件
$ git blame test.md
^c6deafc (NewJackRose 2020-12-10 19:49:24 +0800 1) ljljljljljlj

git grep keys：在工作目录中所有文件中搜索 keys
git grep --cached keys：在暂存区中所有文件中搜索 keys
```

