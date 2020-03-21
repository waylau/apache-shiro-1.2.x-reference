# 3. Architecture 架构

Apache Shiro 设计理念是使程序的安全变得简单直观而易于实现，Shiro的核心设计参照大多数用户对安全的思考模式--如何对某人（或某事）在与程序交互的环境中的进行安全控制。

程序设计通常都以用户故事为基础，也就是说,你会经常设计用户接口或服务api基于用户如何(或应该)与软件交互。 例如,你可能会说,“如果我的应用程序的用户交互是登录,我将展示他们可以单击一个按钮来查看他们的帐户信息。 如果不登录,我将展示一个注册按钮。”

这个陈述例子指出我们开发程序很大程度上是为了满足用户的需求，即使“用户（User）”是另外一个软件系统而并非一个人，你仍然要写代码对当前与你软件交互的谁（或者什么）的动作进行回应。

Shiro 从它的设计中表现了这种理念，为了与软件开发者的直觉相配合，Apache Shiro 在几乎所有程序中保留了直观和易用的特性。

## 高级概述
 
在概念层，Shiro 架构包含三个主要的理念：Subject,SecurityManager和 Realm。下面的图展示了这些组件如何相互作用，我们将在下面依次对其进行描述。

![Shiro 架构](../images/ShiroBasicArchitecture.png)

* **Subject**:就像我们在上一章示例中提到的那样，Subject 本质上是当前运行用户特定的'View'(视图)，而单词“User”经常暗指一个人，Subject 可以是一个人，但也可以是第三方服务、守护进程帐户、时钟守护任务或者其它--当前和软件交互的任何事件。
Subject 实例都和（也需要）一个 SecurityManager 绑定，当你和一个Subject 进行交互，这些交互动作被转换成 SecurityManager 下Subject 特定的交互动作。

* **SecurityManager**: SecurityManager 是 Shiro 架构的核心，配合内部安全组件共同组成安全伞。然而，一旦一个程序配置好了SecurityManager 和它的内部对象，SecurityManager通常独自留下来，程序开发人员几乎花费的所有时间都集中在 Subjet API上。
我们将在以后详细讨论 SecurityManager，但当你和一个 Subject 互动时了解它是很重要的。任何 Subject 的安全操作中 SecurityManager 是幕后真正的举重者，这在上面的图表中可以反映出来。

