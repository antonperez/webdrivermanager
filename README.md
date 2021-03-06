[![Maven Central](https://img.shields.io/maven-central/v/io.github.bonigarcia/webdrivermanager.svg)](http://search.maven.org/#search%7Cga%7C1%7Cg%3Aio.github.bonigarcia%20a%3Awebdrivermanager)
[![Build Status](https://travis-ci.org/bonigarcia/webdrivermanager.svg?branch=master)](https://travis-ci.org/bonigarcia/webdrivermanager)
[![Quality Gate](https://sonarcloud.io/api/badges/gate?key=io.github.bonigarcia:webdrivermanager)](https://sonarcloud.io/dashboard/index/io.github.bonigarcia:webdrivermanager)
[![codecov](https://codecov.io/gh/bonigarcia/webdrivermanager/branch/master/graph/badge.svg)](https://codecov.io/gh/bonigarcia/webdrivermanager)
[![badge-jdk](https://img.shields.io/badge/jdk-7-green.svg)](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
[![License badge](https://img.shields.io/badge/license-Apache2-green.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![Support badge](https://img.shields.io/badge/support-sof-green.svg)](http://stackoverflow.com/questions/tagged/webdrivermanager-java)
[![Twitter](https://img.shields.io/badge/follow-@boni_gg-green.svg)](https://twitter.com/boni_gg)

# WebDriverManager [![][Logo]][GitHub Repository]

This library is aimed to automate the [Selenium Webdriver] binaries management in runtime for Java.

If you use [Selenium Webdriver], you will know that in order to use some browsers such as **Chrome**, **Firefox**, **Opera**, **PhantomJS**, **Microsoft Edge**, or **Internet Explorer**, first you need to download a binary file which allows WebDriver to handle browsers. In addition, the absolute path to this binary must be set as JVM properties, as follows:

```java
System.setProperty("webdriver.chrome.driver", "/absolute/path/to/binary/chromedriver");
System.setProperty("webdriver.gecko.driver", "/absolute/path/to/binary/geckodriver");
System.setProperty("webdriver.opera.driver", "/absolute/path/to/binary/operadriver");
System.setProperty("phantomjs.binary.path", "/absolute/path/to/binary/phantomjs");
System.setProperty("webdriver.edge.driver", "C:/absolute/path/to/binary/MicrosoftWebDriver.exe");
System.setProperty("webdriver.ie.driver", "C:/absolute/path/to/binary/IEDriverServer.exe");
```

This is quite annoying since it forces you to link directly this binary file into your Java source code. In addition, you have to check manually when new versions of the binaries are released. WebDriverManager comes to the rescue, performing in an automated way all this dirty job for you.

WebDriverManager is open source, released under the terms of [Apache 2.0 License].

## Usage

In order to use WebDriverManager in a Maven project, you need to add the following dependency in your `pom.xml` (Java 7 or upper required):

```xml
<dependency>
    <groupId>io.github.bonigarcia</groupId>
    <artifactId>webdrivermanager</artifactId>
    <version>2.2.0</version>
</dependency>
```

WebDriverManager is typically used by tests, and therefore, the typical scope would be *test* (`<scope>test</scope>`).

Once we have included this dependency, you can let WebDriverManager to manage the WebDriver binaries for you. Take a look at this JUnit 4 example which uses Chrome with Selenium WebDriver (in order to use WebDriverManager in conjunction with **JUnit 5**, the extension [selenium-jupiter] is highly recommended):

```java
public class ChromeTest {

    private WebDriver driver;

    @BeforeClass
    public static void setupClass() {
        WebDriverManager.chromedriver().setup();
    }

    @Before
    public void setupTest() {
        driver = new ChromeDriver();
    }

    @After
    public void teardown() {
        if (driver != null) {
            driver.quit();
        }
    }

    @Test
    public void test() {
        // Your test code here
    }

}
```

Notice that simply adding ``WebDriverManager.chromedriver().setup();`` WebDriverManager does magic for you:

1. It checks for the latest version of the WebDriver binary.
2. It downloads the WebDriver binary if it's not present on your system.
3. It exports the required WebDriver Java environment variables needed by Selenium.

So far, WebDriverManager supports **Chrome**, **Firefox**, **Opera**, **PhantomJS**, **Microsoft Edge**, and **Internet Explorer**. For that, it provides several *drivers managers* for these browsers. These *drivers managers* can be used as follows:

```java
WebDriverManager.chromedriver().setup();
WebDriverManager.firefoxdriver().setup();
WebDriverManager.operadriver().setup();
WebDriverManager.phantomjs().setup();
WebDriverManager.edgedriver().setup();
WebDriverManager.iedriver().setup();
```

*NOTE*: The old WebDriverManager API (version 1.x) is still supported (`ChromeDriverManager.getInstance().setup();`, `FirefoxDriverManager.getInstance().setup();`, and so on), although the 2.x fashion (`WebDriverManager.chromedriver()`, `WebDriverManager.firefoxdriver()`, and so on) is recommended. 

Moreover, WebDriverManager provides a generic *driver manager*. This manager which can be parameterized using Selenium driver classes (e.g. `org.openqa.selenium.chrome.ChromeDriver`, `org.openqa.selenium.firefox.FirefoxDriver`, etc), as follows: 

```java
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import io.github.bonigarcia.wdm.WebDriverManager;

// ...

Class<? extends WebDriver> driverClass = ChromeDriver.class;
WebDriverManager.getInstance(driverClass).setup();
WebDriver driver = driverClass.newInstance();
```

This generic *driver manager* can be also parameterized using the enumeration `DriverManagerType`. For instance as follows:

```java
import static io.github.bonigarcia.wdm.DriverManagerType.CHROME;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import io.github.bonigarcia.wdm.WebDriverManager;

// ...

WebDriverManager.getInstance(CHROME).setup();
WebDriver driver = new ChromeDriver();
```

## Examples

Check out [WebDriverManager Examples][WebDriverManager Examples] for some JUnit 4 tests using WebDriverManager.


## WebDriverManager API

WebDriverManager exposes its API by means of the **builder pattern**. This means that given a *WebDriverManger* instance, their capabilities can be tuned using different methods. The following table summarizes the WebDriverManager API, together with the equivalent configuration key:


| Method                               | Description                                                                                                                                                                                                                                                                                  | Equivalent configuration key                                                                                                                                                          |
|--------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ``version(String)``                  | By default, WebDriverManager tries to download the latest version of a given driver binary. A concrete version can be specified using this method.                                                                                                                                           | ``wdm.chromeDriverVersion``, ``wdm.operaDriverVersion``, ``wdm.internetExplorerDriverVersion``, ``wdm.edgeDriverVersion``, ``wdm.phantomjsDriverVersion``, ``wdm.geckoDriverVersion`` |
| ``targetPath(String)``               | Folder in which WebDriver binaries are stored (WedDriverManager *cache*).                                                                                                                                                                                                                    | ``wdm.targetPath``                                                                                                                                                                    |
| ``forceCache()``                     | By default, WebDriverManager connects to the specific driver repository URL to find out what is the latest version of the binary. This can be avoided forcing to use the latest version form the local repository.                                                                           | ``wdm.forceCache=true``                                                                                                                                                               |
| ``forceDownload()``                  | By default, WebDriverManager finds out the latest version of the binary, and then it uses the cached version if exists. This option forces to download again the binary even if it has been previously cached.                                                                               | ``wdm.override=true``                                                                                                                                                                 |
| ``useBetaVersions()``                | By default, WebDriverManager skip beta versions. With this method, WebDriverManager will download also beta versions.                                                                                                                                                                        | ``wdm.useBetaVersions=true``                                                                                                                                                          |
| ``architecture(Architecture)``       | By default, WebDriverManager would try to use the proper binary for the platform running the test case (i.e. 32-bit or 64-bit). This behavior can be changed by forcing a given architecture: 32-bits (``Architecture.x32``) or 64-bits (``Architecture.x64``);                              | ``wdm.architecture``                                                                                                                                                                  |
| ``arch32()``                         | Force to use the 32-bit version of a given driver binary.                                                                                                                                                                                                                                    | ``wdm.architecture=32``                                                                                                                                                               |
| ``arch64()``                         | Force to use the 64-bit version of a given driver binary.                                                                                                                                                                                                                                    | ``wdm.architecture=64``                                                                                                                                                               |
| ``operatingSystem(OperatingSystem)`` | By default, WebDriverManager downloads the binary for the same operative systems than the machine running the test. This can be changed using this method (accepted values: ``WIN``, ``LINUX``, ``MAC``).                                                                                    | ``wdm.os=WIN``, ``wdm.os=LINUX``, ``wdm.os=MAC``                                                                                                                                      |
| ``driverRepositoryUrl(URL)``         | This method allows to change the repository URL in which the binaries are hosted (see next section for default values).                                                                                                                                                                      | ``wdm.chromeDriverUrl``, ``wdm.operaDriverUrl``, ``wdm.internetExplorerDriverUrl``, ``wdm.edgeDriverUrl``, ``wdm.phantomjsDriverUrl``, ``wdm.geckoDriverUrl``                         |
| ``useMirror()``                      | The [npm.taobao.org] site is a mirror which hosts different software assets. Among them, it hosts *chromedriver*, *geckodriver*,  *operadriver*, and *phantomjs* driver. Therefore, this method can be used for Chrome, Firefox, Opera, and PhantomJS to force to use the taobao.org mirror. | ``wdm.useMirror=true``                                                                                                                                                                |
| ``proxy(String)``                    | Use a HTTP proxy for the Internet connection.                                                                                                                                                                                                                                                | ``wdm.proxy``                                                                                                                                                                         |
| ``proxyUser(String)``                | Specify a username for HTTP proxy.                                                                                                                                                                                                                                                           | ``wdm.proxyUser``                                                                                                                                                                     |
| ``proxyPass(String)``                | Specify a password for HTTP proxy.                                                                                                                                                                                                                                                           | ``wdm.proxyPass``                                                                                                                                                                     |
| ``ignoreVersions(String...)``        | Ignore some versions to be downloaded.                                                                                                                                                                                                                                                       | ``wdm.ignoreVersions``                                                                                                                                                                |
| ``gitHubTokenName(String)``          | Token name for authenticated requests (see "Known issues").                                                                                                                                                                                                                                  | ``wdm.gitHubTokenName``                                                                                                                                                               |
| ``gitHubTokenSecret(String)``        | Secret for authenticated requests (see "Known issues").                                                                                                                                                                                                                                      | ``wdm.gitHubTokenSecret``                                                                                                                                                             |
| ``timeout(int)``                     | Timeout (in seconds) to connect and download binaries from online reporsitories                                                                                                                                                                                                              | ``wdm.timeout``                                                                                                                                                                       |
| ``properties(String)``               | Properties file for configuration values (by default ``webdrivermanager.properties``).                                                                                                                                                                                                       | ``wdm.properties``                                                                                                                                                                    |
| ``avoidExport()``                    | Avoid exporting JVM properties with the path of binaries (i.e. ``webdriver.chrome.driver``, ``webdriver.gecko.driver``, etc). Only recommended for interactive mode.                                                                                                                         | ``wdm.avoidExport``                                                                                                                                                                   |
| ``avoidOutputTree()``                | Avoid create tree structure for downloaded binaries (e.g. ``~/.m2/repository/webdriver/chromedriver/linux64/2.37/`` for ``chromedriver``). Used by default in interactive mode.                                                                                                              | ``wdm.avoidOutputTree``                                                                                                                                                               |


The following table contains some examples:

| Example                                                           | Description                                                       |
|-------------------------------------------------------------------|-------------------------------------------------------------------|
| ``WebDriverManager.chromedriver().version("2.26").setup();``      | Force to use version 2.26 of *chromedriver*                       |
| ``WebDriverManager.firefoxdriver().arch32().setup();``            | Force to use the 32-bit version of *geckodriver*                  |
| ``WebDriverManager.operadriver().forceCache().setup();``          | Force to use the cache version of *operadriver*                   |
| ``WebDriverManager.phantomjs().useMirror().setup();``             | Force to use the taobao.org mirror to download *phantomjs* driver |
| ``WebDriverManager.chromedriver().proxy("server:port").setup();`` | Using proxy *server:port* for the connection                      |

Three more methods are exposed by WebDriverManager, namely:

* ``getVersions()``: This method allows to find out the list of of available binary versions for a given browser.
* ``getBinaryPath()``: This method allows to find out the path of the latest resolved binary.
* ``getDownloadedVersion()``: This method allows to find out the version of the latest resolved binary.


## Configuration

Configuration parameters for WebDriverManager are set in the ``webdrivermanager.properties`` file:

```properties
wdm.targetPath=~/.m2/repository/webdriver
wdm.forceCache=false
wdm.override=false
wdm.useMirror=false
wdm.useBetaVersions=false
wdm.avoidExport=false
wdm.avoidOutputTree=false
wdm.timeout=30

wdm.chromeDriverUrl=https://chromedriver.storage.googleapis.com/
wdm.chromeDriverMirrorUrl=http://npm.taobao.org/mirrors/chromedriver
wdm.chromeDriverExport=webdriver.chrome.driver

wdm.geckoDriverUrl=https://api.github.com/repos/mozilla/geckodriver/releases
wdm.geckoDriverMirrorUrl=http://npm.taobao.org/mirrors/geckodriver
wdm.geckoDriverExport=webdriver.gecko.driver

wdm.operaDriverUrl=https://api.github.com/repos/operasoftware/operachromiumdriver/releases
wdm.operaDriverMirrorUrl=http://npm.taobao.org/mirrors/operadriver
wdm.operaDriverExport=webdriver.opera.driver

wdm.phantomjsDriverUrl=https://bitbucket.org/ariya/phantomjs/downloads/
wdm.phantomjsDriverMirrorUrl=http://npm.taobao.org/mirrors/phantomjs
wdm.phantomjsDriverExport=phantomjs.binary.path

wdm.edgeDriverUrl=https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/
wdm.edgeDriverExport=webdriver.edge.driver

wdm.internetExplorerDriverUrl=https://selenium-release.storage.googleapis.com/
wdm.internetExplorerDriverExport=webdriver.ie.driver

```

For instance, the variable ``wdm.targetPath`` is the default folder in which WebDriver binaries are going to be stored. By default the path of the Maven local repository is used. This property can be overwritten by Java system properties, for example:

```java
System.setProperty("wdm.targetPath", "/my/custom/path/to/driver/binaries");
```

... or by command line, for example:

```properties
-Dwdm.override=true
```

By default, WebDriverManager downloads the latest version of the WebDriver binary. But concrete versions of WebDriver binaries can be forced by changing the value of the variables ``wdm.chromeDriverVersion``, ``wdm.operaDriverVersion``,  ``wdm.internetExplorerDriverVersion``, or  ``wdm.edgeDriverVersion`` to a concrete version. For instance:

```properties
-Dwdm.chromeDriverVersion=2.25
-Dwdm.internetExplorerVersion=2.46
-Dwdm.operaDriverVersion=0.2.0
-Dwdm.edgeDriverVersion=3.14366
-Dwdm.phantomjsDriverVersion=2.1.1
-Dwdm.geckoDriverVersion=0.11.1
```

If no version is specified, WebDriverManager sends a request to the server hosting the binary. In order to avoid this request and check if any binary has been previously downloaded, the key `wdm.forceCache` can be used.

As of WebDriverManager 2, the value of these properties can be overridden by means of *environmental variables*. The name of these variables result from putting the name in uppercase and replacing the symbol `.` by `_`. For example, the property ``wdm.targetPath`` can be overridden by the environment variable ``WDM_TARGETPATH``.

Moreover, as of WebDriverManager 2.2.x, configuration value can be customized using a *configuration manager*. This manager can be accessed using ``WebDriverManager.config()``. For example:


```java
WebDriverManager.config().setTargetPath("/path/to/my-wdm-cache");
WebDriverManager.config().setProperties("/path/to/my-wdm.properties");
WebDriverManager.config().setForceCache(true);
WebDriverManager.config().setOverride(true);
```

## Interactive mode

As of version 2.2.0, WebDriverManager can used interactively from the shell to resolve and download binaries for the supported browsers. There are two ways of using this feature:

* Directly from the source code, using Maven. The command to be used is ``mvn exec:java -Dexec.args="browserName"``. For instance:

```
> mvn exec:java -Dexec.args="chrome"
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building WebDriverManager 2.2.0
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- exec-maven-plugin:1.6.0:java (default-cli) @ webdrivermanager ---
[INFO] Using WebDriverManager to resolve chrome
[INFO] Reading https://chromedriver.storage.googleapis.com/ to seek chromedriver
[INFO] Latest version of chromedriver is 2.37
[INFO] Downloading https://chromedriver.storage.googleapis.com/2.37/chromedriver_win32.zip to folder D:\projects\webdrivermanager
[INFO] Binary driver after extraction D:\projects\webdrivermanager\chromedriver.exe
[INFO] Resulting binary D:\projects\webdrivermanager\chromedriver.exe
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 7.306 s
[INFO] Finished at: 2018-03-23T09:53:58+01:00
[INFO] Final Memory: 17M/247M
[INFO] ------------------------------------------------------------------------
```

* Using WebDriverManager as a *fat-jar*. This jar can be created using the command ``mvn compile assembly:single`` from the source code, and then ``java -jar webdrivermanager.jar browserName`` . For instance:

```
> java -jar webdrivermanager-2.2.0-jar-with-dependencies.jar chrome
[INFO] Using WebDriverManager to resolve chrome
[INFO] Reading https://chromedriver.storage.googleapis.com/ to seek chromedriver
[INFO] Latest version of chromedriver is 2.37
[INFO] Downloading https://chromedriver.storage.googleapis.com/2.37/chromedriver_win32.zip to D:\projects\webdrivermanager\target\chromedriver_win32.zip
[INFO] Resulting binary D:\projects\webdrivermanager\target\chromedriver.exe
```

### HTTP Proxy

If you use an HTTP Proxy in your Internet connection, you can configure your settings by exporting the Java environment variable ``HTTPS_PROXY`` using the following notation: ``my.http.proxy:1234`` or ``username:password@my.http.proxy:1234``.
Also you can configure username and password using environment variables (``HTTPS_PROXY_USER`` and ``HTTPS_PROXY_PASS``).

### Known Issues

Some of the binaries (for Opera and Firefox) are hosted on GitHub. When several consecutive requests are made by WebDriverManager, GitHub servers return an **HTTP 403 error** response as follows:

```
Caused by: java.io.IOException: Server returned HTTP response code: 403 for URL: https://api.github.com/repos/operasoftware/operachromiumdriver/releases
    at sun.net.www.protocol.http.HttpURLConnection.getInputStream0(HttpURLConnection.java:1840)
    at sun.net.www.protocol.http.HttpURLConnection.getInputStream(HttpURLConnection.java:1441)
    at sun.net.www.protocol.https.HttpsURLConnectionImpl.getInputStream(HttpsURLConnectionImpl.java:254)
    at io.github.bonigarcia.wdm.BrowserManager.openGitHubConnection(BrowserManager.java:463)
    at io.github.bonigarcia.wdm.OperaDriverManager.getDrivers(OperaDriverManager.java:55)
    at io.github.bonigarcia.wdm.BrowserManager.manage(BrowserManager.java:168)
```

```
Caused by: java.io.IOException: Server returned HTTP response code: 403 for URL: https://api.github.com/repos/mozilla/geckodriver/releases
    at sun.net.www.protocol.http.HttpURLConnection.getInputStream0(HttpURLConnection.java:1840)
    at sun.net.www.protocol.http.HttpURLConnection.getInputStream(HttpURLConnection.java:1441)
    at sun.net.www.protocol.https.HttpsURLConnectionImpl.getInputStream(HttpsURLConnectionImpl.java:254)
    at io.github.bonigarcia.wdm.FirefoxDriverManager.getDrivers(FirefoxDriverManager.java:61)
    at io.github.bonigarcia.wdm.BrowserManager.manage(BrowserManager.java:163)
```

In order to avoid this problem, [authenticated requests] should be done. The procedure is the following:

1. Create a token/secret pair in your [GitHub account]
2. Tell WebDriverManager the value of this pair token/secret. To do that you should use the configuration keys ``wdm.gitHubTokenName`` and ``wdm.gitHubTokenSecret``. You can pass them as command line Java parameters as follows:

```properties
-Dwdm.gitHubTokenName=<your-token-name>
-Dwdm.gitHubTokenSecret=<your-token-secret>
```

... or as environment variables (e.g. in Travis CI) as follows:

```properties
WDM_GITHUBTOKENNAME=<your-token-name>
WDM_GITHUBTOKENSECRET=<your-token-secret>
```

## Help

If you have questions on how to use WebDriverManager properly with a special configuration or suchlike, please consider asking a question on [stackoverflow](https://stackoverflow.com/questions/tagged/webdrivermanager-java) and tag it with  *webdrivermanager-java*.

## About

WebDriverManager (Copyright &copy; 2015-2018) is a project created by [Boni Garcia] and licensed under the terms of the [Apache 2.0 License]. Comments, questions and suggestions are always very [welcome][WebDriverManager issues]!

[Logo]: http://bonigarcia.github.io/img/webdrivermanager.png
[Selenium Webdriver]: http://docs.seleniumhq.org/projects/webdriver/
[Apache 2.0 License]: http://www.apache.org/licenses/LICENSE-2.0
[Boni Garcia]: http://bonigarcia.github.io/
[GitHub Repository]: https://github.com/bonigarcia/webdrivermanager
[authenticated requests]: https://developer.github.com/v3/#rate-limiting
[GitHub account]: https://github.com/settings/tokens
[WebDriverManager issues]: https://github.com/bonigarcia/webdrivermanager/issues
[WebDriverManager Examples]: https://github.com/bonigarcia/webdrivermanager-examples
[npm.taobao.org]: http://npm.taobao.org/mirrors/
[selenium-jupiter]: https://github.com/bonigarcia/selenium-jupiter/
