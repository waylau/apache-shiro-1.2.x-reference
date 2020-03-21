# 21. Beginner's Webapp Tutorial 初学者web应用教程


本文是一篇循序渐进介绍用 Apache Shiro 保护 web 应用程序的教程。 它假定读者已经具备了 Shiro 的入门知识,并假设至少熟悉以下两个介绍性文档:

* 用Shiro保护你的应用安全
* Apache Shiro 十分钟教程

学习本教程应该需要45分钟到1个小时时间。 当你完成后,你将有一个很好的关于 Shiro 是如何在一个 web 应用程序的概念。



## 概述

虽然 Apache Shiro 的核心设计目标允许它被用于任何基于 java 的应用程序的安全,如命令行应用程序、服务器守护进程 ,web 应用程序,等等,本指南将专注于最常见的用例:确保 web 应用程序安全运行在一个 servlet 容器,例如 Tomcat 或 Jetty。

### 先决条件

以下工具将被安装在本地开发机器为了跟随本教程。

* Git(测试版 w/1.7)
* Java SDK 7
* Maven 3
* 你最喜欢的IDE,比如 IntelliJ IDEA 或 Eclipse ,甚至一个简单的文本编辑器用于查看文件和更改。

### 教程格式

这是一个循序渐进的教程。 本教程,和它的所有步骤,存在Git存储库。 当你复制 git 存储库, master 分支是你的起点。 在教程的每一步都是一个独立的分支。 你可以跟随只需查看 git 分支反映本教程一步你审查

### 应用程序

我们将构建的 web 应用程序是一个超级网络应用,可以作为一个起点为您自己的应用程序。 它将展示用户登录,注销,特定于用户的欢迎消息,访问控制web 应用程序的某些部分,plugglable 安全数据存储和集成。

我们将开始通过建立项目,包括构建工具和声明依赖性,以及配置 servlet的 web.xml 文件启动 web 应用程序和 Shiro 的环境。

一旦我们完成设置,我们将层的各个部分的功能,包括集成的安全数据存储,然后让用户登录,注销,访问控制。

## 项目设置

不必手动设置一个目录结构和初始基本文件,我们已为你这样做好了一个 git 存储库。

### 先fork本教程项目

