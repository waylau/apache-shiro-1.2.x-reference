# 2. Tutorial 教程


## 第一个 Shiro 应用

如果你是 Apache Shiro 新手,这个简短的教程将向您展示如何通过Apache Shiro 设置一个初始和非常简单的安全应用程序。接下来我们将讨论 Shiro 的核心概念,帮助你熟悉 Shiro 的设计和 API。

当你跟随本教程时，如果你不想编辑文件,您可以获得一个几乎相同的示例作为参考。 选择一个地址:

* 在Apache Shiro 的 Subversion 存储库: 
[https://svn.apache.org/repos/asf/shiro/trunk/samples/quickstart/](https://svn.apache.org/repos/asf/shiro/trunk/samples/quickstart/)
* 在Apache Shiro 的源码发布 samples/quickstart 目录中。 源码可以[下载](http://shiro.apache.org/download.html)。
* （*译者注*：译者也提供了自己的代码，包含了中文注解，本章所含示例如下）
	* [示例1](https://github.com/waylau/apache-shiro-1.2.x-reference-demos/tree/master/shiro-tutorial)
	* [示例2](https://github.com/waylau/apache-shiro-1.2.x-reference-demos/tree/master/shiro-tutorial-1)
	* [示例3](https://github.com/waylau/apache-shiro-1.2.x-reference-demos/tree/master/shiro-tutorial-2)

### 设置

在这个简单的示例中,我们将创建一个非常简单的命令行应用程序,它将运行并迅速退出,这样你可以领略到 Shiro 的API。

*任何应用程序*

*Apache Shiro设计从一开始就支持**任何**应用程序——从最小的命令行应用程序最大的集群 web 应用程序。对于本教程，尽管我们创建一个简单的应用程序,你都知道运用相同的使用模式来进行应用程序创建或部署。*

本教程需要 Java 1.5 或更高版本。 我们还将使用 Apache [Maven](http://maven.apache.org/) 作为我们的构建工具,当然这对于 Apache Shiro 来说不是必须使用。你可能获得 Shiro 的 jars,把他们以任何方式你喜欢到您的应用程序,例如也许使用Apache [Ant](http://ant.apache.org/) 和  [Ivy](http://ant.apache.org/ivy) 。

对于本教程,请确保您使用 Maven 2.2.1 或更高版本。为了测试 Maven 安装是否正确，命令行下运行 mvn --version  并看到类似如下:

```
$ mvn -version
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: D:\Program Files\apache-maven-3.6.3\bin\..
Java version: 14, vendor: Oracle Corporation, runtime: D:\Program Files\jdk-14
Default locale: zh_CN, platform encoding: GBK
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```

现在,在你的文件系统中创建一个新目录,例如, shiro-tutorial 作为项目名并在目录下保存以下 Maven `pom.xml` 文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>org.apache.shiro.tutorials</groupId>
    <artifactId>shiro-tutorial</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>First Apache Shiro Application</name>
    <packaging>jar</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>1.6</source>
                    <target>1.6</target>
                    <encoding>${project.build.sourceEncoding}</encoding>
                </configuration>
            </plugin>

        <!-- This plugin is only to test run our little application.  It is not
             needed in most Shiro-enabled applications: -->
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.1</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>java</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <classpathScope>test</classpathScope>
                    <mainClass>Tutorial</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
            <version>1.4.1</version>
        </dependency>
        <!-- Shiro uses SLF4J for logging.  We'll use the 'simple' binding
             in this example app.  See http://www.slf4j.org for more info. -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.21</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>jcl-over-slf4j</artifactId>
            <version>1.7.21</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

#### 教程中的 class

我们将运行一个简单的命令行应用程序,因此我们将需要创建一个带 `public static void main(String[] args)` 方法 Java 类。

包含 pom.xml 文件的同一个目录下,创建一个*src/main/java 子目录。 在 src/main/java 创建一个 Tutorial.java 文件，包含以下内容:

src/main/java/Tutorial.java

```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.session.Session;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Tutorial {

    private static final transient Logger log = LoggerFactory.getLogger(Tutorial.class);

    public static void main(String[] args) {
        log.info("My First Apache Shiro Application");
        System.exit(0);
    }
}
```

先不要管引入包的问题。下午将会很快提到。我们先测试下这个应用，会输出 "My First Apache Shiro Application" 并且退出。

### 测试运行

在教程项目的根目录(如 shiro-tutorial )执行以下命令提示符中,输入以下:

```
>mvn compile exec:java
```

你就会看到我们的小教程应用程序的运行和退出。 您应当会看到类似于下面的输出：

```
$ mvn compile exec:java

... a bunch of Maven output ...

1 [Tutorial.main()] INFO Tutorial - My First Apache Shiro Application
```

我们已经验证了应用程序成功运行——现在让我们使 Apache Shiro。当我们继续学习教程,每次我们添加更多的代码之后,您可以运行 mvn compile exec:java 看到我们的变化的结果。

### 启用 Shiro

使用 Shiro 要理解的第一件事情是 Shiro 几乎所有的事情都和一个中心组件 SecurityManager 有关，对于那些熟悉 Java security 的人请注意：这和 java.lang.SecurityManager 不是一回事。

我们将在Architecture章节详细描述 Shiro 的设计，但现在有必要知道 Shrio SecurityManager 是程序中 Shiro 的核心，每一个程序都必定会存在一个 SecurityManager，所以，在我们这个示例程序中必须做的第一件事情是建立一个 SecurityManager 实例。

#### Configuration 配置

虽然我们可以直接对 SecurityManager 实例化，但在 Java 代码中对Shiro 的 SecurityManager 所须的选项和内部组件进行配置会让人感觉有点小痛苦--而将这些 SecurityManager 配置用一个灵活的配置文件实现就会简单地多。

为此，Shiro 默认提供了一个基本的 INI 配置文件的解决方案，人们已经对庞大的 XML 文件有些厌倦了，而一个 INI 文件易读易用，而且所依赖的组件很少，稍后你就会通过一个简单易懂的示例明白 INI 在对简单对象进行配置的时候是非常有效率的，比如 SecurityManager

*多种配置选择*

*Shiro 的 SecurityManager 的实现和其所依赖的组件都是 JavaBean，所以可以用多种形式对 Shiro 进行配置，比如XML（Spring, JBoss, Guice, 等等），[YAML](http://www.yaml.org/), JSON, Groovy Builder markup，及其它，INI 只是 Shiro 一种最基本的配置方式，使得其可以在任何环境中进行配置比如在那些没有以上配置形式的环境中。*

##### shiro.ini

在这个示例中我们使用一个 INI 文件来配置Shiro SecurityManager，首先，在 pom.xml 同目录中创建一个src/main/resources子目录，在该子目录中创建一个 shiro.ini 文件，内容如下：

src/main/resources/shiro.ini

```
# =============================================================================
# Tutorial INI configuration
#
# Usernames/passwords are based on the classic Mel Brooks' film "Spaceballs" :)
# =============================================================================

# -----------------------------------------------------------------------------
# Users and their (optional) assigned roles
# username = password, role1, role2, ..., roleN
# -----------------------------------------------------------------------------
[users]
root = secret, admin
guest = guest, guest
presidentskroob = 12345, president
darkhelmet = ludicrousspeed, darklord, schwartz
lonestarr = vespa, goodguy, schwartz

# -----------------------------------------------------------------------------
# Roles with assigned permissions
# roleName = perm1, perm2, ..., permN
# -----------------------------------------------------------------------------
[roles]
admin = *
schwartz = lightsaber:*
goodguy = winnebago:drive:eagle5
```

可以看到，在该配置文件中最基础地配置了几个静态的帐户，对我们这一个程序已经足够了，在以后的章节中，将会看到如何使用更复杂的用户数据比如数据库、LDAP 和活动目录等。

#### Referencing the Configuration 引用配置

现在我们已经定义了一个 INI 文件，我们可以在我们的示例程序中创建SecurityManager 实例了，将 main 函数中的代码进行如下调整：
	
```java
public static void main(String[] args) {

    log.info("My First Apache Shiro Application");

    //1.
    Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");

    //2.
    SecurityManager securityManager = factory.getInstance();

    //3.
    SecurityUtils.setSecurityManager(securityManager);

    System.exit(0);
}
```

这就是我们要做的--仅仅使用三行代码就把Shiro加进了我们的程序，就是这么简单。

执行mvn compile exec:java 可以看到程序成功的运行（由于 Shiro 默认在 debug 或更底层才记录日志，所以你不会看到任何 Shiro 的日志输出--只要运行时没有错误提示，你就可以知道已经成功了）。

上面所加入的代码做了下面的事情：

* 1. 使用 Shiro 的 IniSecurityManagerFactory 加载了我们的shiro.ini 文件，该文件存在于 classpath 根目录里。这个执行动作反映出 shiro 支持 [Factory Method Design Pattern（工厂模式）](http://en.wikipedia.org/wiki/Factory_method_pattern)。classpath：资源的指示前缀，告诉 shiro 从哪里加载 ini 文件（其它前缀，如 url:和 file: 也被支持）。
* 2. factory.getInstance() 方法被调用，该方法分析 INI 文件并根据配置文件返回一个 SecurityManager 实例。
* 3. 在这个简单示例中，我们将 SecurityManager 设置成了static (memory) singleton，可以通过 JVM 访问，注意如果你在一个 JVM 中加载多个使用 shiro 的程序时不要这样做，在这个简单示例中，这是可以的，但在其它成熟的应用环境中，通常会将 SecurityManager 放在程序指定的存储中（如在 web 中的 ServletContexct 或者 Spring、Guice、 JBoss DI 容器实例）中。

#### Using Shiro 使用

现在我们的 SecurityManager 已经准备好了，我们可以开始进行我们真正关心的事情--执行安全操作了。

为了保护我们的程序安全，我们或许问自己最多的问题就是“谁是当前的用户？”或者“当前用户是否允许做某件事？”通常我们会在写代码或者设计用户接口的时候问这些问题：程序通常建立在用户基础上，程序功能展示（和安全）也基于每一个用户。所以，通常我们考虑我们程序安全的方法也建立在当前用户的基础上，Shiro 的 API 提供了'the current user'概念，即 Subject。

在几乎所有的环境中，你可以通过如下语句得到当前用户的信息：

```java
Subject currentUser = SecurityUtils.getSubject();
```

使用 SecurityUtils.getSubject()，我们可以获取当前执行的Subject，Subject是一个安全术语意思是“当前运行用户的指定安全视图（a security-specific view of the currently executing user）”，这里并不称之为“User”因为“User”这个词通常和一个人相关，但在安全认证中，“Subject”可以认为是一个人，也可以认为是第三方进程、时钟守护任务、守护进程帐户或者其它。它可简单描述为“当前和软件进行交互的事件”，在大多数情况下，你可以认为它是一个“人（User）”。

在一个独立的程序中调用 getSubject() 会在程序指定位置返回一个基于用户数据的 Subject，在服务器环境（如 web 程序）中，它将获取一个和当前线程或请求相关的基于用户数据的 Subject。

现在你得到了Subject，你可以利用它做什么呢？

如果你针对该用户希望一些事情在程序当前会话期内可行，你可以获取他们的 session：

```java
Session session = currentUser.getSession();
session.setAttribute( "someKey", "aValue" );
```

Session 是 shiro 指定的一个实例，提供基本上所有 HttpSession 的功能，但具备额外的好处和不同：它不需要一个 HTTP 环境！

如果发布到一个 web 程序中，默认情况下 Session 将会使用HttpSession 作为基础，但是，在一个非 web 程序中，比如该简单示例程序中，Shiro 将自动默认使用它的 Enterprise Session Management，这意味着你可以在任何程序中使用相同的 API，而根本不需要考虑发布环境！这打开了一个全新的世界，从此任何需要 session 的程序不再需要强制使用 HttpSession 或者 EJB Stateful Session,并且，终端可以共享 session 数据。

现在你可以获取一个 Subject 和它们的 Session，真正填充有用的代码如检测其是否被允许做某些事情如何？比如检查其角色和权限？

我们只能对一个已知用户做这些检测，如上我们获取 Subject 实例表示当前用户，但是当前用户是认证，嗯，他们是任何人--直到他们至少登录一次，我们现在就做这件事情：

```java
if ( !currentUser.isAuthenticated() ) {
	//收集用户的主要信息和凭据，来自GUI中的特定的方式
	//如包含用户名/密码的HTML表格，X509证书，OpenID，等。
	//我们将使用用户名/密码的例子因为它是最常见的。.
	UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");

	//支持'remember me' (无需配置，内建的!):
	token.setRememberMe(true);

	currentUser.login(token);
}
```

就是这样，不能再简单了。

但如果登录失败了呢，你可以捕获所有异常然后按你期望的方式去处理：

```java
try {
	currentUser.login( token );
	//无异常，说明就是我们想要的!
} catch ( UnknownAccountException uae ) {
	//username 不存在，给个错误提示?
} catch ( IncorrectCredentialsException ice ) {
	//password 不匹配，再输入?
} catch ( LockedAccountException lae ) {
	//账号锁住了，不能登入。给个提示?
} 
	... 更多类型异常 ...
} catch ( AuthenticationException ae ) {
	//未考虑到的问题 - 错误?
}
```
这里有许多不同类别的异常你可以检测到，也可以抛出你自己异常。详见
[AuthenticationException JavaDoc](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/AuthenticationException.html)

*小贴士：*

*最好的方式是将普通的失败信息反馈给用户，你总不会希望帮助黑客来攻击你的系统吧。*

好，到现在为止，我们有了一个登录用户，接下来我们还可以做什么？
 
让我们显示他们是谁

```java
//打印主要信息 (本例子是 username):
log.info( "User [" + currentUser.getPrincipal() + "] logged in successfully." );
```


我们也可以判断他们是否拥有某个角色：


```java
if ( currentUser.hasRole( "schwartz" ) ) {
	log.info("May the Schwartz be with you!" );
} else {
	log.info( "Hello, mere mortal." );
}
```


我们也可以判断他们是否拥有某个特定动作或入口的权限：

```java	
if ( currentUser.isPermitted( "lightsaber:weild" ) ) {
	log.info("You may use a lightsaber ring.  Use it wisely.");
} else {
	log.info("Sorry, lightsaber rings are for schwartz masters only.");
}
```

同样，我们还可以执行非常强大的 instance-level （实例级别）的权限检测，检测用户是否具备访问某个类型特定实例的权限：


```java
if ( currentUser.isPermitted( "winnebago:drive:eagle5" ) ) {
	log.info("You are permitted to 'drive' the 'winnebago' with license plate (id) 'eagle5'.  " +
				"Here are the keys - have fun!");
} else {
	log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
}
```

轻而易举，是吧！

最后，当用户不再使用系统，可以退出登录：

```
currentUser.logout(); //清楚识别信息，设置 session 失效.
```

#### 最终的 class

在加入上述代码后，下面的就是我们完整的文件，你可以自由编辑和运行它，可以尝试改变安全检测（以及INI配置）：

src/main/java/Tutorial.java
	
```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.session.Session;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Tutorial {

	private static final transient Logger log = LoggerFactory.getLogger(Tutorial.class);


	public static void main(String[] args) {
		log.info("My First Apache Shiro Application");

		Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
		SecurityManager securityManager = factory.getInstance();
		SecurityUtils.setSecurityManager(securityManager);


		// 获取当前执行用户:
		Subject currentUser = SecurityUtils.getSubject();

		// 做点跟 Session 相关的事
		Session session = currentUser.getSession();
		session.setAttribute("someKey", "aValue");
		String value = (String) session.getAttribute("someKey");
		if (value.equals("aValue")) {
			log.info("Retrieved the correct value! [" + value + "]");
		}

		// 登录当前用户检验角色和权限
		if (!currentUser.isAuthenticated()) {
			UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
			token.setRememberMe(true);
			try {
				currentUser.login(token);
			} catch (UnknownAccountException uae) {
				log.info("There is no user with username of " + token.getPrincipal());
			} catch (IncorrectCredentialsException ice) {
				log.info("Password for account " + token.getPrincipal() + " was incorrect!");
			} catch (LockedAccountException lae) {
				log.info("The account for username " + token.getPrincipal() + " is locked.  " +
						"Please contact your administrator to unlock it.");
			}
			// ... 捕获更多异常
			catch (AuthenticationException ae) {
				//无定义?错误?
			}
		}

		//说出他们是谁:
		//打印主要识别信息 (本例是 username):
		log.info("User [" + currentUser.getPrincipal() + "] logged in successfully.");

		//测试角色:
		if (currentUser.hasRole("schwartz")) {
			log.info("May the Schwartz be with you!");
		} else {
			log.info("Hello, mere mortal.");
		}

		//测试一个权限 (非（instance-level）实例级别)
		if (currentUser.isPermitted("lightsaber:weild")) {
			log.info("You may use a lightsaber ring.  Use it wisely.");
		} else {
			log.info("Sorry, lightsaber rings are for schwartz masters only.");
		}

		//一个(非常强大)的实例级别的权限:
		if (currentUser.isPermitted("winnebago:drive:eagle5")) {
			log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
					"Here are the keys - have fun!");
		} else {
			log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
		}

		//完成 - 退出t!
		currentUser.logout();

		System.exit(0);
	}
}
```


### 总结

非常希望这示例介绍能帮助你理解如何在基础程序中加入 Shiro,并理解Shiro 的设计理念，Subject 和 SecurityManager。

但这个程序太简单了，你可能会问自己，“如果我不想使用 INI 用户帐号，而希望连接更为复杂的用户数据源呢？”

解决这些问题需要更深入地了解 并理解Shiro 的架构和配置机制，我们将在下一节Architecture中介绍。
