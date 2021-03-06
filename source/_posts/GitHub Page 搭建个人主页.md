---
title: GitHub Page 搭建个人主页
tags: GitHub,blog
category: /personal blog/build/2022-03
renderNumberedHeading: true
renderMetaTitle: true
renderMetaTags: true
grammar_highlight: true
grammar_cjkRuby: true
---



**github Pages——用户编写的、托管在github上的静态网页**


----------


## Create Personal Repositories
++repository是一个存放项目文件的仓库，之后博客源码和hexo发布后的静态HTML文件全部保存在里面。++

1. **登录自己的github账号**
![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647518468944.png)
2. **点击右上角的'+'号，选择'New repository'跳转到下面的界面**
![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647518514346.png)	
**Repository name:** 选择-用户名.github.io-的命名格式，之后的site就直接为http://username.github.io
**Public/Private:** 当前创建私有仓库已经免费，如果是个人不想公开的代码和日记等，可以选择Private，已经创建好的仓库也可以转换为另一种权限。
**Initialize this repository with:** 通常会选择'Add a README file'

3. **创建完毕后，打开仓库，点击⚙setting-Pages，此时界面如下：**
![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647519433143.png)
**Choose a theme：** 暂时选择一个默认网页模板，之后可以修改为自己喜欢的主题。
至此，一个只有readme文件的空仓库就创建好了。

