[TOC]

### GIT

#### 前言

>   分布式版本控制 Distributed Version Control System

1. 本地完全克隆代码仓库
2. Git 保存的不是文件的变化或者差异，而是一系列不同时刻的 **快照**
3. 几乎所有操作都在本地，只是最后提交到远程仓库
4. Git保证数据完整性[sha-1散列 基于目录或文件内容]

#### 概念

1.  工作区【已修改】
2.  暂存区【已暂存，保留下次要提交的文件信息】
3.  本地仓库【已提交，永久存储到git目录】
4.  远程仓库

![git流程](./images/git.svg)

#### 安装配置

~~~bash
sudo apt install git-all
git config --global user.name "God Yao"      		[--local]
git config --global user.email "mike@example.com"	[--local]
git config --list
git add -h	[获取指定命令的帮助文档]
~~~

#### 基础命令

1.  仓库

~~~bash
# 本地初始化仓库
mkdir testGit
git init 
# 拉去原创仓库
git clone xx.git
~~~

2.  查看文件状态

~~~bash
git status  
# 1. Untracked 
# 2. new file 
# 3. modified
git status -s
~~~

3. 跟踪文件(暂存区)

~~~bash
git add code.txt   # git add . 提交全部 也支持 git add *.c
~~~

4.  提交本地仓库

~~~bash
git commit -m "version description"
git commit -a -m "version description"	# 跳过add阶段，或者说是自动add
~~~

5.  查看差异

~~~bash
git diff 		   # 工作目录中当前文件和暂存区域快照之间的差异，本身只显示尚未暂存的改动
git diff --staged  # 暂存区和最后一次提交的差异
~~~

6. 查看提交历史


~~~bash
git log
git log -p    # 显示提交差异
git log -p -2 # 两次提交
git log --stat
git log --pretty=oneline
git log --pretty=format:"%h %an %ae %s" --graph [hash author email commitmessage]
git reflog    # 查看所有的版本
~~~

7. 暂存区 -> 工作区

~~~bash
git reset head [file]	# 回到工作区，保留修改
git checkout -- [file] 	# 回到工作区，不保留修改，会退到最后一次提交【危险命令】

git commit --amend # 将暂存区中的文件提交
~~~

8. 版本仓库回退

~~~bash
git reset --hard HEAD^
git reset --hard HEAD^^
git reset --hard HEAD~1
git reset --hard HEAD~10
~~~

9. 忽略文件.gitignore

~~~bash
# 忽略所有的 .a 文件
*.a
# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a
# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO
# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
~~~

10. 远程仓库

~~~bash
git remote -v
git remote show origin # 查看远程分支的详细信息
git remote add [仓库名字] xx.git

git remote rename [仓库原名] [仓库新名]
git remote remove|[rm] [仓库名]

git fetch [仓库名字]   # 不会自动合并
#自动设置本地 master 分支跟踪克隆的远程仓库的 master 分支（或其它名字的默认分支）
git clone xx.git 
# 从最初克隆的服务器上抓取数据并自动尝试合并到当前所在的分支
git pull 
# 推送，推送之前需要pull，因为别人可能已经推送过 
git push origin master
~~~

11. 标签

~~~bash
git tag -l [--list]
git tag -a v1.0.0 -m "tag message"
git show v1.0.0
# 对过去的提交打标签
git tag -a v0.0.8 [commit-hash]

# 必须显式的推送标签
git push origin v1.0.0

# 删除标签
git tag -d <tagname>
git push origin --delete <tagname>
~~~

#### 分支

> Git 的分支，其实本质上仅仅是指向提交对象的可变指针
>
> HEAD 指针，指向当前分支

##### 基本操作

~~~bash
git branch 				# 本地分支
git branch -a 			# 远程分支
git branch testing		# 创建
git branch -b hotfix	# 创建并且切换
git branch -d hotfix	# 删除分支
git merge dev 			# 合并分支
~~~

试图合并两个分支时， 如果顺着一个分支走下去能够到达另一个分支，那么 Git 在合并两者的时候， 只会简单的将指针向前推进（指针右移）因为这种情况下的合并操作没有需要解决的分歧——这就叫做 “快进（fast-forward）”

