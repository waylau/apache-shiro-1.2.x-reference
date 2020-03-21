# 17. CAS


shiro-cas 模块是用来保护一个 [Jasig CAS](http://www.jasig.org/cas) 单点登录服务器。它使一个 Shiro-enabled 程序变成 CAS 客户端

## CAS协议的基本理解

1. 如果你想访问一个应用程序由 CAS 保护，并且如果你不验证在这个应用程序中的客户端,你重定向通过 CAS 客户端 到 CAS 服务器登录页面。 在 CAS 登录 url 定义了应用程序用户希望登录服务参数。

	http://application.examples.com/protected/index.jsp → HTTP 302
	→ https://server.cas.com/login?service=http://application.examples.com/shiro-cas

2. 你填写的登录名和密码和验证CAS服务器，然后将用户重定向到应用程序(服务的 url)和 url 的服务票证。 服务票证是短暂的一次性令牌可赎回在CAS服务器用户标识符(和可选地,用户属性)。

	https://server.cas.com/login?service=http://application.examples.com/shiro-cas → HTTP 302
	→ http://application.examples.com/shiro-cas?ticket=ST-4545454542121-cas
3. 应用程序直接问 CAS 服务器如果服务票是有效和 CAS 服务器响应通过身份验证的用户的身份。 一般来说,CAS 端转发用户最初叫保护页面。

	http://application.examples.com/shiro-cas?ticket=ST-4545454542121-cas → HTTP 302
	→ http://application.examples.com/protected/index.jsp

## 如何配置CAS服务器

### 依赖

	<dependency>
	    <groupId>org.apache.shiro</groupId>
	    <artifactId>shiro-cas</artifactId>
	    <version>version</version>
	</dependency>

( 版本 > = 1.2.0)。

### CasFilter

你必须定义服务应用程序的 ur l(在 CAS 服务器也必须声明)。 该url将被用来接收 CAS 服务票证。 例如: [http://application.examples.com/shiro-cas](http://application.examples.com/shiro-cas)

在 shiro 配置,您必须定义 CasFilter :

	[main]
	casFilter = org.apache.shiro.cas.CasFilter
	casFilter.failureUrl = /error.jsp

(失败的 url 时,服务票验证失败)。

和url可用:

	[urls]
	/shiro-cas = casFilter

这样,当用户通过 CAS服务器使用有效的服务票证(身份验证)后， 被重定向到应用程序服务 url (/shiro-cas),这个过滤器接收服务票证,并创建一个 CasToken 可以被 CasRealm 使用。

### CasRealm

CasRealm 使用 CasFilter 创建的 CasToken 通过 CAS 对CAS服务器服务票证 查验，从而对用户进行身份验证

在shiro配置,您必须添加 CasRealm :

	[main]
	casRealm = org.apache.shiro.cas.CasRealm
	casRealm.defaultRoles = ROLE_USER
	#casRealm.defaultPermissions
	#casRealm.roleAttributeNames
	#casRealm.permissionAttributeNames
	#casRealm.validationProtocol = SAML
	casRealm.casServerUrlPrefix = https://server.cas.com/
	casRealm.casService = http://application.examples.com/shiro-cas

casServerUrlPrefix 是 CAS 服务器的 url(例如: [https://server.cas.com](https://server.cas.com/) )。 

casService 是应用程序服务 url,url 指向应用程序收到 CAS 服务票证(例如: http://application.examples.com/shiro-cas )。 

validationProcol可以是 SAML或 CAS(默认):属性 和 remember me信息 只推到 SAML 验证协议(具体定制除外)。 这取决于 CAS 服务器的版本:SAML 协议可以使用 CAS 服务器版本 >= 3.1。

*如果选择 SAML 验证，需要添加更多依赖*

	<dependency>
	    <groupId>commons-codec</groupId>
	    <artifactId>commons-codec</artifactId>
	</dependency>
	<dependency>
	    <groupId>org.opensaml</groupId>
	    <artifactId>opensaml</artifactId>
	    <version>1.1</version>
	</dependency>
	<dependency>
	    <groupId>org.apache.santuario</groupId>
	    <artifactId>xmlsec</artifactId>
	    <version>1.4.3</version>
	</dependency>

defaultRoles 是默认的角色给了 CAS 认证成功后通过身份验证的用户。
 
defaultPermissions 是默认的权限给了 CAS 认证成功后通过身份验证的用户。 

roleAttributeNames 定义属性的名称来自 CAS 响应定义角色给了身份验证的用户(角色由 comas 进行分隔 )。 

permissionAttributeNames 定义属性的名称来自  CAS 响应它定义权限给身份验证的用户(权限由 comas 进行分隔 )。

### CasSubjectFactory

在 CAS 服务器,你可以“记住我”的支持。 这些信息是通过 SAML 验证或CAS 定制的验证。 
反映在 Shiro CAS-remember 我地位,你必须定义一个特定的 CasSubjectFactory 在你的Shiro配置:

	[main]
	casSubjectFactory = org.apache.shiro.cas.CasSubjectFactory
	securityManager.subjectFactory = $casSubjectFactory

### 应用程序的安全

最后,您必须定义您的应用程序的安全。
 
在 Shiro 配置,您必须保护 url 与角色(例如):

	[urls]
	/protected/** = roles[ROLE_USER]
	/** = anon

和登录 url 如果用户没有经过身份验证的 CAS 服务器上定义与应用程序服务网址:

	[main]
	roles.loginUrl = https://server.cas.com/login?service=http://application.examples.com/shiro-cas

这样,如果你不验证和尝试访问 /保护/ * * url,您被重定向到CAS服务器进行身份验证。

### 完整的配置样例
	
	[main]
	casFilter = org.apache.shiro.cas.CasFilter
	casFilter.failureUrl = /error.jsp
	
	casRealm = org.apache.shiro.cas.CasRealm
	casRealm.defaultRoles = ROLE_USER
	casRealm.casServerUrlPrefix = https://server.cas.com/
	casRealm.casService = http://application.examples.com/shiro-cas
	
	casSubjectFactory = org.apache.shiro.cas.CasSubjectFactory
	securityManager.subjectFactory = $casSubjectFactory
	
	roles.loginUrl = https://server.cas.com/login?service=http://application.examples.com/shiro-cas
	
	[urls]
	/shiro-cas = casFilter
	/protected/** = roles[ROLE_USER]
	/** = anon

## 历史

Version 1.2.0 : shiro-cas 模块第一个发布版本.