在 github，浏览[ tutorial project ](https://github.com/lhazlewood/apache-shiro-tutorial-webapp) 项目,点击 Fork 按钮

### 复制教程存储库

现在您已经将项目 fork 在你的 GitHub 帐户,克隆它在本地机器上:

>$ git clone git@github.com:$YOUR_GITHUB_USERNAME/apache-shiro-tutorial-webapp.git    

(其中 $YOUR_GITHUB_USERNAME 是你的 GitHub 用户名)


用 cd 进入本地的项目目录查看项目结构:

>$ cd apache-shiro-tutorial-webapp

### 审查项目结构

当前项目结构为：

	apache-shiro-tutorial-webapp/
	  |-- src/
	  |  |-- main/
	  |    |-- resources/
	  |      |-- logback.xml
	  |    |-- webapp/
	  |      |-- WEB-INF/
	  |        |-- web.xml
	  |      |-- home.jsp
	  |      |-- include.jsp
	  |      |-- index.jsp
	  |-- .gitignore
	  |-- .travis.yml
	  |-- LICENSE
	  |-- README.md
	  |-- pom.xml

解释下：

* pom.xml :Maven 项目/构建文件。 它有Jetty 配置,这样你就可以马上运行 mvn jetty:run 测试您的 web 应用程序运行。
* README.md :一个简单的项目的自述文件
* LICENSE :该项目是 Apache 2.0 许可协议
* .travis.yml :一个 [Travis CI](https://travis-ci.org/) 配置文件以确保它总是在项目构建时，持续运行集成构建您的项目。
* .gitignore :一个 git 忽略文件,包含的后缀和目录是那些不应该纳入到版本控制中。
* src/main/resources/logback.xml:一个简单的 [Logback](http://logback.qos.ch/) 配置文件。 对于本教程,我们选择 [SLF4J](http://www.slf4j.org/) 的日志 API 和 Logback 日志的实现。 这可能很容易被熟悉 Log4J 或者 JUL 的人所接受。
* src/main/webapp/WEB-INF/web.xml :最初的 web.xml 文件,我们将配置很快使用Shiro。
* src/main/webapp/include.jsp :一个页面,其中包含常见的引入和声明,包括其他的JSP页面。 这让我们在一个地方来管理引入和声明。
* src/main/webapp/home.jsp :应用的简单的默认主页。 包括 include.jsp (如将其他人,因为我们很快就会看到)。
* src/main/webapp/index.jsp :默认站点索引页面-这仅仅是将请求转发给我们 home.jsp 主页。

### 运行

运行

>$ mvn jetty:run 

打开浏览器访问 [localhost:8080](http://localhost:8080/),页面将会输出 Hello, World! 

按`ctl-C` (或者 mac 中的 `cmd-C`) 来关闭应用

## Step 1:  启动 shiro

我们最初的 master 库 只是一个简单的通用的 web 应用程序,可以作为任何应用程序的模板。 让我们添加的最低限度,启动 Shiro web 应用程序。

执行以下git checkout 命令加载 Step1 分支:

>$ git checkout step1

检出的分支，有两点变化

1. 添加了一个 src/main/webapp/WEB-INF/shiro.ini 文件
2. src/main/webapp/WEB-INF/web.xml 改变了.

### 1a: 添加shiro.ini

可以配置 Shiro 在许多不同的方式在一个web应用程序,这取决于您所使用的web和/或MVC框架。 例如,您可以通过Spring配置Shiro,Guice,Tapestry,和许多更多。

为了简单起见,我们将启动一个 Shiro 环境使用Shiro的默认值(非常简单的) INI [配置](https://github.com/waylau/apache-shiro-1.2.x-reference/blob/master/I.%20Overview%20%E6%80%BB%E8%A7%88/4.%20Configuration%20%E9%85%8D%E7%BD%AE.md) 。

如果你签出 Step1 分支,您将看到这个新的的内容 src/main/webapp/WEB-INF/shiro.ini 文件(简短标题删除注释):
	
	[main]
	
	# Let's use some in-memory caching to reduce the number of runtime lookups against Stormpath.
	# A real application might want to use a more robust caching solution (e.g. ehcache or a
	# distributed cache).  When using such caches, be aware of your cache TTL settings: too high
	# a TTL and the cache won't reflect any potential changes in Stormpath fast enough.  Too low
	# and the cache could evict too often, reducing performance.
	cacheManager = org.apache.shiro.cache.MemoryConstrainedCacheManager
	securityManager.cacheManager = $cacheManager

 ini 包含一个简单的  [main] 和一些最小的配置:

* 它定义了一个新的 cacheManager (缓存管理器) 实例。 缓存是Shiro的体系结构的一个重要组成部分,它减少了不断往返通信各种数据存储。 这个示例使用 MemoryConstrainedCacheManager 这是唯一真正好的单个JVM 的应用程序。 如果您的应用程序部署在多个主机(如集群网络服务器),您需要使用集群缓存管理器实现。
* 在Shiro securityManager 它配置新  cacheManager (缓存管理器)  的实例  。 一个Shiro SecurityManager 实例总是存在的,所以它不需要显式地定义。

### 1b: 在web.xml中启动Shiro

当我们有一个 shiro.ini 配置,我们需要加载它,并开始一个新的 Shiro 环境和使 web 应用程序环境的实现。

我们所做的这一切通过添加现有的几件事到 src/main/webapp/WEB-INF/web.xml 文件:

	<listener>
	    <listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>
	</listener>
	
	<filter>
	    <filter-name>ShiroFilter</filter-name>
	    <filter-class>org.apache.shiro.web.servlet.ShiroFilter</filter-class>
	</filter>
	
	<filter-mapping>
	    <filter-name>ShiroFilter</filter-name>
	    <url-pattern>/*</url-pattern>
	    <dispatcher>REQUEST</dispatcher>
	    <dispatcher>FORWARD</dispatcher>
	    <dispatcher>INCLUDE</dispatcher>
	    <dispatcher>ERROR</dispatcher>
	</filter-mapping>

### 1c: 运行

当检出 step1 分支，运行

>$ mvn jetty:run

这一次,你会看到日志输出类似于以下,表明 Shiro 确实是运行在你的应用:

	16:04:19.807 [main] INFO  o.a.shiro.web.env.EnvironmentLoader - Starting Shiro environment initialization.
	16:04:19.904 [main] INFO  o.a.shiro.web.env.EnvironmentLoader - Shiro environment initialized in 95 ms.

按`ctl-C` (或者 mac 中的 `cmd-C`) 来关闭应用

## Step 2: 连接用户存储

检出 step2 分支

>$ git checkout step2

现在我们已经在 webapp 中集成和运行了 Shiro。 但是我们还没有真正告诉 Shiro 做任何事!

之前我们可以登录,注销,或执行基于角色或基于许可的访问控制,或任何其他安全相关的,我们需要用户!

我们需要配置 Shiro 访问 用户存储 的一些类型的,所以它可以查找用户执行登录尝试,或检查角色的安全决策,等等。有许多类型的用户存储任何应用程序可能需要访问:也许你在 MySQL 数据库中存储用户,也许在MongoDB,也许你的公司将用户帐户存储在 LDAP 或 Active Directory,也许你将它们存储在一个简单的文件,或其他专有数据存储。

Shiro 通过所谓的 Realm 来实现这些。 Shiro 的文档:

>Reamls 是 Shiro 和你的程序安全数据之间的“桥”或者“连接”，它用来实际和安全相关的数据如用户执行身份认证（登录）的帐号和授权（访问控制）进行交互，Shiro 从一个或多个程序配置的 Realm 中查找这些东西。 Realm 本质上是一个特定的安全 DAO：它封装与数据源连接的细节，得到Shiro 所需的相关的数据。在配置 Shiro 的时候，你必须指定至少一个Realm 来实现认证（authentication）和/或授权（authorization）。SecurityManager 可以配置多个复杂的 Realm，但是至少有一个是需要的。 Shiro 提供开箱即用的 Realms 来连接安全数据源（或叫地址）如 LDAP、JDBC、文件配置如INI和属性文件等，如果已有的Realm不能满足你的需求你也可以开发自己的Realm实现。 和其它内部组件一样，Shiro SecurityManager 管理如何使用 Realms获取 Subject 实例所代表的安全和身份信息。


因此,我们需要配置一个领域,那么我们可以访问用户。

### 2a: 设置Stormpath

本教程的精神是保持尽可能简单,不引入复杂性或范围干扰了我们的学习Shiro 的目的,我们将使用一个简单的 realm : Stormpath realm。

[Stormpath](http://stormpath.com/) 云托管用户管理服务,以完全自由发展为目的。 这意味着启用 Stormpath 之后,你已经准备好如下:

* 一个用户界面来管理应用程序,目录,帐户和组。 Shiro 不提供这个,所以通过本教程这将是方便和节省你的时间。
* 一个安全的存储用户密码的机制。 您的应用程序不需要担心密码安全、密码比较或存储密码。 虽然 Shiro 可以做这些事情,你必须配置它们,知道密码的概念。 Stormpath 自动化密码安全所以你(Shiro)不需要担心如何“步入正轨”。
* 过电子邮件帐户电子邮件验证和密码重置的安全工作流通。 Shiro不支持这个,因为它通常是特定于应用程序的。
* 主持/管理”always on“基础设施——你不需要设置任何或维持任何东西。

对于本教程,Stormpath 比建立一个独立的 RDBMS 服务器还有担心 SQL 或密码加密问题等等简单的多了。 所以我们将使用它。

当然,Stormpath 只是许多 Shiro 可以连接的后端数据存储之一。 我们将讨论更复杂的数据存储和特定于应用程序的配置之后。

#### 注册Stormpath 

1. 填写 [Stormpath 注册表单](https://api.stormpath.com/register), 它会发邮件确认
2. 确认邮件

#### 获取Stormpath API Key

Stormpath API 所需的关键是 Stormpath Realm 用来与 Stormpath 交流。 获得 Stormpath API Key:

1. 登录到 [Stormpath 管理控制台](https://api.stormpath.com/) 使用你的Stormpath 注册使用的电子邮件地址和密码
2. 在结果页面的右上角,访问 Settings > My Account 。
3. 在账户信息页面, Security Credentials, 点击  Create API Key 。

这将生成 API Key 并下载到你的电脑 apiKey.properties 文件。 如果你在文本编辑器中打开文件,您将看到类似于下面的:

	apiKey.id = 144JVZINOF5EBNCMG9EXAMPLE
	apiKey.secret = lWxOiKqKPNwJmSldbiSkEbkNjgh2uRSNAb+AEXAMPLE

将该文件保存在一个安全的位置,比如在一个隐藏您的主目录 .stormpath 目录中。 例如:

	$HOME/.stormpath/apiKey.properties
	
还改变文件权限,以确保只有你能读这个文件。 例如,在 *nix 操作系统:

	$ chmod go-rwx $HOME/.stormpath/apiKey.properties

#### 通过Stormpath注册web应用

我们必须通过 Stormpath 注册我们的 web 应用程序，用于用户的管理和身份验证。简单通过REST请求, POST 到一个新的 Stormpath 应用程序资源 URL:

	curl -X POST --user $YOUR_API_KEY_ID:$YOUR_API_KEY_SECRET \
	    -H "Accept: application/json" \
	    -H "Content-Type: application/json" \
	    -d '{
	           "name" : "Apache Shiro Tutorial Webapp"
	        }' \
	    'https://api.stormpath.com/v1/applications?createDirectory=true'

其中

* $YOUR_API_KEY_ID 是 apiKey.properties 文件中 apiKey.id 值
* YOUR_API_KEY_SECRET 是 apiKey.properties 文件中 apiKey.secret 值

那样就将会创建你的应用， 下面是一个响应示例：

	{
	    "href": "https://api.stormpath.com/v1/applications/aLoNGrAnDoMAppIdHeRe",
	    "name": "Apache Shiro Tutorial Webapp",
	    "description": null,
	    "status": "ENABLED",
	    "tenant": {
	        "href": "https://api.stormpath.com/v1/tenants/sOmELoNgRaNDoMIdHeRe"
	    },
	    "accounts": {
	        "href": "https://api.stormpath.com/v1/applications/aLoNGrAnDoMAppIdHeRe/accounts"
	    },
	    "groups": {
	        "href": "https://api.stormpath.com/v1/applications/aLoNGrAnDoMAppIdHeRe/groups"
	    },
	    "loginAttempts": {
	        "href": "https://api.stormpath.com/v1/applications/aLoNGrAnDoMAppIdHeR/loginAttempts"
	    },
	    "passwordResetTokens": {
	        "href": "https://api.stormpath.com/v1/applications/aLoNGrAnDoMAppIdHeRe/passwordResetTokens"
	    } 
	}

注意顶层的 href ,如 https://api.stormpath.com/v1/applications/$YOUR_APPLICATION_ID ，接下来我们将在 shiro.ini 配置使用这个 href 。

#### 创建应用测试用户账号

现在有了应用，我们要创建一个简单的测试用户

	curl -X POST --user $YOUR_API_KEY_ID:$YOUR_API_KEY_SECRET \
	    -H "Accept: application/json" \
	    -H "Content-Type: application/json" \
	    -d '{
	           "givenName": "Jean-Luc",
	           "surname": "Picard",
	           "username": "jlpicard",
	           "email": "capt@enterprise.com",
	           "password":"Changeme1"
	        }' \
	 "https://api.stormpath.com/v1/applications/$YOUR_APPLICATION_ID/accounts"

同样的，不要忘了修改 $YOUR_APPLICATION_ID

### 2b: 在shiro.ini中配置Realm 

一旦你选择至少一个用户连接存储,我们将需要配置一个 Realm 来表示数据存储,然后告诉 Shiro SecurityManager 。

如果你已经签出了 step2 分支,你会注意到的 shiro.ini 文件的 (主要) 现在部分有以下补充:
	
	# 配置 Realm 来连接用户存储.本教程只简单的指向 Stormpath
	# 花 5 分钟进行设置:
	stormpathClient = com.stormpath.shiro.client.ClientFactory
	stormpathClient.cacheManager = $cacheManager
	stormpathClient.apiKeyFileLocation = $HOME/.stormpath/apiKey.properties
	stormpathRealm = com.stormpath.shiro.realm.ApplicationRealm
	stormpathRealm.client = $stormpathClient
	
	# 找到这个 你在Stormpath 创建应用时的 URL :
	# Applications -> (choose application name) --> Details --> REST URL
	stormpathRealm.applicationRestUrl = https://api.stormpath.com/v1/applications/$STORMPATH_APPLICATION_ID
	stormpathRealm.groupRoleResolver.modeNames = name
	securityManager.realm = $stormpathRealm

做以下修改:

* 改变 $ HOME 占位符实际主目录路径,例如 /home/jsmith 所以最后 stormpathClient.apiKeyFileLocation 值是类似 /home/jsmith/.stormpath/apiKey.properties 。 这条路必须匹配的位置 apiKey.properties 你在 Step 2a.中从Stormpath下载一个文件。

* 改变 step2 中 Stormpath 返回来的 href 中  $STORMPATH_APPLICATION_ID  占位符中的实际ID值。 最后的 stormpathRealm.applicationRestUrl 值应该类似 https://api.stormpath.com/v1/applications/6hsPwoRZ0hCk6ToytVxi4D (当然有不同的应用程序ID)。

### 2c: 提交修改

替换 $ HOME 和 STORMPATH_APPLICATION_ID 值是特定于您的应用程序。 继续提交这些更改你的分支:

	$ git add . && git commit -m "updated app-specific placeholders" .

### 2d: 运行

运行

	$ mvn jetty:run

看到如下输出：

	16:08:25.466 [main] INFO  o.a.shiro.web.env.EnvironmentLoader - Starting Shiro environment initialization.
	16:08:26.201 [main] INFO  o.a.s.c.IniSecurityManagerFactory - Realms have been explicitly set on the SecurityManager instance - auto-setting of realms will not occur.
	16:08:26.201 [main] INFO  o.a.shiro.web.env.EnvironmentLoader - Shiro environment initialized in 731 ms.

按`ctl-C` (或者 mac 中的 `cmd-C`) 来关闭应用

## Step 3:  启用登录、登出

现在我们有了用户，可以简单的再 UI 里面增加、删除、禁用他们。现在我们要用到登录、登出功能了。

检出 step3 分支

	$ git checkout step3

这次检出的内容，增加了下面两项：

* 新增了一个登录界面 src/main/webapp/login.jsp 包含一个简单的登录框，让我们登入
* shiro.ini 文件更新了，从而能支持  web (URL) 特性

### Step 3a: 启用 Shiro 格式的登录登出

step3 分支中 src/main/webapp/WEB-INF/shiro.ini 文件包含了下面两个内容:
	
	[main]
	
	shiro.loginUrl = /login.jsp
	
	# Stuff we've configured here previously is omitted for brevity
	
	[urls]
	/login.jsp = authc
	/logout = logout

#### shiro.* 行

其中 shiro.loginUrl = /login.jsp 这个是设置 Shiro 的登录页面是  /login.jsp

设置 Shiro 的默认 authc filter (默认是 [FormAuthenticationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/FormAuthenticationFilter.html)) 识别这个登录页面. 这使得 FormAuthenticationFilter 能够正常工作

####  [urls]节

[urls] 是一个新的 web 特性的 INI 

这部分允许您使用一个非常简洁的名称/值对语法告诉 shiro 如何过滤请求任何给定的 UR L路径。 所有的路径 [url] 相对于web应用程序的[HttpServletRequest.getContextPath())( [http://java.sun.com/j2ee/sdk_1.3/techdocs/api/javax/servlet/http/HttpServletRequest.html getContextPath()](http://java.sun.com/j2ee/sdk_1.3/techdocs/api/javax/servlet/http/HttpServletRequest.html#getContextPath()) )的值。

这些名称/值对提供了一个非常强大的方式来过滤请求,允许各种各样的安全规则。 更深入的报道 url 和过滤器链超出了本文的范围,但请做 阅读更多[关于它](https://github.com/waylau/apache-shiro-1.2.x-reference/blob/master/III.%20Web%20Applications/10.%20Web.md), 如果你感兴趣。

现在,我们将讨论的两行补充道:

/login.jsp = authc
/logout = logout

* 第一行意思“每当 Shiro 看到 /login.jsp 的 url 请求,都将会在请求中启用 Shiro authc 过滤器”。
* 第二行意味着“每当 Shiro  看到 /logout 的 url 请求,都将会在请求中启用 Shiro 注销过滤器。”

这两个过滤器是有点特别的:他们实际上并不需要背后的东西。 而不是过滤,他们会完全处理请求。 这就意味着什么都不用为这些 url 请求 做什么——不用写控制器! Shiro 将处理这些请求。

### Step 3b: 添加登录界面

从 step3 启用登录和注销的支持,现在我们需要确保实际上有一个  /login.jsp 页面显示一个登录表单。

step3 分支包含一个新 src/main/webapp/login.jsp 页面。 这是一个简单的足够 bootstrap 风格的 HTML 登录页面,但有四个重要的事情: 

*  form 的 action 值是空字符串。当一个 form 不包含 action 值，则浏览器将会提交 form 请求到相同的 URL。这是可以的,因为我们将告诉 Shiro 该 URL 是什么，所以 Shiro 可以快速自动处理任何登录提交。 /login.jsp = authc 这句是告诉我们 authc 过滤器去处理提交。
* 有一个 username 表单字段。 Shiro authc 过滤器自动寻找 username 的请求参数 在 login 提交时，并且在登录期间使用那个值 (很多 Realms 允许 这个可以是  email 或者是 username ).
* 有一个 password 表单字段。 Shiro authc 过滤器自动寻找 password 的请求参数 在 login 提交时。
* 有一个 rememberMe 的 checkbox，当选中时，值表示 ‘是’  (如 true, t, 1, enabled, y, yes等等).

我们的 login.jsp 表单只使用默认值 username , password , rememberme 表单字段的名称。 名称是可配置的,如果你希望改变他们，看 [FormAuthenticationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/FormAuthenticationFilter.html) 的 javadoc 获取信息。

### Step 3c: 运行

	$ mvn jetty:run

### Step 3d: 尝试登录

浏览器 访问 [localhost:8080/login.jsp](localhost:8080/login.jsp) ,就能看到登录界面

输入 Step 2 中的 登录名称、密码。点击登录，成功则去到主页，失败则返回到登录界面

如果想登录后去到跟主页不同的任意界面，可以设置 authc.successUrl = /任意页面 ，即可

按`ctl-C` (或者 mac 中的 `cmd-C`) 来关闭应用

## Step 4: 用户特定的UI更改

如果想实现界面——当前登录用户是谁——这个功能的话，只需要简单用到 shiro  的 jsp 标签

检出 step4 分支

$ git checkout step4

在 home.jsp 中就几个新增内容:

* 当用户没有登录，则在登录页面显示 ‘Welcome Guest’.
* 当用户登入，则能看到用户名称 ‘Welcome username’ 并且有一个登出的链接t.
* 这个 UI 是非常常见的，操作按钮在屏幕右上方.

### Step 4a:添加Shiro标签库声明

home.jsp 修改包含下面内容:

	<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags" %>
	<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>

允许使用 Core (c:) 和 Shiro (shiro:) 两个 jsp 标签库

### Step 4b: 添加Shiro Guest 和 User 标签

修改 home.jsp 包含下面 <shiro:guest> 和 <shiro:user> 两个标签
	
	p>Hi <shiro:guest>Guest</shiro:guest><shiro:user>
	<%
	    //This should never be done in a normal page and should exist in a proper MVC controller of some sort, but for this
	    //tutorial, we'll just pull out Stormpath Account data from Shiro's PrincipalCollection to reference in the
	    //<c:out/> tag next:
	
	    request.setAttribute("account", org.apache.shiro.SecurityUtils.getSubject().getPrincipals().oneByType(java.util.Map.class));
	
	%>
	<c:out value="${account.givenName}"/></shiro:user>!
	    ( <shiro:user><a href="<c:url value="/logout"/>">Log out</a></shiro:user>
	    <shiro:guest><a href="<c:url value="/login.jsp"/>">Log in</a></shiro:guest> )
	</p>

看起来有点难懂：

* <shiro:guest>: 这个标签只显示 Shiro Subject 在应用里面是 ‘guest’时的内容. Shiro 定义了 guest 是任何 没有登录系统，或者没有被前次登录记住的 Subject (使用 Shiro ‘remember me’ 功能).
* <shiro:user>: 这个标签只显示 Shiro Subject 在应用里面是 ‘user’时的内容. Shiro 定义了 user 是任何登录系统，或者被前次登录记住的 Subject (使用 Shiro ‘remember me’ 功能).

上面的代码片段将呈现以下,如果  Subject 是 guest:

Hi Guest! (Log in)

其中“Log in”是一个超链接到 /login.jsp

它将呈现以下，如果  Subject 是 user:

Hi jsmith! (Log out)

假设 jsmith 的帐户的用户名登录。 “Log out”是一个超链接到 ‘/logout’  注销 过滤器。

正如您可以看到的,你可以关掉或整个页面上部分,特性和 UI 组件。 除了 <shiro:guest> 和 <shiro:user>, Shiro 支持许多其他有用的 JSP 标签 ,您可以使用自定义 UI 知道当前基于各种各样的 Subject。

### Step 4c: 运行

运行:

	$ mvn jetty:run

访问 [localhost:8080]( localhost:8080 )用 guest 身份, 而后登录。成功登录后，看页面显示，知道你先是一个用户了！

按`ctl-C` (或者 mac 中的 `cmd-C`) 来关闭应用

## Step 5: 允许访问授权的用户

虽然您可以更改页面内容基于 Subject 的状态,很多时候你会想要限制你的整个部分应用基于是否有人 证明 他们的身份(身份验证)在他们当前的与 web 应用程序的交互。

这是特别重要的,如果一个用户只能部分的应用显示敏感信息,如帐单细节或控制其他用户的能力。

执行以下git checkout 命令 加载 step5 分支:

	$ git checkout step5

Step 5 包含下面三点变化:

* 我们添加了新的部分 (url 路径) ，想限制只有通过身份验证的用户。
* 改变了 shiro.ini 告诉 shiro 只允许经过身份验证的用户应用 web 应用程序的一部分
* 改变了主页，输出内容是基于 当前 Subject 是否被验证.

### Step 5a: 添加新的受限部分

新的 src/main/webapp/account 目录添加进来了。这个目录及下面的目录，只有登录用户可见。 src/main/webapp/account/index.jsp 文件只是一个占位符一个模拟 “home account” 页面。

### Step 5b: 配置shiro.ini

shiro.ini 修改了：

	/account/** = authc

这 过滤器链定义的意思是“任何请求访问  /account  (或其子路径) 必须经过身份验证”。

但是如果有人试图访问路径或它的任何孩子路径?

但是你还记得在 step3 中添加以下行来 (主要) 部分:

shiro.loginUrl = /login.jsp

这条自动配置 authc 与我们的 webapp 的登录 URL 过滤器。

基于这一配置, authc 过滤器已经足够聪明知道如果当前 Subject  访问 /account 时还没有经过身份验证 ,它将自动重定向到 /login.jsp 页面。 成功登录后,它会自动将用户重定向回他们试图访问的页面( /account )。 方便!

### Step 5c: 更新主页

修改 /home.jsp 页面让用户知道他们可以访问新网站的一部分。添加欢迎以下信息:

	<shiro:authenticated><p>Visit your <a href="<c:url value="/account"/>">account page</a>.</p></shiro:authenticated>
	<shiro:notAuthenticated><p>If you want to access the authenticated-only <a href="<c:url value="/account"/>">account page</a>,
	    you will need to log-in first.</p></shiro:notAuthenticated>

<shiro:authenticated> 标签只会显示内容如果当前  Subject 已经登录在当前会话(身份验证)。 这是为什么 Subject 知道他们可以访问新网站的一部分。

<shiro:notAuthenticated> 标签只会显示内容如果当前  Subject 在当前还没有经过身份验证的会话。

但你注意到 notAuthenticated 还有一个URL的内容  /account ? 没关系,我们 authc 过滤器将处理 登录并且重定向 的流程。如上所述。

试一试!

### Step 5d: 运行应用

	$ mvn jetty:run

访问 [localhost:8080]( localhost:8080 ), 点击 /account 链接重定向强制你登录。成功登录后，看页面显示，知道你已经登录了！尝试登录、登出。

按`ctl-C` (或者 mac 中的 `cmd-C`) 来关闭应用

## Step 6: 基于角色的访问控制

除了控制访问身份验证的基础上,它通常是一个要求限制访问应用程序的某些部分基于角色分配给当前 Subject

检出 step6

	$ git checkout step6

### Step 6a: 添加角色

为了实现基于角色的访问控制,我们需要角色存在。

本教程是最快的方式来填充一些内容到 Stormpath。

要做到这一点,登录到用户界面和导航如下:

**Directories > Apache Shiro Tutorial Webapp Directory > Groups**

添加下面组

* Captains
* Officers
* Enlisted

(这个是 [Star-Trek《星际迷航》](http://www.baidu.com/link?url=BNAZzFmwj-ekabSIip0OXJp8rx-qLv3KbGTDKvfxZeBdjWTKsB9Ig2yeu4FJgiNh) 里面的角色)

一旦你创建了组,添加 Jean-Luc Picard 到 Captains 和 Officers 分组。 您可能想要创建一些特别账户,并将它们添加到您喜欢的任何组。 确保一些帐户不重叠组,这样你就可以看到变化基于单独的组分配到用户帐户。

### Step 6b: RBAC 标签
	
修改 home.jsp 内容

	<h2>Roles</h2> section of the home page:
	
	<h2>Roles</h2>
	
	<p>Here are the roles you have and don't have. Log out and log back in under different user
	    accounts to see different roles.</p>
	
	<h3>Roles you have:</h3>
	
	<p>
	    <shiro:hasRole name="Captains">Captains<br/></shiro:hasRole>
	    <shiro:hasRole name="Officers">Bad Guys<br/></shiro:hasRole>
	    <shiro:hasRole name="Enlisted">Enlisted<br/></shiro:hasRole>
	</p>
	
	<h3>Roles you DON'T have:</h3>
	
	<p>
	    <shiro:lacksRole name="Captains">Captains<br/></shiro:lacksRole>
	    <shiro:lacksRole name="Officers">Officers<br/></shiro:lacksRole>
	    <shiro:lacksRole name="Enlisted">Enlisted<br/></shiro:lacksRole>
	</p>

<shiro:hasRole> 标签只会显示内容如果当前 Subject 分配指定的角色。
<shiro:lacksRole> 如果当前标签只会显示内容 Subject 没有 被分配指定的角色。

### Step 6c: RBAC 过滤器链

留给读者的练习(不是定义步骤)是创建一个新的部分的网站和限制的URL访问部分网站基于角色分配给当前用户。

### Step 6d: 运行

	$ mvn jetty:run

访问 [localhost:8080]( localhost:8080 ),  和不同的用户帐户登录看主页的分配不同的角色， 角色部分内容改变!

按`ctl-C` (或者 mac 中的 `cmd-C`) 来关闭应用

## Step 7: 基于权限的访问控制

基于角色的访问控制是好的,但它存在一个主要问题:你不能在运行时添加或删除角色。 角色名字在角色检查是硬编码的,所以如果你改变了角色名称或角色配置,或添加或删除角色,你必须回去和改变你的代码!

正因为如此,Shiro 有强大的特点:内置的支持 权限 。 Shiro 的 权限是一个原始语句的功能,例如 ‘open a door’ ‘create a blog entry’, ‘delete the jsmith user’等权限反映了应用的原始功能,所以当你改变应用的功能时你只需要更改权限的检查——而不是改变你的角色或用户模型。

为了证明这一点,我们将创建一些权限,并将它们分配给一个用户,然后定制我们的基于用户的授权(权限) web UI 

## Step 7a: 添加权限

Shiro Realms 是只读的组件:每个数据存储模型的角色,组织、权限、账号,以及它们之间的关系不同,所以 Shiro 没有“写”API来修改这些资源。 修改底层模型对象,你只是通过任何 API 直接修改你想要的。

是这样的,因为我们使用 Stormpath 在这个示例应用程序中,我们将权限分配给一个帐户和组在 Stormpath API-specific 方式。

让我们执行 cURL 请求添加一些权限给我们以前创建的  Jean-Luc Picard 帐户。 使用该帐户的 href  URL,我们会请求 apacheShiroPermissions 账户通过 自定义数据 :

	curl -X POST --user $YOUR_API_KEY_ID:$YOUR_API_KEY_SECRET \
	    -H "Accept: application/json" \
	    -H "Content-Type: application/json" \
	    -d '{
	            "apacheShiroPermissions": [
	                "ship:NCC-1701-D:command",
	                "user:jlpicard:edit"
	            ]
	        }' \
	"https://api.stormpath.com/v1/accounts/$JLPICARD_ACCOUNT_ID/customData"

$JLPICARD_ACCOUNT_ID 匹配 创建Jean-Luc Picard 时 的 uid 

添加两个权限到 Stormpath 账户:

* ship:NCC-1701-D:command
* user:jlpicard:edit

第一句含义是，你有权限  ‘command’ 编号是 ‘NCC-1701-D’ 的 ‘ship’。
第二句是有权限 edit 账户是 jlpicard 的 user

想知道 Stormpath 是如何存储权限的，请参阅[ Shiro Stormpath plugin documentation.](https://github.com/stormpath/stormpath-shiro/wiki#permissions)

### Step 7b: Permission 标签

就像我们对角色检查 JSP 标记,并行标记存在权限检查。 我们更新 /home.jsp 页面,让用户知道如果他们允许做一些基于权限分配给他们。 这些消息被添加在一个新的 <h2>Permissions</h2> 部分的主页:
```html
	<h2>Permissions</h2>
	<ul>
	    <li>You may <shiro:lacksPermission name="ship:NCC-1701-D:command"><b>NOT</b> </shiro:lacksPermission> command the <code>NCC-1701-D</code> Starship!</li>
	    <li>You may <shiro:lacksPermission name="user:${account.username}:edit"><b>NOT</b> </shiro:lacksPermission> edit the ${account.username} user!</li>
	</ul>
```

当你访问主页，看到如下输出

You may NOT command the NCC-1701-D Starship!
You may NOT edit the user!

当用 Jean-Luc Picard 账户登录 ,您将看到这个:

You may command the NCC-1701-D Starship!
You may edit the user!


使用 shiro [WildcardPermission](https://github.com/waylau/apache-shiro-1.2.x-reference/blob/master/II.%20Core%20%E6%A0%B8%E5%BF%83/6.1.%20Permissions%20%E6%9D%83%E9%99%90.md) 中的语法 

您可以看到,Shiro解决,通过身份验证的用户权限,和输出在一个适当的方式呈现。

你也可以使用 <shiro:hasPermission> 标记为肯定的权限检查。

最后,我们将呼吁关注一个非常强大的功能和权限检查。 你看到第二个使用权限检查，如何运行时生成权限值?

<shiro:lacksPermission name="user:${account.username}:edit"> ...

${account.username} 在运行时解释,形成最终的值 user:aUsername:edit,然后最后一个字符串值是用于权限检查。

这是极强大的:你可以执行权限检查基于当前用户是谁, 目前正在与什么交互。 这些基于运行时 runtime实例级权限检查基本技术发展高度可定制的、安全的应用程序。

### Step 7c: 运行应用

	$ mvn jetty:run


访问 [localhost:8080]( localhost:8080 ) 并用 Jean-Luc Picard 账户(和其他账户)登录、登出,看到基于权限分配的页面输出变化（或者木有）

按`ctl-C` (或者 mac 中的 `cmd-C`) 来关闭应用


## 总结

我们希望你发现这个入门教程 Shiro-enabled webapps有用。 在未来版本的教程中,我们将介绍:

插入不同的用户数据存储,如 RDBMS 或 NoSQL 数据存储。

### 修复和 pull 请求

请发送任何修复勘误表作为 [GitHub pull 请求](https://help.github.com/articles/creating-a-pull-request) 到 https://github.com/lhazlewood/apache-shiro-tutorial-webapp
 存储库。 我们很感激! ! !


*译者注：本文参考：[http://shiro.apache.org/webapp-tutorial.html](http://shiro.apache.org/webapp-tutorial.html)。如果对本中文翻译有疑议的或发现勘误欢迎指正，[点此](https://github.com/waylau/apache-shiro-1.2.x-reference/issues)提问。*