![basic-branching-4](images/basic-branching-4.png)

直接移动指针即可

![basic-branching-5](images/basic-branching-6.png)



但是合并iss53时，出现分叉，无法靠移动指针合并，找到各分支的位置，以及公共的祖先

![basic-merging-1](images/basic-merging-1.png)

生成一个新的快照，合并过程中可能有冲突出现，解决冲突，然后重新提交

![basic-merging-2](images/basic-merging-2.png)

##### 远程分支

> 以 `<remote>/<branch>` 的形式命名, remote 一般是 origin

~~~bash
git remote show <remote>
git fetch 
git push <remote> <branch>:<remote-branch>

git checkout -b serverfix origin/serverfix
git checkout --track origin/serverfix

# 实际上这个操作包含了前面两个，新建本地，跟踪远程
git checkout serverfix  

# 删除远程分支
git push origin --delete serverfix
~~~

#### 开发流程

##### 常用命令

>   默认分支在master，常见的分支命名: master dev bug hotfix feature

0.  克隆

~~~bash
git clone path.git
~~~

1.  查看分支

~~~bash
git branch
git branch -a # 查看远程分支
~~~

2.  新建分支

~~~bash
git branch dev
git checkout -b hotfix # 新建并且切换
~~~

3.  切换分支

~~~bash
# 如果有远程分支，就会直接拉取，跟踪
git checkout dev 
~~~

4.  合并分支

~~~bash
git checkout master
git merge dev			   # 合并本地分支
git merge origin/serverfix # 合并远程分支
~~~

6.  删除分支

~~~bash
git branch -d dev
git branch -D feature 				# 强行删除没有合并的分支
git push origin --delete serverfix  # 删除远程分支
~~~

7. 推送到远程分支

~~~bash
git push <远程仓库> <本地分支>:<远程分支>
~~~

8. 更新

~~~bash
# 当 git fetch 命令从服务器上抓取本地没有的数据时
# 它并不会修改工作目录中的内容。 它只会获取数据然后让你自己合并 如需彻底更新需合并或使用git pull
git fetch 
~~~

9. 拉取

~~~bash
# 拉取远程主机某分支的更新，再与本地的指定分支合并（相当与fetch加上了合并分支功能的操作）
git pull 
git pull --rebase
# git pull = git fetch + git merge 通常显式使用 git fetch + git merge 比较好
# git pull --rebase = git fetch + git rebase
~~~

#### 冲突

1. 冲突

    > 不同的分支修改同样的地方，并且合并，此时合并回发生冲突

~~~bash
git status # 查看冲突
~~~

2. 解决

    > 可以将提示的部分删除，然后再次提交

3. 查看分支情况

~~~bash
git log
git log --graph
git log --graph --pretty=oneline --abbrev-commit
~~~

4.  当正在编写代码时，突然需要解决一个BUG，此时可以将工作区暂存

~~~bash
git stash
git checkout -b bug
~~~

5. 查看暂存

    > 修复完成bug，然后提交，切换到master然后合并到bug，然后需要回到暂存的工作区

~~~bash
git stash list
~~~

6.  恢复

~~~bash
# 方式一
git stash apply  # stash内容并不删除
git stash drop	 # 需要手动删除
# 方式二
git stash pop
~~~

#### GitFlow

一般我们都会有master分支，dev分支，hotfix分支，feature分支

1.  master/release分支上的代码最稳定的，一般不要轻易合并到它上，一定要严格测试，属于线上环境
2.  dev分支是相对稳定的，属于线上测试
3.  hotfix修复分支
4.  feature新功能分支

#### Tag

1. tag用于发布一次较稳定版本

    > 默认标签是打在最新提交的commit上的【标签总是和某个commit挂钩】

~~~bash
git tag v1.0
~~~

2.  指定在某个commit上tag

~~~bash
 git tag v1.1 f52c633
~~~

3.  tag也可以说明

~~~bash
git tag -a v0.1 -m "version 0.1 released" 1094adb
~~~

