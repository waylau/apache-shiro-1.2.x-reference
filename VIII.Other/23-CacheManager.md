# 23. CacheManager 缓存管理




Shiro 有三个重要的缓存接口：

* [CacheManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/CacheManager.html) - 负责所有缓存的主要管理组件，它返回 Cache 实例。
* [Cache](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/Cache.html) - 维护key/value 对。
* [CacheManagerAware](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/CacheManagerAware.html) - 通过想要接收和使用 CacheManager 实例的组件来实现。

CacheManager 返回Cache 实例，各种不同的 Shiro 组件使用这些Cache 实例来缓存必要的数据。任何实现了 CacheManagerAware 的 Shiro 组件将会自动地接收一个配置好的 CacheManager，该 CacheManager 能够用来获取 Cache 实例。

Shiro 的 [SecurityManager](http://shiro.apache.org/securitymanager.html) 实现及所有 [AuthorizingRealm](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/AuthenticatingRealm.html) 实现都实现了 CacheManagerAware 。如果你在 SecurityManager
上设置了 CacheManger，它反过来也会将它设置到实现了CacheManagerAware 的各种不同的 Realm 上（OO delegation）。例如，在 shiro.ini 中：
	
	securityManager.realms = $myRealm1, $myRealm2, ..., $myRealmN
	...
	cacheManager = my.implementation.of.CacheManager
	...
	securityManager.cacheManager = $cacheManager
	# at this point, the securityManager and all CacheManagerAware
	# realms have been set with the cacheManager instance

我们有开箱即用的 [EhCacheManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/ehcache/EhCacheManager.html)实现 ,所以马上就能使用它。否则，您也可以很好的实现自己的 CacheManager （例如 Coherence，等），配置如上。

## 授权缓存失效

最后请注意, [AuthorizingRealm](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/AuthorizingRealm.html) 有一个 [clearCachedAuthorizationInfo](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/AuthorizingRealm.html#clearCachedAuthorizationInfo(org.apache.shiro.subject.PrincipalCollection)) 方法能够被子类调用，用来清除特殊账户缓存的授权信息。它通常被自定义逻辑调用，如果与之匹配的账户授权数据发生了改变（来确保下次的授权检查能够捕获新数据）。

## 为文档加把手

我们希望这篇文档可以帮助你使用 Apache Shiro 进行工作，社区一直在不断地完善和扩展文档，如果你希望帮助 Shiro 项目，请在你认为需要的地方考虑更正、扩展或添加文档，你提供的任何点滴帮助都将扩充社区并且提升 Shiro。

提供你的文档的最简单的途径是将它发送到用户[论坛](http://shiro-user.582556.n2.nabble.com/)或[邮件列表](http://shiro.apache.org/mailing-lists.html)

*译者注：如果对本中文翻译有疑议的或发现勘误欢迎指正，[点此](https://github.com/waylau/apache-shiro-1.2.x-reference/issues)提问。*
