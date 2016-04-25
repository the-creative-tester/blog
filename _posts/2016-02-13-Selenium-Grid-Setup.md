---
layout: post
title: Selenium Grid Setup
desc: Selenium Grid, ChromeDriver and Python
proj-num: 06
colour: 
---



## Selenium Grid, ChromeDriver and Python

### Introduction

In this post, we will have a look at using [Selenium Grid](http://www.seleniumhq.org/projects/grid/) within a Python context.  SeleniumGrid allows you to run your tests on different machines against different browsers in parallel. If you haven't done so already, ensure that you have followed all the setup steps that are contained in <https://github.com/the-creative-tester/python-lettuce-web-browser-automation-example>.

### Installation

##### Chrome Driver

Download [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/downloads).  Include the ChromeDriver location in your ```PATH``` environment variable.

##### Selenium Grid

Download the latest version of [Selenium Grid](http://selenium-release.storage.googleapis.com/index.html).  This guide will make use of ```selenium-server-standalone-2.52.0.jar```.

A grid consists of a single hub, and one or more nodes. Both are started using the selenium-server.jar executable.

The hub receives a test to be executed along with information on which browser and platform where the test should be run. It knows the configuration of each node that has been registered to the hub. Using this information it selects an available node that has the requested browser-platform combination to run the test against.  In this tutorial, will run Selenium Grid locally.

First, start the hub:

>
~~~ bash
bash-3.2$ java -jar selenium-server-standalone-2.52.0.jar -role hub
21:58:17.788 INFO - Launching Selenium Grid hub
~~~

Second, start the node:

>
~~~ bash
bash-3.2$ java -jar selenium-server-standalone-2.52.0.jar -role node  -hub http://localhost:4444/grid/register
21:59:23.993 INFO - Launching a Selenium Grid node
~~~

Ensure your setup has been successful by navigating to <http://127.0.0.1:4444/grid/console>.

### Initial Setup

Clone a sample project that already has Selenium and Lettuce setup:

>
~~~ bash
bash-3.2$ git clone https://github.com/the-creative-tester/python-lettuce-web-browser-automation-example
Cloning into 'python-lettuce-web-browser-automation-example'...
remote: Counting objects: 25, done.
remote: Total 25 (delta 0), reused 0 (delta 0), pack-reused 25
Unpacking objects: 100% (25/25), done.
Checking connectivity... done.
~~~

### Using Selenium Grid

To make use of Selenium Grid we will have to make a small change in ```terrain.py```.  In this file, modify the ```get_firefox()``` function definiton:

>
~~~ python
def get_firefox():
    try:
        # driver = webdriver.Firefox()
        driver = webdriver.Remote(
            command_executor='http://127.0.0.1:4444/wd/hub',
            desired_capabilities=DesiredCapabilities.CHROME)
    except Exception:
        my_local_firefox_bin = os.environ.get('FIREFOX_BIN')
        firefox_binary = FirefoxBinary(my_local_firefox_bin)
        driver = webdriver.Firefox(firefox_binary=firefox_binary)
    return driver
~~~

### Execution

You can now run ```lettuce``` and you should see the tests run against ChromeDriver through Selenium Grid!

### Full Example

<https://github.com/the-creative-tester/python-selenium-grid-web-browser-automation-example>
