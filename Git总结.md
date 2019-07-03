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

#设置名字和邮箱，每次提交时都会跟踪名字和邮箱
git config --global user.name "linkk"; 
git config --global user.email "linkeke_sea@163.com";

git status;  #查看当前仓库状态
git diff readme.txt;  #查看文件的不同之处
git rm --cached Git总结.md;  #将Git总结.md文件暂存区删除掉
git
```



#### 