4.  查看

~~~bash
git show v1.1
~~~

5.  默认tag是在本地，可以推送到远程

~~~bash
git push origin v1.1
git push origin --tags # 全部推送
~~~

6.  删除tag

~~~bash
# 本地
git tag -d v1.1
# 推送到远程，需要删除，先删本地，在删除远程
git tag -d v1.1
git push origin :refs/tags/v1.9
~~~

#### 多人协作

1.  SSH
2.  用户

~~~bash
git config --global user.name "username"
git config --global user.email "email"
~~~

因此，多人协作的工作模式通常是这样

1.  首先，可以试图用`git push origin <branch-name>`推送自己的修改
2.  如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并
3.  如果合并有冲突，则解决冲突，并在本地提交
4.  没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功

#### 常见问题

**提交暂存区出错**

1.  改乱了工作区某个文件的内容，想直接丢弃工作区的修改时

~~~bash
git checkout -- file
~~~

2.  改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，第一步用命令

~~~bash
git reset HEAD file
~~~

​	就回到了场景1，第二步按场景1操作

3.  已经提交了不合适的修改到版本库时，想要撤销本次提交

~~~bash
git reset –hard 版本号
~~~

**提交本地仓库出错**

1.  提交信息出错

~~~bash
git commit --amend -m "新提交的信息"
~~~

2.  漏提交

~~~bash
git add missed-file 		  # missed-file 为遗漏提交文件
git commit --amend --no-edit  # 表示提交消息不会更改，在 git 上仅为一次提交
~~~

3. reset

    > 提交错误文件，回退上个commit版本，再commit

~~~bash
# 修改版本库，保留暂存区，保留工作区
git reset --soft HEAD~1
# 修改版本库，修改暂存区，修改工作区
git reset --hard HEAD~1
# 回退到特定的版本，可以通过git log查看提交历史，以便确定要回退到哪个版本
git reset --hard commit_id 
~~~

4. git revert

    > 通常使用revert

~~~bash
# 撤销前一次 commit
git revert HEAD
# 撤销前前一次 commit
git revert HEAD^
# 撤销制定版本
git revert commit_id
~~~

5. `git revert` 和 `git reset` 的区别

    `git revert`是用一次新的commit来回滚之前的commit，`git reset`是直接删除指定的commit

    `git revert`是用一次逆向的commit“中和”之前的提交，因此日后合并老的branch时，导致这部分改变不会再次出现，但是

    `git reset`是之间把某些commit在某个branch上删除，因而和老的branch再次merge时,这些被回滚的commit应该还会被引入

#### 例子

0.  注册代码仓库用户

~~~bash
# 目前一般是gitlab
~~~

1.  拉取代码

~~~bash
git clone xx.git
~~~

2.  查看当前分支

~~~bash
git branch -a  # 查看分支, 默认位于master分支
git remote -v  # 查看远程分支情况
~~~

3.  创建本地开发分支

~~~bash
git checkout dev
git checkout -b dev # 创建并且切换
git push origin dev # 创建远程分支
~~~

4.  开发

~~~bash
git add xx.go
git status # 查看状态
git commit -m "添加xx功能，完善测试" # 可能会多次commit
~~~

5.  合并推送

~~~bash
git checkout master
git pull origin master  # 一般先拉去远程代码，有可能远程代码版本比当前新
git checkout master     # 切换回master
git merge dev           # 合并代码，可能有冲突
git push origin master
~~~

6.  删除

~~~bash
git branch -D dev     // 删除本地
git push origin :dev  // 删除远程
~~~

7.  版本回退

~~~bash
git reset --hard head^
git log
git relog
git reset -- hard 44c2ec
~~~

8.  注意

>   上面情况只是基本流程，实际上master分支是不能随意合并的
>
>   通常master是最稳定分支，所有开发全部在远程dev分支上面，本地也是需要新建local-dev分支

~~~bash
git push origin dev:dev   # 推送本地分支到远程
~~~

9.  优化

>   合并代码的时候，本地可能进行多次commit，可以优化

