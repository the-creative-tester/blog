---
layout: post
title: Python Web Browser Automation
desc: Selenium, Page Objects, Lettuce and Nose
proj-num: 02
colour: 
---



## Selenium, Page Objects, Lettuce and Nose in Python

### Introduction

In this post, we will have a look at using [Selenium WebDriver](http://www.seleniumhq.org/projects/webdriver/) within a Python context.  This is my first usage of Python, and I thought it would be useful to share how to use and setup Selenium WebDriver for those that are new to Web Browser Automation.  We will make use of the [Page Objects](http://selenium-python.readthedocs.org/page-objects.html) design pattern which allows for reusability and also reduction in duplicated code.  We will then make use of [Lettuce](http://lettuce.it/), which is a BDD tool based on Cucumber which allows us to describe our features and scenarios in a natural language wrapping steps that ultimately call our Python functions and drive the browser.  We will also make use of [Nose](http://nose.readthedocs.org/en/latest/testing_tools.html), which allows us to use friendly aids to help with our assertions.

### Installation

##### Python

Install [Python 2.7.10](https://www.python.org/downloads/release/python-2710/).  Please ensure that you allow the installer to update your PATH.  As part of your installation, please also ensure that you install pip, which is a tool that allows easy management of any Python packages that you wish to use.  Installers for versions prior to Python 2.7.9 will not have pip bundled, so if you do choose to use an earlier version, please ensure you manually install pip.

Ensure that you have successfully installed Python:  

>
~~~ shell
bash-3.2$ python --version  
Python 2.7.10
~~~

Ensure that you have successfully installed pip: 

>
~~~ shell
bash-3.2$ pip --version
pip 6.1.1 from /Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/site-packages (python 2.7)
~~~

You can now use the following commands to install the Selenium, Lettuce and Nose packages:

>
~~~ shell
bash-3.2$ pip install selenium
bash-3.2$ pip install lettuce
bash-3.2$ pip install nose
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
    __init__.py
    pages/
    	__init__.py
    steps/
    	__init__.py
~~~

The ```__init__.py``` files can be left empty, but will allow for the containing directories to recognised as Python packages.

### Using Lettuce

To make use of Lettuce, we will first have to create a new file ```pypi_automated_tests/terrain.py```.  Have a read about the usage of terrain [here](http://lettuce.it/reference/terrain.html), but in summary, ```terrain.py``` is the place to put all your setup and configuration, but also allows us to make use of ```world```, a place to hold anything that you want to use across your automated tests.  In this file, place the following code:

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
~~~ gherkin
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
~~~ python
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

If you run ```lettuce``` from ```pypi_automated_tests/``` you will now see that the shell of the implemented steps has succesfully executed due to ```assert True```.

### Using Selenium

Let's make some changes to ```pypi_automated_tests/steps/search_steps.py```.  We will add ```from lettuce import step, world``` so that we can make use of the ```world.driver``` that we had setup in ```pypi_automated_tests/terrain.py```. We will also add ```from nose.tools import assert_equal, assert_true``` so that we can use matchers. We can then start to use the [Selenium Python Bindings](http://selenium-python.readthedocs.org/) to drive the browser:

>
~~~ python
from lettuce import step, world
from nose.tools import assert_equal, assert_true
from selenium.webdriver.common.by import By
>
@step('Given I navigate to the PyPi Home page')
def step_impl(step):
    world.driver.get("https://pypi.python.org/pypi")
    assert_equal(world.driver.title, "PyPI - the Python Package Index : Python Package Index")
>
@step('When I search for "([^"]*)"')
def step_impl(step, search_term):
    world.driver.find_element(By.ID, "term").send_keys(search_term)
    world.driver.find_element(By.ID, "submit").click()
>
@step('Then I am taken to the PyPi Search Results page')
def step_impl(step):
    assert_equal(world.driver.title, "Index of Packages Matching 'lettuce' : Python Package Index")
>
@step('And I see a search result "([^"]*)"')
def step_impl(step, search_result):
    assert_true(world.driver.find_element(By.LINK_TEXT, search_result))
>
~~~

If you run ```lettuce``` from ```pypi_automated_tests/``` you will now see that the implemented steps has succesfully executed.  At this point you have completed your Selenium WebDriver test!

### Using Page Objects

To make use of Page Objects, let's first move the functionality that resided in ```pypi_automated_tests/steps/search_steps.py``` to two new files, ```pypi_automated_tests/pages/home_page.py``` and ```pypi_automated_tests/pages/search_results_page.py```.  Firstly, in ```pypi_automated_tests/pages/home_page.py``` make the following updates:

>
~~~ python
from selenium.webdriver.common.by import By
>
class HomePageLocator(object):
    # Home Page Locators
>
    HEADER_TEXT = (By.XPATH, "//h1")
    SEARCH_FIELD = (By.ID, "term")
    SUBMIT_BUTTON = (By.ID, "submit")
>
>
class HomePage(object):
    # Home Page Actions
>
    def __init__(self, browser):
        self.driver = browser
>
    def fill(self, text, *locator):
        self.driver.find_element(*locator).send_keys(text)
>
    def click_element(self, *locator):
        self.driver.find_element(*locator).click()
>
    def navigate(self, address):
        self.driver.get(address)
>
    def get_page_title(self):
        return self.driver.title
>
    def search(self, search_term):
        self.fill(search_term, *HomePageLocator.SEARCH_FIELD)
        self.click_element(*HomePageLocator.SUBMIT_BUTTON)
>
~~~

Secondly, in ```pypi_automated_tests/pages/search_results.py``` make the following updates:

>
~~~ python
from selenium.webdriver.common.by import By
>
class SearchResultsPageLocator(object):
    # Search Results Page Locators
>
    HEADER_TEXT = (By.XPATH, "//h1")
>
>
class SearchResultsPage(object):
    # Search Results Page Actions
>
    def __init__(self, browser):
        self.driver = browser
>
    def get_element(self, *locator):
        return self.driver.find_element(*locator)
>
    def get_page_title(self):
        return self.driver.title
>
    def find_search_result(self, search_result):
        return self.get_element(By.LINK_TEXT, search_result)
>
~~~

Now, let's update ```pypi_automated_tests/steps/search_steps.py``` to make use of the newly added Page Objects:

>
~~~ python
from lettuce import step, world
from nose.tools import assert_equal, assert_true
>
@step('Given I navigate to the PyPi Home page')
def step_impl(step):
    world.home_page.navigate("https://pypi.python.org/pypi")
    assert_equal(world.home_page.get_page_title(), "PyPI - the Python Package Index : Python Package Index")
>
@step('When I search for "([^"]*)"')
def step_impl(step, search_term):
    world.home_page.search(search_term)
>
@step('Then I am taken to the PyPi Search Results page')
def step_impl(step):
    assert_equal(world.search_results_page.get_page_title(), "Index of Packages Matching 'lettuce' : Python Package Index")
>
@step('And I see a search result "([^"]*)"')
def step_impl(step, search_result):
    assert_true(world.search_results_page.find_search_result(search_result))
>
~~~

Finally, in our ```pypi_automated_tests/terrain.py``` we will need to make these Page Objects avaiable through ```world``` by making the following updates:

>
~~~ python
import os
from lettuce import before, world, after
from selenium import webdriver
from selenium.webdriver.firefox.firefox_binary import FirefoxBinary
from features.pages.home_page import HomePage
from features.pages.search_results_page import SearchResultsPage
>
@before.all
def open_shop():
    open_drivers()
    prepare_pages(world.driver)
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
def prepare_pages(driver):
    world.home_page = HomePage(driver)
    world.search_results_page = SearchResultsPage(driver)
>
def close_drivers():
    if world.driver:
        world.driver.quit()
~~~

### Execution

You can now run ```lettuce``` from ```pypi_automated_tests/```, and you should get the following successful results:

>
~~~ shell
bash-3.2$ lettuce
>
Feature: Search                                     # features/search.feature:1
>
  Scenario: Search PyPI                             # features/search.feature:3
    Given I navigate to the PyPi Home page          # features/steps/search_steps.py:5
    When I search for "lettuce"                     # features/steps/search_steps.py:10
    Then I am taken to the PyPi Search Results page # features/steps/search_steps.py:14
    And I see a search result "lettuce 0.2.21"      # features/steps/search_steps.py:18
Total 1 of 1 scenarios passed!
>
1 feature (1 passed)
1 scenario (1 passed)
4 steps (4 passed)
~~~

### Full Example

<https://github.com/the-creative-tester/python-web-browser-automation-example>
