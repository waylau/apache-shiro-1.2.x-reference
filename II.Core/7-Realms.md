# 7. Realms

Realm 是可以访问程序特定的安全数据如用户、角色、权限等的一个组件。Realm 会将这些程序特定的安全数据转换成一种 Shiro 可以理解的形式，Shiro 就可以依次提供容易理解的 [Subject](../IV. Auxiliary Support 辅助支持/14. Custom Subjects 自定义 Subject.md) 程序API而不管有多少数据源或者程序中你的数据如何组织。

Realm 通常和数据源是一对一的对应关系，如关系数据库，LDAP 目录，文件系统，或其他类似资源。因此，Realm 接口的实现使用数据源特定的API 来展示授权数据（角色，权限等），如JDBC，文件IO，Hibernate 或JPA，或其他数据访问API。

*Realm 实质上就是一个特定安全的 [DAO](http://en.wikipedia.org/wiki/Data_Access_Object)*

因为这些数据源大多通常存储身份验证数据（如密码的凭证）以及授权数据（如角色或权限），每个 Shiro Realm 能够执行身份验证和授权操作。

## Realm 配置

如果使用 Shiro 的 ini 配置文件，你可以在[main]区域内像配置其它对象一样定义和引用Realms，但是 Realm 在 secrityManager上的配置有两种方式：显式方式和隐含方式。 

### 显式指定

在迄今所知的INI配置文件的相关知识中，这是一种显示的配置方式。在定义一个或多个Realm后，再将它们在securityManager上进行统一配置。

例如：

	fooRealm = com.company.foo.Realm
	barRealm = com.company.another.Realm
	bazRealm = com.company.baz.Realm
	
	securityManager.realms = $fooRealm, $barRealm, $bazRealm

显式设置是确定性的，你可以非常确切地知道哪个 realm 在使用并且知道它们执行的顺序。可以查看[认证](../II. Core 核心/5. Authentication 认证.md)章节的 Authentication Sequence 了解 Realm 的执行顺序的影响效果

###Implicit Assignment隐含方式(隐式)

*Not Preferred(不推荐)*

*这种方法可能引发意想不到的行为，如果你改变 realm 定义的顺序的话。建议你避免使用此方法，并使用显式分配，它拥有确定的行为。该功能很可能在未来的 Shiro 版本中被废弃或移除。*

如果出于某些原因你不想显式地配置 securityManager.realms 的属性，你可以允许 Shiro 检测所有配置好的 realm 并直接将它们指派给securityManager。

使用这种方法，realm 将会按照它们预先定义好的顺序来指派给 securityManager 实例。

也就是说，对于下面的 shiro.ini 示例：

	blahRealm = com.company.blah.Realm
	fooRealm = com.company.foo.Realm
	barRealm = com.company.another.Realm
	
	# no securityManager.realms assignment here

基本上和下面这一行具有相同的效果：

	securityManager.realms = $blahRealm, $fooRealm, $barRealm

然而，实现隐式分配，只是 realm 定义的顺序直接影响到了它们在身份验证和授权尝试中的访问顺序。如果你改变它们定义的顺序，你将改变主要的[认证](../II. Core 核心/5. Authentication 认证.md)章节的 Authentication Sequence 的Authentication Sequence 是如何起作用的。由于这个原因，以及保证显式的行为，我们推荐使用显式分配而不是隐式分配。

## Realm 认证 

当你理解了Shiro 的主要 [认证](https://github.com/waylau/apache-shiro-1.2.x-reference/blob/master/II.%20Core%20%E6%A0%B8%E5%BF%83/5.%20Authentication%20%E8%AE%A4%E8%AF%81.md) 工作流后，了解在一个授权尝试中当 Authenticator 与 Realm 交互时到底发生了什么是很重要的。

### 支持AuthenticationTokens

正如在认证流程中提到的，在一个 Realm 执行一个验证尝试之前，它的[supports](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/Realm.html#supports(org.apache.shiro.authc.AuthenticationToken))方法被调用。只有在返回值为 true 的时候它的getAuthenticationInfo(token) 方法才会执行。

通常情况下，一个 realm 将检查提交的令牌类型（接口或类）确定自己是否可以处理它，例如，一个处理生物特性数据的Realm 可能一点也不理解 UsernamePasswordTokens，在这种情况下它将从支持函数中返回 false。

### 处理支持的AuthenticationTokens

如果一个Realm支持提交的验证令牌，验证将调用 Realm 的[getAuthenticationInfo(token)](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/Realm.html#getAuthenticationInfo(org.apache.shiro.authc.AuthenticationToken)) 方法，这是Realm 使用后台数据进行验证的一次有效尝试，顺序执行以下动作：

1.检查主要 principal (身份)令牌（用户身份信息）；

2.基于主要 principal (信息)，在数据源中查找对应的用户数据；

3.确定令牌支持的 credentials (凭证数据)和存储的数据相符；

4.如果凭证相符，返回一个[AuthenticationInfo](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/AuthenticationInfo.html)实例，里面封装了 Shiro 可以理解的用户数据。

5.如果证据不符，抛出 [AuthenticationException](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/AuthenticationException.html)异常。


这是所有Realm getAuthenticationInfo 实现的最高级别工作流，Realm 在这个过程中可以自由做自己想做的事情，比如记录日志，修改数据，以及其他，只要对于存储的数据和验证尝试来讲是合理的就行。

仅有一件事情是必须的，如果 credentials （凭证）和给定的 principal （主要信息）匹配，需要返回一个非空的 AuthenticationInfo 实例，用来表示来自数据源的 Subject 账户信息。

*节约时间*

*直接实现 [Realm](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/Realm.html) 接口也许需要时间并容易出错，大部分用户选择继承 [AuthorizingRealm](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/AuthorizingRealm.html) 虚拟类，这个类实现了常用的认证和授权工作流，这会节省你的时间而且不易出错。*


### 凭证匹配

在上述 realm 认证工作流中，一个 Realm 必须较验 Subject 提交的凭证（如密码）是否与存储在数据中的凭证相匹配，如果匹配，验证成功，系统保留已认证的终端用户身份。

*Realm凭证匹配*

*检查提交的凭证是否与后台存储数据相匹配是每一个 Realm 的责任而不是 Authenticator 的责任，每一个 Realm 都具备与凭证形式及存储密切相关的技能，可以执行详细的凭证比对，而 Authenticator 只是一个普通的工作流组件。*

凭证匹配的过程在所有程序中基本上是一样的，通常只是对比数据方式不同。要确保这个过程在必要时是可插拔和可定制的，[AuthenticatingRealm](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/AuthenticatingRealm.html) 以及它的子类支持用 [CredentialsMatcher](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/credential/CredentialsMatcher.html) 来执行一个凭证对比。

在找到用户数据之后，它和提交的 AuthenticationToken 一起传递给一个 CredentialsMatcher ，后者用来检查提交的数据和存储的数据是否相匹配。

Shiro某些 CredentialsMatcher 实现可以使你开箱即用，比如 [SimpleCredentialsMatcher](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/credential/SimpleCredentialsMatcher.html) 和 [HashedCredentialsMatcher](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/credential/HashedCredentialsMatcher.html) 实现，但如果你想配置一个自定义的实现来完成特定的对比逻辑，你可以这样做：

	Realm myRealm = new com.company.shiro.realm.MyRealm();
	CredentialsMatcher customMatcher = new com.company.shiro.realm.CustomCredentialsMatcher();
	myRealm.setCredentialsMatcher(customMatcher);

或者，使用 Shiro 的 INI[配置](../I. Overview 总览/4. Configuration 配置.md)文件

	[main]
	...
	customMatcher = com.company.shiro.realm.CustomCredentialsMatcher
	myRealm = com.company.shiro.realm.MyRealm
	myRealm.credentialsMatcher = $customMatcher
	...

#### 简单证明匹配

所有 Shiro 的开箱即用 Realm 默认使用一个 [SimpleCredentialsMatcher](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/credential/SimpleCredentialsMatcher.html)， SimpleCredentialsMatcher 对存储的用户凭证和从 AuthenticationToken 提交的用户凭证直接执行相等检查。

例如，如果提交了一个[UsernamePasswordToken](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/UsernamePasswordToken.html)，SimpleCredentialsMatcher 检查提交的密码与存储的密码是否完全相等。

SimpleCredentialsMatcher 不仅仅对字符串执行相同对比，它可以对大多数常用类型，如字符串、字符数组、字节数组、文件和输入流等执行对比，查看 JavaDoc 获取更多的信息。

#### 哈希凭证

取代将凭证按它们原始形式存储并执行原始数据的对比，存储终端用户的凭证（如密码）更安全的办法是在存储数据之前，先进行 hash 运算。

这确保终端用户的凭证不会以他们原始的形式存储，没有人能知道其原始值。与明文原始比较相比这是一种更为安全的做法，有安全意识的程序会更喜欢这种方法。

要支持这种加密的 hash 策略，Shiro 为 Realm 配置提供了一个[HashedCredentialsMatcher](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/credential/HashedCredentialsMatcher.html) 实现替代之前的 SimpleCredentialsMatcher。

Hashing 凭证以及 hash 迭代的好处超出了该 Realm 文档的范围，可以在[HashedCredentialsMatcher JavaDoc](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/credential/HashedCredentialsMatcher.html)更详细地了解这些主要内容。

##### 哈希以及相符合的匹配

对于一个使用 Shiro 的程序，如何配置才能简单地做到这些？

Shiro 提供了多个 HashedCredentialsMatcher 子类实现，你必须在你的 Realm 上配置指定的实现来匹配你的凭证所使用的 hash 算法。

例发，假设你的程序使用用户名/密码对来进行验证，基于上述 hash 凭证的好处，你希望当创建用户时以 [SHA-265](http://en.wikipedia.org/wiki/SHA_hash_functions) 方式加密用户的密码，你可以加密用户输入的明文密码并保存加密值：

	import org.apache.shiro.crypto.hash.Sha256Hash;
	import org.apache.shiro.crypto.RandomNumberGenerator;
	import org.apache.shiro.crypto.SecureRandomNumberGenerator;
	...
	
	//我们将使用一个随机数发生器产生的盐。
	//这比使用一个用户名作为盐或不使用盐更加安全。
	//Shiro使这个实现变得容易。
	//
	//注意一个正常的应用程序将引用属性而
	//不是每次创建一个新的RNG：
	RandomNumberGenerator rng = new SecureRandomNumberGenerator();
	Object salt = rng.nextBytes();
	
	//我们的纯文本密码经过散列随机盐和多次迭代，
	//得到Base64编码的值（比Hex需要较少的空间）：
	String hashedPasswordBase64 = new Sha256Hash(plainTextPassword, salt, 1024).toBase64();
	
	User user = new User(username, hashedPasswordBase64);
	//在新帐户保存盐。该 HashedCredentialsMatcher
	//稍后再的登录尝试的时候会处理它：
	user.setPasswordSalt(salt);
	userDAO.create(user);


由于你使用 SHA-256 加密你的密码，你需要告诉 Shiro 使用相应的 HashedCredentialsMatcher 来检查你的 hashing 值，在这个例子中，我们为了加强安全创建了一个随机的 salt 并且执行 1024 Hash 迭代（查看HashedCredentialsMatcher JAVADoc了解为什么），下面的Shiro INI 配置来做这件工作。

	[main]
	...
	credentialsMatcher = org.apache.shiro.authc.credential.Sha256CredentialsMatcher
	# base64 编码, 例子中没有 hex:
	credentialsMatcher.storedCredentialsHexEncoded = false
	credentialsMatcher.hashIterations = 1024
	# 下面属性只在 Shiro 1.0 需要，在 1.1 及以后版本移除了:
	credentialsMatcher.hashSalted = true
	
	...
	myRealm = com.company.....
	myRealm.credentialsMatcher = $credentialsMatcher
	...

##### HSaltedAuthenticationInfo

确保正常运行的最后一件要做的事情是你的Realm实现必须返回一个 [SaltedAuthenticationInfo](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/SaltedAuthenticationInfo.html) 实例而不是普通的AuthenticationInfo，SaltedAuthenticationInfo 接口确保你在创建用户帐户时使用的salt（如上面调用的 user.setPasswordSalt(salt);）能被 HashedCredentialsMatcher 引用。

HashedCredentialsMatcher 需要使用 salt 来对提交的 AuthenticationToken 执行相同的 hashing 技术来对比提交的令牌是否与存储的数据相匹配，所以如果你对用户密码使用 salting（你应该这么做），确保你的 Realm 实现在返回 SaltedAuthenticationInfo 实例时引用它。

### 禁用认证

如果有理由，你不希望某个 Realm 对某个资源执行验证（或者因为你只想 Realm 去执行授权检查），你可以完全禁用 Realm 的认证支持，方法就是在 Realm 的 supports 方法中始终返回 false,这样，你的 Realm 将在整个验证过程中不再被使用。 

当然如果你想验证 Subject，至少要配置一个支持 AuthenticationTokens 的 Realm。

## Realm 授权

SecurityManager将权限或角色检查任务委托给Authorizer，默认为ModularRealmAuthorizer。

### 基于角色的授权

在Subject 上调用重载方法hasRoles或checkRoles方法之一时

1. Subject 委托给SecurityManager以确定是否已分配给定角色
2. 然后，SecurityManager委托给授权者
3. 然后，Authorizer 一一引荐所有授权领域，直到找到分配给该主题的给定角色。如果没有任何领域授予主题，则返回false来拒绝访问
4. 授权Realm AuthorizationInfo getRoles()方法以获取分配给Subject的所有角色
5. 如果在AuthorizationInfo.getRoles调用返回的角色列表中找到给定的Role，则授予访问权限。


### 基于权限的授权

在Subject 上调用重载方法之一isPermitted()或checkPermission()方法时：

1. Subject 将任务委派给SecurityManager授予或拒绝权限
2. 然后，SecurityManager委托给授权者
3. 然后，Authorizer 逐一引用所有授权者领域，直到授予权限为止
4. 如果未由任何授权领域授予许可，则主题被拒绝该许可
5. 授权Realm 执行以下操作以检查是否允许主题的一种。
	* a. 首先，它通过在AuthorizationInfo上调用getObjectPermissions()和getStringPermissions方法并汇总结果来直接标识分配给Subject的所有权限。
	* b. 如果注册了RolePermissionResolver，则可通过调用RolePermissionResolver.resolvePermissionsInRole()来基于分配给Subject的所有角色检索Permissions。
	* c. 对于来自a的汇总权限。和b。调用implies()方法以检查这些权限中的任何一个是否隐含已检查的权限。请参阅WildcardPermission

## 为文档加把手

我们希望这篇文档可以帮助你使用 Apache Shiro 进行工作，社区一直在不断地完善和扩展文档，如果你希望帮助 Shiro 项目，请在你认为需要的地方考虑更正、扩展或添加文档，你提供的任何点滴帮助都将扩充社区并且提升 Shiro。

提供你的文档的最简单的途径是将它发送到用户[论坛](http://shiro-user.582556.n2.nabble.com/)或[邮件列表](http://shiro.apache.org/mailing-lists.html)

*译者注：如果对本中文翻译有疑议的或发现勘误欢迎指正，[点此](https://github.com/waylau/apache-shiro-1.2.x-reference/issues)提问。*