~~~bash
git checkout master
git pull origin master
git checkout dev     
git rebase -i  HEAD~2 # 将2个提交合并为一个
git rebase master     # 将master最新的分支同步到本地,需要解决冲突
git checkout master
git merge dev
git push origin master
~~~

10.  总结

>   1.  git rebase操作实际上是将当前执行rebase分支的所有基于原分支提交点之后的commit打散成一个一个的patch，并重新生成一个新的commit hash值，再次基于原分支目前最新的commit点上进行提交，并不根据两个分支上实际的每次提交的时间点排序，rebase完成后，切到基分支进行合并另一个分支时也不会生成一个新的commit点，可以保持整个分支树的完美线性
>   2.  当开发一个功能时，可能会在本地有无数次commit，而实际上在你的master分支上只想显示每一个功能测试完成后的一次完整提交记录就好了，其他的提交记录并不想将来全部保留在master分支上，rebase可以在rebase时将本地多次的commit合并成一个commit，还可以修改commit的描述

#### 问题

1.  关于Git用户配置

~~~go
git config --local user.name "li yao"
git config --local user.email"liyaoo1995@gmail.com"
git config --global user.name "li yao"
git config --global user.email "liyaoo1995@gmail.com"
~~~

2.  关于Push需要输入用户名和密码问题

>   由于的HTTPS的方式，所以每次需要输入账户密码，可以该用SSH形式

~~~bash
git remote -v
git remote rm origin
git remote add orgin git@xx.git
git push -u orgin master
~~~

3.  上传错文件

~~~bash
git rm -r --cached target # 直接删除
~~~

#### 分支说明

1. master || main 分支：存储正式发布的产品，`master || main` 分支上的产品要求随时处于可部署状态。`master || main` 分支只能通过与其他分支合并来更新内容，禁止直接在 `master || main` 分支进行修改。
2. develop 分支：汇总开发者完成的工作成果，`develop` 分支上的产品可以是缺失功能模块的半成品，但是已有的功能模块不能是半成品。`develop` 分支只能通过与其他分支合并来更新内容，禁止直接在 `develop` 分支进行修改。
3. feature 分支：当要开发新功能时，从 master 分支创建一个新的 `feature` 分支，并在 `feature` 分支上进行开发。开发完成后，需要将该 `feature` 分支合并到 `develop` 分支，最后删除该 `feature` 分支。
4. release 分支：当 `develop` 分支上的项目准备发布时，从 `develop` 分支上创建一个新的 `release` 分支，新建的 `release` 分支只能进行质量测试、bug 修复、文档生成等面向发布的任务，不能再添加功能。这一系列发布任务完成后，需要将 `release` 分支合并到 `master` 分支上，并根据版本号为 `master` 分支添加 `tag`，然后将 `release` 分支创建以来的修改合并回 `develop` 分支，最后删除 `release` 分支。
5. hotfix 分支：当 `master` 分支中的产品出现需要立即修复的 bug 时，从 `master` 分支上创建一个新的 `hotfix` 分支，并在 `hotfix` 分支上进行 BUG 修复。修复完成后，需要将 `hotfix` 分支合并到 `master` 分支和 `develop` 分支，并为 `master` 分支添加新的版本号 `tag`，最后删除 `hotfix` 分支。

#### Commit Message

> 提交信息规范
>
> 提交信息应该描述“做了什么”和“这么做的原因”，必要时还可以加上“造成的影响”，
>
> Header Header 部分只有 1 行，格式为`<type>(<scope>): <subject>`

type 用于说明提交的类型，共有 8 个候选值

1. feat：新功能（ feature ）
2. fix：问题修复
3. docs：文档
4. style：调整格式（不影响代码运行）
5. refactor：重构
6. test：增加测试
7. chore：构建过程或辅助工具的变动
8. revert：撤销以前的提交
9. scope 用于说明提交的影响范围，内容根据具体项目而定。

subject 用于概括提交内容。

### 其他

提交敏感信息，强制回退服务器版本, 然后再次提交即可

~~~
 git reset --soft HEAD~i
 git push origin master --force
~~~

