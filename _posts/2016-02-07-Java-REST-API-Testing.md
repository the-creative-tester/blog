---
layout: post
title: Java REST API Testing
desc: REST Assured, Cucumber-JVM and Gradle
proj-num: 05
colour: 
---



## REST Assured, Cucumber-JVM and Gradle in Java

### Introduction

In this post, we will have a look at using the power of [Gradle](http://gradle.org/) to drive [REST Assured](https://github.com/jayway/rest-assured).  REST Assured is a Java implementation of an API testing framework.  We will then be making use of [Cucumber-JVM](https://github.com/cucumber/cucumber-jvm), which is a Java implementation of Cucumber.  These can be all be used together to create and execute automated yet descriptive REST API tests.

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

As part of this command, you would now also have automatically generated [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html).  Gradle Wrapper allows anybody to work on your project without having to install Gradle.  It ensures that the right version of Gradle that the build was designed for is shipped as part of the project repository.  As a result, you should have now the following project structure created:

>
~~~ shell
bash-3.2$ ls -ogR
total 40
-rw-r--r--  1   1220  7 Feb 23:01 build.gradle
drwxr-xr-x  3    102  7 Feb 23:01 gradle
-rwxr-xr-x  1   4971  7 Feb 23:01 gradlew
-rw-r--r--  1   2404  7 Feb 23:01 gradlew.bat
-rw-r--r--  1    646  7 Feb 23:01 settings.gradle
drwxr-xr-x  4    136  7 Feb 23:01 src
>
./gradle:
total 0
drwxr-xr-x  4   136  7 Feb 23:01 wrapper
>
./gradle/wrapper:
total 120
-rw-r--r--  1   53636  7 Feb 23:01 gradle-wrapper.jar
-rw-r--r--  1     232  7 Feb 23:01 gradle-wrapper.properties
>
./src:
total 0
drwxr-xr-x  3   102  7 Feb 23:01 main
drwxr-xr-x  3   102  7 Feb 23:01 test
>
./src/main:
total 0
drwxr-xr-x  3   102  7 Feb 23:01 java
>
./src/main/java:
total 8
-rw-r--r--  1   299  7 Feb 23:01 Library.java
>
./src/test:
total 0
drwxr-xr-x  3   102  7 Feb 23:01 java
>
./src/test/java:
total 8
-rw-r--r--  1   488  7 Feb 23:01 LibraryTest.java
>
~~~

You can remove the ```src/main``` directory, and also the ```src/test/java/LibraryTest.java``` file.  Create the following new folders, ```src/test/resources```, ```src/test/java/restassuredexample```, ```src/test/java/restassuredexample/cucumber``` and ```src/test/java/restassuredexample/cucumber/steps```.

### Open Notify - International Space Station Current Location

[Open Notify](http://open-notify.org/) is an open source project to provide an API for NASA data.  One such API is the [International Space Station Current Location](http://open-notify.org/Open-Notify-API/ISS-Location-Now/) which returns the current longitude and latitude of the ISS.  This is the target API that we will write our test against.

### Gradle Setup

We are going to write our first automated test against <http://api.open-notify.org/iss-now>.  We will first need to make some changes to ```build.gradle``` to download and setup our dependencies:

>
~~~ config
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
    compile 'org.slf4j:slf4j-api:1.7.13'
    testCompile 'junit:junit:4.12'
    testCompile 'info.cukes:cucumber-java:1.2.0'
    testCompile 'info.cukes:cucumber-junit:1.2.0'
    testCompile 'com.jayway.restassured:rest-assured:2.4.1'
}
>
configurations {
    cucumberRuntime {
        extendsFrom testRuntime
    }
}
>
// setup the cucumber task
task cucumber() {
    dependsOn assemble, compileTestJava
    doLast {
        javaexec {
            main = "cucumber.api.cli.Main"
            classpath = configurations.cucumberRuntime + sourceSets.main.output + sourceSets.test.output
            args = ['--plugin', 'pretty', '--glue', 'restassuredexample', 'src/test/resources']
        }
    }
}
>
test {
    systemProperties = System.properties
    testLogging.showStandardStreams = true
}
~~~

### Cucumber-JVM Setup

Create a new file, ```Cucumber.java``` in ```src/test/java/restassuredexample/cucumber``` with the following contents:

>
~~~  java
package restassuredexample.cucumber;
>
import org.junit.Test;
import org.junit.runner.RunWith;
import cucumber.api.junit.Cucumber;
import cucumber.api.CucumberOptions;
>
@RunWith(Cucumber.class)
@CucumberOptions(
    features = {"src/test/resources"}
)
public class CucumberRunner {
>
}
~~~

Let's now create a new feature file, ```iss-current-location.feature``` in ```src/test/resources``` to test <http://api.open-notify.org/iss-now>:

>
~~~ gherkin
Feature: International Space Station Current Location
>
  Scenario: Retrieve International Space Station Current Location
    Given I access the ISS Current Location
    When I retrieve the ISS Current Location
    Then I see the ISS Current Location
~~~

Write the matching step definitions, ```InternationalSpaceStationCurrentLocationSteps.java```, in ```src/test/java/restassuredexample/cucumber/steps```:

>
~~~ java
package restassuredexample.cucumber.steps;
>
import cucumber.api.java.en.Given;
import cucumber.api.java.en.When;
import cucumber.api.java.en.Then;
import restassuredexample.cucumber.InternationalSpaceStationCurrentLocationDefinition;
>
public class InternationalSpaceStationCurrentLocationSteps {
>
    InternationalSpaceStationCurrentLocationDefinition service;
>
    @Given("^I access the ISS Current Location$")
    public void i_access_the_ISS_Current_Location() throws Throwable {
        service = new InternationalSpaceStationCurrentLocationDefinition();
    }
>
    @When("^I retrieve the ISS Current Location$")
    public void i_retrieve_the_ISS_Current_Location() throws Throwable {
        service.requestInternationalSpaceStationCurrentLocation();
    }
>
    @Then("^I see the ISS Current Location$")
    public void i_see_the_ISS_Current_Location() throws Throwable {
        service.validateInternationalSpaceStationCurrentLocationContents();
    }
}
~~~

### REST Assured Setup

Write the matching service configuration, ```InternationalSpaceStationCurrentLocationConfiguration.java```, in ```src/test/java/restassuredexample/cucumber```:

>
~~~ java
package restassuredexample.cucumber;
>
public abstract class InternationalSpaceStationCurrentLocationConfiguration {
>
    public static final String OPEN_NOTIFY_API_URI = "http://api.open-notify.org";
>
}
~~~

Write the matching service definition, ```InternationalSpaceStationCurrentLocationDefinition.java```, in ```src/test/java/restassuredexample/cucumber```:

>
~~~ java
package restassuredexample.cucumber;
>
import com.jayway.restassured.RestAssured;
import com.jayway.restassured.response.Response;
>
import static com.jayway.restassured.RestAssured.*;
import static org.hamcrest.Matchers.containsString;
import static org.hamcrest.Matchers.equalTo;
>
public class InternationalSpaceStationCurrentLocationDefinition {
>
    public InternationalSpaceStationCurrentLocationDefinition() {
        RestAssured.baseURI = InternationalSpaceStationCurrentLocationConfiguration.OPEN_NOTIFY_API_URI;
    }
>
    public void requestInternationalSpaceStationCurrentLocation() {
        Response response =
                given().
                        contentType("application/json").
                when().
                        get("/iss-now/").
                then().
                        statusCode(200).
                        extract().response();
    }
>
    public void validateInternationalSpaceStationCurrentLocationContents() {
        Response response =
                given().
                        contentType("application/json").
                when().
                        get("/iss-now/").
                then().
                        body(containsString("iss_position")).
                        body(containsString("message")).
                        body(containsString("timestamp")).
                        body(("message"), equalTo("success")).
                extract().response();
    }
}
~~~

### Execution

You can now run your REST Assured with Cucumber-JVM test by using ```./gradlew cucumber``` or ```./gradlew test```, and you should see something similar to the following results:

>
~~~ shell
bash-3.2$ ./gradlew cucumber
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:jar UP-TO-DATE
:assemble UP-TO-DATE
:compileTestJava UP-TO-DATE
:cucumber
Feature: International Space Station Current Location
>
  Scenario: Retrieve International Space Station Current Location # iss-current-location.feature:3
    Given I access the ISS Current Location                       # InternationalSpaceStationCurrentLocationSteps.i_access_the_ISS_Current_Location()
    When I retrieve the ISS Current Location                      # InternationalSpaceStationCurrentLocationSteps.i_retrieve_the_ISS_Current_Location()
    Then I see the ISS Current Location                           # InternationalSpaceStationCurrentLocationSteps.i_see_the_ISS_Current_Location()
>
1 Scenarios (1 passed)
3 Steps (3 passed)
0m3.523s
>
>
BUILD SUCCESSFUL
>
Total time: 9.271 secs
>
~~~

### Full Example

<https://github.com/the-creative-tester/rest-assured-api-testing-example>
