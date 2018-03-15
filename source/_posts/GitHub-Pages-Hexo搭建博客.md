---
title: GitHub Pages + Hexo搭建博客
date: 2018-03-15 10:40:00
tags: hexo 
---
# GitHub Pages + Hexo搭建博客
## 一、 前言
这是一篇是单纯地利用GitHub Pages搭建自己独立博客的过程，并没有额外使用独立域名之类的。

如果是小小白，可以先花时间去了解下：

*  [Git](https://git-scm.com/book/zh/v2)
*  [GitHub](https://github.com)
*  [GitHub Pages](https://pages.github.com)
*  [Hexo](https://hexo.io/zh-cn/)
*  [Markdown](https://www.appinn.com/markdown/#autoescape)

## 二、 必要配置
### 2.1 GitHub Pages 仓库
#### 2.1.1 创建对应仓库
在自己的GitHub账号下创建一个新的仓库，命名为username.github.io（username是你的账号名)。

在这里，要知道，GitHub Pages有两种类型：User/Organization Pages 和 Project Pages，而我所使用的是User Pages。

简单来说，User Pages 与 Project Pages的区别是：
1. User Pages 是用来展示用户的，而 Project Pages 是用来展示项目的。
2. 用于存放 User Pages 的仓库必须使用username.github.io的命名规则，而 Project Pages 则没有特殊的要求。
3. User Pages 将使用仓库的 master 分支，而 Project Pages 将使用 gh-pages 分支。
4. User Pages 通过 http(s)://username.github.io 进行访问，而 Projects Pages通过 http(s)://username.github.io/projectname 进行访问。

#### 2.1.2 相关资料
* [GitHub Pages Basics / User, Organization, and Project Pages](https://help.github.com/articles/user-organization-and-project-pages/)

### 2.2 Git
#### 2.2.1 安装 Git
在windows下安装git比较常用的有两种方式：
1. [Git 官方版本的安装](https://git-scm.com/download/win)
2. [GitHub for Windows](https://desktop.github.com)

#### 2.2.2 配置 Git
当安装完Git应该做的第一件事情就是设置用户名称和邮件地址。这样做很重要，因为每一个Git的提交都会使用这些信息，并且它会写入你的每一次提交中，不可更改：

```
$ git config --global user.name "username"
$ git config --global user.email "username@example.com"
```
对于user.email，因为在GitHub的commits信息上是可见的，所以如果你不想让人知道你的email，可以Keeping your email address private:
1. 在GitHub右上方点击你的头像，选择”Settings”；
2. 在右边的”Personal settings”侧边栏选择”Emails”；
3. 选择”Keep my email address private”。

这样，你就可以使用如下格式的email进行配置：

```
$ git config --global user.email "username@users.noreply.github.com"
```
#### 2.2.3 相关资料
* [安装 Git](https://git-scm.com/book/zh/v2/起步-安装-Git)
* [配置 Git](https://git-scm.com/book/zh/v2/起步-初次运行-Git-前的配置)
* [Setting your email in Git](https://help.github.com/articles/setting-your-commit-email-address-in-git/)
* [Keeping your email address private](https://help.github.com/articles/setting-your-commit-email-address-on-github/)

### 2.3 Git 与 GitHub
#### 2.3.1 git与github的区别
这里，我们要区分清楚git与github。

git是一个版本控制的工具，而github有点类似于远程仓库，用于存放用git管理的各种项目。

#### 2.3.2 与github建立联系
为了能够在本地使用git管理github上的项目，需要进行一些配置，这里介绍SSH的方法。
 
##### 2.3.2.1 检查电脑是否已经有SSH KEYS。

```
$ ls -al ~/.ssh
# Lists the files in your .ssh directory, if they exist
```
默认情况下，public keys的文件名是以下的格式之一：id_dsa.pub、id_ecdsa.pub、id_ed25519.pub、id_rsa.pub。因此，如果列出的文件有public和private钥匙对（例如id_ras.pub和id_rsa），证明已存在SSH keys。

##### 2.3.2.2 如果没有SSH KEY，则生成新的SSH KEY。

```
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
# Creates a new ssh key, using the provided email as a label
```
之后一路回车即可。

##### 2.3.2.3 向SSH-AGENT添加KEY。
首先确保ssh-agent可运行：

```
# start the ssh-agent in the background
$ ssh-agent -s
```
然后添加SSH key：

```
$ ssh-add ~/.ssh/id_rsa
```
##### 2.3.2.4 在GITHUB添加SSH KEY。
首先，拷贝key：

```
clip < ~/.ssh/id_rsa.pub
# Copies the contents of the id_rsa.pub file to your cllipboard
```
然后，在GitHub右上方点击头像，选择”Settings”，在右边的”Personal settings”侧边栏选择”SSH Keys”。接着粘贴key，点击”Add key”按钮。最后，测试链接：

```
$ ssh -T git@github.com
# Attempts to ssh to GitHub
```
如果你看到：

```
The authenticity of host 'github.com (207.97.227.239)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)?
```
就键入：yes。之后将会看到如下信息：

```
Hi username! You've successfully authenticated, but GitHub does not
provide shell access.
```
#### 2.3.3 相关资料
* [Generating SSH keys](https://help.github.com/articles/connecting-to-github-with-ssh/)

### 2.4 Hexo
#### 2.4.1 安装Hexo
安装Hexo相当简单。在安装之前，必须检查电脑中是否已经安装下列应用程序：

* [Node.js](https://nodejs.org/en/)
* [Git](https://git-scm.com)

如果您的电脑中已经安装上述必备程序，那么恭喜您！接下来只需要使用 npm 即可完成 Hexo 的安装。

```
$ npm install -g hexo-cli
```
#### 2.4.2 使用Hexo建站
安装完后，在你喜欢的文件夹内（例如D：\Hexo），点击鼠标右键选择Git bash，输入以下指令：

```
$ hexo init
```
该命令会在目标文件夹内建立网站所需要的所有文件。接下来是安装依赖包：

```
$ npm install
```
这样，我们就已经搭建起本地的Hexo博客了。可以先执行以下命令（在对应文件夹下），然后再浏览器输入localhost:4000查看。

```
$ hexo generate
$ hexo server
```
这个博客只是本地的，别人是浏览不了的，之后需要部署到GitHub上。

#### 2.4.3 相关资料
* [Hexo 官方文档](https://hexo.io/zh-cn/docs/)

### 三、一般的搭建方法
在上面，我们已经配置好了所需的所有东西，也成功地搭建了一个本地Hexo博客。现在，需要使用GitHub Pages搭建一个别人能够访问的Hexo博客了。

#### 3.1 使用默认theme
我们继续使用上面的文件夹D:\Hexo（也可以新建一个文件夹重新生成），然后编辑该文件夹下的_config.yml。

默认生成的_config.yml：

```
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type:
```

修改后的_config.yml：

```
deploy:
  type: git
  repo: 对应仓库的SSH地址（可以在GitHub对应的仓库中复制）
  branch: 分支（User Pages为master，Project Pages为gh-pages）

```
为了能够使Hexo部署到GitHub上，需要安装一个插件：

```
$ npm install hexo-deployer-git --save
```
然后，执行下列指令即可完成部署：

```
$ hexo generate
$ hexo deploy
```
之后，可以通过在浏览器键入：username.github.io进行浏览，开心吧~

#### 3.2 其他theme
如果想要使用其他主题，可以使用git clone将别人的主题拷贝到D:\Hexo\themes下，然后将_config.yml中的theme: landscape改为对应的主题名字。

详细步骤可以参考网上的指南。

### 四、 优化部署与管理
#### 4.1 概述
Hexo部署到GitHub上的文件，是.md（你的博文）转化之后的.html（静态网页）。因此，当你重装电脑或者想在不同电脑上修改博客时，就不可能了（除非你自己写html o(^▽^)o ）。

其实，Hexo生成的网站文件中有.gitignore文件，因此它的本意也是想我们将Hexo生成的网站文件存放到GitHub上进行管理的（而不是用U盘或者云备份啦(╬▔皿▔)凸）。这样，不仅解决了上述的问题，还可以通过git的版本控制追踪你的博文的修改过程，是极赞的。

但是，如果每一个GitHub Pages都需要创建一个额外的仓库来存放Hexo网站文件，我感觉很麻烦（10个项目需要20个仓库(ˉ▽ˉ)…）。

所以，我利用了分支！！！

简单地说，每个想建立GitHub Pages的仓库，起码有两个分支，一个用来存放Hexo网站的文件，一个用来发布网站。

下面以我的博客作为例子详细地讲述。
#### 4.2 我的博客搭建流程
1. 创建仓库，liuxc123.github.io；
2. 创建两个分支：master 与 hexo；
3. 设置hexo为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件）；
4. 使用git clone git@github.com:liuxc123/liuxc123.github.io.git拷贝仓库；
5. 修改_config.yml中的deploy参数，分支应为master；
6. 依次执行git add .、git commit -m “…”、git push origin hexo提交网站相关的文件；
7. 执行hexo generate -d生成网站并部署到GitHub上。
8. 这样一来，在GitHub上的CrazyMilk.github.io仓库就有两个分支，一个hexo分支用来存放网站的原始文件，一个master分支用来存放生成的静态网页。完美( •̀ ω •́ )y！

#### 4.3 我的博客管理流程
##### 4.3.1 日常修改

在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理：

1. 依次执行`git add .`、`git commit -m “…”`、`git push origin hexo`指令将改动推送到GitHub（此时当前分支应为hexo）；
2. 然后才执行`hexo generate -d`发布网站到master分支上。

虽然两个过程顺序调转一般不会有问题，不过逻辑上这样的顺序是绝对没问题的（例如突然死机要重装了，悲催….的情况，调转顺序就有问题了）。

##### 4.3.2 本地资料丢失
当重装电脑之后，或者想在其他电脑上修改博客，可以使用下列步骤：

1. 使用`git clone git@github.com:liuxc123/liuxc123.github.io.git`拷贝仓库（默认分支为hexo）；
2. 在本地新拷贝的liuxc123.github.io文件夹下通过Git bash依次执行下列指令：`npm install hexo`、`npm install`、`npm install hexo-deployer-git`（记得，不需要`hexo init`这条指令）。

###五、结尾
ok,现在你就可以轻松的搭建自己的博客了。(ˉ▽ˉ)



