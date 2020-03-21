# 16. Guice


Shiro Guice 集成是在 Shiro 1.2 添加的。 这个页面覆盖了 Shiro 融入 基于 Guice 的应用程序使用标准方法 Guice 的约定和机制。 阅读这个集成文档之前,你应该至少有点熟悉 Guice。

## 概述

shiro-guice 提供了三个 Guice 模块可以包含在您的应用程序。

* ShiroModule
	* 提供基本的集成设置 SecurityManager ,任何 Realms 和任何其他配置 Shiro。
	* 使用这个模块通过扩展它并添加您自己的自定义配置。
* ShiroWebModule
	* 的延伸 ShiroModule 设置网络环境,还允许过滤器链配置。 这使用 [Guice Servlet Module](http://code.google.com/p/google-guice/wiki/ServletModule) 配置过滤器,因此要求设置。
	* 就像 ShiroModule 使用这个模块,通过扩展它并添加您自己的自定义配置。
* ShiroAopModule
	* 使用 [Guice AOP](http://code.google.com/p/google-guice/wiki/AOP) 实现Shiro AOP 注释。 这个模块主要是关心适应Shiro AnnotationMethodInterceptors 到 Guice 方法拦截器模型。
	* 通常使用这个模块通过简单地安装它。 然而,如果你有你自己的 Shiro AnnotationMethodInterceptors ,他们可以很容易地注册通过扩展它。

## 开始

最简单的配置是扩展 ShiroModule 安装您自己的 Realm 。
	
	 class MyShiroModule extends ShiroModule {
	        protected void configureShiro() {
	            try {
	                bindRealm().toConstructor(IniRealm.class.getConstructor(Ini.class));
	            } catch (NoSuchMethodException e) {
	                addError(e);
	            }
	        }
	
	        @Provides
	        Ini loadShiroIni() {
	            return Ini.fromResourcePath("classpath:shiro.ini");
	        }
	    }

在这种情况下,会在用户和角色配置 shiro.ini 文件。

	
*Shiro.ini 使用 Guice*

*重要的是要注意,在上面的配置,只有 用户 和 角色 部分使用ini文件。*

然后,Guice 模块是用来创建一个注射器,注射器是用来获得一个 SecurityManager 。 下面的例子具有同样目的的前三行 [快速入门](http://shiro.apache.org/10-minute-tutorial.html#10MinuteTutorial-Quickstart.java) 的例子。

	 Injector injector = Guice.createInjector(new MyShiroModule());
	 SecurityManager securityManager = injector.getInstance(SecurityManager.class);
	 SecurityUtils.setSecurityManager(securityManager);

## AOP

Shiro 包括几个注释和执行授权对于通过 AOP 方法拦截器非常有用。 它还提供了一个简单的 API 编写 Shiro-specific 方法拦截器。 shiro-guice 支持这个的 ShiroAopModule 。

要使用它,只需实例化并安装模块和应用程序模块和你 ShiroModule 。

    Injector injector = Guice.createInjector(new MyShiroModule(), new ShiroAopModule(), new MyApplicationModule());

如果你有写自定义拦截器,符合 Shiro 的 api,您可能会发现它有用的扩展 ShiroAopModule 。
	
	class MyShiroAopModule extends ShiroAopModule {
	        protected void configureInterceptors(AnnotationResolver resolver)
	        {
	            bindShiroInterceptor(new MyCustomAnnotationMethodInterceptor(resolver));
	        }
    
## Web

shiro-guice 的网络集成设计集成 Shiro 及其过滤范式与 Guice servlet 模块。 如果您正在使用 Shiro 在 web 环境中,并使用 Guice servlet 模块,那么你应该延长 ShiroWebModule 而不是 ShiroModule。 你的网络。 xml 应该设置 Guice servlet 模块的建议。
	
	 class MyShiroWebModule extends ShiroWebModule {
	        MyShiroWebModule(ServletContext sc) {
	            super(sc);
	        }
	
	        protected void configureShiroWeb() {
	            try {
	                bindRealm().toConstructor(IniRealm.class.getConstructor(Ini.class));
	            } catch (NoSuchMethodException e) {
	                addError(e);
	            }
	
	            addFilterChain("/public/**", ANON);
	            addFilterChain("/stuff/allowed/**", AUTHC_BASIC, config(PERMS, "yes"));
	            addFilterChain("/stuff/forbidden/**", AUTHC_BASIC, config(PERMS, "no"));
	            addFilterChain("/**", AUTHC_BASIC);
	        }
	
	        @Provides
	        Ini loadShiroIni() {
	            return Ini.fromResourcePath("classpath:shiro.ini");
	        }
	    }

在前面的代码中,我们已经绑定的 IniRealm 和设置四个过滤器链。 这些链相当于以下 ini 配置。

	[urls]
    /public/** = anon
    /stuff/allowed/** = authcBasic, perms["yes"]
    /stuff/forbidden/** = authcBasic, perms["no"]
    /** = authcBasic

shiro-guice,过滤器名称 Guice 的钥匙。 所有可用默认的 Shiro 过滤器是常量,但是你并不仅限于这些。 为了使用一个自定义过滤器过滤器链,你要做的事情

	Key customFilter = Key.get(MyCustomFilter.class);

    addFilterChain("/custom/**", customFilter);
 
我们仍然需要告诉 guice-servlets Shiro 过滤器。 自 ShiroWebModule 是私人的,guice-servlets 不给我们揭露一个过滤器映射的方法,我们必须手动绑定它。

    ShiroWebModule.guiceFilterModule()

或者,在一个应用程序模块,

    ShiroWebModule.bindGuiceFilter(binder())

## 属性

许多 Shiro 类暴露通过 setter 方法配置参数。 shiro-guice 注入这些如果找到一个绑定 @Named("shiro.{propName}")。 例如,设置会话超时,你可以做以下的事情。

    bindConstant().annotatedWith(Names.named("shiro.globalSessionTimeout")).to(30000L);

如果这个范式对你不起作用,你也可以考虑使用提供者直接实例化对象和调用setter。

## 注入对象

shiro-guice 使用 Guice TypeListener 对本地执行注射 Shiro 类(类的子目录 org.apache.shiro 但不是 org.apache.shiro.guice )。 然而, Guice 只考虑显式绑定类型作为候选人 TypeListeners ,所以如果你有一 个Shiro 对象,你想要注射,你必须显式声明它。 例如,设置 credentialsmatcher 领域,我们需要添加以下绑定:

	  bind(CredentialsMatcher.class).to(HashedCredentialsMatcher.class);
	  bind(HashedCredentialsMatcher.class);
	  bindConstant().annotatedWith(Names.named("shiro.hashAlgorithmName")).to(Md5Hash.ALGORITHM_NAME);

## 为文档加把手

我们希望这篇文档可以帮助你使用 Apache Shiro 进行工作，社区一直在不断地完善和扩展文档，如果你希望帮助 Shiro 项目，请在你认为需要的地方考虑更正、扩展或添加文档，你提供的任何点滴帮助都将扩充社区并且提升 Shiro。

提供你的文档的最简单的途径是将它发送到用户[论坛](http://shiro-user.582556.n2.nabble.com/)或[邮件列表](http://shiro.apache.org/mailing-lists.html)

*译者注：如果对本中文翻译有疑议的或发现勘误欢迎指正，[点此](https://github.com/waylau/apache-shiro-1.2.x-reference/issues)提问。*

 
