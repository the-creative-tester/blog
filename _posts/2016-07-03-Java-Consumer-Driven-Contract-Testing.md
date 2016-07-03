---
layout: post
title: Java Consumer Driven Contract Testing
desc: Pact-JVM and Gradle
proj-num: 10
colour: 
---



## Pact-JVM and Gradle in Java

### Introduction

In this post, we will write a Consumer Driven Contract Test between a [Gradle](http://gradle.org/) [Spring Boot](https://spring.io/guides/gs/spring-boot/) dummy provider (RESTful Web Service) and a [Gradle](http://gradle.org/) dummy consumer.  

We will make use of [Pact-JVM](https://github.com/DiUS/pact-jvm), which will provide a mock service for the consumer project to generate a Pact, and verification ability for the provider project to verify the Pact.

### Installation

##### Gradle

A Gradle Wrapper allows anybody to work on your project without having to install Gradle.  It ensures that the right version of Gradle that the build was designed for is shipped as part of this project repository.  First clone the sample repository that includes both the dummy provider and the dummy consumer:

>
~~~ bash
bash-3.2$ git clone https://github.com/the-creative-tester/consumer-driven-contract-testing-example.git
Cloning into 'consumer-driven-contract-testing-example'...
remote: Counting objects: 47, done.
remote: Compressing objects: 100% (33/33), done.
remote: Total 47 (delta 2), reused 42 (delta 1), pack-reused 0
Unpacking objects: 100% (47/47), done.
Checking connectivity... done.
~~~

### Provider Setup

Navigate into the ```provider``` directory, where you will see a sample RESTful Web Service that was built using Sprint Boot and Gradle through this [guide](https://spring.io/guides/gs/rest-service/).  Try running the service by using the following command:

>
~~~ shell
bash-3.2$ ./gradlew clean build && java -jar build/libs/gs-actuator-service-0.1.0.jar
..
2016-07-03 14:49:59.367  INFO 32114 --- [           main] hello.HelloWorldConfiguration            : Started HelloWorldConfiguration in 6.782 seconds (JVM running for 7.452)  
~~~

You should now be able to reach to service by navigating to <http://localhost:8080/hello-world> or by using ```curl``` (observe that the id increments for every request made to the service):

>
~~~ shell
bash-3.2$ curl http://localhost:8080/hello-world
{"id":2,"content":"Hello World!"}
~~~

### Consumer Setup

The generation of a Pact is always done from the consumer point of view.  This generation can be broken down into three parts.

##### Part 1: Pact Rule

In this part, we define a mock server's host and port to represent the provider:

>
~~~ java
@Rule
public PactRule rule = new PactRule(Configuration.MOCK_HOST, Configuration.MOCK_HOST_PORT, this);
private DslPart helloWorldResults;
~~~

##### Part 2: Pact Fragment

In this part, we construct a Pact Fragment, which defines the expected contract from a provider:

>
~~~ java
@Pact(state = "HELLO WORLD", provider = Configuration.DUMMY_PROVIDER, consumer = Configuration.DUMMY_CONSUMER)
public PactFragment createFragment(ConsumerPactBuilder.PactDslWithProvider.PactDslWithState builder)
{
    helloWorldResults = new PactDslJsonBody()
            .id()
            .stringType("content")
            .asBody();
>
    return builder
            .uponReceiving("get hello world response")
            .path("/hello-world")
            .method("GET")
            .willRespondWith()
            .status(200)
            .headers(Configuration.getHeaders())
            .body(helloWorldResults)
            .toFragment();
}
~~~

##### Part 3: Pact Verification

In this part, we ensure that the constructed fragment in Part 2, matches the response from the mock server:

>
~~~ java
@Test
@PactVerification("HELLO WORLD")
public void shouldGetHelloWorld() throws IOException
{
    DummyConsumer restClient = new DummyConsumer(Configuration.SERVICE_URL);
    assertEquals(helloWorldResults.toString(), restClient.getHelloWorld());
}
~~~

Try generating a Pact using Gradle from the ```consumer``` directory:

>
~~~ shell
bash-3.2$ ./gradlew test
:compileJava UP-TO-DATE
:processResources UP-TO-DATE
:classes UP-TO-DATE
:compileTestJava
:processTestResources UP-TO-DATE
:testClasses
:test
>
BUILD SUCCESSFUL
>
Total time: 10.67 secs
~~~

The Pact will then be generated and stored in the ```pacts``` directory, but it looks something like this:

>
~~~ json
{
  "provider" : {
    "name" : "dummy-provider"
  },
  "consumer" : {
    "name" : "dummy-consumer"
  },
  "interactions" : [ {
    "providerState" : "HELLO WORLD",
    "description" : "get hello world response",
    "request" : {
      "method" : "GET",
      "path" : "/hello-world"
    },
    "response" : {
      "status" : 200,
      "headers" : {
        "Content-Type" : "application/json;charset=UTF-8"
      },
      "body" : {
        "id" : 5677679801,
        "content" : "dugNvVPasiFRnzqpPNuq"
      },
      "responseMatchingRules" : {
        "$.body.id" : {
          "match" : "type"
        },
        "$.body.content" : {
          "match" : "type"
        }
      }
    }
  } ],
  "metadata" : {
    "pact-specification" : {
      "version" : "2.0.0"
    },
    "pact-jvm" : {
      "version" : "2.1.12"
    }
  }
}
~~~

### Provider

Now that the Pact has been generated by the Consumer, and assuming the Provider (RESTful Web Service) is running.  You can run the following command from the ```provider``` directory to ensure that the Provider meets all expectations of the Consumer:

>
~~~ shell
bash-3.2$ ./gradlew pactVerify
:pactVerify_dummyProvider
>
Verifying a pact between dummy-consumer and dummyProvider
  [Using file /Users/jasonthye/Dev/pact-example-gradle/pacts/dummy-consumer-dummy-provider.json]
  Given HELLO WORLD
         WARNING: State Change ignored as there is no stateChange URL
  get hello world response
    returns a response which
      has status code 200 (OK)
      includes headers
        "Content-Type" with value "application/json;charset=UTF-8" (OK)
      has a matching body (OK)
:pactVerify
>
BUILD SUCCESSFUL
>
Total time: 9.936 secs
~~~

### Full Example

<https://github.com/the-creative-tester/consumer-driven-contract-testing-example>
