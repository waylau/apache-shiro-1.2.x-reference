# 8. Session Management



Apache Shiro 提供安全框架界独一无二的东西：一个完整的企业级Session 解决方案，从最简单的命令行及智能手机应用到最大的集群企业Web 应用程序。

这对许多应用有着很大的影响——直到 Shiro 出现，如果你需要 session 支持，你需要部署你的应用程序到 Web 容器或使用EJB 有状态会话Bean。Shiro 的 Session 支持比这两种机制的使用和管理更为简单，而且它在适用于任何程序，不论容器。

即使你在一个 Servlet 或 EJB 容器中部署你的应用程序，仍然有令人信服的理由来使用 Shiro 的Session 支持而不是容器的。下面是一个
Shiro 的 Session 支持的最可取的功能列表：

特性

* **POJO/J2SE based(IoC friendly)** - Shiro 的一切（包括所有Session 和Session Management 方面）都是基于接口和
POJO 实现。这可以让你轻松地配置所有拥有任何 JavaBeans 兼容配置格式（如JSON，YAML，Spring XML 或类
似的机制）的会话组件。你也可以轻松地扩展 Shiro 的组件或编写你自己所需的来完全自定义 session management。
* **Easy Custom Session Storage** - 因为Shiro 的Session 对象是基于 POJO 的，会话数据可以很容易地存储在任意数量的数据源。这允许你自定义你的应用程序会话数据的确切位置——例如，文件系统，联网的分布式缓存，关系数据库，或专有的数据存储。
* **Container-Independent Clustering!** - Shiro 的会话可以很容易地聚集通过使用任何随手可用的网络缓存产品，像 Ehcache + Terracotta，Coherence，GigaSpaces，等等。这意味着你可以为Shiro 配置会话群集一次且仅一次，无论你部署到什么容器中，你的会话将以相同的方式聚集。不需要容器的具体配置！
* **Heterogeneous Client Access** - 与 EJB 或 web 会话不同，Shiro 会话可以被各种客户端技术“共享”。例如，一个桌面应用程序可以“看到”和“共享”同一个被使用的物理会话通过在 Web 应用程序中的同一用户。我们不知道除了 Shiro 以外的其他框架能够支持这一点。
* **Event Listeners** - 事件监听器允许你在会话生命周期监听生命周期事件。你可以侦听这些事件和对自定义应用程序的行为作出反应——例如，更新用户记录当他们的会话过期时。
* **Host Address Retention** - Shiro Sessions 从会话发起地方保留IP 地址或主机名。这允许你确定用户所在，并作出相应的反应（通常是在IP 分配确定的企业内部网络环境）。
* **Inactivity/Expiration Support** - 由于不活动导致会话过期如预期的那样，但它们可以延续很久通过 touch() 方法来保持它们“活着”，如果你希望的话。这在 RIA (富互联网应用)环境非常有用，用户可能会使用桌面应用程序，但可能不会经常与服务器进行通信，但该服务器的会话不应过期。
* **Transparent Web Use** - Shiro 的网络支持，充分地实现和支持关于Sessions（HttpSession 接口和它的所有相关的API）的 Servlet2.5 规范.这意味着你可以使用在现有 Web 应用程序中使用Shiro 会话，并且你不需要改变任何现有的 Web 代码。
* **Can be used for SSO** - 由于 Shiro 会话是基于POJO 的，它们可以很容易地存储在任何数据源，而且它们可以跨
程序“共享”如果需要的话。我们称之为"poor man's SSO"，并它可以用来提供简单的登录体验，由于共享的会话能够保留身份验证状态。

## 使用Sessions

几乎与所有其他在Shiro 中的东西一样，你通过与当前执行的Subject 交互来获取Session：
	
	Subject currentUser = SecurityUtils.getSubject();
	
	Session session = currentUser.getSession();
	session.setAttribute( "someKey", someValue);

subject.getSession() 方法是调用 currentUser.getSubject(true)的快捷方式。

对于那些熟悉 HttpServletRequest API 的，Subject.getSession(boolean create) 方法与 HttpServletRequest.getSession(boolean create) 方法有着异曲同工之效。

*  如果该Subject 已经拥有一个Session，则boolean 参数被忽略且Session 被立即返回。
* 如果该Subject 还没有一个Session 且create 参数为true，则创建一个新的会话并返回该会话。
* 如果该Subject 还没有一个Session 且create 参数为false，则不会创建新的会话且返回null。

*Any Application 任何应用*

*getSession 要求能够在任何应用程序工作，甚至是非 Web 应用程序。*

当开发框架代码来确保一个 Session 没有被创建是没有必要的时候，subject.getSession(false) 可以起到很好的作用。
当你获取了一个 Subject 的 Session 后，你可以用它来做许多事情，像设置或取得 attribute，设置其超时时间，以及
更多。请参见 Session 的 [JavaDoc](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/Session.html)来了解一个单独的会话能够做什么。

