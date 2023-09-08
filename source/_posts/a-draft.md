---
title: A draft
date: 2023-09-06 20:00:32
tags:
categories: 
thumbnail: ../images/catH.png
---




# 一、关于Hexo
## 1.简介
**什么是Hexo？**
>官网介绍：快速、简洁且高效的博客框架

Hexo是一个基于Node.js的静态网站生成器，它能够将Markdown等格式的文本内容转化成静态网页，方便快速构建博客或其他类型的静态网站。
## 2.文件架构
```
├── node_modules：    #依赖包-安装插件及所需nodejs模块。
├── public            #最终网页信息。即存放被解析markdown、html文件。
├── scaffolds         #模板文件夹。即当您新建文章时，根据 scaffold生成文件。
├── source            #资源文件夹。即存放用户资源。
|   └── _posts        #博客文章目录。
└── themes            #存放主题。Hexo根据主题生成静态页面。
├── _config.yml       #网站的配置信息。标题、网站名称等。
├── db.json：         #source解析所得到的缓存文件。
├── package.json      # 应用程序信息。即配置Hexo运行需要js包。
```
## 3.命令总览

+ hexo-cli：hexo命令行，作用是：

  + 启动hexo命令进程和参数解析机制。每次我们输入hexo xxx命令后，都会通过node调用hexo-cli中的entry函数(比如，可以把’hexo init’视为’node hexo-cli/entry.js init’)，hexo init命令仅仅在安装时调用

  + 实现hexo命令的三个初始参数：init/version/plugins

  + 加载hexo核心模块，并初始化

+ hexo core：hexo核心，作用是：

  + 实现hexo的new、generate、publish等功能

+ hexo plugins: 指一些能够扩展hexo的插件。插件可以按功能分成两类:
  + 扩展hexo命令的参数，如hexo-server(安装这个插件以后才能使用hexo server命令)
  + 扩展hexo解析文件的”能力”，如增加jade模版解析功能的hexo-render-jade插件
  ![命令总览](../images/hexo%20cmd.png)
## 4.部署流程
+ `hexo g`：生成静态文件。将我们的数据和界面模板相结合生成静态文件的过程。Hexo（node.js程序）遍历主题文件中你的source目录（js、css、img等静态资源），建立索引，再根据索引生成由html、js、css、img建立的纯静态文件并放在public文件夹里。public就是你的博客了，而这些恰好能被gitpages识别。
+ `hexo d`：部署文件。主要是根据在_config.yml中配置的git仓库或者coding的地址，将public文件通过git方式push到上传到github或coding的指定分支，然后在根据pages服务呈现出页面。当然把public文件部署至你的服务器也是可以的。

---
# 二、关于Git
## 1.管理机制
Git 是一种分布式版本控制系统，用于跟踪和管理项目的源代码以及各种文件。在博客管理中，Git 用于管理博客的源文件和相关资源，而不仅仅是生成的网页。`hexo d`上传部署到github的其实是hexo编译后的文件，是用来生成网页的，不包含源文件。也就是上传的是在本地目录里自动生成的.deploy_git。其他文件 ，包括我们写在source 里面的，和配置文件，主题文件，都没有上传到github。所以可以利用git的分支管理，将源文件上传到github的另一个分支即可。
## 2.Git常用命令

### `git init`

初始化一个新的 Git 仓库。
该命令在当前目录下创建一个名为 `.git` 的文件夹，用于存储 Git 仓库的配置和历史记录。

### `git clone <仓库URL>`

克隆远程 Git 仓库到本地。
通过提供仓库的 URL，将远程仓库的所有文件和历史记录复制到本地。

### `git add <文件或目录>`

将文件或目录的更改添加到 Git 的暂存区。
在提交前，使用该命令将要保存的更改移动到暂存区。

### `git commit -m "提交消息"`

提交暂存区中的更改，并创建一个新的提交。
提交消息通常用于描述此次提交的目的或内容。

### `git status`

显示工作目录和暂存区中文件的状态。
它会告诉你哪些文件已修改、哪些文件已添加到暂存区、哪些文件未跟踪等信息。

### `git log`

显示仓库的提交历史记录。
它会列出每个提交的作者、日期、提交消息以及哈希值。

### `git branch`

