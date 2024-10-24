### GitHub搜索和Git常用命令

#### GitHub搜索

1. 基础搜索
   - in:name xxx // 按照项目名搜索
     in:readme xxx // 按照README搜索
     in:description xxx // 按照description搜索

2. 添加条件
   1. stars:>xxx // stars数大于xxx
      forks:>3000 // forks数大于xxx
      language:xxx // 编程语言是xxx
      pushed:>YYYY-MM-DD // 最后更新时间大于YYYY-MM-DD

#### Git常用命令

1. git add /remove/pull/add/commit
2. git branch -a  查看所有的分支.

#### 远程同步

```bash
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]
 例如: git pull origin main

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
```

#### 项目初始化

1. 第一次提交

~~~bash
echo "# springboot2learn" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:a908236075/springboot2learn.git
git push -u origin main
~~~

2. 将已存在的git的project提交

   1. ~~~bash
      git remote add origin git@github.com:a908236075/springboot2learn.git
      git branch -M main
      git push -u origin main
      ~~~

#### 分支操作

~~~shell
## 查看本地分支
git branch
## 查看远程分支
git branch -r
## 创建分支
git branch branchname
## 切换分支
git checkout branchname
## 合并分支
git merge branchname
## 分支删除
git branch -D branchname
~~~



#### 修改分支名称

**方法1:使用git命令操作修改本地分支名称**

~~~shell
git branch -m oldBranchName newBranchName
~~~

**方法2:使用git命令操作修改远程分支名称**

将本地分支的远程分支删除

~~~shell
git push origin :oldBranchName
~~~

将改名后的本地分支推送到远程，并将本地分支与之关联

~~~shell
git push origin :oldBranchName
git push --set-upstream origin newBranchName
~~~

