# 10. Web

## 配置

将 Shiro 集成到任何 Web 应用程序的最简单的方法是在 web.xml 中配置 ContextListener 和 Filter，理解如何读取 Shiro 的 INI 配置文件。大部分的 INI 配置格式定义在 [Configuration](../I. Overview 总览/4. Configuration 配置.md) 页的 INI Sections 节，但我在这里我们将介绍一些额外的Web 的特定部分。

*Using Spring?*

*Spring 框架用户将不执行此设置。如果你使用 Spring，你将要阅读关于[Spring 特定的 Web 配置](../V. Integration 整合/15. Spring Framework.md)*

### Web.xml

### Shiro 1.2 and later

在Shiro 1.2 及以后版本，标准的 Web 应用程序通过添加下面的 XML 块到 web.xml 来初始化 Shiro：
	
	<listener>
	    <listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>
	</listener>
	
	...
	
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

这假设一个 Shiro INI  [Configuration](../I. Overview 总览/4. Configuration 配置.md) 文件在以下两个位置任意一个，并使用最先发现的那个：

1. /WEB-INF/shiro.ini
2. 在classpath 根目录下shiro.ini 文件

下面是上述配置所做的事情：

* EnvironmentLoaderListener 初始化一个Shiro WebEnvironment 实例（其中包含 Shiro 需要的一切操作，包括 SecurityManager ），使得它在 ServletContext 中能够被访问。如果你需要在任何时候获得WebEnvironment 实例，你可以调用WebUtils.getRequiredWebEnvironment（ServletContext）。
* ShiroFilter 将使用此 WebEnvironment 对任何过滤的请求执行所有必要的安全操作。
* 最后，filter-mapping 的定义确保了所有的请求被 ShiroFilter 过滤，建议大多数 Web 应用程序使用以确保任何请求是安全的。

*ShiroFilter filter-mapping*

*它通常可取的做法是在任何其他 filter-mapping 声明之前定义 ShiroFilter filter-mapping，以确保 Shiro 也能在那些过滤器
下工作的很好。*

#### 自定义 WebEnvironment Class

默认情况下，EnvironmentLoaderListener 将创建一个IniWebEnvironment 实例，呈现 Shiro 基于INI 文件的[Configuration](../I. Overview 总览/4. Configuration 配置.md)。如果你愿意，你可以在 web.xml 中指定一个自定义的 ServletContext context-param：

	<context-param>
		<param-name>shiroEnvironmentClass</param-name>
		<param-value>com.foo.bar.shiro.MyWebEnvironment</param-value>
	</context-param>

这允许你自定义一个如何解析和代表 WebEnvironment 实例的配置格式。你可以为自定义的行为对现有的 IniWebEnvironment 创建子类，或完全支持不同的配置格式。例如，如果有人想在XML 中配置Shiro 而不是在INI 中，他们可以创建一个基于XML 的实现，如com.foo.bar.shiro.XmlWebEnviroment。

#### 自定义  Configuration Locations

IniWebEnvironment 将会去读取和加载 INI 配置文件。默认情况下，这个类会自动地在下面两个位置寻找 Shiro.ini 配置（按顺序）。

1. /WEB-INF/shiro.ini
2. classpath:shiro.ini

它将使用最先发现的那个。
然而，如果你想把你的配置放在另一位置，你可以在 web.xml 中用contex-param 指定该位置。

	<context-param>
	    <param-name>shiroConfigLocations</param-name>
	    <param-value>YOUR_RESOURCE_LOCATION_HERE</param-value>
	</context-param>

默认情况下，在 ServletContext.getResource 方法定义的规则下，param-value 是可以被解析的。例如，
/WEB-INF/some/path/shiro.ini。

但你也可以指定具体的文件系统，如classpath 或URL 位置，通过使用Shiro 支持的合适的[ResourceUtils 类](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/io/ResourceUtils.html)，例如：

* file://home/foobar/myapp/shiro.ini
* classpath:com/foo/bar/shiro.ini
* url:http://confighost.mycompany.com/myapp/shiro.ini

### Shiro 1.1 and earlier

