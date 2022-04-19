[TOC]

### Git协作规范

#### 分支

|  分支   |     名称     | 描述                             | 环境可访问 | 权限 |
| :-----: | :----------: | -------------------------------- | :--------: | :--: |
| master  |    主分支    | 最稳定的发布分支，经过严格的测试 |     是     |  高  |
| release |  预上线分支  | 预发布、仿真环境测试             |     是     |  高  |
| hotfix  | 紧急修复分支 |                                  |     否     |      |
| develop |   测试分支   | 次稳定的开发分支, 线上测试       |     是     |      |
| feature | 需求开发分支 |                                  |     否     |      |

#### 分支说明

##### master

1. 存储正式发布的系统，`master` 分支要求随时处于可部署状态 
2. **`master ` 分支只能通过与其他分支合并来更新内容，禁止直接在 `master` 分支进行修改**

##### develop

1. 汇总开发者完成的子任务，该分支上的产品可以是缺失功能模块的半成品，但是已有的功能模块不能是半成品
2. `develop`该分支只能通过与其他分支合并来更新内容，禁止直接在 `develop` 分支进行修改

##### feature 

1. 当要开发新功能时，从 master 分支创建一个新的 `feature` 分支，并在 `feature` 分支上进行开发
2. 开发完成后，需要将该 `feature` 分支合并到 `develop` 分支，最后删除该 `feature` 分支

##### release 

1. 当 `develop` 分支上的项目准备发布时，从 `develop` 分支上创建一个新的 `release` 分支，新建的 `release` 分支只能进行质量测试、bug 修复、文档生成等面向发布的任务，不能再添加功能
2. 这一系列发布任务完成后，需要将 `release` 分支合并到 `master` 分支上，并根据版本号为 `master` 分支添加 `tag`，然后将 `release` 分支创建以来的修改合并回 `develop` 分支，最后删除 `release` 分支(保留现场亦可不删)

##### hotfix(bug) 

1. 当 `master` 分支中的产品出现需要立即修复的 bug 时，从 `master` 分支上创建一个新的 `hotfix` 分支，并在 `hotfix` 分支上进行 BUG 修复

2. 修复完成后，需要将 `hotfix` 分支合并到 `master` 分支和 `develop` 分支，并为 `master` 分支添加新的版本号 `tag`，最后删除 `hotfix` 分支

##### issue

1. 某些非紧急问题

#### Commit Message

> 提交信息应该描述「做了什么」和「这么做的原因」，必要时还可以加上「所属模块」「造成的影响」「关联影响」
>

**日志**

1.  日志信息重要
2.  格式参考

~~~
<type> (scope)：<subject>
~~~

|  格式   |                   意义                    |
| :-----: | :---------------------------------------: |
|  type   |      表示 动作类型，可分为如下的动作      |
|  scope  | 表示 影响范围，可分为：模块、类库、方法等 |
| subject |    表示 简短描述，最好不要超过 60 个字    |

type 用于说明提交的类型，共有 8 个候选值

|   type   |            说明            |
| :------: | :------------------------: |
|   feat   |    新功能（ feature ）     |
|   fix    |          问题修复          |
|   docs   |            文档            |
|  style   | 调整格式（不影响代码运行） |
| refactor |            重构            |
|   test   |          增加测试          |
|  chore   |  构建过程或辅助工具的变动  |
|  revert  |       撤销以前的提交       |

#### 提交代码

**前提**

1.  不要提交非代码以及配置外的文件
2.  不要提交未编译成功的代码
3.  提交之前需要认真检查整个流程无错误
4.  提交之前需要更新

#### 开发流程

##### 个人配置

- [x] gitlab (或其他服务器) Account
- [x] SSH 本地生成以及gitlab配置

##### 正常开发

1. 获取源代码

~~~bash
$ git clone example.git
~~~

2. 配置提交者信息

~~~bash
$ git config --local user.name "GodYao"
$ git config --local user.email "GodYao@gmail.com"
~~~

3. 查看分支情况

~~~bash
# 查看本地分支
$ git branch	

# 查看远程分支
$ git branch -a 			
~~~

2.  新建本地分支(检测并且追踪远程分支)，基于某个分支创建, 见上文各个分支作用

~~~bash
# 远程已经存在dev，就会新建本地dev，并且追踪远程dev，通常采用此方法
$ git branch feature-xx

# 新建并且切换
$ git checkout -b feature-xx 	
~~~

3.  切换分支

~~~bash
$ git checkout dev 
~~~

4.  合并分支

~~~bash
# 通常情况下，是没有权限合并master分支的，只能进行CR后合并dev分支
$ git checkout dev	

# 合并feature to dev本地分支
$ git merge feature-xx		
~~~

6.  删除分支

~~~bash
# 删除本地分支
$ git branch -d feature-xx	

# 强行删除没有合并的分支
$ git branch -D feature-xx	 	

# 删除远程分支
$ git push origin --delete serverfix  
~~~

7. 推送到远程分支

~~~bash
$ git push origin <本地分支>:<远程分支>
~~~

8. 查看分支

~~~bash
$ git log
$ git log --graph
$ git log --graph --pretty=oneline --abbrev-commit
~~~

9. 更新


~~~bash
# 默认拉去当前分支，并且合并
$ git pull 
~~~

10. 临时任务

~~~bash
# 当正在编写代码时，突然需要解决一个BUG，此时又不想提交代码，那么可以将工作区暂存
$ git stash
$ git checkout -b hotgix

# 修复完成bug，然后提交，切换到master然后合并到bug，然后需要回到暂存的工作区
$ git stash list

# 恢复 方式一
$ git stash apply  
# stash内容并不删除 需要手动删除
$ git stash drop	 

# 方式二
$ git stash pop
~~~

##### 解决冲突

> 在merge合并的时候，可能会出现修改同一个地方导致冲突

1. 查看冲突


~~~bash
$ git status  
~~~

2. 解决: 可以将提示的部分删除，然后再次提交

##### 常见问题

**提交暂存区出错**

1.  改乱了工作区某个文件的内容，想直接丢弃工作区的修改时「危险操作，丢失修改」

~~~bash
$ git checkout -- [file]	
~~~

2.  改乱了工作区某个文件的内容，还添加到了暂存区时，回到工作区

~~~bash
$ git reset HEAD file
~~~

3.  已经提交了不合适的修改到版本库时，想要撤销本次提交

~~~bash
$ git reset –hard "版本号"
~~~

**提交本地仓库出错**

1.  提交信息出错

~~~bash
$ git commit --amend -m "新提交的信息"
~~~

2.  漏提交

~~~bash
# missed-file 为遗漏提交文件
$ git add missed-file 	
# 表示提交消息不会更改，在 git 上仅为一次提交
$ git commit --amend --no-edit  
~~~

3. 版本回退

~~~bash
# 修改版本库，保留暂存区，保留工作区
$ git reset --soft HEAD~1

# 修改版本库，修改暂存区，修改工作区
$ git reset --hard HEAD~1

# 回退到特定的版本，可以通过 git log查看提交历史，以便确定要回退到哪个版本
$ git reset --hard commit_id 
~~~

4. 不覆盖回滚

~~~bash
# git revert 是用一次新的commit来回滚之前的commit，git reset 是直接删除指定的commit
# 撤销前一次 commit
$ git revert HEAD	 	

# 撤销前前一次 commit
$ git revert HEAD^ 

# 撤销制定版本
$ git revert commit_id	
~~~
