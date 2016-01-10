---
layout: post
title: REST API Testing
desc: Node.js, Frisby and Jasmine
proj-num: 04
colour: 
---



## Node.js, Frisby and Jasmine in JavaScript

### Introduction

In this post, we will have a look at using the [Node.js](https://nodejs.org/en/) JavaScript runtime with two packages.  The first, [Frisby](http://frisbyjs.com/), is an API testing framework and the second, [Jasmine](http://jasmine.github.io/), is a test runner.  These can be used together to create and execute automated REST API tests.

### Installation

##### Node.js

Install [Node.js](https://nodejs.org/en/).  As part of the installation, you will also install [npm](https://docs.npmjs.com/getting-started/what-is-npm), which is the package manager for Node.js, allowing for easy access, installation and management of packages in the Node.js repository.

Ensure that you have successfully installed Node.js:  

>
~~~
bash-3.2$ node --version
v4.2.4
~~~

Ensure that you have successfully installed npm: 

>
~~~
bash-3.2$ npm --version
2.14.12
~~~

You can now use the following npm commands to install the Frisby and Jasmine packages:

>
~~~
bash-3.2$ sudo npm install frisby -g
bash-3.2$ sudo npm install jasmine-node -g
~~~

##### Sublime Text

Install [Sublime Text 3](http://www.sublimetext.com/3).

### Open Notify - International Space Station Current Location

[Open Notify](http://open-notify.org/) is an open source project to provide an API for NASA data.  One such API is the [International Space Station Current Location](http://open-notify.org/Open-Notify-API/ISS-Location-Now/) which returns the current longitude and latitude of the ISS.  This is the target API that we will write our test against.

### Initial Setup

We are going to write our first automated test against <http://api.open-notify.org/iss-now>.  Create a new directory for your API test automation project, and open that directory in Sublime Text 3.  Now create a new file in that directory.  Tests scripts are usually named ```*spec.js``` in order for jasmine-node to find them. For this post, I will use ```iss-spec.js```.

### Using Frisby

In your newly created file, place the following code to allow you to utilise the previously installed Frisby module:

>
~~~ 
var frisby = require('frisby');
~~~

Let's write a simple test. We will first make use of the Frisby create() method to define the new test, and the parameter supplied defines the name of the test. Next, the get() method performs a HTTP GET request on the supplied URL. We can use the expectStatus() method to verify the returned HTTP status code, and the expectHeaderContains() method can be used to verify contents within the returned header. Since the API returns its payload in JSON, the content-type should be 'application/json' on the request. The last method we will use is toss(), which is used to execute the test. Here's an example of these methods being used:

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

Let's now make this test a bit more useful, by using the expectJSONTypes() method to check the structure of the response rather than the data:

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

We can also use the expectJSON() method to check the data of the response:

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
