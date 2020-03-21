# 13. Testing 测试

文档的这一部分介绍了在单元测试中如何使用Shiro。

## 对于测试我们需要了解什么

由于我们已经涉及到了 Subject ，我们知道 Subject 是“当前执行”用户的特定安全视图，且该 Subject 实例绑定到一个线程来确保我们知道在线程执行期间的任何时间是谁在执行逻辑。

这意味着三个基本的东西必须始终出现，为了能够支持访问当前正在执行的Subject：

1. 必须创建一个 Subject 实例
2. Subject 实例必须绑定当前执行的线程。
3. 在线程完成执行后（或如果该线程执行抛出异常），该 Subject 必须解除绑定来确保该线程在任何线程池环境中保持'clean'。

Shiro 拥有为正在运行的应用程序自动地执行 绑定/解除绑定 逻辑的构建。例如，在 Web 应用程序中，当过滤一个请求时，Shiro 的根过滤器执行该逻辑。但由于测试环境和框架不同，我们需要自己选择自己的测试框架来执行此 绑定/解除绑定 逻辑。

## 测试设置

我们知道在创建一个 Subject 实例后，它必须被绑定线程。在该线程（或在这个例子中，是一个 test ）完成执行后，我们必须解除 Subject 的绑定来保持线程的 'clean'.

幸运的是，现代测试框架如 JUnit 和 TestNG 已经能够在本地支持'setup'和'teardown'的概念。我们可以利用这一支持来模拟 Shiro 在一个“完整的”应用程序中会做些什么。我们已经在下面创建了一个你能够在你自己的测试中使用的抽象基类——随意复制和修改如果你觉得合适的话。它能够在单元测试和集成测试中使用（我在本例中使用 JUnit，
但 TestNG 也能够工作得很好）：

### AbstractShiroTest
	
	
	import org.apache.shiro.SecurityUtils;
	import org.apache.shiro.UnavailableSecurityManagerException;
	import org.apache.shiro.mgt.SecurityManager;
	import org.apache.shiro.subject.Subject;
	import org.apache.shiro.subject.support.SubjectThreadState;
	import org.apache.shiro.util.LifecycleUtils;
	import org.apache.shiro.util.ThreadState;
	import org.junit.AfterClass;
	
	/**
	 * Abstract test case enabling Shiro in test environments.
	 */
	public abstract class AbstractShiroTest {
	
	    private static ThreadState subjectThreadState;
	
	    public AbstractShiroTest() {
	    }
	
	    /**
	     * Allows subclasses to set the currently executing {@link Subject} instance.
	     *
	     * @param subject the Subject instance
	     */
	    protected void setSubject(Subject subject) {
	        clearSubject();
	        subjectThreadState = createThreadState(subject);
	        subjectThreadState.bind();
	    }
	
	    protected Subject getSubject() {
	        return SecurityUtils.getSubject();
	    }
	
	    protected ThreadState createThreadState(Subject subject) {
	        return new SubjectThreadState(subject);
	    }
	
	    /**
	     * Clears Shiro's thread state, ensuring the thread remains clean for future test execution.
	     */
	    protected void clearSubject() {
	        doClearSubject();
	    }
	
	    private static void doClearSubject() {
	        if (subjectThreadState != null) {
	            subjectThreadState.clear();
	            subjectThreadState = null;
	        }
	    }
	
	    protected static void setSecurityManager(SecurityManager securityManager) {
	        SecurityUtils.setSecurityManager(securityManager);
	    }
	
	    protected static SecurityManager getSecurityManager() {
	        return SecurityUtils.getSecurityManager();
	    }
	
	    @AfterClass
	    public static void tearDownShiro() {
	        doClearSubject();
	        try {
	            SecurityManager securityManager = getSecurityManager();
	            LifecycleUtils.destroy(securityManager);
	        } catch (UnavailableSecurityManagerException e) {
	            //we don't care about this when cleaning up the test environment
	            //(for example, maybe the subclass is a unit test and it didn't
	            // need a SecurityManager instance because it was using only 
	            // mock Subject instances)
	        }
	        setSecurityManager(null);
	    }
	}

*Testing & Frameworks*

