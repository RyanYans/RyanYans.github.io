---
layout:     post
title:      "利用GitHub&Jekyll建立个人博客"
subtitle:   " \"快速搭建属于你自己的Blog。\""
date:       2016-03-27 12:30:00
author:     "RyanYans"
header-img: "img/Githubpage.jpg"
catalog: true
tags:
    - Jekyll
    - Github
---

### 在阅读之前，请先确认：  
* 我知道Git是干嘛用的，或者我可以 现在开始学习Git——[Git Pro 中文版](https://git-scm.com/book/zh/v2) 
* 我有基本的计算机知识，改个环境变量什么的没问题。  
* 我愿意折腾
* ....

![GithubPage](http://i9.qhimg.com/t018049f9215df32183.jpg)


## 一、为什么在GitHub上托管个人博客
  
* Github很好的将代码和社区联系在了一起，于是发生了很多有趣的事情，**世界也因为他美好了一点点**。Github作为现在最流行的代码仓库，已经得到很多大公司和项目的青睐，比如jQuery、Twitter等。为使项目更方便的被人理解，介绍页面少不了，甚至会需要完整的文档站，Github替你想到了这一点，他提供了Github Pages的服务，不仅可以方便的为项目建立介绍站点，也可以用来建立个人博客。


### Github Pages有以下几个优点：
* 轻量级的博客系统，没有麻烦的配置
* 使用标记语言，比如Markdown
* 无需自己搭建服务器
* 根据Github的限制，对应的每个站有300MB空间
* 可以绑定自己的域名
 
### 当然他也有缺点：  
* 使用Jekyll模板系统，相当于静态页发布，通常仅适合博客，文档介绍等。
* 动态程序的部分相当局限，比如没有评论，不过还好我们有解决方案。
* 基于Git，很多东西需要动手，不像Wordpress有强大的后台。

## 二、快速搭建一个博客原型
### 首先确认：
* 拥有一个GitHub账号
* 已经安装 Git 

### 1.建立一个项目
![Chekout](http://img1.tuicool.com/vaEvum.jpg!web)	
* 先在GitHub创建一个项目，名字写你注册时的用户名，尽量全部小写字母，避免随后会碰到的URL冲突问题

* 本地运行Git，随意选个目录，将刚才创建的项目clone下来

		cd /f/Blog
		git clone https://github.com/UserName/UserName.github.io.git
		cd Blog

### 2.了解Jekyll模板的基本结构

		|-- _config.yml
		|-- _includes
		|-- _layouts
		|   |-- default.html
		|   |-- post.html
		|-- _posts
		|   |-- 2007-10-29-why-every-programmer-should-play-nethack.textile
		|   |-- 2009-04-26-barcamp-boston-4-roundup.textile
		|-- _site
		|-- index.html

简单介绍一下他们的作用：

* _config.yml

 配置文件，用来定义你想要的效果，设置之后就不用关心了。

* _includes

 可以用来存放一些小的可复用的模块，方便通过{ % include file.ext %}（去掉前两个{中或者{与%中的空格，下同）灵活的调用。这条命令会调用_includes/file.ext文件。

* _layouts

 这是模板文件存放的位置。模板需要通过YAML front matter来定义，后面会讲到，{ { content }}标记用来将数据插入到这些模板中来。

* _posts

 你的动态内容，一般来说就是你的博客正文存放的文件夹。他的命名有严格的规定，必须是2012-02-22-artical-title.MARKUP这样的形式，MARKUP是你所使用标记语言的文件后缀名，根据_config.yml中设定的链接规则，可以根据你的文件名灵活调整，文章的日期和标记语言后缀与文章的标题的独立的。

* _site

 这个是Jekyll生成的最终的文档，不用去关心。最好把他放在你的.gitignore文件中忽略它。

### 3.建立配置文件_config.yml
* 注意：之后建立的所有文档务必使用UTF-8 无 BOM 的编码保存  
在项目的根目录下新建文件 _config.yml ，填写 baseurl: /blogdemo ， blogdemo是你的项目名称，这一行内容规定了整个网站的根路径，稍后会详细解释这样做的意义。 

### 4.建立主页
在根目录下新建文件 index.html ， 内容像这样：

		---
		title: Hello, My Blog
		---

		\{{ page.title }}  

先不用急着奇怪为什么一个html文件会出现”{}“这样的标签，这里使用的是 Liquid 模板语言 ，{{ page.title }} 表示“本页面的标题”，更详细的介绍我们以后再讲，不妨这样理解：

	#--- begin of page's head
	title = "Hello, My Blog"
	#--- end of page's head

	print(page.title)

OK，那么博客主页设计完毕！（别吵…我答应过你要快速搭建完成的…先弄个毛胚房意思意思，美化啊功能啊什么的晚点再说嘛。。）  

### 5.发布到GitHub

* 回到git bash, 检查一下 git status ，确认 _config.yml 与 index.html 无误后 add，commit，保持使用Git的良好习惯，记得添加提交描述 commit -m "Your view"

* 然后推送到GitHub，这里注意，因为我们使用的是GitHub Pages中的 Project Pages, GitHub仅会将分支 gh-pages 下的内容进行自动生成操作， 所以本地的 master 分支应推送到远端的 gh-pages 分支

稍微等待一下，最多10分钟（通常不用那么久啦），访问 yourname.github.io// ，（其中 yourname 是你的GitHub帐户名）就会看见你的博客主页（确实很丑…而且完全不像一个博客的样子，不过别急，慢慢来比较快~）

---  **另外，如果不幸发现你的中文页面出现了乱码的情况，别着急，还是该死的UTF-8问题，后面我们会一劳永逸的解决他的，暂时先手动调整一下浏览器的编码。**

### 6.在_posts内撰写文章

在这段时间里，我们继续为你的博客添砖加瓦，让他拥有最基本的文章阅读功能，另外不断F5页面的同时也可以关注一下自己在GitHub注册时所用的邮箱，如果之前推送的内容有误的话，GitHub将以邮件形式提醒你生成失败。

回到项目根目录， mkdir _posts 新建一个目录，看名字也知道啦，这里存放你所有的文章。

进入_posts目录，新建一篇文章。注意默认的文件名格式是 year-month-day-postTitle 这样。比如 2016-03-27-Hello-World.md，尽量避免空格或者其他乱七八糟的字符，这个文件名将作为URL的生成依据。文件名的格式可以通过修改 _config.yml 中的 permalink 属性而改变，默认值为 date，也就是我们刚刚创建的文件的样子，具体的规则可以看这里，后面我们也会讲到。

如果你发现了我刚才创建的文件后缀名是 .md(.markdown) ，熟悉GitHub或者StackOverFlow的朋友应该知道Markdown 格式，推荐使用GitHub托管博客的原因之一也正是如此，我们可以在大部分时候避开恼人的HTML，转而使用更加直观的Markdown语法。如果不熟悉也没关系，可以参见这份Markdown 语法说明，应该说是相当易学，并且在熟悉之后非常易用的。(当然博主在windows下用的是markdownpad2)

回到主题，打开刚才创建的文件，输入如下内容：
	---
	title: 我的第一篇文章
	---

	# {{ page.title }}
	
	## 目录
	+ [第一部分](#partI)
	+ [第二部分](#partII)
	+ [第三部分](#partIII)
	
	----------------------------------
	
	## 第一部分 <p id="partI"></p>
	这里是第一部分的内容
	
	----------------------------------
	
	## 第二部分 <p id="partII"></p>
	这里是第二部分的内容
	
	----------------------------------
	
	## 第三部分 <p id="partIII"></p>
	这里是第三部分的内容
	
	{{ page.date|date_to_string }}

这段内容中使用了最常用的几种Markdown语法，比如使用 # ，## 表示 HTML 中的h1>/h1>,h2>/h2>。使用[text](link)创建超链接，使用 连续多个 - 创建水平线（注意：不包括最上端包围title所使用的横线，那里表示一个页面的“头属性”），等等。更多详细的语法可以在之前提到的页面查询，这里不再赘述，总之，这是一种更加贴近真实写作的语法，推荐大家尝试。

啊对了，最后面的那个 {{ page.date|date_to_string }} 是指显示本页的日期属性，并且转换为可读的字符串形式。同样也是Liquid语法。

OK，那么第一篇文章也写好了，再把最新的repo推送到github，稍等片刻，就可以…等下，忘记给文章加上入口的链接了。

重新打开我们的 index.html 文件，添加内容，变成下面这样：
	---
	title: My Blog
	---
	
	{{ \page.title }}
	
	{% for post in site.posts %}
	
	{{ \post.date|date_to_string }} <a href='{{ \site.baseurl }}{{ \post.url }}'>{{ \post.title }}</a>
	
	{% endfor %}

### 7.增加模板套装_layouts
很抱歉我欺骗了你，这已经超出快速的范围了，可能有很多人已经坚持不到这步了。。不过至少我们进展很快～接下来——如果你仍有兴趣的话，让我们为网站增加一些统一性的风格设置。

回到项目根目录，新建文件夹 _layouts，顾名思义，“布局”是也。在 _layouts 中新建一个最基本的布局文件，姑且命名为default.html好了（记得首先解决了UTF-8的编码问题哦）：  
	
	<!DOCTYPE html>
	<html>
	    <head>
	      <meta http-equiv="content-type" content="text/html; charset=utf-8" />
	      <title>{{ \page.title }}</title>
	    </head>
	    <body>
	
	      {{ content }}
	
	    </body>
	</html>

然后我们修改index.html和刚写完的那篇文章，只要在头属性上加一句就好：

	---
	title: xxoo
	layout: default.html
	---

我们当然还可以把这个结构变得更灵活一些，比如继续新增两个模板分别叫做l_post.html与l_index.html，他们首先引用default.html，但在其基础上做出一定的修改。然后首页使用l_index模板，而所有的post文件则使用l_post模板，等等等等，请随意发挥。但始终记得加上 {{ content }} 标签

再次推送到服务端，查看效果。

至少这一点我没骗你，要发布最新的更改实在是太简单了，只需要一次push而已。哈哈~o(^▽^)o~

**到此为止，基本的步骤和项目结构已经全部讲完了，祝你能折腾愉快！**

###### 多谢大家的耐心观看，如果有任何的意见或疑问，请直接在下方留言，或是通过页面点击我的社交图标与我联系。下次再见～