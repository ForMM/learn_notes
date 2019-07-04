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





```