*在AbstractShiroTest 类中的代码使用 Shiro 的 ThreadState 概念及一个静态的S ecurityManager。这些技术在测试和框架代码中是很有用的，但几乎不曾在应用程序代码中使用。*

*大多数使用 Shiro 工作的需要确保线程的一致性的终端用户，几乎总是使用 Shiro 的自动管理机制，即 Subject.associateWith 和Subject.execute 方法。这些方法包含在 [Subject thread  association](http://shiro.apache.org/subject.html#Subject-ThreadAssociation) 参考文献中。*

## 单元测试
单元测试主要是测试你的代码，且你的代码是在有限的作用域内。当你考虑到 Shiro 时，你真正要关注的是你的代码能够与 Shiro 的API 正确的运行——你不会想做关于Shiro 的实现是否工作正常（这是 Shiro 开发团队在Shiro 的代码库必须确保的东西）的必要测试。

测试 Shiro 的实现是否与你的实现协同工作是真实的集成测试（下面讨论）。

### ExampleShiroUnitTest

由于单元测试适用于测试你的逻辑（而不是你可能调用的任何实现），这对于模拟你逻辑所依赖的任何 API 来说是个很好的主意。这能够与 Shiro 工作得很好——你可以模拟 Subject 实例，并使它反映任何情况下你所需的反应，这些反应是处于测试的代码做出的。

但正如上文所述，在 Shiro 测试中关键是要记住在测试执行期间任何Subject 实例（模拟的或真实的）必须绑定到线程。因此，我们所需要做的是绑定模拟的 Subject 以确保如预期进行。

（这个例子使用 EasyMock，但 Mockito 也同样地工作得很好）：
	
	import org.apache.shiro.subject.Subject;
	import org.junit.After;
	import org.junit.Test;
	
	import static org.easymock.EasyMock.*;
	
	/**
	 * Simple example test class showing how one may perform unit tests for code that requires Shiro APIs.
	 */
	public class ExampleShiroUnitTest extends AbstractShiroTest {
	
	    @Test
	    public void testSimple() {
	
	        //1.  Create a mock authenticated Subject instance for the test to run:
	        Subject subjectUnderTest = createNiceMock(Subject.class);
	        expect(subjectUnderTest.isAuthenticated()).andReturn(true);
	
	        //2. Bind the subject to the current thread:
	        setSubject(subjectUnderTest);
	
	        //perform test logic here.  Any call to 
	        //SecurityUtils.getSubject() directly (or nested in the 
	        //call stack) will work properly.
	    }
	
	    @After
	    public void tearDownSubject() {
	        //3. Unbind the subject from the current thread:
	        clearSubject();
	    }
	
	}

正如你所看到的，我们没有设立一个 Shiro SecurityManager 实例或配置一个Realm 或任何像这样的东西。我们简单地创建一个模拟Subject 实例，并通过调用setSubject 方法将它绑定到线程。这将确保任何在我们测试代码中的调用或在代码中我们正测试的SecurityUtils.getSubject()正常工作。

请注意，setSubject 方法实现将绑定你的模拟 Subject 到线程，且它仍将存在，直到你通过一个不同的 Subject 调用 setSubject 或直到你明确地通过调用 clearSubject() 将它从线程中清除。
保持 Subject 绑定到该线程多长时间（或在一个不同的测试中用来交换一个新的实例）取决于你及你的测试需求。

### tearDownSubject()

在实例中的 tearDownSubject() 方法使用了 Junit 4 的注释来确保该Subject 在每个测试方法执行后被清除，不管发生
什么。这要求你设立一个新的 Subject 实例并将它设置到每个需要执行的测试中。

然而这也不是绝对必要的。例如，你可以只每个测试开始时绑定一个新的Subject 实例（通过setSubject），也就是说，使用 @Before-annotated 方法。但如果你将要这么做，你可以同时使用 @After tearDownSubject() 方法来保持对称及'clean'。

你可以手动地在每个方法中混合及匹配该 setup/teardown 逻辑或使用@Before 和 @After 注释只要你认为合适。所有测试完成后，AbstractShiroTest 超类在无论怎样都会将 Subject 从线程解除绑定，因为 @After 注释在它的 tearDownShiro() 方法中。

## 集成测试

现在我们讨论了单元测试的设置，让我们讨论一些关于集成测试的东西。集成测试是指测试跨API 边界的实现。例如，测试当调用B 实现时A 实现是否工作，且B 实现是否做它该做的事情。你同样也可以在Shiro 中轻松地执行集成测试。Shiro 的SecurityManager 实例及它所包含的东西（如 Realms 和 SessionManager 等）都是占用很少内存的非常轻量级的POJO。这意味着你可以为每一个你执行的测试类创建并销毁一个SecurityManager 实例。当你的集成测试运行时，它们将使用“真实的” SecurityManager，且与你应用程序中相像的 Subject 实例将会在运行时使用。

### ExampleShiroIntegrationTest

下面的实例代码看起来与上面的单元测试实例几乎相同，但这3 个步骤却有些不同：

1. 现在有了step '0'，它用来设立一个“真实的” SecurityManager 实例。
2. Step 1 现在通过Subject.Builder 构造一个“真实的” Subject 实例，并将它绑定到线程。

线程的绑定与解除绑定（step 2 和 3 ）与单元测试实例中的作用一样。

	import org.apache.shiro.config.IniSecurityManagerFactory;
	import org.apache.shiro.mgt.SecurityManager;
	import org.apache.shiro.subject.Subject;
	import org.apache.shiro.util.Factory;
	import org.junit.After;
	import org.junit.BeforeClass;
	import org.junit.Test;
	
	public class ExampleShiroIntegrationTest extends AbstractShiroTest {
	
	    @BeforeClass
	    public static void beforeClass() {
	        //0.  Build and set the SecurityManager used to build Subject instances used in your tests
	        //    This typically only needs to be done once per class if your shiro.ini doesn't change,
	        //    otherwise, you'll need to do this logic in each test that is different
	        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:test.shiro.ini");
	        setSecurityManager(factory.getInstance());
	    }
	
	    @Test
	    public void testSimple() {
	        //1.  Build the Subject instance for the test to run:
	        Subject subjectUnderTest = new Subject.Builder(getSecurityManager()).buildSubject();
	
	        //2. Bind the subject to the current thread:
	        setSubject(subjectUnderTest);
	
	        //perform test logic here.  Any call to 
	        //SecurityUtils.getSubject() directly (or nested in the 
	        //call stack) will work properly.
	    }
	
	    @AfterClass
	    public void tearDownSubject() {
	        //3. Unbind the subject from the current thread:
	        clearSubject();
	    }
	}

正如你所看到的，一个具体的 SecurityManager 实现被实例化，并通过setSecurityManager 方法使其余的测试能够对其进行访问。然后测试方法能够使用该 SecurityManager，当使用 Subject.Builder 后通过调用getSecurityManager() 方法。

还要注意 SecurityManager 实例在 @BeforeClass 设置方法中只被设置一次——一个对于大多数测试类较为普遍的做法。如果你想，你可以创建一个新的 SecurityManager 实例并在任何时候从任何测试方法通过setSerurityManager 来设置它——例如，你可能会引用两个不同的.ini 文件来构建一个根据你的测试需求而来的新 SecurityManager。

最后，与单元测试例子一样，AbstractShiroTest 超类将会清除所有Shiro 产物（任何存在的 SecurityManager 及 Subject
实例）通过它的 @AfterClass tearDownShiro() 方法来确保该线程在下个测试类运行时是 'clean' 的。

## 为文档加把手

我们希望这篇文档可以帮助你使用 Apache Shiro 进行工作，社区一直在不断地完善和扩展文档，如果你希望帮助 Shiro 项目，请在你认为需要的地方考虑更正、扩展或添加文档，你提供的任何点滴帮助都将扩充社区并且提升 Shiro。

提供你的文档的最简单的途径是将它发送到用户[论坛](http://shiro-user.582556.n2.nabble.com/)或[邮件列表](http://shiro.apache.org/mailing-lists.html)

*译者注：如果对本中文翻译有疑议的或发现勘误欢迎指正，[点此](https://github.com/waylau/apache-shiro-1.2.x-reference/issues)提问。*

 