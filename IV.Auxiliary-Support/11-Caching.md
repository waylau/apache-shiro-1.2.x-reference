# 11. Caching 缓存


Shiro 开发团队明白在许多应用程序中性能是至关重要的。Caching 是从第一天开始第一个建立在 Shiro 中的一流功能，以确保安全操作保持尽可能的快。

然而，Caching 作为一个概念是 Shiro 的基本组成部分，实现一个完整的缓存机制是安全框架核心能力之外的事情。为此，Shiro 的缓存支持基本上是一个抽象的（包装）API，它将“坐”在一个基本的缓存机制产品（例如，Ehcache，OSCache，Terracotta，Coherence，GigaSpaces，JBossCache 等）之上。这允许Shiro 终端用户配置他们喜欢的任何缓存机制。

## 缓存API

Shiro 有三个重要的缓存接口：

* [CacheManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/CacheManager.html) - 负责所有缓存的主要管理组件，它返回 Cache 实例。
* [Cache](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/Cache.html) - 维护key/value 对。
* [CacheManagerAware](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/CacheManagerAware.html) - 通过想要接收和使用 CacheManager 实例的组件来实现。

CacheManager 返回Cache 实例，各种不同的Shiro 组件使用这些Cache 实例来缓存必要的数据。任何实现了 CacheManagerAware 的 Shiro 组件将会自动地接收一个配置好的 CacheManager，该 CacheManager 能够用来获取 Cache 实例。

Shiro 的 [SecurityManager](http://shiro.apache.org/securitymanager.html) 实现及所有 [AuthorizingRealm](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/AuthenticatingRealm.html) 实现都实现了 CacheManagerAware 。如果你在 SecurityManager
上设置了 CacheManger，它反过来也会将它设置到实现了CacheManagerAware 的各种不同的 Realm 上（OO delegation）。例如，在 shiro.ini 中：
	
	securityManager.realms = $myRealm1, $myRealm2, ..., $myRealmN
	...
	cacheManager = my.implementation.of.CacheManager
	...
	securityManager.cacheManager = $cacheManager
	# at this point, the securityManager and all CacheManagerAware
	# realms have been set with the cacheManager instance

## CacheManager 实现

Shiro提供了许多开箱即用 CacheManager 实现，你可能会发现有用的而不是执行你自己的。

### MemoryConstrainedCacheManager

[MemoryConstrainedCacheManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/MemoryConstrainedCacheManager.html) 是一个 缓存管理器 实现适合单个jvm 生产环境。 这不是集中/分布式,所以,如果你的应用程序跨越多个 JVM(例如 web 应用程序运行在多个 web 服务器),你想要缓存实体跨 JVM 访问,您将需要使用一个分布式缓存实现。

MemoryConstrainedCacheManager 管理 [MapCache](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/MapCache.html) 情况下,一个 MapCache 每个命名的缓存实例。 每一个 MapCache Shiro 的实例支持 SoftHashMap 它可以自动调整大小本身基于应用程序的运行时内存约束/需求(通过利用 JDK 吗 softreference 实例)。

因为 MemoryConstrainedCacheManager 可以根据应用程序的内存配置文件自动调整大小本身,它的使用是安全的,在单个 jvm 的生产应用程序以及测试的需要。 然而,它没有更多的高级功能 suche 生存时间或 Time-to-Expire 设置缓存实体。 对于这些更先进的缓存管理功能,您可能会希望使用下面更先进的缓存管理器产品。

	...
	cacheManager = org.apache.shiro.cache.MemoryConstrainedCacheManager
	...
	securityManager.cacheManager = $cacheManager

### HazelcastCacheManager

*Shiro 1.3 or later*

*这个特性在当前版本中还没有发布。 Shiro 1.3 以后将会出现*

待定。

### EhCacheManager

待定。

## 授权缓存失效

最后请注意, [AuthorizingRealm](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/AuthorizingRealm.html) 有一个 [clearCachedAuthorizationInfo](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/AuthorizingRealm.html#clearCachedAuthorizationInfo(org.apache.shiro.subject.PrincipalCollection)) 方法能够被子类调用，用来清除特殊账户缓存的授权信息。它通常被自定义逻辑调用，如果与之匹配的账户授权数据发生了改变（来确保下次的授权检查能够捕获新数据）。