## SessionManager

SessionManager，名如其意，在应用程序中为所有的 subject 管理Session —— 创建，删除，inactivity(失效)及验证，等等。如同其他在Shiro 中的核心结构组件一样，SessionManager 也是一个由
SecurityManager 维护的顶级组件。

默认的 SecurityManger 实现是默认使用开箱即用的[DefaultSessionManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/mgt/DefaultSecurityManager.html)。DefaultSessionManager 的实现提供一个应用程序所需的所有企业级会话管理，如 Session 验证，orphan cleanup，等等。这可以在任何应用程序中使用。

*Web Applications*
*Web 应用程序使用不同SessionManager 实现。请参见 [Web](https://github.com/waylau/apache-shiro-1.2.x-reference/blob/master/III.%20Web%20Applications/10.%20Web.md) 文档获取web-specific Session Management 信息。*

像其他被 SecurityManager 管理的组件一样，SessionManager 可以通过 JavaBean 风格的 getter/setter 方法在所有Shiro
默认 SecurityManager 实现（getSessionManager()/setSessionManager()）上获取或设置值。或者例如，如果在使用
shiro.ini 配置：

	[main]
	...
	sessionManager = com.foo.my.SessionManagerImplementation
	securityManager.sessionManager = $sessionManager

但从头开始创建一个 SessionManager 是一个复杂的任务且是大多数人不想亲自做的事情。Shiro 的开箱即用的SessionManager 实现是高度可定制的和可配置的，并满足大多数的需要。本文档的其余部分假定你将使用 Shiro 的默认 SessionManager 实现，当覆盖配置选项时。但请注意，你基本上可以创建或插入任何你想要的东西。

### Session 超时

默认地，Shiro 的 SessionManager 实现默认是 30 分钟会话超时。也就是说，如果任何 Session 创建后闲置（未被使用，它的[lastAccessedTime](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/Session.html#getLastAccessTime())未被更新）的时间超过了 30 分钟，那么该 Session 就被认为是过期的，且不允许再被使用。

你可以设置 SessionManager 默认实现的 globalSessionTimeout 属性来为所有的会话定义默认的超时时间。例如，如果你想超时时间是一个小时而不是 30 分钟：

	[main]
	...
	# 3,600,000 milliseconds = 1 hour
	securityManager.sessionManager.globalSessionTimeout = 3600000

#### Per-Session 超时

上面的 globalSessionTimeout 值默认是为新建的 Session 使用的。你可以在每一个会话的基础上控制超时时间通过设置单独的会话 [timeout](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/Session.html#setTimeout(long))值。与上面的 globalSessionTimeout 一样，该值以毫秒（不是秒）为时间单位。

### Session 监听器

Shiro 支持 SessionListener 概念来允许你对发生的重要会话作出反应。你可以实现 [SessionListener](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/SessionListener.html) 接口（或扩展易用的SessionListenerAdapter ）并与相应的会话操作作出反应。
由于默认的 SessionManager sessionListeners 属性是一个集合，你可以对 SessionManager 配置一个或多个 listener 实
现，就像其他在 shiro.ini 中的集合一样：

	[main]
	...
	aSessionListener = com.foo.my.SessionListener
	anotherSessionListener = com.foo.my.OtherSessionListener
	
	securityManager.sessionManager.sessionListeners = $aSessionListener, $anotherSessionListener, etc.

*All Session Events 所有Session 事件*

*当任何会话发生事件时，SessionListeners 都会被通知——不仅仅是对一个特定的会话*

### Session 存储

每当一个会话被创建或更新时，它的数据需要持久化到一个存储位置以便它能够被稍后的应用程序访问。同样地，当一个会话失效且不再被使用时，它需要从存储中删除以便会话数据存储空间不会被耗尽。SessionManager 实现委托这些 Create/Read/Update/Delete(CRUD) 操作为内部组件，同时，[SessionDAO](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/eis/SessionDAO.html)，反映了数据访问对象（[DAO](http://en.wikipedia.org/wiki/Data_access_object)）设计模式。

SessionDAO 的权力是你能够实现该接口来与你想要的任何数据存储进行通信。这意味着你的会话数据可以驻留在内存中，文件系统，关系数据库或NoSQL 的数据存储，或其他任何你需要的位置。你得控制持久性行为。

你可以将任何 SessionDAO 实现作为一个属性配置在默认的SessionManager 实例上。例如，在shiro.ini 中：

	[main]
	...
	sessionDAO = com.foo.my.SessionDAO
	securityManager.sessionManager.sessionDAO = $sessionDAO

然而，正如你可能期望的那样，Shiro 已经有一些很好的SessionDAO 实现，你可以立即使用或实现你需要的子类。

*Web Applications*

*上述的 securityManager.sessionManager.sessionDAO = $sessionDAO 作业仅在使用一个本地的 Shiro 会话管理器时才
工作。Web 应用程序默认不会使用本地的会话管理器，而是保持不支持SessionDAO 的 Servlet Container 的默认会话
管理器。如果你想基于 Web 应用程序启用 SessionDAO 来自定义会话存储或会话群集，你将不得不首先配置一个本
地的Web 会话管理器。例如：*

	[main]
	...
	sessionManager = org.apache.shiro.web.session.mgt.DefaultWebSessionManager
	securityManager.sessionManager = $sessionManager
	
	# Configure a SessionDAO and then set it:
	securityManager.sessionManager.sessionDAO = $sessionDAO

*Configure a SessionDAO! 配置*

*Shiro 的默认配置本地 SessionManagers 使用仅内存 Session 存储。这是不适合大多数应用程序的。大多数生产应用程序想要配置提供的 EHCache（见下文）支持或提供自己的SessionDAO 实现。*

*请注意 Web 应用程序默认使用基于 servlet 容器的 SessionManager，且没有这个问题。这也是使用 Shiro 本地
SessionManager 的唯一问题。*

### EHCache SessionDAO

EHCache 默认是没有启用的，但如果你不打算实现你自己的 SessionDAO，那么强烈地建议你为 Shiro 的 SessionManagerment 启用 EHCache 支持。EHCache SessionDAO 将会在内存中保存会话，并支持溢出到磁盘，若内存成为制约。这对生产程序确保你在运行时不会随机地“丢失”会话是非常好的。

*Use EHCache as your default 设置 EHCache 为默认*

*如果你不准备编写一个自定义的 SessionDAO，则明确地在你的 Shiro 配置中启用 EHCache。EHCache 带来的好处远不止在 Sessions，缓存验证和授权数据方面。更多信息，请参见 [Caching](../IV. Auxiliary Support 辅助支持/11. Caching 缓存.md) 文档。*

*Container-Independent Session Clustering 独立容器的Session聚类*

*如果你急需独立的容器会话集群，EHCache 会是一个不错的选择。你可以显式地在 EHCache 之后插入[TerraCotta](http://www.terracotta.org/)，并拥有一个独立于容器集群的会话缓存。不必再担心 Tomcat，JBoss，Jetty，WebSphere 或WebLogic 特定的会话集群！

为会话启用 EHCache 是非常容易的。首先，确保在你的 classpath 中有shiro-ehcache-<version>.jar 文件（请参见
[Download](http://shiro.apache.org/download.html) 页面或使用 Maven 或 Ant+Ivy）。*

当在 classpath 中后，这第一个 shiro.ini 实例向你演示怎样为所有Shiro 的缓存需要（不只是会话支持）使用 EHCache：

	[main]
	
	sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
	securityManager.sessionManager.sessionDAO = $sessionDAO
	
	cacheManager = org.apache.shiro.cache.ehcache.EhCacheManager
	securityManager.cacheManager = $cacheManager

最后一行，securityManager.cacheManager = $cacheManager，为所有 Shiro 的需要配置了一个 CacheManager。该CacheManager 实例会自动地直接传送到 SessionDAO（通过 EnterpriseCacheSessionDAO 实现 [CacheManagerAware](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/CacheManagerAware.html) 接口的性质）。

然后，当 SessionManager 要求 EnterpriseCacheSessionDAO 去持久化一个 Session 时，它使用一个 EHCache 支持的 Cache 实现去存储Session 数据。

*Web Applications*

*当使用 Shiro 本地的 SessionManager 实现时不要忘了分配SessionDAO 是一项功能。Web 应用程序默认使用基于容器的 SessionManager，它不支持 SessionDAO。如果你想在 Web 应用程序中使用基于 EHCache 的会话存储，配置一个
如上所述的 Web SessionManager。*

#### EHCache Session Cache Configuration

默认地，EhCacheManager 使用一个 Shiro 特定的 [ehcache.xml](https://svn.apache.org/repos/asf/shiro/trunk/support/ehcache/src/main/resources/org/apache/shiro/cache/ehcache/ehcache.xml) 文件来建立 Session 缓存区以及确保 Sessions 正常存取的必要设置。

然而，如果你想改变缓存设置，或想配置你自己的 ehcache.xml 或EHCache net.sf.ehcache.CacheManager 实例，你需要配置缓存区来确保Sessions 被正确地处理。

如果你查看默认的 [ehcache.xml](https://svn.apache.org/repos/asf/shiro/trunk/support/ehcache/src/main/resources/org/apache/shiro/cache/ehcache/ehcache.xml) 文件，你会看到接下来的 shiro-activeSessionCache 缓存配置：

	<cache name="shiro-activeSessionCache"
	       maxElementsInMemory="10000"
	       overflowToDisk="true"
	       eternal="true"
	       timeToLiveSeconds="0"
	       timeToIdleSeconds="0"
	       diskPersistent="true"
	       diskExpiryThreadIntervalSeconds="600"/>

如果你希望使用你自己的 ehcache.xml 文件，那么请确保你已经为 Shiro 所需的定义了一个类似的缓存项。很有可能
你会改变 maxElementsInMemory 的属性值来吻合你的需要。然而，至少下面两个存在于你自己配置中的属性是非常重要的：

* overflowToDisk="true" - 这确保当你溢出进程内存时，会话不丢失且能够被序列化到磁盘上。
* eternal="true" - 确保缓存项（ Session 实例）永不过期或被缓存自动清除。这是很有必要的，因为 Shiro 基于计划过程完成自己的验证。如果我们关掉这项，缓存将会在 Shiro 不知道的情况下清扫这些 Sessions，这可能引起麻烦

#### EHCache Session Cache Name

默认地，EnterpriseCacheSessionDAO 向 CacheManager 寻求一个名为"shiro-activeSessionCache"的 Cache。该缓存的 name/region 将在 ehcache.xml 中配置，如上所述。

如果你想使用一个不同的名字而不是默认的，你可以在EnterpriseCacheSessionDAO 上配置名字，例如：

	[main]
	...
	sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
	sessionDAO.activeSessionsCacheName = myname
	...

只要确保在 ehcahe.xml 中有一项与名字匹配且你已经配置好了如上所述的 overflowToDisk="true" 和 eternal="true"。

#### 自定义 Session ID

Shiro 的 SessionDAO 实现使用一个内置的 [SessionIdGenerator](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/eis/SessionIdGenerator.html) 组件来产生一个新的 Session ID 当每次创建一个新的会话的时候。该 ID 生成后，被指派给新近创建的Session 实例，然后该Session 通过SessionDAO 被保存下来。

默认的SessionIdGenerator 是一个 [JavaUuidSessionIdGenerator](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/eis/JavaUuidSessionIdGenerator.html)，它能产生基于Java [UUIDs](http://download.oracle.com/javase/6/docs/api/java/util/UUID.html) 的 String IDs。该实现能够支持所有的生产环境。

如果它不符合你的需要，你可以实现 SessionIdGenerator 接口并在Shiro 的 SessionDAO 实例上配置该实现。例如，
在 shiro.ini 中：

	[main]
	...
	sessionIdGenerator = com.my.session.SessionIdGenerator
	securityManager.sessionManager.sessionDAO.sessionIdGenerator = $sessionIdGenerator

### Session  验证和计划

Sessions 必须被验证，这样任何无效(过期或停止)的会话能够从会话数据存储中删除。这保证了数据存储不会由于不能再次使用的会话而导致写入超时。

由于性能上的原因，仅仅在Sessions 被访问（也就是subject.getSession()）时验证它们是否停止或过期。这意味着，
如果没有额外的定期验证，Session orphans(孤儿)将会开始填充会话数据存储。

一个常见的说明孤儿的例子是 Web 浏览器中的场景：比方说，用户登录到Web 应用程序并创建了一个会话来保留
数据（身份验证状态，购物车等）。如果用户不注销，并在应用程序不知道的情况下关闭了浏览器，则他们的会话
实质上是“躺在”会话数据存储的（孤儿）。SessionManager 没有办法检测用户不再使用他们的浏览器，同时该会话永远不会被再次访问（它是孤儿了）。

会话孤儿，如果它们没有定期清除，将会填充会话数据存储（这是很糟糕的）。因此，为了防止丢放孤儿，SessionManager 实现支持 [SessionValidationScheduler](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/SessionValidationScheduler.html) 的概念。SessionValidationScheduler 负责定期地验证会话以确保
它们是否需要清理。

#### 默认 SessionValidationScheduler 

默认可用的 SessionValidationScheduler 在所有环境中都是[ExecutorServiceSessionValidationScheduler](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/ExecutorServiceSessionValidationScheduler.html)，它使用 JDK
[ScheduledExecutorService](http://download.oracle.com/javase/6/docs/api/java/util/concurrent/ScheduledExecutorService.html) 来控制验证频率。

默认地，该实现每小时执行一次验证。你可以通过指定一个新的 ExecutorServiceSessionValidationScheduler 实例并指
定不同的间隔（以毫秒为单位）改变速率来更改验证频率：

	[main]
	...
	sessionValidationScheduler = org.apache.shiro.session.mgt.ExecutorServiceSessionValidationScheduler
	# Default is 3,600,000 millis = 1 hour:
	sessionValidationScheduler.interval = 3600000
	
	securityManager.sessionManager.sessionValidationScheduler = $sessionValidationScheduler

#### 自定义 SessionValidationScheduler 

如果你希望提供一个自定义的 SessionValidationScheduler 实现，你可以指定它作为默认的 SessionManager 实例的一个属性。例如，在shiro.ini 中：

	[main]
	…
	sessionValidationScheduler = com.foo.my.SessionValidationScheduler
	securityManager.sessionManager.sessionValidationScheduler = $sessionValidationScheduler

#### 禁用 Session 验证 

在某些情况下，你可能希望禁用会话验证项，由于你建立了一个超出了Shiro 控制的进程来为你执行验证。例如，也许你正在使用一个企业的 Cache 并依赖于缓存的Time To Live 设置来自动地去除旧的会话。或者也许你已经制定了一个计划任务来自动清理一个自定义的数据存储。在这些情况下你可以关掉 session validation scheduling：

	[main]
	...
	securityManager.sessionManager.sessionValidationSchedulerEnabled = false
		
当会话从会话数据存储取回数据时它仍然会被验证，但这会禁用掉 Shiro 的定期验证。

*Enable Session Validation somewhere*

*如果你关闭了 Shiro 的session validation scheduler，你必须通过其他的机制（计划任务等）来执行定期的会话验证。
这是保证会话孤儿不会填充数据存储的唯一方法。*

####  删除无效的Session

正如我们上面所说的，进行定期的会话验证主要目的是为了删除任何无效的（过期或停止）会话来确保它们不会占用会话数据存储。

默认地，某些应用程序可能不希望 Shiro 自动地删除会话。例如，如果一个应用程序已经提供了一个 SessionDAO 备份数据存储查询，也许是应用程序团队希望旧的或无效的会话在一定的时间内可用。这将允许团队对数据存储运行查询来判断，例如，在上周某个用户创建了多少个会话，或一个用户会话的持续时间，或与之类似报告类型的查询。

在这些情形中，你可以关闭 invalid session deletion 项。例如，在shiro.ini 中：
	
	[main]
	...
	securityManager.sessionManager.deleteInvalidSessions = false

请注意！如果你关闭了它，你得为确保你的会话数据存储不耗尽它的空间。你必须自己从你的数据存储中删除无效的会话！

还要注意，即使你阻止了 Shiro 删除无效的会话，你仍然应该使用某种会话验证方式——要没通过 Shiro 的现有验证机制，要么通过一个你自己提供的自定义的机制（见上述的"Disabling Session Validation"获取更多）。验证机制将会更新你的会话记录以反映无效的状态（例如，什么时候它是无效的，它最后一次被访问是什么时候，等等），即使你在其他的一些时间将手动删除它们。

*如果你配置 Shiro 来让它不会删除无效的会话，你得为确保你的会话数据存储不会耗尽它的空间负责。你必须亲自从你的数据存储删除无效的会话！
另外请注意，禁用会话删除并不等同于禁用 session validation schedule（会话验证调度）。你应该总是使用一个会话验证调度机制——无论是 Shiro 直接支持或者是你自己的。*

## Session 集群

Apache Shiro 会话能力一个非常令人兴奋的事情是,你可以原生的集群 Subject 会话,不需要再担心你的容器环境。也就是说,如果您使用 Shiro 的原生会话并配置一个会话集群,可以,说,部署到 Jetty 和 Tomcat 开发环境,JBoss 或 Geronimo 的生产环境,或任何其他环境，不用担心容器/特定于环境的集群安装或配置。 Shiro 会话集群配置一次，无论您的部署环境如何，都能正常运行

因为 Shiro 的基于 pojo 的 n 层体系结构,使会话集群的集群机制非常简单,使会话持久性的水平。 也就是说,如果您配置集群 SessionDAO ,DAO 可以与集群交互机制, Shiro 的 SessionManager 不需要知道集群的问题。

###  分布式缓存

分布式缓存比如 [Ehcache + TerraCotta](http://ehcache.org/documentation/get-started/about-distributed-cache) , [GigaSpaces](http://www.gigaspaces.com/) [Oracle Coherence](http://www.oracle.com/technetwork/middleware/coherence/overview/index.html) , [memcached](http://memcached.org/) (和许多其他)已经解决 distributed-data-at-the-persistence-level(分布式数据持久层) 问题。 因此在 Shiro 会话使用集群配置如同使用分布式缓存一样简单。

这使您的灵活性选择确切的集群机制,适用于你的环境。

*缓存*

*请注意,当启用分布式/企业缓存在您的会话集群数据存储,下列两种情况之一必须是 true 的:*

* *整个集群范围的分布式缓存有足够内存来保留所有的 活动/当前 会话*

* *如果整个集群范围的分布式缓存没有足够的内存保留所有活动会话,它必须支持磁盘溢出,会话是不会丢失。*

*如果这两种情况都失败，会导致会话被随机丢失,对于终端用户来说这可能令人沮丧。*

### EnterpriseCacheSessionDAO

如您所料,Shiro 已经提供了 SessionDAO 的实现，将保存数据到 企业/分布式缓存。 [EnterpriseCacheSessionDAO](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/eis/EnterpriseCacheSessionDAO.html) 预计 Shiro 缓存 或 缓存管理器已经配置，所以它可以利用缓存机制。

例如,在 shiro.ini :

	#This implementation would use your preferred distributed caching product's APIs:
	activeSessionsCache = my.org.apache.shiro.cache.CacheImplementation
	
	sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
	sessionDAO.activeSessionsCache = $activeSessionsCache
	
	securityManager.sessionManager.sessionDAO = $sessionDAO

虽然你可以将缓存实例直接注入到 SessionDAO 如上所示,它通常是更常见的配置一般是缓存管理器 使用 Shiro 的所有缓存的需求(会话以及身份验证和授权数据)。 在这种情况下,而不是 直接配置 缓存实例,您会告诉 EnterpriseCacheSessionDAO 缓存管理器 中的缓存的名称，应该用于存储活动会话。
例如:
	
	# This implementation would use your caching product's APIs:
	cacheManager = my.org.apache.shiro.cache.CacheManagerImplementation
	
	# Now configure the EnterpriseCacheSessionDAO and tell it what
	# cache in the CacheManager should be used to store active sessions:
	sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
	# This is the default value.  Change it if your CacheManager configured a different name:
	sessionDAO.activeSessionsCacheName = shiro-activeSessionsCache
	# Now have the native SessionManager use that DAO:
	securityManager.sessionManager.sessionDAO = $sessionDAO
	
	# Configure the above CacheManager on Shiro's SecurityManager
	# to use it for all of Shiro's caching needs:
	securityManager.cacheManager = $cacheManager

但是上面的配置有一些有点奇怪。 你注意到吗?

有趣的关于这个配置,在配置我们实际上告诉 SessionDAO 实例使用一个 缓存 或 缓存管理器 ! 所以如何 SessionDAO 使用分布式缓存吗?

当 Shiro 初始化 SecurityManager ,它将检查看看 SessionDAO 是否 实现了 [CacheManagerAware](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/CacheManagerAware.html) 接口。 如果是这样,它将自动提供任何可用的全局配置 缓存管理器 。

所以,当 Shiro 评估 securityManager.cacheManager = $cacheManager  行,它就会发现 EnterpriseCacheSessionDAO 实现了 CacheManagerAware 接口和调用 setCacheManager 和你的配置方法 缓存管理器 作为方法的参数。

然后在运行时,当 EnterpriseCacheSessionDAO 需要 activeSessionsCache 它会问 缓存管理器 使用实例返回它 activeSessionsCacheName 作为查找得到的关键 缓存 实例。 那 缓存 实例(支持分布式缓存/企业产品的API)将用于存储和检索会话的所有 SessionDAO CRUD 操作。

### Ehcache + Terracotta

这样一个分布式缓存解决方案,人们取得了成功在使用 Shiro 的Ehcache + Terracotta 配对。 看到 Ehcache-hosted [分布式缓存与Terracotta ](http://ehcache.org/documentation/get-started/about-distributed-cache)文档的全部细节和 Ehcache 如何启用分布式缓存。

一旦你得到 Terracotta 集群处理 Ehcache ,Shiro-specific 部分非常简单。 阅读并遵守 EHCache SessionDAO 文档,但是我们需要做出一些改变

Ehcache 会话缓存配置 引用之前 不工作 —— Terracotta 特定的配置是必要的。 下面是一个示例配置,测试正常工作。 其内容保存在一个文件并将其保存在一个 ehcache.xml 文件:

	<ehcache>
	    <terracottaConfig url="localhost:9510"/>
	    <diskStore path="java.io.tmpdir/shiro-ehcache"/>
	    <defaultCache
	        maxElementsInMemory="10000"
	        eternal="false"
	        timeToIdleSeconds="120"
	        timeToLiveSeconds="120"
	        overflowToDisk="false"
	        diskPersistent="false"
	        diskExpiryThreadIntervalSeconds="120">
	        <terracotta/>
	   </defaultCache>
	   <cache name="shiro-activeSessionCache"
	       maxElementsInMemory="10000"
	       eternal="true"
	       timeToLiveSeconds="0"
	       timeToIdleSeconds="0"
	       diskPersistent="false"
	       overflowToDisk="false"
	       diskExpiryThreadIntervalSeconds="600">
	       <terracotta/>
	   </cache>
	   <!-- Add more cache entries as desired, for example,
	        Realm authc/authz caching: -->
	</ehcache>

当然你想要改变你 <terracottaConfig url="localhost:9510"/> e 条目引用适当的主机/端口 Terracotta 服务器阵列。 还要注意,与 以前的 配置中, ehcache-activeSessionCache 元素不设置 diskPersistent 或 overflowToDisk 属性为 true 的。他们都应该是 假 作为真实值不支持在集群配置。

在你保存了 ehcache.xml 文件,我们需要在 Shiro 的配置中引用它。 假设你已经作了 terracotta 特定 ehcache.xml 文件在类路径的根,这是最后的 Shiro 配置,使 Terracotta + Ehcache 集群的 Shiro 的需要(包括会话):
	
	sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
	# This name matches a cache name in ehcache.xml:
	sessionDAO.activeSessionsCacheName = shiro-activeSessionsCache
	securityManager.sessionManager.sessionDAO = $sessionDAO
	
	# Configure The EhCacheManager:
	cacheManager = org.apache.shiro.cache.ehcache.EhCacheManager
	cacheManager.cacheManagerConfigFile = classpath:ehcache.xml
	
	# Configure the above CacheManager on Shiro's SecurityManager
	# to use it for all of Shiro's caching needs:
	securityManager.cacheManager = $cacheManager

请记住, 顺序很重要 。 通过配置 缓存管理器 在 SecurityManager 最后,我们确保可以传播到所有之前配置缓存管理器 CacheManagerAware 组件(如 EnterpriseCachingSessionDAO )。

### Zookeeper

用户报告使用  [Apache Zookeeper](http://zookeeper.apache.org/) 来管理/协调分布式会话。 如果你有任何文档/评论关于这个工作,请提交到到 [Shiro 邮件列表](http://shiro.apache.org/mailing-lists.html)

## Session 和 Subject 状态

### 有状态应用

默认地，Shiro 的SecurityManager 实现使用一个Subject 的Session 作为一种策略来为接下来的引用存储Subject 的身份 ID（PrincipalCollection）和验证状态（subject.isAuthenticated()）。这通常发生在一个Subject 登录后或当一个 Subject
的身份 ID 通过Remember 服务被发现后。

这个默认的方法有几个好处:

* 任何服务于请求，调用或消息的应用程序可以用请求/调用/消息的有效载荷关联会话ID，且这是Shiro 用入站
请求关联用户所有所必须的。例如，如果使用Subject.Builder，这是需要获取相关的Subject 所需的一切：

	Serializable sessionId = //get from the inbound request or remote method invocation payload
	Subject requestSubject = new Subject.Builder().sessionId(sessionId).buildSubject();

这给大多数Web 应用程序及任何编写远程处理或消息框架的人带来了令人难以置信的方便（这事实上是Shiro
的Web 支持在自己的框架代码内关联Subject 和ServletRequest）。

* 任何"RememberMe"身份基于一个能够在第一次访问就能持久化到会话的初始请求。这确保了Subject 被记住的身份可以跨请求保存而不需要反序列化及将它解释到每个请求。例如，在一个 Web 应用程序中，没有必要去读取每一个请求的加密RememberMe Cookie，如果该身份在会话中是已知的。这可是一个很好的性能提升。

### 无状态应用
虽然上述的默认策略对于大多数应用程序而言是很好的（通常是可取的），但这对于尝试尽可能无状态的应用程序来说是不合适的。许多无状态的架构规定在请求中不能存在持久状态，这种情况下的 Sessions 不会被允许（一个会话其本质代表了持久状态）。

但这一要求带来一个便利的代价—— Subject 状态不能跨请求保留。这意味着有这一要求的应用程序必须确保 Subject 状态可以在每一个请求中以其他的方式代表。

这几乎总是通过验证每个由应用程序处理的请求/调用/消息来完成的。例如，大多数无状态 Web 应用程序通常支持这一点通过执行 HTTP 基本验证，允许浏览器验证每一个代表最终用户的请求

#### 禁用 Subject 状态会话存储

在 Shiro 1.2 及以后开始，应用程序想禁用 Shiro 的内部实现策略——将Subject 状态持久化到会话，可以禁用所有 Subject 的这一项，通过下面的操作：

在shiro.ini 中，在 securityManager 上配置下面的属性：

	[main]
	...
	securityManager.subjectDAO.sessionStorageEvaluator.sessionStorageEnabled = false
	...

这将防止 Shiro 使用 Subject 的会话来存储所有跨请求/调用/消息的Subject 状态。只要确保你对每个请求进行了身份验证，这样 Shiro 将会对给定的请求/调用/消息知道它的 Subject 是谁。

*Shiro的需求 vs. 你的需求*

*使用 Sessions 作为存储策略将禁用 Shiro 本身的实现。它没有完全地禁用 Sessions。如果你的任何代码显式地调用
subject.getSession() 或 subject.getSession(true) ，一个session 仍然会被创建。*

### 一个混合的方法

上面的shiro.ini 配置中的(securityManager.subjectDAO.sessionStorageEvaluator.sessionStorageEnabled = false) 这一行将会禁用Shiro 为所有的Subject 使用Session 作为一种实现策略。

但，如果你想使用混合的方法呢？如果某些对象应该有会话而某些没有？这种混合法方法能够给许多应用程序带来好处。例如：

* 也许 human Subject（如 Web 浏览器用户）由于上面提供的好处能够使用Session。
* 也许non-human Subject（如 API 客户端或第三方应用程序）不应该创建session 由于它们与软件的交互可能会
间歇或不稳定。
* 也许所有某种确定类型的 Subject 或从某一确定位置访问系统的应该将状态保持在会话中，但所有其他的不应
该。
如果你需要这个混合方法，你可以实现一个 SessionStorageEvaluator。

#### SessionStorageEvaluator

在你想究竟控制哪个 Subject 能够在它们的 Session 中保存它们的状态的情况下，你可以实现

org.apache.shiro.mgt.SessionStorageEvaluator 接口，并告诉Shiro 哪个 Subject 支持会话存储。

该接口只有一个方法：

	public interface SessionStorageEvaluator {
	
	    public boolean isSessionStorageEnabled(Subject subject);
	
	}

关于更详细的API 说明，请参见 SessionStorageEvaluator 的[JavaDoc](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/mgt/SessionStorageEvaluator.html)。
你可以实现这一接口，并检查 Subject，为了你可能做出这一决定的任何信息

##### Subject Inspection

但实现 isSessionStorageEnabled(subject)接口方法时，你可以一直查看 Subject 并访问任何你需要用来作出决定的东西。

当然所有期望的 Subject 方法都是可用的（gePrincipals()等），但特定环境的 Subject 实例也是有价值的。

例如，在 Web 应用程序中，如果该决定必须基于当前 ServletRequest 中的数据，你可以获取该 request 或该 response，因为运行时的Subjce 实例实际上就是一个 [WebSubject](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/subject/WebSubject.html) 实例：

	...
    public boolean isSessionStorageEnabled(Subject subject) {
        boolean enabled = false;
        if (WebUtils.isWeb(Subject)) {
            HttpServletRequest request = WebUtils.getHttpRequest(subject);
            //set 'enabled' based on the current request.
        } else {
            //not a web request - maybe a RMI or daemon invocation?
            //set 'enabled' another way...
        }

        return enabled;
    }

N.B.框架开发人员应该考虑到这种类型的访问，并确保任何请求/调用/消息上下文对象可用是同过特定环境下的 Subject 实现的。联系 Shiro 用户邮件列表，如果你想帮助设置它，为了你的框架/环境。

#### 配置 

在你实现了 SessionStorageEvaluator 接口后，你可以在 shiro.ini 中配置它：

	[main]
	...
	sessionStorageEvaluator = com.mycompany.shiro.subject.mgt.MySessionStorageEvaluator
	securityManager.subjectDAO.sessionStorageEvaluator = $sessionStorageEvaluator
	
	...

### Web 应用

通常 Web 应用程序希望在每一个请求的基础上容易地启用或禁用会话的创建，不管是哪个 Subject 正在执行请求。这经常在支持 REST 及Messaging/RMI 构架上使用来产生很好的效果。例如，也许正常的终端用户（使用浏览器的人）被允许创建和使用会话，但远程的 API 客户端使用REST 或 SOAP，不该拥有会话（因为它们在每一个请求上验证，
常见于 REST/SOAP 体系结构）。

为了支持这种 hybrid/per-request （混合/每次请求）的能力，noSessionCreation 过滤器被添加到 Shiro 的默认“池”g过滤器中，为 Web 应用程序启用的。该过滤器将会阻止在请求期间创建新的会话来保证无状态的体验。在shiro.ini 的[urls]项中，你通常定义该过滤器在所有其它过滤器之前来确保会话永远不会被使用。

举例：
	
	[urls]
	...
	/rest/** = noSessionCreation, authcBasic, ...

这个过滤器允许现有会话的任何会话操作，但不允许在过滤的请求创建新的会话。也就是说，在请求或没有会话存在的Subject 调用下面四个方法中的任何一个时，将会自动地触发一个 DisabledSessionException 异常：

* httpServletRequest.getSession()
* httpServletRequest.getSession(true)
* subject.getSession()
* subject.getSession(true)

如果一个 Subject 在访问 noSessionCreation-protected-URL（无会话创建保护的 URL） 之前已经有一个会话，则上述的四种调用仍然会如预期工作。

最后，在所有情况下，下面的调用将始终被允许：

* httpServletRequest.getSession(false)
* subject.getSession(false)

