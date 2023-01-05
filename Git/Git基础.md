## 更改 git 远程拉取时的网址

```
git remote set-url origin [url]
```



## 合并两个不相关的分支

```
git merge test2 --allow-unrelated-histories
```



## 将本地代码push到github

```
a. git init 在本地创建一个Git仓库；

b. git add . 将项目添加到暂存区；

c. git commit -m "注释内容" 将项目提交到Git仓库；

远程仓库：
a. 添加SSH KEY；

b. 新建repositories；

本地仓库：
a. git remote add origin git@github.com:UserName/projectName.git 将本地仓库与远程仓库关联；

b. git push -u origin master 将本地项目推送到远程仓库。
```

