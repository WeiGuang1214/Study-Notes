### Git概述：

​	1、使用场景：备份、代码还原、协同开发、追溯问题代码+责任人

​	2、完全分布式，分布式版本控制，pull下拉，push上传

### 常用命令：

​	1、clone克隆：从远程仓库中克隆代码到本地

​	2、checkout检出：从本地仓库检出一个仓库分支然后修订

​	3、add添加：在提交前把代码保存到暂存区

​	4、commit提交：提交到本地仓库，本地仓库保存修改的各个历史版本

​	5、fetch抓取：从远程库抓取到本地仓库，不做任何合并动作，一般操作较少

​	6、pull拉取：从远程库拉到本地库，自动进行合并merge，然后放到工作区，相当于fetch+merge

​	7、push推送：修改完成后，需要和团队成员共享代码，将代码推送到远程仓库

### 基本配置

##### 设置用户信息,‌**在Git中，用户名和邮箱的主要作用是记录每次提交的作者信息**‌。

​	git config --global user.name"itcast"

​	git config --global user.email"@.com"

##### 指令可以设置别名（用来简写）：

​	touch ~/.bashrc   创建文件

​	在bashrc文件中写入：

​	#用于输出git提交日志

​	alias git-log='git log --pretty=oneline --all --graph --abbrev-commit'

​	#输出当前目录所有文件及基本信息

​	alias ll='ls -al'

##### 解决中文乱码问题

​	1、git config --global core.quotepath false

​	2、${git_home}/etc/bash.bashrc下面加两行

​	export LANG="zh_CN.UTF-8"

​	export LC_ALL="zh_CN.UTF-8"

##### 获取本地仓库

​	创建目录之后进入目录，Git bash，执行git init，创建本地仓库

##### 在工作目录对于代码的修改：

​	除了.git都是工作目录

​	新创建文件是未跟踪的untracked，使用git add 提交到暂存区，git commit提交到仓库

​	git add （工作区->暂存区）

​	git commit (暂存区->本地仓库)

#####  git status      查看状态  

##### git add .       通配符，添加所有目录至暂存区

##### git log          查看提交日志

​	options：--all  显示所有分支

​			--pretty=oneline  将信息显示为一行

​			--abbrev-commit  使得输出的commit id 更简短

​			--graph 以图表的形式显示

##### git commit -m  “add file”     添加注释提交到本地仓库

#####  版本回退   git reset --hard commitID   commitid可以通过git-log 或者git log 指令查看

##### git reflog  可以查看已经删除的提交记录

##### touch. gitignore 不被git管理

##### vi .gitignore 内写指定不被git管理的文件

##### gitignore 指定不被git管理

### 分支，避免影响主线开发，开辟工作代码线

##### 	git branch 查看本地分支

##### 	git branch dev01 创建新分支，命名

##### 	git-log可以查看分支

##### 	git checkout 分支名，可以切换分支

##### 	git checkout -b 分支名，懒得创建，直接创建并且切换

​	head指向谁谁是当前分支

##### merge合并，其它分支合并到master

​	先切换到master，然后再git merge dev01

##### 删除分支，只能删其它，不能删当前

​	git branch -d b1 做检查删除

​	git branch -D b1 不做检查直接删

##### 解决冲突

​	两个分支代码版本冲突，

​	master（生产分支）

​	develop（开发分支）

​	feature（同期并发分支）

​	hotfix（线上bug修复分支）

### Github远程仓库

​	创建仓库、配置SSH公钥、添加远程仓库

##### 	1. 生成SSH密钥

​	ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

- `-t rsa` 指定密钥类型为RSA。
- `-b 4096` 指定密钥的位数为4096位，这是一个相对安全的位数。
- `-C` 后面是你的电子邮件，用于标识密钥。

​	提示你输入一个文件名来保存密钥（通常使用默认文件名`id_rsa`即可），然后再次输入一个密码（可选，如果你希望每次使用密钥时都需要输入密码的话）

##### 	2. 添加SSH密钥到ssh-agent

​	eval "$(ssh-agent -s)"

​	ssh-add ~/.ssh/id_rsa

##### 	3、将公钥添加到远程仓库

​	cat ~/.ssh/id_rsa.pub

##### 	4. 配置Git使用SSH连接远程仓库

​	git config --global url."git@github.com:".insteadOf "https://github.com/"

##### 	5. 克隆或推送仓库

​	git clone git@github.com:username/repository.git

​	或者，如果你已经有一个通过HTTPS克隆的仓库，你可以更改其远程URL为SSH：

​	git remote set-url origin git@github.com:username/repository.git

​	添加远程仓库：

​	git remote add 别名 地址

​	git remote 查看仓库列表

##### 	推送

​	git push 别名 分支

​	git push 别名 分支：分支，把本地分支推送到远程分支

​	如果分分支同名直接git push 别名 分支

​		-f  强制覆盖

​		--set-upstream 推送到远端的同时建立远端分支的关联关系

​		如果当前分支和远端分支建立关联，则可以省略分支名和远端名

​		git push

​		git branch -vv  查看本地和远端分支关联关系

##### 克隆

​	git clone 仓库地址 本地目录

抓取fetch

​	git fetch 

​	可以抓取远端分支最新的修改，再merge到本地

​	git merge origin/master

拉取pull(一键fetch+merge)

git pull remote-name branch-name

##### 解决合并冲突

​	先pull，合并到本地最新更改，解决冲突，再push到云

#### IDEA结合GIT使用

​	基于提交点创建分支，new branch

​	
