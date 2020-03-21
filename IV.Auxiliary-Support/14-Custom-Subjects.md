# 14. Custom Subjects 自定义 Subject



毫无疑问，在 Apache Shiro 中最重要的概念就是 Subject。 'Subject' 仅仅是一个安全术语，是指应用程序用户的特定安全的“视图”。一个 Shiro Subject 实例代表了一个单一应用程序用户的安全状态和操作。

这些操作包括：

* authentication(login)
* authorization(access control)
* session access
* logout

我们原本希望把它称为"User"由于这样“很有意义”，但是我们决定不这样做：太多的应用程序现有的 API 已经有自己的 User classes/frameworks，我们不希望和这些起冲突。此外，在安全领域，"Subject" 这一词实际上是公认的术语。

Shiro 的 API 为应用程序提供 Subject 为中心的编程范式支持。当编码应用程序逻辑时，大多数应用程序开发人员想知道谁才是当前正在执行的用户。虽然应用程序通常能够通过它们自己的机制（ UserService 等）来查找任何用户，但涉及到安全性时，最重要的问题是“谁才是当前的用户？”。

虽然通过使用 SecurityManager 可以捕获任何 Subject，但只有基于当前 用户/Subject 的应用程序代码更自然，更直观。

## 当前执行的Subject

几乎在所有环境下，你能够获得当前执行的 Subject 通过使用

	org.apache.shiro.SecurityUtils:Subject currentUser

getSubject() 方法调用一个独立的应用程序，该应用程序可以返回一个在应用程序特有位置上基于用户数据的Subject，在服务器环境中（如，Web 应用程序），它基于与当前线程或传入的请求相关的用户数据上获得 Subject 。

当你获得了当前的 Subject 后，你能够拿它做些什么？

如果你想在他们当前的 session 中使事情对用户变得可用，你可得的他们的 session:

	Session session = currentUser.getSession();
	session.setAttribute( "someKey", "aValue" );

Session 是一个 Shiro 的具体实例，它提供了大多数你经常要和HttpSessions 用到的东西，但有一些额外的好处和一个很大的区别：它不需要一个 HTTP 环境！

如果在 Web 应用程序内部部署，默认的 Session 将会是基于HttpSession 的。但是，在一个非 Web 环境中，像这个简单的 Quickstart，Shiro 将会默认自动地使用它的 Enterprise Session Management。这意味着你可以在你的应用程序中使用相同的 API，在任何层，无论部署环境。这打开了应用程序的全新世界，由于任何需要 session 的应用程序不再被强迫使用 HttpSession 或 EJB Stateful Session Beans。而且，任何客户端技术现在能够共享会话数据。

所以，你现在可以获取一个 Subject 以及他们的 Session。对于真正有用的东西像检查会怎么样呢，如果他们被允许做某些事——如对角色和权限的检查？

嗯，我只能对已知的用户做这些检查。我们的 Subject 实例代表了当前的用户，但谁又是实际上的当前用户呢？呃，他们都是匿名的——也就是说，直到他们至少登录一次。那么，让我们像下面这样做：

	if ( !currentUser.isAuthenticated() ) {
	    //collect user principals and credentials in a gui specific manner
	    //such as username/password html form, X509 certificate, OpenID, etc.
	    //We'll use the username/password example here since it is the most common.
	    //(do you know what movie this is from? ;)
	    UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
	    //this is all you have to do to support 'remember me' (no config - built in!):
	    token.setRememberMe(true);
	    currentUser.login(token);
	}

那就是了！它再简单不过了。

但如果他们的登录尝试失败了会怎么样？你可以捕获各种各样的具体的异常来告诉你到底发生了什么：

	try {
	    currentUser.login( token );
	    //if no exception, that's it, we're done!
	} catch ( UnknownAccountException uae ) {
	    //username wasn't in the system, show them an error message?
	} catch ( IncorrectCredentialsException ice ) {
	    //password didn't match, try again?
	} catch ( LockedAccountException lae ) {
	    //account for that username is locked - can't login.  Show them a message?
	} 
	    ... more types exceptions to check if you want ...
	} catch ( AuthenticationException ae ) {
	    //unexpected condition - error?
	}