显示当前仓库的所有分支。
当前分支会用星号 `*` 标记。

### `git checkout <分支或提交>`

切换到指定分支或提交。
用于切换工作目录中的文件版本，可以是分支名或提交的哈希值。

### `git pull`

从远程仓库拉取最新的更改并合并到当前分支。
通常与 `git fetch` 一起使用，以更新本地仓库。

### `git push`

将本地分支的更改推送到远程仓库。
通常需要指定远程仓库的名称和分支名，例如 `git push origin master`。

---
# 三、基于Github的Hexo环境搭建


## 1.安装Git

Git是目前世界上最先进的分布式版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理。

+ Windows:到[git官网](https://gitforwindows.org/)下载。
+ Linux:
`
sudo apt-get install git
`
+ 检查:`git --version`

## 2.安装nodejs
Node.js（通常简称 Node）是一个开源的、跨平台的 JavaScript 运行时环境，它允许开发者使用 JavaScript 编写服务器端应用程序。在 Hexo 中，Node.js 的主要作用是作为 Hexo 的运行时环境。
+ Windows:到[nodejs官网](https://nodejs.org/en/download)下载，选择LTS版本。
+ Linux:
```
sudo apt-get install nodejs
sudo apt-get install npm
```
+ 检查:
```
node -v
npm -v
```

## 3.安装Hexo
先创建一个文件夹blog，然后cd到这个文件夹下（或者在这个文件夹下直接右键git bash打开）。

+ 输入命令：
```
npm install -g hexo-cli
```
+ 检查:
```
hexo -v
```
+ 初始化

```
hexo init myblog
cd myblog //进入这个myblog文件夹
npm install
```
+ 运行测试
```
hexo g
hexo server
```
输入localhost:4000验证。

## 4.将Hexo部署到Github
+ 在GitHub中创建一个和你用户名相同的仓库，后面加 .github.io。< name >.github.io。

+ 回到你的git bash中，配置 Git 全局用户信息：
```
git config --global user.name "yourname"
git config --global user.email "youremail"
```
+ 检查信息:
```
git config user.name
git config user.email
```
+ 创建ssh[^1]

```
ssh-keygen -t rsa -C "youremail"
```
+ 在GitHub的setting中，找到SSH keys的设置选项，点击New SSH key
把你的id_rsa.pub里面的信息复制进去。
+ 检查：
 ```
ssh -T git@github.com
 ```


[^1]:SSH（Secure Shell）是一种加密网络协议，用于在不安全的网络中建立安全的远程连接。SSH协议的主要目的是通过加密和身份验证来保护数据的传输和远程访问。
运行这个命令后，系统会生成一个RSA密钥对，包括一个私钥（默认保存在 ~/.ssh/id_rsa）和一个公钥（默认保存在 ~/.ssh/id_rsa.pub）。私钥是私有的，必须妥善保管，而公钥可以共享给其他人或远程服务器以进行身份验证。
生成SSH密钥对后，你可以将公钥添加到远程服务器上，以便通过SSH进行安全的远程访问。私钥则会用于本地SSH客户端与远程服务器进行通信时的身份验证。
+ 打开站点配置文件 _config.yml，修改deploy部分为：
```
deploy:
  type: git
  repo: https://github.com/YourgithubName/YourgithubName.github.io.git
  branch: master
```
+ 安装deploy-git ，也就是部署的命令,这样你才能用命令部署到GitHub。
```
  npm install hexo-deployer-git --save
```
+ 运行：
```
hexo clean
hexo generate
hexo deploy
```
访问http://yourname.github.io


## 5.Git分支进行多终端工作

+ 首先，先在github上新建一个hexo分支。
+ 然后在这个仓库的settings中，选择默认分支为hexo分支。
+ 然后在本地的任意目录下输入：
  ```
  git clone git@github.com:ZJUFangzh/ZJUFangzh.github.io.git
  ```
+ 接下来在克隆到本地的ZJUFangzh.github.io中，把除了.git 文件夹外的所有文件都删掉
。把之前我们写的博客源文件全部复制过来，除了.deploy_git。这里应该说一句，复制过来的源文件应该有一个.gitignore，用来忽略一些不需要的文件，如果没有的话，自己新建一个，在里面写上如下，表示这些类型文件不需要git：
```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```
注意，如果你之前克隆过theme中的主题文件，那么应该把主题文件中的.git文件夹删掉，因为git不能嵌套上传，最好是显示隐藏文件，检查一下有没有，否则上传的时候会出错，导致你的主题文件无法上传，这样你的配置在别的电脑上就用不了了。
+ 上传：
```
git add .
git commit –m "add branch"
git push 
```
## 6.绑定域名
+ 在[阿里云](https://wanwang.aliyun.com/?spm=5176.8142029.digitalization.2.e9396d3e46JCc5)上**购买域名**。
+ 进入阿里云控制台找到域名管理，进入解析**添加 DNS 记录**：
  + A 记录（地址记录）：将域名指向博客托管服务的 IP 地址。你需要向 A 记录中添加托管服务提供的 IP 地址。这个 IP 地址通常可以在托管服务的文档或设置中找到。
    + 记录类型为A
    + 主机记录为：@
    + 记录值为自己<>.git.io的ip地址。IP地址在cmd中ping一下即可,`ping -4 XIciA.github.io`
  + CNAME 记录（别名记录）：如果你的博客托管服务有自定义域名设置，你可能需要添加一个 CNAME 记录，将域名指向托管服务的自定义域名。这通常在服务的文档中有详细说明。
    + 记录类型为CNAME
    + 主机记录为www
    + 记录值为自己的github访问地址。
+ **TTL 设置**：TTL（生存时间）是 DNS 记录在缓存中保存的时间。你可以根据需要设置 TTL 的值。较短的 TTL 可以更快地生效域名设置变更，但也可能导致 DNS 请求增加。一般来说，你可以选择默认设置或者设置为较短的时间，然后再根据需要更改。
+ **Hexo配置域名**:
在自己博客的跟目录source文件夹下创建一个名字CNAME记事本
内容输入自己的个人域名，保存关闭。删除后缀.txt
+ **仓库绑定域名**：在GitHub中找到自己的博客仓库，点击setting，拉到最下面找到GitHub Pages点进去,在Custom domain输入自己的域名。
## 7.发布文章

+ **创建草稿**：首先，在 Hexo 博客项目的根目录下，使用以下命令创建一篇新的草稿文章：

```
hexo new draft my-new-draft
```

+ **发布草稿**：发布草稿： 当你准备好将草稿发布为一篇正式文章时，使用以下命令发布它：

```
hexo publish draft my-new-draft
```
这将移动草稿文件从 source/_drafts 目录到 source/_posts 目录，并将其日期信息更新为当前日期和时间。


+ **生成静态网站文件**：在命令行中，进入你的 Hexo 项目的根目录，然后运行以下命令来生成静态网站文件：

```
hexo generate
```

这个命令会根据你在 Hexo 项目中的配置生成静态 HTML 文件和其他资源文件。生成的文件会存储在 Hexo 项目的 `public` 目录下。

+ **预览你的修改**（可选）：在生成静态文件后，你可以在本地预览你的网站，以确保修改没有问题。你可以使用以下命令启动本地服务器：
```
hexo server
```

然后在浏览器中访问 `http://localhost:4000` 来查看网站。

+ **部署你的网站**：一旦你确认修改没有问题，你可以使用以下命令来将网站部署到远程服务器或托管平台：

```
hexo deploy
```
这会自动将生成的静态文件推送到 GitHub Pages 上。如果你使用其他托管服务或者服务器，你需要相应地配置和执行部署命令。

+ **提交你的修改到 Git 仓库**：在你发布网站之后，确保将你的修改和生成的文件提交到你的 Git 仓库，以便跟踪网站的历史和与团队成员协作。你可以使用以下命令来提交：
```
git add .
git commit -m "描述你的修改"
git push 
```
---
# 四、相关脚本
“一键保存”
```
import subprocess
import os

# 设置 Hexo 项目的目录路径
hexo_project_directory = r"D:\BLOG\branch\XIciA.github.io"
# 切换到 Hexo 项目目录
os.chdir(hexo_project_directory)
# 获取当前工作目录的路径
current_directory = os.getcwd()
print("Path:",current_directory)

commit_message = input("Please enter Git submit description:")
# 定义要执行的命令列表
commands = [
    'hexo clean',  
    'hexo generate',  
    'hexo deploy', 
    'git add .',
    f'git commit -m "{commit_message}"',
    'git push',
    'git log',
]

# 使用 subprocess 执行命令
try:
    for cmd in commands:
        print(f"Running: {cmd}")
        result = subprocess.run(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True, check=True)
        print(f"Result: {result.stdout}")
    print('Successful Completion')
except subprocess.CalledProcessError as e:
    # 如果命令执行失败，打印错误信息
    print(f"Error running command: {e}")
```
# 五、客制化笔记

## 1.修改前段界面的关键路径
+ 汉化文件夹：`D:\BLOG\branch\XIciA.github.io\themes\hueman\languages\zh-CN.yml`
+ ejs文件夹：`D:\BLOG\branch\XIciA.github.io\themes\hueman\layout\common\post`
+ css文件夹：`D:\BLOG\branch\XIciA.github.io\themes\hueman\source\css\_partial`
## 2.修改sidebar-top
+ 可在主题配置文件`D:\BLOG\branch\XIciA.github.io\themes\hueman\_config.yml`中的`social_links`进行基本的更改。这里注释掉所有相关功能，让其不再显示图标。

+ 在`D:\BLOG\branch\XIciA.github.io\themes\hueman\layout\common\sidebar.ejs`中修改`sidebar.follow`为`sidebar.sentence`并删除`</p>`前的冒号。

+ 在`D:\BLOG\branch\XIciA.github.io\themes\hueman\languages\zh-CN.yml`中的sidebar部分添加变量`sidebar.sentence`并加上自己喜欢的句子。

+ 可以在`D:\BLOG\branch\XIciA.github.io\themes\hueman\source\css\_partial\sidebar.styl`调整css样式。

# 六、踩雷与排雷
## 1.静态博客在线版本与源文件的Git管理
 **踩雷**：master与branch,hexo deploy与git push,Git管理与静态页面混淆不清,导致分支上混子静态页面，试图用分支展示静态页面解决，进一步修改配置文件部署部分到hexo分支，最后运行git push和github分支页面时均报错。  
  
  **排雷**：Hexo 部署应该将静态博客文件部署到 master 分支或者您用于托管博客的主分支。这是博客的在线版本。另一方面，可以在 hexo 分支上管理 Hexo 博客的源文件、配置和原始内容。这个分支用于 Hexo 生成静态博客文件，但不直接用于部署。您可以使用 hexo d 或者其他 Hexo 部署命令来将生成的静态文件提交到 master 分支或其他托管博客的主分支。

## 2.消除`git add`产生的一种warning
**踩雷**：windows平台进行 `git add` 时，控制台打印警告`LF will be replaced by CRLF the next time Git touches it
`




**排雷**:这个警告是由于你的 Git 配置中设置了自动换行 (line endings) 规则，但是在你的项目中发现了与该规则不符的文件。这个警告意味着 Git 将会在下一次操作时更改这些文件的换行符。要解决这个问题，你可以在项目的根目录中创建一个名为 .gitattributes 的文件，并在其中指定换行符的规则。例如，如果你想在整个项目中使用 LF 换行符，可以添加以下内容：
```
* text eol=lf
```
这将告诉 Git 对所有文本文件使用 LF 换行符。然后，提交这个 .gitattributes 文件到你的版本控制系统，以确保团队中的所有人都使用相同的换行符规则。在添加了 .gitattributes 文件后，你可能需要重新执行一次 git add . 来重新标记那些文件，然后提交它们。这样可以确保在以后的提交中使用正确的换行符规则。需要注意的是，如果你在 Windows 上使用文本编辑器，它可能会自动将文件保存为 CRLF 格式。你可以在编辑器的设置中查找选项，以确保它以 LF 格式保存文件，或者手动更改文件的换行符格式。这可以帮助减少 Git 警告的出现。

# 七、一点感受
>雷军：知识不全是线性的，大部分是网状的，知识点之间也不一定有绝对的先后关系，前面的内容看不懂，跳过去，并不影响学后面的；后面的学会了，有时候更容易看懂前面的。

说点什么

>Alex:程序员看重的不仅仅是成功解决问题，代码的健壮性与可扩张性更能体现能力。