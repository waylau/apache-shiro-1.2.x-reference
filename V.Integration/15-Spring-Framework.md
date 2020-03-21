# 15. Spring Framework



本页涵盖了将 Shiro 集成到基于 [Spring](http://www.springframework.org/) 的应用程序的方法。

Shiro 的 JavaBean 兼容性使得它非常适合通过 Spring XML 或其他基于 Spring 的配置机制。 Shiro 应用程序需要一个具有单例SecurityManager 实例的应用程序。请注意，这不会是一个静态的单例，但应该只有一个应用程序能够使用的实例，无论它是否是静态单例的。

## 单独的应用

这里是在 Spring 应用程序中启用应用程序单例 SecurityManager 的最简单的方法：

	<!-- 定义连接后台安全数据源的realm -->
	<bean id="myRealm" class="...">
	   ...
	</bean>
	<bean id="securityManager" class="org.apache.shiro.mgt.DefaultSecurityManager">
	   <!-- 单realm应用。如果有多个realm，使用‘realms’属性代替 -->
	   <property name="realm" ref="myRealm"/>
	</bean>
	<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>
	<!-- 最简单的集成，是将securityManager bean配置成一个静态单例，也就是让            SecurityUtils.* 
	下的所有方法在任何情况下都起作用。在web应用中不要将securityManager bean配置为静态单例，
	具体方式请参阅下文中的‘Web Application’部分 -->
	<bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
	   <property name="staticMethod" value="org.apache.shiro.SecurityUtils.setSecurityManager"/>
	   <property name="arguments" ref="securityManager"/>
	</bean>

## Web 应用

Shiro 拥有对 Spring Web 应用程序的一流支持。在 Web 应用程序中，所有 Shiro 可访问的 web 请求必须通过一个主要的 Shiro 过滤器。该过滤器本身是极为强大的，允许临时的自定义过滤器链基于任何 URL 路径表达式执行。

在Shiro 1.0 之前，你不得不在 Spring web 应用程序中使用一个混合的方式，来定义 Shiro 过滤器及所有它在 web.xml 中的配置属性，但在Spring XML 中定义 SecurityManager。这有些令人沮丧，由于你不能把你的配置固定在一个地方，以及利用更为先进的 Spring 功能的配置能力，如 PropertyPlaceholderConfigurer 或抽象 bean 来固定通用配置。

现在在 Shiro 1.0 及以后版本中，所有 Shiro 配置都是在 Spring XML 中完成的，用来提供更为强健的 Spring 配置机制。

以下是如何在基于 Spring web 应用程序中配置 Shiro：

### web.xml

除了其他 Spring web.xml 中的元素（ ContextLoaderListener，Log4jConfigListener 等等），定义下面的过滤器及过滤器映射：

	<!-- filter-name对应applicationContext.xml中定义的名字为“shiroFilter”的bean -->
	<filter>
	    <filter-name>shiroFilter</filter-name>
	    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	    <init-param>
	        <param-name>targetFilterLifecycle</param-name>
	        <param-value>true</param-value>
	    </init-param>
	</filter>
	...
	<!-- 使用“/*”匹配所有请求，保证所有的可控请求都经过Shiro的过滤。通常这个filter-mapping
	放置到最前面(其他filter-mapping前面)，保证它是过滤器链中第一个起作用的 -->
	<filter-mapping>
	    <filter-name>shiroFilter</filter-name>
	    <url-pattern>/*</url-pattern>
	</filter-mapping>

### applicationContext.xml

在你的 applicationContext.xml 文件中，定义 web 支持的SecurityManager 和 'shiroFilter' bean 将会被 web.xml 引用。

	<bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
	    <property name="securityManager" ref="securityManager"/>
	    <!-- 可根据项目的URL进行替换 -->
	    <property name="loginUrl" value="/login.jsp"/>
	    <property name="successUrl" value="/home.jsp"/>
	    <property name="unauthorizedUrl" value="/unauthorized.jsp"/> 
	    <!-- 因为每个已经定义的javax.servlet.Filter类型的bean都可以在链的定义中通过bean名
	    称获取，所以filters属性不是必须出现的。但是可以根据需要通过filters属性替换filter
	    实例或者为filter起别名 -->
	    <property name="filters">
	        <util:map>
	            <entry key="anAlias" value-ref="someFilter"/>
	        </util:map>
	    </property> 
	    <property name="filterChainDefinitions">
	        <value>
	            # some example chain definitions:
	            /admin/** = authc, roles[admin]
	            /docs/** = authc, perms[document:read]
	            /** = authc
	            # more URL-to-FilterChain definitions here
	        </value>
	    </property>
	</bean>
	<!-- 定义应用上下文的 javax.servlet.Filter beans。这些beans 会被上面定义的shiroFilter自
	动感知，并提供给“filterChainDefinitions”属性使用。或者也可根据需要手动的将他们添加在
	shiroFilter bean的“filters”属性下的Map标签中。 -->
	<bean id="someFilter" class="..."/>
	<bean id="anotherFilter" class="..."> ... </bean>
	...
	<bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
	    <!-- 单realm应用。如果需要配置多个realm，使用“realms”属性 -->
	    <property name="realm" ref="myRealm"/>
	    <!-- 默认使用servlet容器session。下面是使用shiro 原生session的例子(细节请参考帮助文档)-->
	    <!-- <property name="sessionMode" value="native"/> -->
	</bean>
	<bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>
	<!-- 定义连接后台安全数据源的realm -->
	<bean id="myRealm" class="...">
	...
	</bean>

## 启用Shiro注解

不论独立的应用程序还是 web 应用程序，都可以使用 Shiro 提供的注解进行安全检查。比如 @RequiresRoles, @RequiresPermissions 等。这些注解需要借助 Spring 的 AOP扫描使用 Shiro 注解的类，并在必要时进行安全逻辑验证。

下面让我们看下如何开启注解，其实很简单，只要在applicationContext.xml 中定义两个 bean 即可。

	<!-- 开启Shiro注解的Spring配置方式的beans。在lifecycleBeanPostProcessor之后运行 -->
	<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator" depends-on="lifecycleBeanPostProcessor"/>
	<bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
	    <property name="securityManager" ref="securityManager"/>
	</bean>

## Spring远程安全

Shiro 的 Spring 远程支持有两部分组成：远程调用的客户端配置和接收、处理远程调用的服务器端配置。

### 服务端配置

当 Shiro 的 Server 端接收到一个远程方法调用时，与远程调用相关的Subject 必须在接收线程执行时绑定到接收线程上，这项工作通过在applicationContext.xml 中定义 SecureRemoteInvocationExecutor bean 完成。

	<!-- Spring远程安全确保每个远程方法调用都与一个负责安全验证的Subject绑定 -->
	<!-- can be associated with a Subject for security checks. -->
	<bean id="secureRemoteInvocationExecutor" class="org.apache.shiro.spring.remoting.SecureRemoteInvocationExecutor">
	    <property name="securityManager" ref="securityManager"/>
	</bean>

SecureRemoteInvocationExecutor 定义完成后，需要将它加入到Exporter 中，这个 Exporter 用于暴露向外提供的服务，而且 Exporter 的实现类由具体使用的远程处理机制和协议决定。定义 Exporter beans 请参照 Spring 的 [Remoting](http://static.springsource.org/spring/docs/2.5.x/reference/remoting.html) 章节。

以基于 HTTP 的远程 SecureRemoteInvocationExecutor 为例。( remoteInvocationExecutor 属性引用自secureRemoteInvocationExecutor)

	<bean name="/someService" class="org.springframework.remoting.httpinvoker.HttpInvokerServiceExporter">
	    <property name="service" ref="someService"/>
	    <property name="serviceInterface" value="com.pkg.service.SomeService"/>
	    <property name="remoteInvocationExecutor" ref="secureRemoteInvocationExecutor"/>
	</bean>

### 客户端配置

当远程调用发生时，负责鉴别信息的Subject需要告知server远程方法是谁发起的 。如果客户端是基于Spring的，那么这种关联可以通过Shiro的SecureRemoteInvocationFactory 完成。

	<bean id="secureRemoteInvocationFactory" class="org.apache.shiro.spring.remoting.SecureRemoteInvocationFactory"/>

然后将SecureRemoteInvocationFactory 添加到与协议相关的Spring远程ProxyFactoryBean 中。

以基于HTTP协议的远程ProxyFactoryBean为例。(remoteInvocationExecutor 属性引用自secureRemoteInvocationExecutor)

	<bean id="someService" class="org.springframework.remoting.httpinvoker.HttpInvokerProxyFactoryBean">
	    <property name="serviceUrl" value="http://host:port/remoting/someService"/>
	    <property name="serviceInterface" value="com.pkg.service.SomeService"/>
	    <property name="remoteInvocationFactory" ref="secureRemoteInvocationFactory"/>
	</bean>

## 为文档加把手

我们希望这篇文档可以帮助你使用 Apache Shiro 进行工作，社区一直在不断地完善和扩展文档，如果你希望帮助 Shiro 项目，请在你认为需要的地方考虑更正、扩展或添加文档，你提供的任何点滴帮助都将扩充社区并且提升 Shiro。

提供你的文档的最简单的途径是将它发送到用户[论坛](http://shiro-user.582556.n2.nabble.com/)或[邮件列表](http://shiro.apache.org/mailing-lists.html)

*译者注：如果对本中文翻译有疑议的或发现勘误欢迎指正，[点此](https://github.com/waylau/apache-shiro-1.2.x-reference/issues)提问。*

 

