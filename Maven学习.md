### maven项目管理工具，把项目开发和管理过程抽象成一个项目对象模型POM

依赖管理本地仓库-->私服-->中央仓库；

集成jar、源代码、文档、xml，

1、项目构建：提供标准的、跨平台的自动化项目构建方式

2、依赖管理：管理项目依赖的资源jar包，避免资源间的版本冲突问题

3、统一开发结构：提供标准的、统一的项目结构

#### 建议自己安装maven，因为IDEA自带的maven版本较低，高版本可以向下兼容

#### maven基本概念：

#### 1、仓库

​	用于存储资源，jar包等等；本地是中央仓库的子集，私服具有版权保护，仅对内部使用

#### 2、坐标

​	资源在仓库中的位置，类似URL

​	https://repo1.maven.org/maven2/

​	Maven坐标组成：

​	groupId：所属组织，通常是域名反写，org.mybatis

​	artifactId：当前maven项目名称，通常是模块名称，CRM，SMS

​	versin：当前项目版本号

### 配置本地仓库（jar包下载位置）

### 配置阿里镜像仓库（下载来源）

在mvnrepository.com中找

#### 配置ID、仓库类型、镜像名称、URL：http://maven.aliyun.com/nexus/context/groups

project下

--src

​	--main

​		--java源程序

​		--resources配置文件

​	--test测试文件

​		--java源程序

​		--resources配置文件

src同层要写pom.xml配置，可以在maven下soft，jar包打开，找pom文件复用，写项目信息和依赖包

#### Maven项目命令（主要记idea的使用）了解流程即可

```cmd
mvn compile //下载插件，编译
mvn clean //清理编译完成的东西
mvn test  //下载测试插件
mvn package //打包
mvn install //安装到本地
```

### 原型创建工程(IDEA生成)

##### 1、直接利用idea自带的maven骨架，导入对应的插件即可

idea使用支持的版本才能做

##### java的maven配置

##### web的maven配置，

##### 学会导包，导依赖，写xml，里面修改打包方式、项目id、版本号、当前工程依赖、具体的插件

### 依赖管理

### 生命周期与插件
