---
layout: post
title: Java Web Automation (Cucumber)
desc: Selenium, Page Objects, Cucumber-JVM and Gradle
proj-num: 11
colour: 
---



## Selenium, Page Objects, Cucumber-JVM and Gradle in Python

### Introduction

In this post, we will write an automated test that utilises [Selenium WebDriver](http://www.seleniumhq.org/), [Cucumber-JVM](https://cucumber.io/docs/reference/jvm) and [Gradle](https://gradle.org/).  We will also make use of [Selenium Grid](https://github.com/SeleniumHQ/selenium/wiki/Grid2) in Gradle.

In this guide we will automate a browser navigating [Cucumber](https://cucumber.io/).  We will start off with a very basic test, but will then slowly build on top of it to incorporate Cucumber and Page Objects.

### Installation

##### Gradle

Download and extract [Gradle](http://gradle.org/gradle-download/).  You will also have to set your add GRADLE_HOME/bin to your PATH environment variable.

Ensure that you have successfully installed Gradle:  

>
~~~ shell
bash-3.2$ gradle -v
>
------------------------------------------------------------
Gradle 2.10
------------------------------------------------------------
~~~

In a directory of your choice, you are now going to use Gradle's [Build Init Plugin](https://docs.gradle.org/current/userguide/build_init_plugin.html) to bootstrap the process of creating a new Gradle build:

>
~~~ shell
bash-3.2$ gradle init --type java-library
:wrapper
:init
>
BUILD SUCCESSFUL
~~~

As part of this command, you would now also have automatically generated [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html).  Gradle Wrapper allows anybody to work on your project without having to install Gradle.  It ensures that the right version of Gradle that the build was designed for is shipped as part of the project repository.  Remove ```Library.java``` from ```/src/main/java/``` and ```LibraryTest.java``` from ```/src/test/java/````, this will be replaced by our own implementation later.

##### Selenium Grid

We will now be able touse Gradle to manage Selenium Grid.  Update your ```build.gradle``` file to contain the following:

>
~~~ groovy
// apply the java plugin to add support for Java
apply plugin: 'java'
>
// in this section declare where to find the dependencies of your project
repositories {
    jcenter()
}
>
// in this section declare the dependencies for your production and test code
dependencies {
    compile "org.seleniumhq.selenium:selenium-support:2.53.1"
    compile "org.seleniumhq.selenium:selenium-firefox-driver:2.53.1"
    compile "org.seleniumhq.selenium:selenium-chrome-driver:2.53.1"
    compile "org.seleniumhq.selenium:selenium-remote-driver:2.53.1"
    compile "org.seleniumhq.selenium:selenium-server:2.53.1"
    compile "info.cukes:cucumber-java:1.2.4"
    compile "info.cukes:cucumber-core:1.2.4"
    compile "info.cukes:cucumber-junit:1.2.4"
    compile 'info.cukes:gherkin:2.12.2'
    testCompile 'junit:junit:4.12'
}
>
task startSeleniumHub() << {
    ant.java(classname : 'org.openqa.grid.selenium.GridLauncher', 
            fork:true,
            classpath : configurations.runtime.asPath) {
        arg(value : '-role')
        arg(value : 'hub')
    }
}
>
task startSeleniumNode() << {
    ant.java(classname : 'org.openqa.grid.selenium.GridLauncher', 
            fork:true,
            classpath : configurations.runtime.asPath) {
        arg(value : '-role')
        arg(value : 'node')
        arg(value : '-hub')
        arg(value : 'http://localhost:4444/grid/register')
    }
}
~~~

A grid consists of a single hub, and one or more nodes. Both are started using the selenium-server.jar executable.

The hub receives a test to be executed along with information on which browser and platform where the test should be run. It knows the configuration of each node that has been registered to the hub. Using this information it selects an available node that has the requested browser-platform combination to run the test against.  In this tutorial, will run Selenium Grid locally.

First, start the hub:

>
~~~ bash
bash-3.2$ ./gradlew startSeleniumHub
~~~

Second, start the node:

>
~~~ bash
bash-3.2$ ./gradlew startSeleniumNode
~~~

Ensure your setup has been successful by navigating to <http://127.0.0.1:4444/grid/console>.

### Introducing Selenium

We can now write a very simple test that utilises WebDriver by creating ```ExampleTest.java``` in ```/src/test/java/```:

>
~~~ java
import java.net.MalformedURLException;
import java.net.URL;
import org.junit.*;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.firefox.FirefoxDriver;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import static org.junit.Assert.assertTrue;
>
public class ExampleTest 
{
>
  WebDriver driver;
>
  @Before
  public void setUp()
  {
    driver = new FirefoxDriver();
  }
>
  @After
  public void tearDown() 
  {
    driver.quit();
  }
>
  @Test    
  public void thisIsTheActualTest() 
  {
    driver.get("https://cucumber.io/");
    WebElement element = driver.findElement(By.linkText("Docs"));
    element.click();
    assertTrue(driver.getTitle().contains("Cucumber"));
  }
>
}
>
~~~

Now, try running the test and observe that Firefox is launched and the test is run:

>
~~~ bash
bash-3.2$ ./gradlew clean build
:clean
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:jar
:assemble
:compileTestJava
:processTestResources UP-TO-DATE
:testClasses
:test
:check
:build
>
BUILD SUCCESSFUL
>
Total time: 18.906 secs
~~~

Let's now update ```ExampleTest.java``` to utilise RemoteWebDriver and the Selenium Grid that you have started in the background:

>
~~~ java
import java.net.MalformedURLException;
import java.net.URL;
import org.junit.*;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import static org.junit.Assert.assertTrue;
>
public class ExampleTest 
{
> 
  DesiredCapabilities capabilities;
  RemoteWebDriver driver;
>
  @Before
  public void setUp() throws MalformedURLException
  {
    capabilities = DesiredCapabilities.firefox();
    capabilities.setCapability("browserName", "firefox"); 
    driver = new RemoteWebDriver(new URL("http://127.0.0.1:4444/wd/hub"), capabilities);
  }
>
  @After
  public void tearDown() 
  {
    driver.quit();
  }
>
  @Test    
  public void thisIsTheActualTest() 
  {
    driver.get("https://cucumber.io/");
    WebElement element = driver.findElement(By.linkText("Docs"));
    element.click();
    assertTrue(driver.getTitle().contains("Cucumber"));
  }
>
}
>
~~~

Now, try running the test and observe that Firefox is launched and the test is run:

>
~~~ bash
bash-3.2$ ./gradlew clean build
:clean
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:jar
:assemble
:compileTestJava
:processTestResources UP-TO-DATE
:testClasses
:test
:check
:build
>
BUILD SUCCESSFUL
>
Total time: 24.002 secs
~~~

### Introducing Cucumber

Let's next introduce Cucumber-JVM!  First, create the file ```Cucumber.feature``` in ```/src/test/resources/``` and place the following contents:

>
~~~ gherkin
Feature: Cucumber Home Page
>
  Scenario: Docs
    Given I navigate to "https://cucumber.io/"
    When I take a look at the Docs
    Then I see a browser title containing "Cucumber"
~~~

Create the file ```CucumberRunner.java``` in ```/src/test/java/``` and place the following contents:

>
~~~ java
import cucumber.api.CucumberOptions;
import org.junit.runner.RunWith;
>
@CucumberOptions(features = {"src/test/resources"})
@RunWith(cucumber.api.junit.Cucumber.class)
public class CucumberRunner
{
>
}
>
~~~

Rename the file ```ExampleTest.java``` to ```CucumberSteps.java```, and move the file to ```/src/test/java/steps/``` and place the following contents:

>
~~~ java
package steps;
>
import cucumber.api.java.After;
import cucumber.api.java.Before;
import cucumber.api.java.en.Given;
import cucumber.api.java.en.Then;
import cucumber.api.java.en.When;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import java.net.MalformedURLException;
import java.net.URL;
>
import static org.junit.Assert.assertTrue;
>
public class CucumberSteps
{
>
    RemoteWebDriver driver;
    DesiredCapabilities capabilities;
>
    @Before
    public void setUp() throws MalformedURLException
    {
        capabilities = DesiredCapabilities.firefox();
        capabilities.setCapability("browserName", "firefox");
        driver = new RemoteWebDriver(new URL("http://127.0.0.1:4444/wd/hub"), capabilities);
    }
>
    @After
    public void tearDown()
    {
        driver.close();
    }
>
    @Given("^I navigate to \"([^\"]*)\"$")
    public void i_navigate_to(String page) throws Throwable {
        driver.get(page);
    }
>
    @When("^I take a look at the Docs$")
    public void i_take_a_look_at_the_docs() throws Throwable {
        WebElement element = driver.findElement(By.linkText("Docs"));
        element.click();
    }
>
    @Then("^I see a browser title containing \"([^\"]*)\"$")
    public void i_see_a_browser_title_containing(String text) throws Throwable {
        assertTrue(driver.getTitle().contains(text));
    }
>
}
>
~~~

Now, try running the test and observe that Firefox is launched and the test is run:

>
~~~ bash
bash-3.2$ ./gradlew clean test
:clean
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:compileTestJava
:processTestResources
:testClasses
:test
>
BUILD SUCCESSFUL
>
Total time: 22.261 secs
~~~

### Introducing Page Objects

Let's now introduce the concept of Page Objects.  Create the file ```CucumberHomePage.java``` in ```/src/test/java/pages/``` and place the following contents:

>
~~~ java
package pages;
>
import org.openqa.selenium.WebElement;
import org.openqa.selenium.support.FindBy;
>
public class CucumberHomePage
{
>
    @FindBy(linkText = "Docs") WebElement docsLink;
>
    public void clickDocsLink()
    {
        docsLink.click();
    }
>
}
>
~~~

In the file ```CucumberSteps.java``` update the following contents to import the new Page Object that is initialised by PageFactory:

>
~~~ java
package steps;
>
import cucumber.api.java.After;
import cucumber.api.java.Before;
import cucumber.api.java.en.Given;
import cucumber.api.java.en.Then;
import cucumber.api.java.en.When;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import java.net.MalformedURLException;
import java.net.URL;
import pages.CucumberHomePage;
import org.openqa.selenium.support.PageFactory;
>
import static org.junit.Assert.assertTrue;
>
public class CucumberSteps
{
>
    RemoteWebDriver driver;
    DesiredCapabilities capabilities;
    CucumberHomePage cucumberHomePage;
>
    @Before
    public void setUp() throws MalformedURLException
    {
        capabilities = DesiredCapabilities.firefox();
        capabilities.setCapability("browserName", "firefox");
        driver = new RemoteWebDriver(new URL("http://127.0.0.1:4444/wd/hub"), capabilities);
        cucumberHomePage = PageFactory.initElements(driver, CucumberHomePage.class);
    }
>
    @After
    public void tearDown()
    {
        driver.close();
    }
>
    @Given("^I navigate to \"([^\"]*)\"$")
    public void i_navigate_to(String page) throws Throwable {
        driver.get(page);
    }
>
    @When("^I take a look at the Docs$")
    public void i_take_a_look_at_the_docs() throws Throwable {
        // WebElement element = driver.findElement(By.linkText("Docs"));
        // element.click();
        cucumberHomePage.clickDocsLink();
    }
>
    @Then("^I see a browser title containing \"([^\"]*)\"$")
    public void i_see_a_browser_title_containing(String text) throws Throwable {
        assertTrue(driver.getTitle().contains(text));
    }
>
}
>
~~~

Now, try running the test and observe that Firefox is launched and the test is run:

>
~~~ bash
bash-3.2$ ./gradlew clean test
:clean
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:compileTestJava
:processTestResources
:testClasses
:test
>
BUILD SUCCESSFUL
>
Total time: 19.391 secs
~~~

### Introducing DriverFactory

Let's now create the ability to share an instance of a RemoteWebDriver across different step definition and feature files.  Create the file ```DriverFactory.java``` in ```/src/test/java/utils/``` and place the following contents:

>
~~~ java
package utils;
>
import cucumber.api.java.After;
import cucumber.api.java.Before;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.openqa.selenium.support.PageFactory;
import pages.CucumberHomePage;
>
import java.net.MalformedURLException;
import java.net.URL;
>
public class DriverFactory
{
>
    public static RemoteWebDriver driver;
    DesiredCapabilities capabilities;
>
    @Before
    public void setUp() throws MalformedURLException
    {
        capabilities = DesiredCapabilities.firefox();
        capabilities.setCapability("browserName", "firefox");
        driver = new RemoteWebDriver(new URL("http://127.0.0.1:4444/wd/hub"), capabilities);
    }
>
    @After
    public void tearDown()
    {
        driver.close();
    }
>
}
>
~~~

Now update ```CucumberSteps.java``` to make use of DriverFactory with the following contents:

>
~~~ java
package steps;
>
import cucumber.api.java.After;
import cucumber.api.java.Before;
import cucumber.api.java.en.Given;
import cucumber.api.java.en.Then;
import cucumber.api.java.en.When;
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import java.net.MalformedURLException;
import java.net.URL;
import pages.CucumberHomePage;
import utils.DriverFactory;
import org.openqa.selenium.support.PageFactory;
>
import static org.junit.Assert.assertTrue;
>
public class CucumberSteps
{
>
    RemoteWebDriver driver = DriverFactory.driver;
    CucumberHomePage cucumberHomePage = PageFactory.initElements(driver, CucumberHomePage.class);
>
    @Given("^I navigate to \"([^\"]*)\"$")
    public void i_navigate_to(String page) throws Throwable {
        driver.get(page);
    }
>
    @When("^I take a look at the Docs$")
    public void i_take_a_look_at_the_docs() throws Throwable {
        // WebElement element = driver.findElement(By.linkText("Docs"));
        // element.click();
        cucumberHomePage.clickDocsLink();
    }
>
    @Then("^I see a browser title containing \"([^\"]*)\"$")
    public void i_see_a_browser_title_containing(String text) throws Throwable {
        assertTrue(driver.getTitle().contains(text));
    }
>
}
>
~~~

Now, try running the test and observe that Firefox is launched and the test is run:

>
~~~ bash
bash-3.2$ ./gradlew clean test
:clean
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:compileTestJava
:processTestResources
:testClasses
:test
>
BUILD SUCCESSFUL
>
Total time: 23.717 secs
~~~

### Full Example

<https://github.com/the-creative-tester/java-web-browser-automation-example>