你，作为 应用程序/GUI 开发人员，可以基于异常选择是否显示消息给终端用户（例如，“在系统中没有与该用户名对应的帐户。”）。有许多不同种类的异常供你检查，或者你可以抛出你自己自定义的异常，这些异常可能是Shiro 还未提供的。有关详情，请查看AuthenticationException 的[JavaDoc](http://www.jsecurity.org/api/org/jsecurity/authc/AuthenticationException.html)。

好了，现在，我们有了一个登录的用户，我们还有什么可以做的呢？

比方说，他们是谁：

	//print their identifying principal (in this case, a username):
	log.info( "User [" + currentUser.getPrincipal() + "] logged in successfully." );

我们还可以测试他们是否有特定的角色：

	if ( currentUser.hasRole( "schwartz" ) ) {
	    log.info("May the Schwartz be with you!" );
	} else {
	    log.info( "Hello, mere mortal." );
	}

我们还能够判断他们是否有[权限](http://shiro.apache.org/permissions.html)对一个确定类型的实体进行操作

	if ( currentUser.isPermitted( "lightsaber:weild" ) ) {
	    log.info("You may use a lightsaber ring.  Use it wisely.");
	} else {
	    log.info("Sorry, lightsaber rings are for schwartz masters only.");
	}

此外，我们可以执行一个非常强大的实例级权限检查——它能够判断用户是否能够访问一个类型的具体实例：

	if ( currentUser.isPermitted( "winnebago:drive:eagle5" ) ) {
	    log.info("You are permitted to 'drive' the 'winnebago' with license plate (id) 'eagle5'.  " +
	                "Here are the keys - have fun!");
	} else {
	    log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
	}

小菜一碟，对吧？
最后，当用户完成了对应用程序的使用时，他们可以注销：

	currentUser.logout(); //removes all identifying information and invalidates their session too.

这个简单的API 包含了 90% 的 Shiro 终端用户在使用 Shiro 时将会处理的东西。

## 自定义 Subject 实例

Shiro 1.0 中添加了一个新特性，能够在特殊情况下构造 自定义/临时 的subject 实例。

*只在特殊情况下使用!*

*你应该总是通过调用 SubjectUtils.getSubject() 来获得当前正在执行的 Subject；
创建自定义的 Subject 实例只应在特殊情况下进行。*

当一些“特殊情况”是，这是可以很有用的：

* 系统启动/引导——当没有用户月系统交互时，代码应该作为一'system'或daemon 用户来执行。创建 Subject实例来代表一个特定的用户是值得的，这样引导代码能够以该用户（如admin）来执行。
鼓励这种做法是由于它能保证 utility/system 代码作为一个普通用户以同样的方式执行，以确保代码是一致的。
这使得代码更易于维护，因为你不必担心 system/daemon 方案的自定义代码块。
* 集成测试——你可能想创建 Subject 实例，在必要时可以在集成测试中使用。请参阅[测试](13. Testing 测试.md)文档获取更多的内容。
* Daemon/background 进程的工作——当一个 daemon 或 background 进程执行时，它可能需要作为一个特定的用户来执行。

*如果你已经有一个 Subject 的实例，并希望它提供给其他线程，你应该使用 `Subject.associateWith*` 方法，而不是创建一个新的Subject 实例。*

好了，假设你仍然需要创建自定义的Subject 实例的情况下，让我们看看如何来做：

### Subject.Builder

Subject.Builder 被制定得非常容易创建 Subject 实例，而无需知道构造细节。
Builder 最简单的用法是构造一个匿名的，session-less（无会话） Subject 的实例。

	Subject subject = new Subject.Builder().buildSubject()

上面所展示的默认的Subject.Builder 无参构造函数将通过SecurityUtils.getSubject() 方法使用应用程序当前可访问的
SecurityManager 。你也可以指定被额外的构造函数使用的SecurityManager 实例，如果你需要的话：

	SecurityManager securityManager = //acquired from somewhere
	Subject subject = new Subject.Builder(securityManager).buildSubject();

所有其他的 Subject.Builder 方法可以在 buildSubject()方法之前被调用，它们来提供关于如何构造 Subject 实例的上下文。例如，假如你拥有一个 session ID ，想取得“拥有”该 session 的Subject（假设该 session 存在且未过期）：

	Serializable sessionId = //acquired from somewhere
	Subject subject = new Subject.Builder().sessionId(sessionId).buildSubject();


同样地，如你想创建一个Subject 实例来反映一个确定的身份：

	Object userIdentity = //a long ID or String username, or whatever the "myRealm" requires
	String realmName = "myRealm";
	PrincipalCollection principals = new SimplePrincipalCollection(userIdentity, realmName);
	Subject subject = new Subject.Builder().principals(principals).buildSubject();

然后，你可以使用构造的 Subject 实例，如预期一样对它进行调用。但请注意：

构造的 Subject 实例不会由于应用程序（线程）的进一步使用而自动地绑定到应用程序（线程）。如果你想让它对于任何代码都能够方便地调用SecurityUtils.getSubject()，你必须确保创建好的 Subject 有一个线程与之关联。

### 线程关联

如上所述，只是构建一个 Subject 实例，并不与一个线程相关联——一个普通的必要条件是在线程执行期间任何对 SecurityUtils.getSubject() 的调用是否能正常工作。确保一个线程与一个 Subject 关联有三种途径：

* Automatic Association(自动关联)—— 通过 Sujbect.execute* 方法执行一个Callable 或Runnable 方法会自动地绑定和解除绑定到线程的Subject，在Callable/Runnable 异常的前后。
* Manual Association(手动关联)——你可以在当前执行的线程中手动地对Subject 实例进行绑定和解除绑定。这通常对框架开发人员非常有用。
* Different Thread(不同的线程)——通过调用Subject.associateWith* 方法将 Callable 或 Runnable 方法关联到
Subject，然后返回的 Callable/Runnable 方法在另一个线程中被执行。如果你需要为Subject 在另一个线程上执行工作的话，这是首选的方法。

了解线程关联最重要的是，两件事情必须始终发生：
1. Subject 绑定到线程，所以它在线程的所有执行点都是可用的。Shiro 做到这点通过它的 ThreadState 机制，该机制是在 ThreadLocal 上的一个抽象。
2. Subject 将在某点解除绑定，即使线程的执行结果是错误的。这将确保线程保持干净，并在pooled/reusable 线程环境中清除任何之前的Subject 状态。

这些原则保证在上述三个机制中发生。接下来阐述它们的用法。

#### 自动关联

如果你只需要一个 Subject 暂时与当前的线程相关联，同时你希望线程绑定和清理自动发生，Subject 的 Callable 或 Runnable 的直接执行正是你所需要的。在 Subject.execute 调用返回后，当前线程被保证当前状态与执行前的状态是一样的。这个机制是这三个中使用最广泛的。

例如，让我们假定你有一些逻辑在系统启动时需要执行。你希望作为一个特定用户执行代码块，但一旦逻辑完成后，你想确保 线程/环境 自动地恢复到正常。你可以通过调用 Subject.execute* 方法来做到：

	Subject subject = //build or acquire subject
	subject.execute( new Runnable() {
	    public void run() {
	        //subject is 'bound' to the current thread now
	        //any SecurityUtils.getSubject() calls in any
	        //code called from here will work
	    }
	});
	//At this point, the Subject is no longer associated
	//with the current thread and everything is as it was before

当然，Callable 的实例也能够被支持，所以你能够拥有返回值并捕获异常：

	Subject subject = //build or acquire subject
	MyResult result = subject.execute( new Callable<MyResult>() {
	    public MyResult call() throws Exception {
	        //subject is 'bound' to the current thread now
	        //any SecurityUtils.getSubject() calls in any
	        //code called from here will work
	        ...
	        //finish logic as this Subject
	        ...
	        return myResult;        
	    }
	});
	//At this point, the Subject is no longer associated
	//with the current thread and everything is as it was before

这种方法在框架开发中也是很有用的。例如，Shiro 对 secure Spring remoting 的支持确保了远程调用能够作为一个特
定的 Subject 来执行：

	Subject.Builder builder = new Subject.Builder();
	//populate the builder's attributes based on the incoming RemoteInvocation
	...
	Subject subject = builder.buildSubject();
	
	return subject.execute(new Callable() {
	    public Object call() throws Exception {
	        return invoke(invocation, targetObject);
	    }
	});

#### 手动关联
虽然 Subject.execute* 方法能够在它们返回后自动地清理线程的状态，但有可能在一些情况下，你想自己管理 ThreadState。当结合 w/Shiro 时，这几乎总是在框架开发层次使用，但它很少在 bootstrap/daemon 情景下使用（上面 Subject.execute(callable) 例子使用得更为频繁）。

*Guarantee Cleanup*

*关于这一机制最重要的是，你必须一直保证当前的线程在逻辑执行完后被清理，以确保在一个可重复使用或线程池的环境中没有一个线程状态腐化。*

最好的做法是在try/finally 块保证清理：

	Subject subject = new Subject.Builder()...
	ThreadState threadState = new SubjectThreadState(subject);
	threadState.bind();
	try {
	    //execute work as the built Subject
	} finally {
	    //ensure any state is cleaned so the thread won't be 
	    //corrupt in a reusable or pooled thread environment
	    threadState.clear();
	}

有趣的是，这正是 Subject.execute* 方法实际上所做的——它们只是在Callable 或 Runnable 执行前后自动地执行这个逻辑。Shiro 的 ShiroFilter 为 Web 应用程序执行几乎相同的逻辑（ShiroFilter 使用 Web 特定的 ThreadState 的实现，超出了本节的范围）

*Web Use*

*不要在一个处理 Web 请求的进程中使用上述 ThreadState 代码示例。 Web 特定的 ThreadState 的实现使用 Web 请求代替。相反，确保ShiroFilter 拦截 Web 请求以确保 Subject 的 building/binding/cleanup 能够好好的完成。*

#### 一个不同的线程

如果你有一个 Callable 或 Runnable 实例要以 Subject 来执行，你将自己执行 Callable 或 Runnable（或这将它移交给线程池或执行者或ExcutorService），你应该使用 Subject.associateWith* 方法。这些方法确保在最终执行的线程中保
留 Subject，且该 Subject 是可访问的。

Callable 例子：

	Subject subject = new Subject.Builder()...
	Callable work = //build/acquire a Callable instance.
	//associate the work with the built subject so SecurityUtils.getSubject() calls works properly:
	work = subject.associateWith(work);
	ExecutorService executorService = new java.util.concurrent.Executors.newCachedThreadPool();
	//execute the work on a different thread as the built Subject:
	executor.execute(work);

Runnable 例子：

	Subject subject = new Subject.Builder()...
	Runnable work = //build/acquire a Runnable instance.
	//associate the work with the built subject so SecurityUtils.getSubject() calls works properly:
	work = subject.associateWith(work);
	Executor executor = new java.util.concurrent.Executors.newCachedThreadPool();
	//execute the work on a different thread as the built Subject:
	executor.execute(work);

*Automatic Cleanup*

*associateWith 方法自动执行必要的线程清理，以取保现在在线程池环境中的clean。*

## 为文档加把手

我们希望这篇文档可以帮助你使用 Apache Shiro 进行工作，社区一直在不断地完善和扩展文档，如果你希望帮助 Shiro 项目，请在你认为需要的地方考虑更正、扩展或添加文档，你提供的任何点滴帮助都将扩充社区并且提升 Shiro。

提供你的文档的最简单的途径是将它发送到用户[论坛](http://shiro-user.582556.n2.nabble.com/)或[邮件列表](http://shiro.apache.org/mailing-lists.html)

*译者注：如果对本中文翻译有疑议的或发现勘误欢迎指正，[点此](https://github.com/waylau/apache-shiro-1.2.x-reference/issues)提问。*

 