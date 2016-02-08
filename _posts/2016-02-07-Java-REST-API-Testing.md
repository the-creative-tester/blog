---
layout: post
title: Java REST API Testing
desc: REST Assured, Cucumber-JVM and Gradle
proj-num: 05
colour: 
---



## REST Assured, Cucumber-JVM and Gradle

### Introduction

In this post, we will have a look at using the power of [Gradle](http://gradle.org/) to drive [REST Assured](https://github.com/jayway/rest-assured).  REST Assured is a Java implementation of an API testing framework.  We will then be making use of [Cucumber-JVM](https://github.com/cucumber/cucumber-jvm), which is a Java implementation of Cucumber.  These can be all be used together to create and execute automated yet descriptive REST API tests.

### Installation

##### Gradle

Download and extract [Gradle](http://gradle.org/gradle-download/).  You will also have to set your add GRADLE_HOME/bin to your PATH environment variable.

Ensure that you have successfully installed Gradle:  

>
~~~
bash-3.2$ gradle -v
>
------------------------------------------------------------
Gradle 2.10
------------------------------------------------------------
~~~

In a directory of your choice, you are now going to use Gradle's [Build Init Plugin](https://docs.gradle.org/current/userguide/build_init_plugin.html) to bootstrap the process of creating a new Gradle build:

>
~~~
bash-3.2$ gradle init --type java-library
:wrapper
:init
>
BUILD SUCCESSFUL
~~~

As part of this command, you would now also have automatically generated [Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html).  Gradle Wrapper allows anybody to work on your project without having to install Gradle.  It ensures that the right version of Gradle that the build was designed for is shipped as part of the project repository.  As a result, you should have now the following project structure created:

>
~~~
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

### Open Notify - International Space Station Current Location

[Open Notify](http://open-notify.org/) is an open source project to provide an API for NASA data.  One such API is the [International Space Station Current Location](http://open-notify.org/Open-Notify-API/ISS-Location-Now/) which returns the current longitude and latitude of the ISS.  This is the target API that we will write our test against.

### Initial Setup

We are going to write our first automated test against <http://api.open-notify.org/iss-now>.  We will first need to make some changes to


Create a new directory for your API test automation project, and open that directory in Sublime Text 3.  Now create a new file in that directory.  Tests scripts are usually named ```*spec.js``` in order for jasmine-node to find them. For this post, I will use ```iss-spec.js```.

### Using Frisby

In your newly created file, place the following code to allow you to utilise the previously installed Frisby module:

>
~~~ 
var frisby = require('frisby');
~~~

Let's write a simple test. We will first make use of the Frisby ```create()``` method to define the new test, and the parameter supplied defines the name of the test. Next, the ```get()``` method performs a HTTP GET request on the supplied URL. We can use the ```expectStatus()``` method to verify the returned HTTP status code, and the ```expectHeaderContains()``` method can be used to verify contents within the returned header. Since the API returns its payload in JSON, the content-type should be 'application/json' on the request. The last method we will use is ```toss()```, which is used to execute the test. Here's an example of these methods being used:

>
~~~ 
var frisby = require('frisby');
>
frisby.create('GET: International Space Station Current Location')
          .get('http://api.open-notify.org/iss-now')
          .expectStatus(200)
          .expectHeaderContains('content-type', 'application/json')
          .toss();
~~~

Let's now make this test a bit more useful, by using the ```expectJSONTypes()``` method to check the structure of the response rather than the data:

>
~~~
var frisby = require('frisby');
>
frisby.create('GET: International Space Station Current Location')
          .get('http://api.open-notify.org/iss-now')
          .expectStatus(200)
          .expectHeaderContains('content-type', 'application/json')
          .expectJSONTypes({
            message: String,
            timestamp: Number,
            iss_position: {
                latitude: Number,
                longitude: Number
            }
          })
          .toss();
~~~

We can also use the ```expectJSON()``` method to check the data of the response:

>
~~~
var frisby = require('frisby');
>
frisby.create('GET: International Space Station Current Location')
          .get('http://api.open-notify.org/iss-now')
          .expectStatus(200)
          .expectHeaderContains('content-type', 'application/json')
          .expectJSONTypes({
            message: String,
            timestamp: Number,
            iss_position: {
                latitude: Number,
                longitude: Number
            }
          })
          .expectJSON({
            message: "success"
          })
          .toss();
~~~

### Execution

You can now run your Frisby test by using jasmine-node, and you should see something similar to the following results:

>
~~~
bash-3.2$ jasmine-node iss_spec.js 
>
Finished in 1.736 seconds
1 test, 7 assertions, 0 failures, 0 skipped
~~~

### Full Example

<https://github.com/the-creative-tester/frisby-api-testing-example>