在Web 应用程序中使用Shiro 1.1 或更早版本的最简单的方法是定义IniShiroFilter 并指定一个filter-mapping:
	
	<filter>
	    <filter-name>ShiroFilter</filter-name>
	    <filter-class>org.apache.shiro.web.servlet.IniShiroFilter</filter-class>
	</filter>
	
	...
	
	<!-- Make sure any request you want accessible to Shiro is filtered. /* catches all -->
	<!-- requests.  Usually this filter mapping is defined first (before all others) to -->
	<!-- ensure that Shiro works in subsequent filters in the filter chain:             -->
	<filter-mapping>
	    <filter-name>ShiroFilter</filter-name>
	    <url-pattern>/*</url-pattern>
	    <dispatcher>REQUEST</dispatcher> 
	    <dispatcher>FORWARD</dispatcher> 
	    <dispatcher>INCLUDE</dispatcher> 
	    <dispatcher>ERROR</dispatcher>
	</filter-mapping>


该定义期望你的INI 配置是一个在 classpath 根目录的Shiro.ini 文件（如：classpath:shiro.ini）。

#### 自定义 Path
如果你不想将你的 INI 配置放在 /WEB-INF/shiro.ini 或classpath:shiro.ini，你可以指定一个自定义的资源位置，如果必
要的话。添加一个 configPath 的 init-param，并指定资源位置。

	<filter>
	    <filter-name>ShiroFilter</filter-name>
	    <filter-class>org.apache.shiro.web.servlet.IniShiroFilter</filter-class>
	    <init-param>
	        <param-name>configPath</param-name>
	        <param-value>/WEB-INF/anotherFile.ini</param-value>
	    </init-param>
	</filter>

	...

不合格的（不完整的组合或'non-prefixed'）configPath 值被假定为ServletContext 的资源路径，通过 ServletContext.[getResource](http://download.oracle.com/javaee/6/api/javax/servlet/ServletContext.html#getResource(java.lang.String)) 方法所定义的规则来解析。

*ServletContext resource paths - Shiro 1.2+*

*ServletContext 资源路径是一个在 Shiro 1.2 即将推出的功能。在 1.2 被发布后，所有的configPath 定义必须指定一个 classpath:，file:或url:前缀。*

通过分别地使用 classpath:，url:，或file:前缀来指明classpath，url，或 filesystem 位置，你也可以指定其他非 ServletContext 资源位置。例如：

	...
	<init-param>
	    <param-name>configPath</param-name>
	    <param-value>url:http://configHost/myApp/shiro.ini</param-value>
	</init-param>
	...

#### 内嵌 Config

最后，也可以将你的 INI 配置嵌入到 web.xml 中而不使用一个独立的 INI 文件。你可以通过使用 init-param 做到这点，而不是 configPath：

	<filter>
	    <filter-name>ShiroFilter</filter-name>
	    <filter-class>org.apache.shiro.web.servlet.IniShiroFilter</filter-class>
	    <init-param><param-name>config</param-name><param-value>
	
	    # INI Config Here
	
	    </param-value></init-param>
	</filter>
	...

内嵌配置对于小型的或简单的应用程序通常是很好用的，但是由于以下原因一般把它具体化到一个专用的 Shiro.ini 文件中：

* 你可能编辑了许多安全配置，不希望为 web.xml 添加版本控制。
* 你可能想从余下的 web.xml 配置中分离安全配置。
* 你的安全配置可能变得很大，你想保持 web.xml 的苗条并易于阅读。
* 你有个负责的编译系统，相同的 shiro 配置可能需要在多个地方被引用。

这取决于你——使用什么使你的项目更有意义。

###  Web INI配置

除了在主要的 [Configuration](https://github.com/waylau/apache-shiro-1.2.x-reference/blob/master/I.%20Overview%20%E6%80%BB%E8%A7%88/4.%20Configuration%20%E9%85%8D%E7%BD%AE.md) 章节描述的标准的[main]，[user]和[roles]项外，你可以在shiro.ini 文件中指定具有 web
特性的[urls]项：
	
	# [main], [users] and [roles] above here
	...
	[urls]
	...

[urls]项允许你做一些在我们已经见过的任何 Web 框架都不存在的东西：在你的应用程序中定义自适应过滤器链来匹配URL 路径！

这将更为灵活，功能更为强大，比你通常在 web.xml 中定义的过滤器链更为简洁：即使你从未使用任何 Shiro 提供的其他功能并仅仅使用了这个，但它即使是单独使用也是值得的。

[urls]

在urls 项的每一行格式如下：

URL_Ant_Path_Expression = Path_Specific_Filter_Chain

举例：

	...
	[urls]
	
	/index.html = anon
	/user/create = anon
	/user/** = authc
	/admin/** = authc, roles[administrator]
	/rest/** = authc, rest
	/remoting/rpc/** = authc, perms["remote:invoke"]

接下来我们将讨论这些行的具体含义。

#### URL 路径表达式

等号左边是一个与Web 应用程序上下文根目录相关的 [Ant](http://ant.apache.org/) 风格的路径表达式。

例如，假设你有如下的[urls]行：

	/account/** = ssl, authc

此行表明，“任何对我应用程序的/accout 或任何它的子路径（/account/foo, account/bar/baz，等等）的请求都将触发'ssl, authc'过滤器链”。我们将在下面讨论过滤器链。

请注意，所有的路径表达式都是相对于你的应用程序的上下文根目录而言的。这意味着如果某一天你在某个位置部署了你的应用程序，如 www.somehost.com/myapp ，然后又将它部署到了 www.anotherhost.com （没有'myapp'子目录），这样的匹配模式仍将继续工作。所有的路径都是相对于 [HttpServletRequest.getContextPath()](http://java.sun.com/j2ee/sdk_1.3/techdocs/api/javax/servlet/http/HttpServletRequest.html#getContextPath()) 的值来的。

*Order Matters! 秩序*

*URL 路径表达式按事先定义好的顺序判断传入的请求，并遵循 FIRST MATCH WINS 这一原则。例如，让我们假设有如下链的定义：*

	/account/** = ssl, authc
	/account/signup = anon

*如果传入的请求旨在访问 /account/signup/index.html（所有 'anon'ymous 用户都能访问），那么它将永不会被处理！原因是因为/account/** 的模式第一个匹配了传入的请求，“短路”了其余的定义。
始终记住基于FIRST MATCH WINS 的原则定义你的过滤器链！*

#### 过滤器链定义

等号右边是逗号隔开的过滤器列表，用来执行匹配该路径的请求。它必须符合以下格式：

filter1[optional_config1], filter2[optional_config2], ..., filterN[optional_configN]

并且：

* filterN 是一个定义在[main]项中的 filter bean 的名字
* [optional_configN]是一个可选的括号内的对特定的路径，特定的过滤器有特定含义的字符串（每个过滤器，每个路径的具体配置！）。若果该过滤器对该 URL 路径并不需要特定的配置，你可以忽略括号，于是 filteNr[] 就变成了 filterN.

因为过滤器标志符定义了链（又名列表），所以请记住顺序问题！请按顺序定义好你的逗号分隔的列表，这样请求就能够流通这个链。

最后，每个过滤器按照它期望的方式自由的处理请求，即使不具备必要的条件（例如，执行一个重定向，响应一个 HTTP 错误代码，直接渲染等）。否则，它有可能允许该请求继续通过这个过滤器链到达最终的视图。

*确保能够对路径的具体配置作出反应，即[optional_configN]一个过滤器标志符的一部分，是一个对所有Shiro 过滤器
适用的独有的功能。*

*你也可以这样做，如果你想创建你自己的 javax.servlet.Filter 的实现的话，确保你的过滤器子类
[org.apache.shiro.web.filter.PathMatchingFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/PathMatchingFilter.html)*

#### 可用的过滤器

在过滤器链中能够使用的过滤器“池”被定义在[main]项。在[main]项中指派给它们的名字就是在过滤器链定义中使
用的名字。例如：

	[main]
	...
	myFilter = com.company.web.some.FilterImplementation
	myFilter.property1 = value1
	...
	
	[urls]
	...
	/some/path/** = myFilter

## 默认过滤器

当运行一个 Web 应用程序时，Shiro 将会创建一些有用的默认 Filter 实例，并自动地在[main]项中将它们置为可用。

你可以在 main 中配置它们，当作在你的链的定义中你是否有任何其他的bean 和 reference。例如：

	[main]
	...
	# Notice how we didn't define the class for the FormAuthenticationFilter ('authc') - it is instantiated and available already:
	authc.loginUrl = /login.jsp
	...
	
	[urls]
	...
	# make sure the end-user is authenticated.  If not, redirect to the 'authc.loginUrl' above,
	# and after successful authentication, redirect them back to the original account page they
	# were trying to view:
	/account/** = authc
	...
	
自动地可用的默认的Filter 实例是被 [DefaultFilter 枚举](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/mgt/DefaultFilter.html)定义的，枚举的名称字段是可供配置的名称。它们是：



<table class="confluenceTable"><tbody><tr><th colspan="1" rowspan="1" class="confluenceTh"> Filter Name </th><th colspan="1" rowspan="1" class="confluenceTh"> Class </th><iframe id="tmp_downloadhelper_iframe" style="display: none;"></iframe></tr><tr><td colspan="1" rowspan="1" class="confluenceTd"> anon </td><td colspan="1" rowspan="1" class="confluenceTd"> <a class="external-link" href="http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/AnonymousFilter.html">org.apache.shiro.web.filter.authc.AnonymousFilter</a> </td></tr><tr><td colspan="1" rowspan="1" class="confluenceTd"> authc </td><td colspan="1" rowspan="1" class="confluenceTd"> <a class="external-link" href="http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/FormAuthenticationFilter.html">org.apache.shiro.web.filter.authc.FormAuthenticationFilter</a> </td></tr><tr><td colspan="1" rowspan="1" class="confluenceTd"> authcBasic </td><td colspan="1" rowspan="1" class="confluenceTd"> <a class="external-link" href="http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/BasicHttpAuthenticationFilter.html">org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter</a> </td></tr><tr><td colspan="1" rowspan="1" class="confluenceTd"> logout </td><td colspan="1" rowspan="1" class="confluenceTd"> <a class="external-link" href="http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/LogoutFilter.html">org.apache.shiro.web.filter.authc.LogoutFilter</a> </td></tr><tr><td colspan="1" rowspan="1" class="confluenceTd"> noSessionCreation </td><td colspan="1" rowspan="1" class="confluenceTd"> <a class="external-link" href="http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/session/NoSessionCreationFilter.html">org.apache.shiro.web.filter.session.NoSessionCreationFilter</a> </td></tr><tr><td colspan="1" rowspan="1" class="confluenceTd"> perms </td><td colspan="1" rowspan="1" class="confluenceTd"> <a class="external-link" href="http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/PermissionsAuthorizationFilter.html">org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter</a> </td></tr><tr><td colspan="1" rowspan="1" class="confluenceTd"> port </td><td colspan="1" rowspan="1" class="confluenceTd"> <a class="external-link" href="http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/PortFilter.html">org.apache.shiro.web.filter.authz.PortFilter</a> </td></tr><tr><td colspan="1" rowspan="1" class="confluenceTd"> rest </td><td colspan="1" rowspan="1" class="confluenceTd"> <a class="external-link" href="http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/HttpMethodPermissionFilter.html">org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter</a> </td></tr><tr><td colspan="1" rowspan="1" class="confluenceTd"> roles </td><td colspan="1" rowspan="1" class="confluenceTd"> <a class="external-link" href="http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/RolesAuthorizationFilter.html">org.apache.shiro.web.filter.authz.RolesAuthorizationFilter</a> </td></tr><tr><td colspan="1" rowspan="1" class="confluenceTd"> ssl </td><td colspan="1" rowspan="1" class="confluenceTd"> <a class="external-link" href="http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/SslFilter.html">org.apache.shiro.web.filter.authz.SslFilter</a> </td></tr><tr><td colspan="1" rowspan="1" class="confluenceTd"> user </td><td colspan="1" rowspan="1" class="confluenceTd"> <a class="external-link" href="http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/UserFilter.html">org.apache.shiro.web.filter.authc.UserFilter</a> </td></tr></tbody></table>


## 启动、禁用过滤器

由于这是与任何过滤器链定义机制（web.xml，Shiro 的INI 等）相关的例子，你通过在过滤器链中包含它来启用过滤器，通过在过滤器链中移除它来禁用过滤器。

但在Shiro 1.2 中新增的一个新功能是不通过从过滤器链中移除过滤器来启用或禁用过滤器。如果启用（默认设置），那么请求将如预期一样过滤。如果禁用，那么该过滤器将允许请求立即通过到FilterChain 的下一个元素。你可以基于一般配置属性触发过滤器的启用状态，或者你甚至可以在每一个请求的基础上触发。

这是一个强大的概念，因为基于特定需求启用或禁用一个过滤器比更改静态过滤器链（这是永久的且固定的）定义更为方便。

Shiro 通过它的 [OncePerRequestFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/servlet/OncePerRequestFilter.html) 抽象父类来完成这点。所有 Shiro 的不受规范限制的过滤器实现子类实现这一点，因此不需要从过滤器链移除它们实现启用或禁用。如果你需要实现此功能，你可以为自己的过滤器实现继承这个类的子类。

[SHIRO-224](https://issues.apache.org/jira/browse/SHIRO-224) 将有望为任何过滤器使用这项功能，不仅仅只是那些OncePerRequestFilter 的子类。如果这对你很重要，请为这个issue 投票。

###General Enabling/Disabling

[OncePerRequestFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/servlet/OncePerRequestFilter.html)（及其所有子类）支持 Enabling/Disabling 所有请求及 per-request 基础。
一般为所有的请求启用或禁用一个过滤器是通过设置其 enabled 属性为true 或 false。默认的设置是true 由于大多
数过滤器本质上是需要执行的，如果他们被配置在一个过滤器链中。
例如，在shiro.ini 中：
	
	[main]
	...
	# configure Shiro's default 'ssl' filter to be disabled while testing:
	ssl.enabled = false
	
	[urls]
	...
	/some/path = ssl, authc
	/another/path = ssl, roles[admin]
	...

该例表明，许多潜在的URL 路径都需要请求必须通过SSL 连接保证。在开发中设置SSL 是令人沮丧且费时的。在开发时，你可以禁用ssl 过滤器。当部署产品时，你可以启用它通过一个配置属性——这比手动更改所有URL 路径或
维护两个Shiro 配置要容易得多。

### Request-specific Enabling/Disabling

OncePerRequestFilter 实际上决定过滤器启用或禁用是基于它的isEnabled(request, response) 方法。该方法默认返回enabled 属性的值，该属性通常是用来enabling/disabling 上面提及的所有请求。如果你想启用或禁用一个基于特定标准的请求的过滤器，你可以通过覆盖OncePerRequestFilter 的 isEnabled(request, response) 方法来
执行更多特定的检查。

### Path-specific Enabling/Disabling

Shiro 的 [PathMatchingFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/PathMatchingFilter.html)（一个 OncePerRequestFilter 的子类）能够对基于被过滤的特定路径的配置作出反应。这
意味着你可以启用或禁用一个过滤器基于路径和特定路径配置，除了传入的request 和 response。
如果你需要能够对匹配的路径和特定路径配置作出反应来判断一个过滤器是否是启用的或禁用的，而不是通过覆盖
OncePerRequestFilter 的 isEnabled(request, reponse)方法，你应该是覆盖  PathMatchingFilter 的 isEnabled(request,response,path,pathConfig) 方法。

## Session管理

### Servlet容器会话

在 web 环境中,Shiro 的默认的会话管理器 [SessionManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/SessionManager.html) 实现是 [ServletContainerSessionManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/session/mgt/ServletContainerSessionManager.html) 。 这个非常简单的实现代表所有会话管理职责(包括会话集群如果 servlet 容器支持)运行 servlet  容器。 它本质上是一个桥 Shiro 会话 API 的 servlet 容器,没有别的。

使用这个默认的一个好处是,使用现有的 servlet 容器的应用程序会话配置(超时,任何特定容器集群机制等)将正常工作。

这个默认的缺点是,你与 servlet 容器的特定会话行为。 举个例子,如果你想集群会话,但你使用 Jetty 在生产、测试和 Tomcat 容器配置(或代码)将不具有可移植性。

#### Servlet 容器会话超时

如果使用默认 servlet 容器支持,您配置将在您的web应用程序的会话超时 web . xml 文件。 例如:

	<session-config>
	  <!-- web.xml expects the session timeout in minutes: -->
	  <session-timeout>30</session-timeout>
	</session-config>

### 本地会话

如果你想让你的会话配置设置和集群便携式在 servlet 容器(比如 Jetty 在测试中,但 Tomcat 或 JBoss 在生产环境中),或你想控制特定的会话/聚类特性,您可以启用 Shiro 的原生会话管理。

“Native”这个词在这里意味着 Shiro 的企业将被用来支持所有会话管理实现 Subject 和 httpservletrequest 会话和完全绕过 servlet容器。 但放心,Shiro 实现 Servlet 规范的相关部分直接所以任何现有的 web/http  相关代码符合预期和不需要“知道”,Shiro 透明地管理会话。

#### DefaultWebSessionManager

启用本地会话管理您的web应用程序,您需要配置一个本地 web-capable 会话管理器覆盖默认的基于 servlet 容器。 你可以通过配置的一个实例 [DefaultWebSessionManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/session/mgt/DefaultWebSessionManager.html) Shiro 的 SecurityManager 。 例如,在 shiro.ini :
	
	[main]
	...
	sessionManager = org.apache.shiro.web.session.mgt.DefaultWebSessionManager
	# configure properties (like session timeout) here if desired
	
	# Use the configured native session manager:
	securityManager.sessionManager = $sessionManager

一旦宣布,您可以配置 DefaultWebSessionManager 实例与本地会话选项会话超时和集群配置的描述 [会话管理](http://shiro.apache.org/session-management.html) 部分。

#### 本地会话超时

配置后 DefaultWebSessionManager 例如,会话超时配置中描述 [会话管理:会话超时](http://shiro.apache.org/session-management.html#SessionManagement-sessionTimeout)

#### Session Cookie

DefaultWebSessionManager 支持两种网络自身配置属性:

* sessionIdCookieEnabled (一个 boolean)
* sessionIdCookie, 一个 [Cookie](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/servlet/Cookie.html) 实例.
	
*Cookie as a template 作为模板*

*sessionIdCookie 属性本质上是一个模板,你配置 Cookie 实例属性,该模板将在运行时用一个适当的会话ID值设置在实际 HTTP Cookie header中* 

#### Session Cookie 配置

DefaultWebSessionManager 的 sessionIdCookie 默认实例是一个 [SimpleCookie](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/servlet/SimpleCookie.html) 。 这个简单的实现允许 JavaBeans-style 属性配置为所有你想要的相关属性配置在 http Cookie。

例如,您可以设置 Cookie 域

	[main]
	...
	securityManager.sessionManager.sessionIdCookie.domain = foo.com

查看 [SimpleCookie JavaDoc](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/servlet/SimpleCookie.html) 额外的属性。

 cookie 的默认名称 JSESSIONID  根据servlet规范。 此外,Shiro 的cookie 支持 [HttpOnly](http://en.wikipedia.org/wiki/HTTP_cookie#HttpOnly_cookie) 标识。 sessionIdCookie 设置 在默认情况下 httponly 为 true 为了额外的安全。

*Shiro 的 Cookie  概念支持 HttpOnly 标识 甚至在 Servlet 2.4和 2.5 环境(而对原生的 Servlet API 只支持它在 2.6 或更高版本)。*

#### 禁用 Session Cookie 

如果您不希望使用会话 cookie,您可以禁用它们被配置使用 sessionIdCookieEnabled  属性为 false。 例如

	[main]
	...
	securityManager.sessionManager.sessionIdCookieEnabled = false

## Remember Me 服务

Shiro 将执行 'rememberMe' 服务如果 AuthenticationToken 实现了
[org.apache.shiro.authc.RememberMeAuthenticationToken](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/RememberMeAuthenticationToken.html) 接口。该接口指定了一个方法

	boolean isRememberMe();

如果该方法返回 true，Shiro 将会在整个会话中记住终端用户的身份ID

*UsernamePasswordToken and RememberMe*

*经常使用的 UsernamePasswordToken 已经实现了RememberMeAuthenticationToken 接口，并支持 rememberMe 登录。*

### 编程支持

要有计划性地使用 rememberMe ，你可以在一个支持该配置的类上把它的值设为 true。例如，使用标准的 UsernamePasswordToken：

	UsernamePasswordToken token = new UsernamePasswordToken(username, password);
	
	token.setRememberMe(true);
	
	SecurityUtils.getSubject().login(token);

### 基于表单的登录

对于 Web 应用程序而言，authc 过滤器默认是FormAuthenticationFilter 。它支持将 'rememberMe' 的布尔值作为一个 form/request 参数读取。默认地，它期望该 request 参数被命名为 rememberMe。下面是一个支持这点的 shiro.ini 配置的例子：

	[main]
	authc.loginUrl = /login.jsp
	
	[urls]
	
	# your login form page here:
	login.jsp = authc

同时在你的 web form 中有一个名为 'rememberMe' 的checkbox。

	<form ...>
	
	   Username: <input type="text" name="username"/> <br/>
	   Password: <input type="password" name="password"/>
	   ...
	   <input type="checkbox" name="rememberMe" value="true"/>Remember Me? 
	   ...
	</form>

默认地，FormAuthenticationFilter 将会寻找名为 username，password 及 rememberMe 的request 参数。如果这些不同于你使用的form 中的表单域名，你可能想在 FormAuthenticationFilter 上配置这些参数名。例如，在 shiro.ini 中：
	
	[main]
	...
	authc.loginUrl = /whatever.jsp
	authc.usernameParam = somethingOtherThanUsername
	authc.passwordParam = somethingOtherThanPassword
	authc.rememberMeParam = somethingOtherThanRememberMe
	...

### Cookie 配置

你可以通过设定 {{RememberMeManager}} 的各方面的 cookie 属性来配置 rememberMe cookie 是如何工作的。例如，
在shiro.ini 中：

	[main]
	...
	
	securityManager.rememberMeManager.cookie.name = foo
	securityManager.rememberMeManager.cookie.maxAge = blah
	...

请参见 [CookieRememberMeManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/mgt/CookieRememberMeManager.html) 及 [SimpleCookie](http://shiro.apache.org/static/current/apidocs/src-html/org/apache/shiro/web/servlet/SimpleCookie.html) 的 JavaDoc 支持来获取更多的配置属性

## JSP/GSP 标签库

Apache Shiro 提供了一个 Subject-aware JSP/GSP 标签库，它允许你控制你的 JSP，JSTL 或 GSP 页面基于当前 Subject 的状态进行输出。这对于根据身份个性化视图及当前用户所浏览的页面授权状态是相当有用的。

### 标签库配置

标签库描述文件 (TLD)被打包在 META-INF/shiro.tld 文件中的 shiro-web.jar 文件中。要使用任何标签，添加下面一行到你 JSP 页面（或任何你定义的页面指令）的顶部。

	<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags" %>

我们使用 shiro 前缀用以表明 shiro 标签库命名空间，当然你可以指定任何你喜欢的名字。

现在我们将讨论每一个标签，并展示它是如何用来渲染页面的。

### guest标签

guest 标签将显示它包含的内容，仅当当前的 Subject 被认为是 'guest' 时。'guest' 是指没有身份 ID 的任何 Subject。也
就是说，我们并不知道用户是谁，因为他们没有登录并且他们没有在上一次的访问中被记住（RememberMe 服务）。
例子：
	
	<shiro:guest>
	    Hi there!  Please <a href="login.jsp">Login</a> or <a href="signup.jsp">Signup</a> today!
	</shiro:guest>

guest 标签与user 标签逻辑相反。

### user 标签

user 标签将显示它包含的内容，仅当当前的 Subject 被认为是 'user' 时。'user' 在上下文中被定义为一个已知身份 ID 的 Subject，或是成功通过身份验证及通过'RememberMe'服务的。请注意这个标签在语义上与authenticated 标签是
不同的，authenticated 标签更为严格。

	<shiro:user>
	Welcome back John! Not John? Click <a href="login.jsp">here<a> to login.
	</shiro:user>

usre 标签与 guest 标签逻辑相反。

### authenticated 标签

仅仅只当当前用户在当前会话中成功地通过了身份验证 authenticated 标签才会显示包含的内容。它比 'user' 标签更为严格。它在逻辑上与'notAuthenticated'标签相反。

authenticated 标签只有当当前 Subject 在其当前的会话中成功地通过了身份验证才会显示包含的内容。它比 user 标签更为严格，authenticated 标签通常在敏感的工作流中用来确保身份 ID 是可靠的。

例子：

	<shiro:authenticated>
	    <a href="updateAccount.jsp">Update your contact information</a>.
	</shiro:authenticated>

authenticated 标签与 notAuthenticated 标签逻辑相反。

### notAuthenticated 标签


notAuthenticated 标签将会显示它所包含的内容，如果当前 Subject 还没有在其当前会话中成功地通过验证。

例子：

	<shiro:notAuthenticated>
	    Please <a href="login.jsp">login</a> in order to update your credit card information.
	</shiro:notAuthenticated>

notAuthenticated 标签与 Authenticated 标签逻辑相反。

### principal 标签

principal 标签将会输出 Subject 的主体（标识属性）或主要的属性。
若没有任何标签属性，则标签将使用 principal 的 toString()值来呈现页面。例如（假设 principal 是一个字符串的用户名）：

	Hello, <shiro:principal/>, how are you today?

这（大部分地）等价于下面：

	Hello, <%= SecurityUtils.getSubject().getPrincipal().toString() %>, how are you today?

#### Typed principal

principal 标签默认情况下，假定该 principal 输出的是subject.getPrincipal() 的值。但若你想输出一个不是主要 principal
的值，而是属于另一个 Subject 的 principal collection，你可以通过类型来获取该 principal 并输出该值。

例如，输出 Subject 的用户 ID（并不是 username ），假设该 ID 属于 principal collection：

	User ID: <principal type="java.lang.Integer"/>

这（大部分地）等价于下面：

	User ID: <%= SecurityUtils.getSubject().getPrincipals().oneByType(Integer.class).toString() %>

#### Principal property

但如果该 principal （是默认主要的 principal 或是上面的'typed' principal）是一个复杂的对象而不是一个简单的字符串，而且你希望引用该 principal 上的一个属性该怎么办呢？你可以使用 property 属性来来表示 property 的名称来理解（必须通过 JavaBeans 兼容的 getter 方法访问）。例如（假主要的principal 是一个User 对象）：

	Hello, <shiro:principal property="firstName"/>, how are you today?

这（大部分地）等价于下面：

	Hello, <%= SecurityUtils.getSubject().getPrincipal().getFirstName().toString() %>, how are you today?

或者，结合类型属性：

	Hello, <shiro:principal type="com.foo.User" property="firstName"/>, how are you today?

这也很大程度地等价于下面：

	Hello, <%= SecurityUtils.getSubject().getPrincipals().oneByType(com.foo.User.class).getFirstName().toString() %>, how are you today?

###   hasRole 标签

hasRole 标签将会显示它所包含的内容，仅当当前 Subject 被分配了具体的角色。

例如：

	<shiro:hasRole name="administrator">
	    <a href="admin.jsp">Administer the system</a>
	</shiro:hasRole>

hasRole 标签与lacksRole 标签逻辑相反。

###   lacksRole 标签

lacksRole 标签将会显示它所包含的内容，仅当当前 Subject 未被分配具体的角色。

例如：

	<shiro:lacksRole name="administrator">
	Sorry, you are not allowed to administer the system.
	</shiro:lacksRole>

###   hasAnyRole 标签

hasAnyRole 标签将会显示它所包含的内容，如果当前的 Subject 被分配了任意一个来自于逗号分隔的角色名列表中的具体角色。

例如：

	<shiro:hasAnyRoles name="developer, project manager, administrator">
	    You are either a developer, project manager, or administrator.
	</shiro:lacksRole>

###   hasPermission 标签

hasPermission 标签将会显示它所包含的内容，仅当当前Subject“拥有”（蕴含）特定的权限。也就是说，用户具有特定的能力。

例如：

	<shiro:hasPermission name="user:create">
	    <a href="createUser.jsp">Create a new User</a>
	</shiro:hasPermission>

hasPermission 标签与lacksPermission 标签逻辑相反。

###   lacksPermission 标签

lacksPermission 标签将会显示它所包含的内容，仅当当前Subject 没有拥有（蕴含）特定的权限。也就是说，用户没有特定的能力。

例如：

	<shiro:lacksPermission name="user:delete">
	    Sorry, you are not allowed to delete user accounts.
	</shiro:hasPermission>

## 为文档加把手

我们希望这篇文档可以帮助你使用 Apache Shiro 进行工作，社区一直在不断地完善和扩展文档，如果你希望帮助 Shiro 项目，请在你认为需要的地方考虑更正、扩展或添加文档，你提供的任何点滴帮助都将扩充社区并且提升 Shiro。

提供你的文档的最简单的途径是将它发送到用户[论坛](http://shiro-user.582556.n2.nabble.com/)或[邮件列表](http://shiro.apache.org/mailing-lists.html)

*译者注：如果对本中文翻译有疑议的或发现勘误欢迎指正，[点此](https://github.com/waylau/apache-shiro-1.2.x-reference/issues)提问。*
