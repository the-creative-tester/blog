---
layout: post
title: Python Web Browser Automation
desc: Selenium, Page Objects, Lettuce and Nose
proj-num: 02
colour: 
---



## Selenium, Page Objects and BDD in Python

### Introduction

In this post, we will have a look at using [Selenium WebDriver](http://www.seleniumhq.org/projects/webdriver/) within a Python context.  This is my first usage of Python, and I thought it would be useful to share how we can use Selenium WebDriver for those that a new to Web Browser Automation.  We will make use of the [Page Objects](http://selenium-python.readthedocs.org/page-objects.html) design pattern which allows for reusability and also reduction in duplicatted code.  We will then make us of [Lettuce](http://lettuce.it/), which is a BDD tool based on Cucumber which will allow us to describe our features and scenarios in a natural language with steps that ultimately call our Python functions and drive the browser.  We will also make use of [Nose](http://nose.readthedocs.org/en/latest/testing_tools.html), which will allow us to use friendly aids to help with our assertions.

### Installation

##### Python

Install [Python 2.7.10](https://www.python.org/downloads/release/python-2710/).  Please ensure that you allow the installer to update your PATH.  As part of your installation, pleaes also ensure that you install pip, which is a tool that allows easy management of any Python packages that you wish to use.  Installers for versions prior to Python 2.7.9 will not have pip bundled, so if you do choose to use an earlier version, please ensure you manually install pip.

Ensure that you have successfully installed Python:  

>
~~~
bash-3.2$ python --version  
Python 2.7.10
~~~

Ensure that you have successfully installed pip: 

>
~~~
bash-3.2$ pip --version
pip 6.1.1 from /Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages (python 2.7)
~~~

You can now use the following commands to install the Selenium, Lettuce and Nose packages:

>
~~~
pip install selenium
pip install lettuce
pip install nose
~~~

##### Sublime Text

Install [Sublime Text 3](http://www.sublimetext.com/3).

##### Firefox

Install [Firefox](https://www.mozilla.org/en-US/firefox/all/).

### Initial Setup

We are going to write our first automated test against [PyPI](https://pypi.python.org/pypi).  Create a new directory for your test automation project, and open that directory in Sublime Text 3.  Now create a folder structure similar to this:

>
~~~
pypi_automated_tests/
  features/
    pages/
    steps/
~~~

### Using Lettuce

To make use of Lettuce, we will first have to create a new file ```pypi_automated_tests/terrain.py```.  Have a read about the usage of terrain [here](http://lettuce.it/reference/terrain.html), but in summary, ```terrain.py``` is the place to put all your setup and configuration, but also allows us to make use of ```world```, a place to dump stuff that you want to use across your automated tests.  In this file, place the following code:

>
~~~ python
import os
from lettuce import before, world, after
from selenium import webdriver
from selenium.webdriver.firefox.firefox_binary import FirefoxBinary
>
@before.all
def open_shop():
    open_drivers()
>
@after.all
def close_shop(total):
    print "Total %d of %d scenarios passed!" % (total.scenarios_passed, total.scenarios_ran)
    close_drivers()
>
def open_drivers():
    world.driver = get_firefox()
    world.driver.set_page_load_timeout(10)
    world.driver.implicitly_wait(10)
    world.driver.maximize_window()
>
def get_firefox():
    # Locate Firefox from the default directory otherwise use FIREFOX_BIN #
    try:
        driver = webdriver.Firefox()
    except Exception:
        my_local_firefox_bin = os.environ.get('FIREFOX_BIN')
        firefox_binary = FirefoxBinary(my_local_firefox_bin)
        driver = webdriver.Firefox(firefox_binary=firefox_binary)
    return driver
>
def close_drivers():
    if world.driver:
        world.driver.quit()
>
~~~

Run ```lettuce``` from ```pypi_automated_tests/```.  You should have successfully launched an instance of Firefox! Now, let's create a new file ```pypi_automated_tests/features/search.feature```.  In this file, let's describe the scenario that we want to test:

>
~~~
Feature: Search
>
  Scenario: Search PyPI
    Given I navigate to the PyPi Home page
    When I search for "lettuce"
    Then I am taken to the PyPi Search Results page
    And I see a search result "lettuce 0.2.21"
~~~

If you run ```lettuce``` from ```pypi_automated_tests/``` you will see that we now have to implement the steps for the above feature.  Now, let's create a new file ```pypi_automated_tests/steps/search_steps.py```.  In this file, let's first define a shell for our steps:

>
~~~
from lettuce import step
>
@step('Given I navigate to the PyPi Home page')
def step_impl(step):
    assert True, 'This step must be implemented'
>
@step('When I search for "([^"]*)"')
def step_impl(step, search_term):
    assert True, 'This step must be implemented'
>
@step('Then I am taken to the PyPi Search Results page')
def step_impl(step):
    assert True, 'This step must be implemented'
>
@step('And I see a search result "([^"]*)"')
def step_impl(step, search_result):
    assert True, 'This step must be implemented'
>
~~~

### Using Selenium

Let's make some changes to ```pypi_automated_tests/steps/search_steps.py```.  We will import ```world``` so that we can make use of the ```world.driver``` that we had setup in ```pypi_automated_tests/terrain.py```. We can start to use the [Selenium Python Bindings](http://selenium-python.readthedocs.org/) to also drive the browser:

>
~~~

~~~

### Using Page Objects

### Using Nose

### Execution
