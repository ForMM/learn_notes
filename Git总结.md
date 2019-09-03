# Git总结

#### Git理解的三个区：工作区、暂存区、版本库。

1. 工作区就是要提交的文件目录，

2. 暂存区是Git仓库里的一个概念区，git  add操作就是把文件同步到这个区。

3. 版本库是Git仓库里的本地仓库。



##### Git相关命令：

```shell
git init;  # 新建一个仓库
git add Git总结.md;  #添加Git总结.md文件到暂存区
git commit -m “first time”;  #将暂存区文件提交到本地仓库
#注意：提交到本地仓库整体流程，git add-->git commit两个命令都要执行

#设置名字和邮箱，每次提交时都会跟踪名字和邮箱
git config --global user.name "linkk"; 
git config --global user.email "linkeke_sea@163.com";
#设置每个库单独用户名和邮箱
git config user.name linkeke_sea@163.com;
git config user.email linkeke_sea@163.com;

git status;  #查看当前仓库状态
git diff readme.txt;  #查看文件的不同之处
git rm --cached Git总结.md;  #将Git总结.md文件暂存区删除掉

#版本回退
git log;  #提交日志记录
git reset --hard HEAD^;  #回退到上一个版本
git reflog;  #查看命令历史记录
git reset --hard 1092a; #1092a代表commit id

#撤销修改
git checkout -- Git总结.md;  #撤销工作区的修改，有两种情况：1.修改后没有提交暂存区，撤销恢复到版本库一致；2修改后提交了暂存区，撤销恢复到暂存区时的内容。

#删除文件
rm Git总结.md;  #工作去删除文件
git rm Git总结.md;  #从版本库中删除文件，若删除错了，用checkout恢复

#git和github之间通过ssh加密传输，需要在本地生成ssh key放到github上
#添加远程仓库
ssh-keygen -t rsa -C "linkeke_sea@163.com";  #本地生成ssh私钥、公钥，将公钥放到github上
git remote add origin git@github.com:ForMM/learn_notes.git;  #关联远程仓库
git push -u origin master;  #推送到远程仓库
git pull upstream dev; #upstream远程仓库对应的名字

#远程库克隆项目
git clone git@github.com:ForMM/HelloBoy.git;  #克隆远程项目到本地

#创建与合并分支
git branch;  #查看分支
git checkout master;  #切换分支到master
git checkout -b dev;  #创建分支并切换分支到dev,然后修改文件提交,需要切换到master
git checkout master;  #切换分支到master
git merge dev;  #解决冲突，快速合并到master
git branch -d dev;  #删除分支dev
git update-index --assume-unchanged src/main/resources/application.properties; #忽略某个文件修改
git update-index --no-assume-unchanged src/main/resources/application.properties; #解除忽略某个文件修改

#解决冲突
git log --graph --pretty=oneline --abbrev-commit;  #可以查看合并信息
git merge --no-ff -m "merge with no-ff" dev;  #普通模式合并，到时查看日志可以看得出具体分支合并

#bug分支管理（当在dev分支上开发一个版本，突然来了紧急bug，需要立刻处理）
git stash;  #保存工作目录
git checkout master;
git checkout -b bug-1;
git add Git总结.md;
git commit -m "bug-1";
git checkout master;
git merge --no-ff -m "merged bug fix 1" bug-1;
git checkout dev;
git stash pop;  #恢复dev的工作区，继续在之前保存的基础上开发
#当开发新功能，同样创建新分支，开发完成后，新功能不需要了，需要删除分支
git branch -D feture-1;  #强制删除

#多人协作
git remote -v;  #查看远程分支信息
git pull;  #把远程分支内容拿下来（当拿下来后有冲突，需要解决冲突）

#rebase让提交日志变成一条直线
git rebase;

#标签管理（标签对应commit）
git tag v1.0;  #创建标签
git tag;  #查看标签
git log --pretty=oneline --abbrev-commit;  #一条直线显示提交日志
git tag -d v1.0;  #删除标签
git push origin v1.0;  #推送到远程库







```







 