* **Realms**： Reamls 是 Shiro 和你的程序安全数据之间的“桥”或者“连接”，它用来实际和安全相关的数据如用户执行身份认证（登录）的帐号和授权（访问控制）进行交互，Shiro 从一个或多个程序配置的 Realm 中查找这些东西。
Realm 本质上是一个特定的安全 [DAO](http://en.wikipedia.org/wiki/Data_access_object)：它封装与数据源连接的细节，得到Shiro 所需的相关的数据。在配置 Shiro 的时候，你必须指定至少一个Realm 来实现认证（authentication）和/或授权（authorization）。SecurityManager 可以配置多个复杂的 Realm，但是至少有一个是需要的。
Shiro 提供开箱即用的 Realms 来连接安全数据源（或叫地址）如 LDAP、JDBC、文件配置如INI和属性文件等，如果已有的Realm不能满足你的需求你也可以开发自己的Realm实现。
和其它内部组件一样，Shiro SecurityManager 管理如何使用 Realms获取 Subject 实例所代表的安全和身份信息。

## 详细架构

下面的图表展示了 Shiro 的核心架构思想，下面有简单的解释。

![详细架构](../images/ShiroArchitecture.png)

* **Subject** ([org.apache.shiro.subject.Subject](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html))正在与软件交互的一个特定的实体“view”（用户、第三方服务、时钟守护任务等）。
* **SecurityManager** ([org.apache.shiro.mgt.SecurityManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/mgt/SecurityManager.html))如同上面提到的，SecurityManager 是 Shiro 的核心，它基本上就是一把“保护伞”用来协调它管理的组件使之平稳地一起工作，它也管理着 Shiro 中每一个程序用户的视图，所以它知道每个用户如何执行安全操作。
* **Authenticator**([org.apache.shiro.authc.Authenticator](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/Authenticator.html))Authenticator 是一个组件，负责执行和反馈用户的认证（登录），如果一个用户尝试登录，Authenticator 就开始执行。Authenticator 知道如何协调一个或多个保存有相关用户/帐号信息的 Realm，从这些 Realm中获取这些数据来验证用户的身份以确保用户确实是其表述的那个人。
    * **Authentication Strategy**([org.apache.shiro.authc.pam.AuthenticationStrategy](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/pam/AuthenticationStrategy.html))如果配置了多个 Realm，AuthenticationStrategy 将会协调 Realm 确定在一个身份验证成功或失败的条件（例如，如果在一个方面验证成功了但其他失败了，这次尝试是成功的吗？是不是需要所有方面的验证都成功？还是只需要第一个？）
* **Authorizer**([org.apache.shiro.authz.Authorizer](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Authorizer.html))Authorizer 是负责程序中用户访问控制的组件，它是最终判断一个用户是否允许做某件事的途径，像 Authenticator 一样，Authorizer 也知道如何通过协调多种后台数据源来访问角色和权限信息，Authorizer 利用这些信息来准确判断一个用户是否可以执行给定的动作。
* **SessionManager**([org.apache.shiro.session.mgt.SessionManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Authorizer.html))SessionManager 知道如何创建并管理用户 Session 生命周期而在所有环境中为用户提供一个强有力的 Session 体验。这在安全框架领域是独一无二--Shiro 具备管理在任何环境下管理用户 Session 的能力，即使没有 Web/Servlet 或者 EJB 容器。默认情况下，Shiro 将使用现有的session（如Servlet Container），但如果环境中没有，比如在一个独立的程序或非 web 环境中，它将使用它自己建立的 session 提供相同的作用，sessionDAO 用来使用任何数据源使 session 持久化。
    * **SessionDAO**([org.apache.shiro.session.mgt.eis.SessionDAO](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/eis/SessionDAO.html))SessionDAO 代表 SessionManager 执行 Session 持久（CRUD）动作，它允许任何存储的数据挂接到 session 管理基础上。
* **CacheManager**([org.apache.shiro.cache.CacheManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/CacheManager.html))CacheManager 为 Shiro 的其他组件提供创建缓存实例和管理缓存生命周期的功能。因为 Shiro 的认证、授权、会话管理支持多种数据源，所以访问数据源时，使用缓存来提高访问效率是上乘的选择。当下主流开源或企业级缓存框架都可以继承到 Shiro 中，来获取更快更高效的用户体验。
* **Cryptography** ([org.apache.shiro.crypto.*](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/crypto/package-summary.html))Cryptography 在安全框架中是一个自然的附加产物，Shiro 的 crypto 包包含了易用且易懂的加密方式，Hashes（即digests）和不同的编码实现。该包里所有的类都亦于理解和使用，曾经用过 Java 自身的加密支持的人都知道那是一个具有挑战性的工作，而 Shiro 的加密 API 简化了 java 复杂的工作方式，将加密变得易用。
* **Realms** ([org.apache.shiro.realm.Realm](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/Realm.html))如同上面提到的，Realm 是 shiro 和你的应用程序安全数据之间的“桥”或“连接”，当实际要与安全相关的数据进行交互如用户执行身份认证（登录）和授权验证（访问控制）时，shiro 从程序配置的一个或多个Realm 中查找这些数据，你需要配置多少个 Realm 便可配置多少个 Realm（通常一个数据源一个），shiro 将会在认证和授权中协调它们。

## SecurityManager

因为 Shiro API 鼓励以 Subject 为中心的开发方式，大部分开发人员将很少会和 SecurityManager 直接交互（尽管框架开发人员也许发现它非常有用），尽管如此，知道 SecurityManager 如何工作，特别是当在一个程序中进行配置的时候，是非常重要的。

## 设计

如前所述，程序中 SecurityManager 执行操作并且管理所有程序用户的状态，在 Shiro 基础的 SecurityManager 实现中，包含以下内容：

* 认证（Authentication）
* 授权（Authorization）
* 会话管理（Session Management）
* 缓存管理（Cache Management）
* Realm协调（Realm coordination）
* 事件传导（Event propagation ）
* "RememberMe" 服务（"Remember Me" Services）
* 建立Subject(Subject creation)
* 退出登录（Logout）
* 等等

但这些功能都在一个单独的组件中管理，并且，当所有功能集中在一个类中实现是灵活和可定制是非常困难的。

为了实现配置的简单、灵活、可插拔，Shiro在设计时实现了高模块化--尽管模块化，SecurityManager（包括它的继承类）并没有做到，相反地，SecurityManager实现更像一个轻量级的‘容器（container）’，代表几乎所有嵌套/封装组件的行为，这种‘封装（wrapper）’设计在上面的架构图表中已有反映。

当组件执行逻辑的时候，SecurityManager 知道如何以及何时去协调组件做出正确的动作。

SecurityManager 和 JavaBean 兼容，这允许你（或者配置途径）通过标准的J avaBean 访问/设置方法（get*/set*）很容易地定制插件，这意味着 Shiro 模块可以根据用户行为转化成简易的配置。

*简易的配置*

*因为适合JavaBean，任何支持Javabean配置的组件都有非常简单的途径配置 SecurityManager，如 Spring、Guice、JBoss,等等。*

我们将在下一节讨论配置（[Configuration](4. Configuration 配置.md) ）

## 为文档加把手

我们希望这篇文档可以帮助你使用 Apache Shiro 进行工作，社区一直在不断地完善和扩展文档，如果你希望帮助 Shiro 项目，请在你认为需要的地方考虑更正、扩展或添加文档，你提供的任何点滴帮助都将扩充社区并且提升 Shiro。

提供你的文档的最简单的途径是将它发送到用户[论坛](http://shiro-user.582556.n2.nabble.com/)或[邮件列表](http://shiro.apache.org/mailing-lists.html)

*译者注：如果对本中文翻译有疑议的或发现勘误欢迎指正，[点此](https://github.com/waylau/apache-shiro-1.2.x-reference/issues)提问。*

