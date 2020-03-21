# 4. Configuration 配置


Shiro 可以在任何环境下工作，从简单的命令行程序到大型企业级集群项目，因为环境的多样化，可以通过许多途径来配合当前环境的配置方式进行配置，在本章我们来了解一下 Shiro 核心支持的配置方式。

*多种配置选择*

*Shiro 的 SecurityManager 的实现和其所依赖的组件都是 JavaBean，所以可以用多种形式对 Shiro 进行配置，比如XML（Spring, JBoss, Guice, 等等），[YAML](http://www.yaml.org/), JSON, Groovy Builder markup，及其它，INI 只是 Shiro 一种最基本的配置方式，使得其可以在任何环境中进行配置比如在那些没有以上配置形式的环境中。*

## 编程方式配置

创建一个 SecurityManager 并使之可用最简单的方法就是创建一个org.apache.shiro.mgt.DefaultSecurityManager 对象并且将它写入代码，例如：

	Realm realm = //实例化或获得一个Realm的实例。我们将稍后讨论Realm。
	
	SecurityManager securityManager = new DefaultSecurityManager(realm);
	
	//使SecurityManager实例通过静态存储器对整个应用程序可见：
	SecurityUtils.setSecurityManager(securityManager);

仅仅三行代码，你就可以拥有一个适用于任何程序的功能全面的 Shiro 环境，多么简单。

### SecurityManager对象图

如同我们在Architecture中讨论过的，Shiro SecurityMangger 本质上是一个由一套安全组件组成的对象模块视图（graph），因为与 JavaBean兼容，所以可以对所有这些组件调用的 getter 和 setter 方法来配置SecurityManager 和它的内部对象视图。

例如，你想用一个自定义的 SessionDAO 来定制 [Session Management](http://shiro.apache.org/session-management.html)从而配置一个 SecurityManager 实例，你就可以使用 SessionManager 的 setSessionDAO 方法直接 set 这个 SessionDAO。
	

```java
Realm realm = //instantiate or acquire a Realm instance.  We'll discuss Realms later.
SecurityManager securityManager = new DefaultSecurityManager(realm);

//Make the SecurityManager instance available to the entire application via static memory: 
SecurityUtils.setSecurityManager(securityManager);
```

使用这些函数，你可以配置 SecurityManager 视图（graph）中的任何一部分。

虽然在程序中配置很简单，但它并不是我们现实中配置的完美解决方案。在几种情况下这种方法可能并不适合你的程序：

* 它需要你确切知道并实例化一个直接实现（direct implementation），然而更好的做法是你并不需要知道这些实现也不需要知道从哪里找到它们。
* 因为JAVA类型安全的特性，你必须对通过 get* 获取的对象进行强制类型转换，这么多强制转换非常的丑陋、累赘并且会和你的类紧耦合。
* SecurityUtils.setSecurityManager 方法会将 SecurityManager 实例化为虚拟机的单独静态实例，在大多数程序中没有问题，但如果有多个使用 Shiro 的程序在同一个 JVM 中运行时，各程序有自己独立的实例会更好些，而不是共同引用一块静态内存。
* 改变配置就需要重新编译你的程序。

然而，尽管有这些不足，在程序中定制的这种方法在限制内存（memory-constrained ）的环境中还是很有价值的，像智能电话程序。如果你的程序不是运行在一个限制内存的环境中，你会发现基于文本的配置会更易读易用。

## INI配置

大多数程序已经改为使用基于文本的配置，不需要依靠代码就可进行修改，对于不熟悉Shiro API的人来说，也易于理解。

为了确保具有共性的基于文本配置的途径适用于任何环境而且减少对第三方的依赖，Shiro 支持使用 INI 创建 SecurityManager 对象视图（graph）以及它支持的组件，INI 易读易配置，很容易创建并且对大多数程序都很适合。

### 通过INI资源创建 SecurityManager

这里举两个通过INI配置创建SecurityManager的例子。

#### 从INI资源创建SecurityManager

我们可以从一个INI资源路径创建一个 SecurityManager 实例，资源可以通过文件系统（前缀为file:）、类路径(classpath:)或者URL(url:)获得，下面的例子使用一个 Factory 从类路径根目录加载 shiro.ini 并返回一个 SecurityManager 实例。


```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.util.Factory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.config.IniSecurityManagerFactory;

...

Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
SecurityManager securityManager = factory.getInstance();
SecurityUtils.setSecurityManager(securityManager);
```

#### 通过INI实例创建SecurityManager

INI 配置可以通过[org.apache.shiro.config.Ini](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/config/Ini.html) 类用程序方式创建，这个 INI 类类似于 JDK 的[java.util.Properties](http://download.oracle.com/javase/6/docs/api/java/util/Properties.html)类，但支持通过section 名分割。例子如下：


```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.util.Factory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.config.Ini;
import org.apache.shiro.config.IniSecurityManagerFactory;

...

Ini ini = new Ini();
//populate the Ini instance as necessary
...
Factory<SecurityManager> factory = new IniSecurityManagerFactory(ini);
SecurityManager securityManager = factory.getInstance();
SecurityUtils.setSecurityManager(securityManager);
```


现在我们知道如何使用 INI 配置文件创建一个 SecurityManager，让我们仔细了解一下如何定义一个 shiro INI配置文件。

### INI区域

INI　基于文本配置，在独立命名的区域内通过成对的键名/键值组成。键名在每个区域内必须唯一，但在整个配置文件中并不需要这样（这点和JDK的Properties不同），每一个区域（section）可以看作是一个独立的Properties 定义。

注释行可以用“#”或“;”标识。

这里是一个 Shiro 可以理解的各 section 的示例。


```java
# =======================
# Shiro INI configuration
# =======================

[main]
# Objects and their properties are defined here,
# Such as the securityManager, Realms and anything
# else needed to build the SecurityManager

[users]
# The 'users' section is for simple deployments
# when you only need a small number of statically-defined
# set of User accounts.

[roles]
# The 'roles' section is for simple deployments
# when you only need a small number of statically-defined
# roles.

[urls]
# The 'urls' section is used for url-based security
# in web applications.  We'll discuss this section in the
# Web documentation
```


#### `[main]`

[main]区域是配置程序 SecurityManager 实例及其支撑组件的地方，如 Realm。 

通过INI配置像 SecurityManager 的对象实例及其支撑组件听起来是一件很困难的事情，因为在这里我们只能用键名/键值对。但通过定义一些对象视图（graphs）可以理解的惯例，你发现你完全可以这样做。Shiro 利用这些假定的惯例来实现一个简单而简明的配置途径。

我们经常将这种方法认为是“可怜人的（poor man's）”的依赖注入，虽然不及成熟的Spring/Guice/JBoss的XML文件强大，但你会发现它可以做很多事情而且并不复杂，当然当那配置途径也可以使用，但对 Shiro 来讲并不是必须的。 

仅仅吊一下胃口，这里是一个简单的可以使用的[main]配置，下面我们会详细介绍，但你可能发现你仅凭直觉就可以理解一些。


```
[main]
sha256Matcher = org.apache.shiro.authc.credential.Sha256CredentialsMatcher

myRealm = com.company.security.shiro.DatabaseRealm
myRealm.connectionTimeout = 30000
myRealm.username = jsmith
myRealm.password = secret
myRealm.credentialsMatcher = $sha256Matcher

securityManager.sessionManager.globalSessionTimeout = 1800000
```


##### 定义一个对象

在[main]中包含以下片段。

```
[main]
myRealm = com.company.shiro.realm.MyRealm
...
```

这一行实例化了一个类型为 com.company.shiro.realm.MyRealm 的对象实例并且使对象使用 myRealm 作为名称以便于将来引用和配置。

如果对象实例化时实现了 org.apache.shiro.util.Nameable 接口，Nameable.setName方法将被以该名（在此例中为myRealm）命名的对象调用。

##### 设置对象属性

###### 原始值

简单的原始值属性可以使用下面的等于符号进行设置：

```
...
myRealm.connectionTimeout = 30000
myRealm.username = jsmith
...
```

这些配置行转换为方法调用就是：

```
...
myRealm.setConnectionTimeout(30000);
myRealm.setUsername("jsmith");
...
```

怎么做到的呢？它假定所有对象都是兼容 [JavaBean](http://en.wikipedia.org/wiki/JavaBean) 的 [POJO](http://en.wikipedia.org/wiki/Plain_Old_Java_Object)。在设置这些属性时，Shiro 默认使用 Apache 通用的 [BeanUtils](http://commons.apache.org/beanutils/) 来完成这项复杂的工作，所以虽然 INI 值是文本，BeanUtils 知道如何将这些字符串值转换为适合的原始值类型并调用合适的 JavaBeans 的 setter 方法。

###### 引用值

如果你想设置的值并不是一个原始值，而是另一个对象怎么办呢？你可以使用一个 `$` 符来引用一个之前定义的实例，如：

```
...
sha256Matcher = org.apache.shiro.authc.credential.Sha256CredentialsMatcher
...
myRealm.credentialsMatcher = $sha256Matcher
...
```

这定义了名为 sha256Matcher 的对象并且使用 BeanUtils 将其设置到myRealm 的实例中（通过调用 myRealm.setCredentialsMatcher(sha256Matcher) 方法）。

###### 嵌套属性

通过在等号左侧使用点符号，你可以得到你希望设置对象视图最终的对象/属性，例如下面这行配置：


```
...
securityManager.sessionManager.globalSessionTimeout = 1800000
...
```

转换逻辑为（通过BeanUtils）：


```
securityManager.getSessionManager().setGlobalSessionTimeout(1800000);
```

用这种方法访问的层数需要多深可以有多深：`object.property1.property2....propertyN.value = blah`

*BeanUtils 属性支持*

*BeanUtils 支持任何指定的属性操作，在 Shiro [main] 区域中[setProperty](http://commons.apache.org/beanutils/v1.8.2/apidocs/org/apache/commons/beanutils/BeanUtils.html#setProperty%28java.lang.Object,%20java.lang.String,%20java.lang.Object%29)方法将被调用，包括集合（set）/列表（list）/图（map），查看[Apache Commons BeanUtils Website](http://commons.apache.org/beanutils/)和文档了解更多的信息。*

###### 字节数组值

因为原始的字节数组不能直接在文本中定义，我们必须使用字节数组的文本编码。可以使用64位编码（默认）或者16位编码，默认为64位编码因为使用64位编码实际文字会少一些--它拥有很大的编码表，这意味着你的标识会更短（对于文本配置来讲会好一些）。

```
# The 'cipherKey' attribute is a byte array.    By default, text values
# for all byte array properties are expected to be Base64 encoded:

securityManager.rememberMeManager.cipherKey = kPH+bIxk5D2deZiIxcaaaA==
...
```

如果你想使用16位编码，你必须在字串前面加上 `0x` 前缀：

```
securityManager.rememberMeManager.cipherKey = 0x3707344A4093822299F31D008
```

###### 集合属性

列表（Lists）、集合（Sets）、图（Maps）可以像其它属性一样设置--直接设置或者像嵌套属性一样，对于列表和集合，只需指定一个逗号分割的值集或者对象引用集。

如定义一些SessionListeners：


```
sessionListener1 = com.company.my.SessionListenerImplementation
...
sessionListener2 = com.company.my.other.SessionListenerImplementation
...
securityManager.sessionManager.sessionListeners = $sessionListener1, $sessionListener2
```

对于图（Maps），你可以指定以逗号分割的键-值对列表，每个键-值之间用冒号分割

```
object1 = com.company.some.Class
object2 = com.company.another.Class
...
anObject = some.class.with.a.Map.property

anObject.mapProperty = key1:$object1, key2:$object2
```

在上面的例子中，$object1 引用的对象将存于键 key1 之下，也就是map.get("key1") 将返回 object1。你也可以使用其它对象作为键值：

```
anObject.map = $objectKey1:$objectValue1, $objectKey2:$objectValue2
...
```

##### 注意事项

###### 顺序问题

上述 INI 格式和约定非常方便也非常易懂，但它并没有另外一种 text/XML的配置路径强大，通过上述途径进行配置需要知道非常重要的一件事情就是顺序问题！

*小心*

*每一个对象实例以及每一个指定的值都将按照其在 [main] 区域中产生的顺序的执行，这些行最终转换为 JavaBeans 的 getter/setter 方法调用，这些方法按同样的顺序调用。*

当你写配置文件的时候要牢记于此。

###### 覆盖实例

每一个对象都可以被后定义的新实例覆盖，例如，第二个myRealm定义将重写第一个：

```
...
myRealm = com.company.security.MyRealm
...
myRealm = com.company.security.DatabaseRealm
...
```

这样的结果是 myRealm 是 com.company.security.DatabaseRealm 实例而前面的实例不会被使用（会作为垃圾回收）。

###### 默认 SecurityManager

你可能注意到在以上所有例子中都没有定义 SecurityManager，而我们直接设置其嵌套属性

```
myRealm = ...

securityManager.sessionManager.globalSessionTimeout = 1800000
...
```

这是因为securityManager实例是特殊的--它已经为你实例化过了并且准备好了，所以你并不需要知道指定的实例化SecurityManager的实现类。

当然，如果你确实想指定你自己的实现类，你可以像上面的覆盖实例那样定义你自己的实现：

```
...
securityManager = com.company.security.shiro.MyCustomSecurityManager
...
```

当然，很少需要这样--Shiro 的 SecurityManager 实现可以按需求进行定制，你可能要问一下自己（或者用户群）你是否真的需要这样做。

#### `[users]`

[users]区域允许你定义一组静态的用户帐号，这对于那些只有少数用户帐号并且用户帐号不需要在运行时动态创建的环境来说非常有用。下面是一个例子：

```
[users]
admin = secret
lonestarr = vespa, goodguy, schwartz
darkhelmet = ludicrousspeed, badguy, schwartz
```

*自动生成IniRealm*

定义非空的[users]或[roles]区域将自动创建org.apache.shiro.realm.text.IniRealm 实例,在[main]区域下生成一个可用的 iniRealm ，你可以像上面配置其它对象那样配置它。

##### 行格式

[users]区域下每一行必须和下面的形式一致：

```
username = password, roleName1, roleName2, …, roleNameN
```

* 等号左边的值是用户名；
* 等号右侧第一个值是用户密码，密码是必须的；
* 密码之后用逗号分割的值是赋予用户的角色名，角色名是可选的。

##### 密码加密

如果你不希望[users]区域下的密码以明文显示，你可以用你喜欢的哈希算法（MD5, Sha1, Sha256, 等）来加密它们，将加密后的字符串作为密码值，默认的，密码建议用16位编码算法，但也可以用64位编码算法替代（如下）

*简单的安全密码*

*为了节约时间获得最佳实践，你可以使用 Shiro 的 [Command Line Hasher](http://shiro.apache.org/command-line-hasher.html)，它可以加密密码和其它类型的资源，尤其使给 INI[user] 密码加密变得非常简单。*

一旦你指定了加密后的密码值，你必须告诉 shiro 它们是加密的，你可以通过配置配置在[main]隐含创建的iniRealm相应的CredentialsMatcher 实现来告知你使用的哈希算法：
	
```
[main]
...
sha256Matcher = org.apache.shiro.authc.credential.Sha256CredentialsMatcher
...
iniRealm.credentialsMatcher = $sha256Matcher
...

[users]
# user1 = sha256-hashed-hex-encoded password, role1, role2, ...
user1 = 2bb80d537b1da3e38bd30361aa855686bde0eacd7162fef6a25fe97bf527a25b, role1, role2, ...
```


你可以像配置其他对象那样配置 CredentialsMatcher 的所有属性，例如，指定使用salting或者有多少hash iterations执行，可以查看[org.apache.shiro.authc.credential.HashedCredentialsMatcher](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/credential/HashedCredentialsMatcher.html) Java文档更好地理解 hashing 策略，可能会很有帮助。

例如，如果你用64位编码方式取代了16位编码方式，你应该指定：

```
[main]
...
# true = hex, false = base64:
sha256Matcher.storedCredentialsHexEncoded = false
```

#### `[roles]`

[roles]区域允许你将权限和在[users]定义的角色对应起来，同样的，这对于那些只有少数用户帐号并且用户帐号不需要在运行时动态创建的环境来说非常有用。下面是一个例子：

```
[roles]
# 'admin' role has all permissions, indicated by the wildcard '*'
admin = *
# The 'schwartz' role can do anything (*) with any lightsaber:
schwartz = lightsaber:*
# The 'goodguy' role is allowed to 'drive' (action) the winnebago (type) with
# license plate 'eagle5' (instance specific id)
goodguy = winnebago:drive:eagle5
```


##### 行格式

[roles]区域下的每一行必须用下面的格式定义角色-权限的键/值对应关系。


```
rolename = permissionDefinition1, permissionDefinition2, …, permissionDefinitionN
```


权限定义可以是非常随意的字符串，但大部分用户还是希望使用易用而灵活的和 [org.apache.shiro.authz.permission.WildcardPermission](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/permission/WildcardPermission.html)形式一致的字符串格式。查看 [Permissions](../II. Core 核心/6.1. Permissions 权限.md) 文档获取更多关于权限的信息和你可以如何利用它为你服务。

*内部用法*

*注意如果一个特定的权限定义需要用到逗号分隔（如：printer:5thFloor:print,info），你需要将该定义用双引号括起来从而避免出错："printer:5thFloor:print,info"。*

*没有权限的角色*

*如果你有不需要权限的角色，不需要将它们列入[roles]区域，仅仅在 [users]区域定义角色名就可以创建它们（如果它们尚不存在）。*

#### `[urls]`

该区域选项将在[Web](../III. Web Applications/10. Web.md)章节讨论。

## 为文档加把手

我们希望这篇文档可以帮助你使用 Apache Shiro 进行工作，社区一直在不断地完善和扩展文档，如果你希望帮助 Shiro 项目，请在你认为需要的地方考虑更正、扩展或添加文档，你提供的任何点滴帮助都将扩充社区并且提升 Shiro。

提供你的文档的最简单的途径是将它发送到用户[论坛](http://shiro-user.582556.n2.nabble.com/)或[邮件列表](http://shiro.apache.org/mailing-lists.html)

*译者注：如果对本中文翻译有疑议的或发现勘误欢迎指正，[点此](https://github.com/waylau/apache-shiro-1.2.x-reference/issues)提问。*