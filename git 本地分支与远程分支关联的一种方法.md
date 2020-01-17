git 本地分支与远程分支关联的一种方法


ithub上已经有master分支 和dev分支

在本地

git checkout -b dev 新建并切换到本地dev分支

git pull origin dev 本地分支与远程分支相关联

在本地新建分支并推送到远程

git checkout -b test

git push origin test   这样远程仓库中也就创建了一个test分支

 ================================================================================================================


1. 克隆代码
git clone https://github.com/master-dev.git  
# 这个git路径是无效的，示例而已

2. 查看所有分支
git branch --all  
# 默认只有master分支，所以会看到如下两个分支
# master[本地主分支] origin/master[远程主分支]
# 新克隆下来的代码默认master和origin/master是关联的，也就是他们的代码保持同步

3. 创建本地新的dev分支
git branch dev  # 创建本地分支
git branch  # 查看分支
# 这是会看到master和dev，而且master上会有一个星号
# 这个时候dev是一个本地分支，远程仓库不知道它的存在
# 本地分支可以不同步到远程仓库，我们可以在dev开发，然后merge到master，使用master同步代码，当然也可以同步

4. 发布dev分支
发布dev分支指的是同步dev分支的代码到远程服务器
git push origin dev:dev  # 这样远程仓库也有一个dev分支了

5. 在dev分支开发代码
git checkout dev  # 切换到dev分支进行开发
# 开发代码之后，我们有两个选择
# 第一个：如果功能开发完成了，可以合并主分支
git checkout master  # 切换到主分支
git merge dev  # 把dev分支的更改和master合并
git push  # 提交主分支代码远程
git checkout dev  # 切换到dev远程分支
git push  # 提交dev分支到远程
# 第二个：如果功能没有完成，可以直接推送
git push  # 提交到dev远程分支
# 注意：在分支切换之前最好先commit全部的改变，除非你真的知道自己在做什么

6. 删除分支
git push origin :dev  # 删除远程dev分支，危险命令哦
# 下面两条是删除本地分支
git checkout master  # 切换到master分支
git branch -d dev  # 删除本地dev分支

7.git查看本地分支关联（跟踪）的远程分支之间的对应关系
git branch -vv

8.建立当前分支与远程分支的映射关系
git branch -u origin/addFile
或者git branch --set-upstream-to origin/addFile

9.撤销本地分支与远程分支的映射关系
git branch --unset-upstream

10.将远程git仓库里的指定分支拉取到本地（本地不存在的分支)
git checkout -b 本地分支名 origin/远程分支名

 git@gitlab.idc.safecenter.cn:A-EMM/A-EMM.git


 ###########################################

 同一个git库在不同分支放不同的项目

 新建项目执行 git init
 新增项目文件并提交 git add .  / git commit -m 'add file'
 此时默认master分支，新建项目分支；
 git branch newProject
 切换到新建项目分支上
 git checkout newProject
 关联本地仓库 到远程仓库
 git remote add origin  git@gitlab.idc.safecenter.cn:A-EMM/A-EMM.git
 提交新建分支到远程仓库
 git push origin newProject
