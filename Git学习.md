## Git概述：

​	1、使用场景：备份、代码还原、协同开发、追溯问题代码+责任人

​	2、完全分布式，分布式版本控制，pull下拉，push上传

#### 常用命令：

​	1、clone克隆：从远程仓库中克隆代码到本地

​	2、checkout检出：从本地仓库检出一个仓库分支然后修订

​	3、add添加：在提交前把代码保存到暂存区

​	4、commit提交：提交到本地仓库，本地仓库保存修改的各个历史版本

​	5、fetch抓取：从远程库抓取到本地仓库，不做任何合并动作，一般操作较少

​	6、pull拉取：从远程库拉到本地库，自动进行合并merge，然后放到工作区，相当于fetch+merge

​	7、push推送：修改完成后，需要和团队成员共享代码，将代码推送到远程仓库

#### 基本配置

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

####  版本回退   git reset --hard commitID   commitid可以通过git-log 或者git log 指令查看

#### git reflog  可以查看已经删除的提交记录