**参考链接** 
[不同权限之间的转换方式](https://www.cnblogs.com/05-hust/articles/13607712.html)


----------


## Git安装
1. **安装过程参考本节最后的链接，安装好的效果如下：**
![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647520903724.png)
	
2. **打开 Git Bash, 输入git，验证安装是否成功:**
![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647520989806.png)
   ●进行任何操作时，先切换到仓库目录
   ●声明为Git仓库+信息提交
    <mark>git init</mark> → 初始化本地git仓库 
	<mark>git add  filename</mark> → 将file添加到[临时缓冲区]
	<mark>git status</mark> → 获取仓库当前的状态
	<mark>git commit -m "text commit" </mark> → 将[临时缓冲区]的内容提交到仓库
   ●关于分支的git指令
   <mark>git branch a</mark> → 创建分支a
   <mark>git checkout a</mark> → 切换到分支a
   <mark>git checkout -b a</mark> → 创建分支a的同时进行切换
   <mark>git merge a</mark> → 在主分支下执行，将a分支与主分支进行合并
   <mark>git branch -d/-D a</mark> → 删除分支a


**参考链接** 
[git下载安装指南](https://zhuanlan.zhihu.com/p/103325381)


----------


## 绑定GitHub
1. **本地内容怎么投放到github上对应的仓库呢？通过什么协议将本地和自己的github进行绑定呢？**
	●**利用SSH登录远程主机：**
	<mark>method1</mark>：口令登录；(需要反复输入密码  
	<mark>method2</mark>: 公钥登录；
	●**安全外壳协议(SSH)**
2. **确认本机是否是否安装 SSH**
![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647522852374.png)

3. **生成密钥**
	<mark>ssh -keygen -t rsa</mark>(通过RSA算法实现)
	<mark>id_ras</mark> → 密钥(本地)
	<mark>id_rsa.pub</mark> → 公钥(添加到github)
	
	生成文件的所在路径和目录文件如下：
![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647561811480.png)

4. **将公钥添加到自己的github**
		●通过找到相应路径，用文本模式打开id_rsa.pub复制其中的内容
		● 直接在git bash中打开，指令如下
		<mark>$ cd ~/.ssh</mark>
		<mark>$ ls</mark>
		<mark>$ cat id_rsa.pub</mark>
		(注：git中的复制粘贴不是 Ctrl+C 和 Ctrl+V，而是 Ctrl+insert 和Shift+insert)
		
	●点击头像-setting-SSH and GPG keys-New SSH key
![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647562157635.png)
![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647563056861.png)

   **Title:** 选填
   **Key:** 将公钥文件的内容复制到这里
	
5. **验证是否绑定成功**
<mark>ssh -T git@github.com</mark> (之后键入yes)
		


----------


## 实现提交文件
1. **基本指令**
	<mark>pull</mark> → 本地 to 远程仓库
	<mark>push #F44336</mark> → 远程仓库 to 本地
	example: <mark>git pull/push origin master</mark>(branch name)
	
	●**情况一:当本地没有 git 仓库**
			a. 直接将远程仓库 clone 到本地(无需通过init进行初始化，并已经自动关联了远程仓库)
			b. 将文件添加并 commit 到本地仓库
				<mark>git add</mark>
				<mark>git commit -m ""</mark>
			c. 将本地仓库的内容push到远程仓库
			
	●**情况二:本地已有Git仓库，并且进行了多次提交**
			a. 关联远程仓库：
			<mark>git remote add origin http:// </mark>
			b. 同步远程和本地仓库：
			<mark>git pull origin master(this is branch name) </mark>
			c. 新建file
			<mark>git add test,txt</mark>
			<mark>git commit -m "add test file"</mark>
			d. 将修改后的内容推到远程仓库： 
			<mark>git push origin master</mark>(同上)
			(注：先pull后push)
			
**参考文献**
[将本仓库更新到远程参考](https://zhuanlan.zhihu.com/p/265454741)
[两种情况的详细指南](https://zhuanlan.zhihu.com/p/103391101)			
	


----------


## 具体的BLOG Pages搭建

1. **安装nodejs**
	[下载地址](https://nodejs.org/en/)
![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647563732842.png)
  ●测试是否安装成功：用 node -v 和 npm -v 命令检查版本
![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647563776437.png)
	●关于设置npm在安装全局模块时的路径和环境变量(当没有安装在默认路径)[参考链接](https://zhuanlan.zhihu.com/p/105715224)
		

2. **安装Hexo**
	● 通过npm安装hexo：
		<mark>npm install -g hexo-cli #F44336</mark>
		●输入 <mark>hexo init #F44336</mark> 命令初始化博客
	●.相关指令 
	    <mark>hexo g</mark> → 静态部署
	    <mark>hexo s </mark> → 本地预览静态网页
		<mark>hexo d</mark>→ 上传网页文件到github

3. **将博客发布到github上环境配置**
	● 配置blog目录中的_config.yml(注意":"号后的空格)
	![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647564081849.png)
		**repo:** 目标仓库名
		**branch：** 所在分支(注：这里提交的是hexo相关的框架文件，并不是blog目录文件)

	●Deployer配置：
			error : Deployer not find:git
			reason: 没安装hexo-deployer-git插件
			solution: npm install hexo-deployer-git --save
			
	●输入以下指令：
			<mark>hexo clean</mark> → 清除缓存文件 db.json 和已生成的静态文件 public
			<mark>hexo g</mark>→ 生成网站静态文件到默认设置的 public 文件夹(hexo generate 的缩写)
			<mark>hexo d</mark> → 自动生成网站静态文件，并部署到设定的仓库(hexo deploy 的缩写)

	●配置url
		**情况一：xxx为用户名**
		       url: http://xxx.github.io
			   root: /
	    **情况二：xxx不是用户名**
			  url: .../xxx.github.io
			  root: /xxx.github.io
			  
**参考链接**
[config.yml文件内容详解](https://zhuanlan.zhihu.com/p/127786638)

4. **主题更换**
	●config.yml文件中相关的部分： 
![Extensions part](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647564319699.png)
	●themes文件夹中相关的部分：
	当前该目录下又两个主题，在config.yml中选择了“fexo”主题
	![enter description here](https://raw.githubusercontent.com/echolumj/blogImg/main/blog/1647564488865.png)
	●主题配置的过程
	 a. 在下面的hexo主题集合中选择好自己喜欢的主题
	 b. 打开博客根目录Blog文件夹，右键Git Bash，输入如下代码将目标主题下载到目录Blog/themes：
	 <mark>git clone (http) themes/next</mark>
	 c. 修改blog目录下的config.yml文件themes(注意冒号后的空格)
	 d. 主题页面的修改文件为主题文件内的_config.yml文件，如何修改参鉴clone地址的readme信息
	 
**参考链接：**
[hexo主题集合](https://github.com/FoxerLee/awesome-hexo-themes)


----------


## 之后要增加的内容
<mark>
怎么把具体的博客内容发送到自己的pages上
怎么把hexo原代码也发送到远程仓库
其他一些操作指令
小书匠图库的配置
.... 
</mark>
