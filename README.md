---
title: 简洁优雅的git工作流
date: 2016-04-28 10:06:45
tags: git-work-flow legit
---

# git相关概念
## 1. Git是什么
GIT是一个版本控制系统,
版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。<br/>
版本控制系统分为3类:

+ 本地版本控制系统
+ 集中化版本控制系统
+ 分布式版本控制系统

git属于分布式版本控制系统, 相比于本地版本控制, git可以实现多人协作;
相比于集中化的版本控制系统, git每一台工作机上都有一个完整的git仓库,
不会因为主仓库duang掉就导致所有人无法开发提交, 更加安全。

## 2. Git模型
Git中有三个重要的"分区": **工作区**, **暂存区**, **历史记录区** <br/>
当我们<code>git init</code>初始化一个仓库时, 会创建一个**.git**目录,
Git仓库的特性就是因为.git目录而存在。<br/>
![.git](http://7xj431.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-28%20%E4%B8%8A%E5%8D%8810.40.01.png)

**工作区** <br/>
Git仓库所在的目录, 程序员进行开发改动的地方.

**暂存区** <br/>
.git目录下的index文件, 暂存区会记录<code>git
add</code>添加文件的相关信息(文件名、大小、timestamp...),不保存文件实体(文件实体保存在object对象区),
通过id指向每个文件实体。<code>git commit</code>后同步index的目录树到object. <br/>
可以使用<code>git status</code>查看暂存区的状态(git status具体见文末补充1)。

**历史记录区** <br/>
保存commit记录

## 3. Git基础命令
### <code>git init</code>
初始化一个git仓库, 会创建**.git**目录(版本库)
### <code>.gitignore文件</code>
.gitignore中存在的后缀名文件会在提交时被忽视, 后面涉及到具体技术的工作流时会利用好这个.gitignore文件。
### <code>git add</code>
<code>git add files</code>将files添加到暂存区, <code>git add
.</code>添加所有(除被.gitignore忽略)的文件到暂存区(index).<br/>
![index](http://7xj431.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-28%20%E4%B8%8A%E5%8D%8811.02.03.png) <br/>
可以看到对象区(object)中已经存在文件实体和id.(更多关于对象区的概念见文末补充2)
### <code>git status</code>
<code>git status</code>可以查看当前暂存区的文件状态<br/>
![status](http://7xj431.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-28%20%E4%B8%8A%E5%8D%8811.05.11.png) <br/>
### <code>git撤销</code>
有两个git撤销操作

+ <code>git reset</code>
+ <code>git revert</code>

#### git reset
<code>git reset [commit head]</code>用于将版本库回滚到某次commit时的状态, 有两个参数选项<code>--hard/soft</code>,
默认是soft。这两个选项的区别在于--hard会强制删除这次commit,
而--soft则是先将改动保存在暂存区等待稍后checkout到工作区. <br/>
注意, git reset撤销操作后, 撤销的这次commit之后的所有commit都将回退到暂存区。

#### git revert
<code>git revert</code>用于撤销某次操作, 用一次新的commit去进行撤销, 且直接撤销,
不会回退到暂存区。
<hr/>
那么使用哪个撤销呢? 一个总的原则就是,
如果你的提交已经被别人通过版本库clone(即存在于别人的提交记录中),
那么使用git revert而不用git reset, 因为git reset会毁坏你的提交历史,
导致冲突的发生, git revert用新的commit去执行撤销是一种比较简洁的方式。

### 仓库相关

+ <code>git remote add origin git_url</code> 添加远程仓库地址(url), 并用origin指代
+ <code>git remote rm origin</code> 删除origin对应的远程仓库

### 分支相关

+ <code>git branch</code> 显示所有分支
+ <code>git checkout -b branch</code> 创建一个新分支branch
+ <code>git checkout branch</code> 进入branch分支
+ <code>git branch -d branch</code> 删除branch分支

### 合并相关
<code>git merge</code> 和 <code>git rebase</code><br/>
merge和rebase(变基)都可以用于分支的合并。merge会把两个分支的最新快照以及二者最近的共同
祖先进行三方合并，合并的结果是生成一个新的快照（并提交）。而rebase命令则将提交到某一
分支上的所有修改都移至另一分支上。变基以后的提交树只有一条主干。<br/>
<code>git merge</code><br/>
![merge](https://git-scm.com/book/en/v2/book/03-git-branching/images/basic-rebase-2.png) <br/>
<code>git rebase</code><br/>
![rebase](https://git-scm.com/book/en/v2/book/03-git-branching/images/basic-rebase-3.png) <br/>

# 我总结的git工作流
git工作流其实就是一套使用git进行版本控制的流程规范, 有利于团队的项目协作。<br/>
结合木犀的产品开发, 总结的git工作流(场景如下)

    1. 创建远程git仓库(github服务)
    2. 确定主分支(一般为master分支)
        : 主分支用于发布**正式版本**(可以打上tag)
        : 主分支确定后一定不要更换
    3. 确定开发分支(develop)
        : 用于功能开发
        : merge某个开发者分支的功能实现
        : 完成特定阶段功能后合并到主分支
        : 发布隔夜(Night)版本
    4. 开发者fork+clone主仓库
        : clone仓库后checkout新分支
        : 新分支的名字清楚明了(比如: 修bug分支叫bug, 个人开发分支叫develop)
        : 合并到develop分支后新分支就可以删除
    5. hotfix分支
        : 修复master正式版本出现的bug
        : 修bug的开发者checkout hotfix分支
        : bug修复后合并到develop和master两个分支
    6. 其他
        : 预发布分支: 进行AB test

**Git工作时的注意事项** <br/>

1. 删除操作时尽量使用<code>git revert</code>
2. <code>rebase虽好, 可不要贪杯哦</code>, 绝大多数情况下merge就足够了
3. 频繁提交, 这样不至于一次提交时和其他开发者的版本库相差太大
4. 项目的管理者在进行开发时也要pull request! pull request的代码才可以保证质量(代码测试、code review)

**最后但仍然重要的是** <br/>
CI(持续集成), 这个现在还在摸索, 但是很重要,
更多关于CI可以看[一峰大师的博客](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)

# 基于legit实现简洁优雅的git工作流
[legit](https://github.com/kennethreitz/legit) 是一个git work flow工具,
封装了git的命令,
用于快速实现git工作流操作(比如一个命令将功能分支合并到开发分支)。官方称"for human",
作者是Python界大名鼎鼎的Kennethreitz(requests作者, heroku创始人)。

# 总结
团队一直使用git作为版本管理工具, 但是没有一个项目是很好的遵守Git工作流的, 学而先前主要
是两个人开发, 还好。冲突在官网这边体现的比较多, 可以看一下官网的分支和commit: <br/>
![分支](http://7xj431.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-30%20%E4%B8%8A%E5%8D%8812.09.39.png)<br/>
![commit](http://7xj431.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-30%20%E4%B8%8A%E5%8D%8812.10.07.png)<br/>
官网现在是git单分支。目前聪聪和斯蒂芬两个人,
如果再加2个人估计就要麻烦了。这个要想想办法, clean一下。

# 参考
+ [Git-内部原理-Git-对象](https://git-scm.com/book/zh/v2/Git-内部原理-Git-对象)
+ [详解Git工作区、暂存区、历史记录区以及git reset、git revert、git checkout等撤销命令的区别](http://josh-persistence.iteye.com/blog/2215214)
+ [关于Git工作流的随笔](https://blog.tonyseek.com/post/jottings-about-git-flow/)
+ [变基](https://git-scm.com/book/zh/v2/Git-分支-变基)

# 补充
## 1. git status的具体过程

> 当执行 "git status" 命令扫描工作区改动的时候，先依据 .git/index 文件中记录的（工作区跟踪文件的）时间戳、长度等信息判断工作区文件是否改变。如果工作区的文件时间戳改变，说明文件的内容可能被改变了，需要打开文件，读取文件内容，和更改前的原始文件相比较（本地文件和与之对应的object库中的文件的内容进行对比），判断文件内容是否被更改。如果文件内容没有改变，则将该文件新的时间戳记录到 .git/index 文件中。因为判断文件是否更改，使用时间戳、文件长度等信息进行比较要比通过文件内容比较要快的多，所以 Git 这样的实现方式可以让工作区状态扫描更快速的执行，这也是 Git 高效的因素之一。

## 2. git object详解
参考[progit2这篇文章](https://git-scm.com/book/zh/v2/Git-内部原理-Git-对象)

