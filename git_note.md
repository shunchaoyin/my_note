# git的三种状态
工作区、暂存区（add）、git仓库（commit）
git init
git add .
git commit -m "meassage"
git commit -am " meassage"
git status 
git log
git diff HEAD -- ReadMe.md

## 移除暂存区文件
git restore --staged ReadMe.md
或者 git reset HEAD ReadMe.md (取消上一次操作)

## 版本回退
git log --pretty=oneline
git reset --hard HEAD^ (回退一个版本)
git reset --hard HEAD^^ (回退两个版本)
git reset --hard HEAD~n . (回退到n个版本)
git reset --hard commitID (回退到指定的commit ID)
git reflog (包含reset的log信息)

## 文件删除
git ls-files
1. 在工作区删除文件，然后git add
2. git rm ReadMe.md

## ssh 绑定github需要时候，查google

## 分支操作
开发时应该保持主干可以随时发布，branch分支，进行功能开发，功能验证pass后才可以和入主干。

git branch    //本地分支
git branch -a //查看所有分支，包括远端
git checkout branch   //切换到指定分支
git checkout -b new_branch  //创建新分支，并且切换
git branch -d branchname    //删除分支
git merge branch            //合并分支，不要merge主干，要在主干上去merge其他分支 
git branch -m oldbranch newbranch   //更改分支名称，如果分支名已经存在要使用M

## 远端分支操作
git push origin branch_name   //推送本地分支到远端
git push origin :remote_branch_name //删除远端分支
git checkout -b local_branch_name  remote_branch_name  //切换到远端分支，并建立本地分支

## 冲突合并
流程：切换到主分支，合并开发分支，如果有冲突，修改即可。
git checkout master 
git merge dev
//修改文件，解决冲突 
git pull       //每次push前最好执行git pull
git commit -m "message"
git push 
git log --graph --pretty=oneline

## 标签管理
git tag tag_name      //新建标签，默认为HEAD
git tag tag_name commitId    //指定提交位置打tag
git tag -a tag_name -m "message"  //新建标签，并且添加标签描述信息
git tag              //查看所有标签
git tag -d tag_name  //删除标签
git push origin tage_name  //推送远端tag
git push origin --tags     //推送所有tags
git push origin :refs/tags/tage_name  //删除远端tag

