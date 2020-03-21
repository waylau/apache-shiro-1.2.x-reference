# 18. Command Line Hasher
 

Shiro 1.2.0 及以后版本提供了一个命令行程序,可以哈希字符串和资源(文件、url、classpath 、 实体)几乎任何类型。 要使用它,您必须安装一个 Java 虚拟机，并且“Java”命令必须能访问访问 $PATH 环境变量。

## 使用

确保你可以访问shiro-tools-hasher-version-cli.jar  文件。 你可以发现这在 buildroot/tools/hasher/target 目录的源码构建或通过Maven下载。

一旦你获得 jar,您可以运行下面的命令:

>java -jar shiro-tools-hasher-X.X.X-cli.jar

这将打印所有可用选项标准(MD5、SHA1)和更复杂的密码散列的场景。

## 常见的场景

请参阅上面的命令打印指令。 它将提供一个详尽的清单的指令将帮助您根据您的需要使用厨师。 然而,我们已经提供了一些快速参考用途/场景下面方便。

shiro.ini 用户密码

最好是保持用户的密码 shiro.ini (用户) 部分安全。 添加一个新用户帐户,使用上面的命令 p (或 ——密码 )选项:

>java -jar shiro-tools-hasher-X.X.X-cli.jar -p

它会问你输入密码确认

	Password to hash:
	Password to hash (confirm):

当命令执行，将会输出 securely-salted-iterated-and-hashed
密码，举例：	

	$shiro1$SHA-256$500000$eWpVX2tGX7WCP2J+jMCNqw==$it/NRclMOHrfOvhAEFZ0mxIZRdbcfqIBdwdwdDXW2dM=

把这个值,并将其作为密码的用户定义行(其次是任何可选角色)中定义的 [INI 用户配置](https://github.com/waylau/apache-shiro-1.2.x-reference/blob/master/I.%20Overview%20%E6%80%BB%E8%A7%88/4.%20Configuration%20%E9%85%8D%E7%BD%AE.md#users) 文档。 例如

	[users]
	...
	user1 = $shiro1$SHA-256$500000$eWpVX2tGX7WCP2J+jMCNqw==$it/NRclMOHrfOvhAEFZ0mxIZRdbcfqIBdwdwdDXW2dM=

您还需要确保隐式 iniRealm 使用一个 CredentialsMatcher  知道如何执行安全散列密码比较。 所以配置的 [main] 部分:

	[main]
	...
	passwordMatcher = org.apache.shiro.authc.credential.PasswordMatcher
	iniRealm.credentialsMatcher = $passwordMatcher
	...

### MD5 校验和

虽然在 JVM上您可以用任意算法执行任何散列,但默认的散列算法是 MD5,常用于文件校验和。 只使用 - r (或 ——资源 )选项显示下面的值是一个资源位置(而不打印出你希望散列):

>java -jar shiro-tools-hasher-X.X.X-cli.jar -r RESOURCE_PATH

默认情况下 RESOURCE_PATH 将是文件路径,但您可以指定 classpath 或URL 资源通过将 classpath: 或 url: 设置为前缀。

一些例子:

	<command> -r fileInCurrentDirectory.txt
	<command> -r ../../relativePathFile.xml
	<command> -r ~/documents/myfile.pdf
	<command> -r /usr/local/logs/absolutePathFile.log
	<command> -r url:http://foo.com/page.html
	<command> -r classpath:/WEB-INF/lib/something.jar