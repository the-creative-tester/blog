---
layout: post
title: Python Web Automation (Behave)
desc: Selenium, Page Objects, Behave and Nose
proj-num: 08
colour: 
---



## Selenium, Page Objects, Behave and Nose in Python

### Introduction

Earlier, I wrote a [post](http://the-creative-tester.github.io/Python-Web-Browser-Automation-Lettuce/) about using Selenium with Lettuce in a Python context.  In this post, we will have a look at using [Selenium WebDriver](http://www.seleniumhq.org/projects/webdriver/) with [Behave](https://github.com/behave/behave).  Behave is very similar to Lettuce, in that it allows for tests to be written in a natural language style, but it does seem a bit simpler to use and setup.  We will also make use of [Nose](http://nose.readthedocs.org/en/latest/) for its assertions.

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

You can now use the following commands to install the Selenium, Behave and Nose packages:

>
~~~ shell
bash-3.2$ pip install selenium
bash-3.2$ pip install behave
bash-3.2$ pip install nose
~~~

##### Sublime Text

Install [Sublime Text 3](http://www.sublimetext.com/3).

##### Firefox

Install [Firefox](https://www.mozilla.org/en-US/firefox/all/).

### Initial Setup

We are going to write our first automated test against [PyPI](https://pypi.python.org/pypi).  Create a new directory for your test automation project, and open that directory in Sublime Text 3.  

##### Basic Selenium WebDriver Usage

Create a new file called ```sample-test.py``` and place the following contents:

>
~~~ python
from selenium import webdriver
from selenium.webdriver.common.by import By
>
driver = webdriver.Firefox()
driver.get("https://pypi.python.org/pypi")
driver.find_element(By.ID, "term").send_keys("Selenium")
driver.find_element(By.ID, "submit").click()
driver.quit()
~~~

Run the test using ```python sample-test.py``` and observe that Firefox is started:

>
~~~ shell
bash-3.2$ python sample-test.py
~~~

##### Using Python's Unit Testing Framework (unittest)

Here we will start to make use of nose.  The nose library extends unittest to act as a test runner, which provides more meaningful results at execution.

>
~~~ python
import unittest
from selenium import webdriver
from selenium.webdriver.common.by import By
>
class SampleTest(unittest.TestCase):
>
    def pypi_test(self):
        self.driver = webdriver.Firefox()
        self.driver.get("https://pypi.python.org/pypi")
        self.driver.find_element(By.ID, "term").send_keys("Selenium")
        self.driver.find_element(By.ID, "submit").click()
        self.driver.quit()
>
~~~

Run the test using ```nosetests``` or ```nosetests sample-test.py```:

>
~~~ shell
bash-3.2$ nosetests
.
----------------------------------------------------------------------
Ran 1 test in 4.294s
>
OK
~~~

Let's now update the test to include unittest's ```setUp()``` and ```tearDown()``` methods.  Update ```sample-test.py``` with the following contents:

>
~~~ python
import unittest
from selenium import webdriver
from selenium.webdriver.common.by import By
>
class SampleTest(unittest.TestCase):
>
    def setUp(self):
        self.driver = webdriver.Firefox()
>
    def pypi_test(self):
        self.driver.get("https://pypi.python.org/pypi")
        self.driver.find_element(By.ID, "term").send_keys("Selenium")
        self.driver.find_element(By.ID, "submit").click()
>
    def tearDown(self):
        self.driver.quit()
>
~~~

Run the test using ```nosetests``` or ```nosetests sample-test.py```:

>
~~~ shell
bash-3.2$ nosetests
.
----------------------------------------------------------------------
Ran 1 test in 4.563s
>
OK
~~~

### Using Behave

Create a folder structure similar to this:

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

The ```__init__.py``` files can be left empty, but will allow for the containing directories to recognised as Python packages.  To make use of Behave, we will first have to create a new file ```pypi_automated_tests/features/environment.py```.  This file can be used by Behave to define the functions that run ```before_all()``` or ```after_all()``` events in your test, which is very similar to unittest's ```setUp()``` and ```tearDown()``` methods.  In this file, place the following code:

>
~~~ python
from selenium import webdriver
>
def before_all(context):
    context.browser = webdriver.Firefox()
    # context.browser = webdriver.Chrome() if you have set chromedriver in your PATH
    context.browser.set_page_load_timeout(10)
    context.browser.implicitly_wait(10)
    context.browser.maximize_window()
>
def after_all(context):
    context.browser.quit()
>
~~~

Create a new file ```pypi_automated_tests/features/search.feature```:

>
~~~ gherkin
Feature: Search
>
  Scenario: Search PyPI
    Given I navigate to the PyPi Home page
    When I search for "behave"
    Then I am taken to the PyPi Search Results page
    And I see a search result "behave 1.2.5"
~~~

Now, let's create a new file ```pypi_automated_tests/features/steps/search_steps.py```.  In this file, let's first define a shell for our steps:

>
~~~ python
from nose.tools import assert_equal, assert_true
from selenium.webdriver.common.by import By
>
@step('I navigate to the PyPi Home page')
def step_impl(context):
    context.browser.get("https://pypi.python.org/pypi")
    assert_equal(context.browser.title, "PyPI - the Python Package Index : Python Package Index")
>
@step('I search for "{search_term}"')
def step_impl(context, search_term):
    context.browser.find_element(By.ID, "term").send_keys(search_term)
    context.browser.find_element(By.ID, "submit").click()
>
@step('I am taken to the PyPi Search Results page')
def step_impl(context):
    assert_equal(context.browser.title, "Index of Packages Matching 'behave' : Python Package Index")
>
@step('I see a search result "{search_result}"')
def step_impl(context, search_result):
    assert_true(context.browser.find_element(By.LINK_TEXT, search_result))
~~~

If you run ```behave``` from ```pypi_automated_tests/``` you will now see that the test was run successfully:

>
~~~ shell
bash-3.2$ behave
Feature: Search # features/search.feature:1
>
  Scenario: Search PyPI                             # features/search.feature:3
    Given I navigate to the PyPi Home page          # features/steps/search_steps.py:4 1.498s
    When I search for "behave"                      # features/steps/search_steps.py:9 2.556s
    Then I am taken to the PyPi Search Results page # features/steps/search_steps.py:14 0.011s
    And I see a search result "behave 1.2.5"        # features/steps/search_steps.py:18 0.120s
>
1 feature passed, 0 failed, 0 skipped
1 scenario passed, 0 failed, 0 skipped
4 steps passed, 0 failed, 0 skipped, 0 undefined
Took 0m4.184s
~~~

### Using Page Objects

To make use of Page Objects, let's first create ```pypi_automated_tests/features/browser.py```.  Let's move our WebDriver functionality from ```environment.py``` to ```browser.py``` (which also makes it accessible to our Page Objects):

>
~~~ python
from selenium import webdriver
>
class Browser(object):
>
    driver = webdriver.Firefox()
    # driver = webdriver.Chrome() if you have set chromedriver in your PATH
    driver.implicitly_wait(30)
    driver.set_page_load_timeout(30)
    driver.maximize_window()
>
    def close(context):
        context.driver.close()
>
~~~

You can now also update ```environment.py``` to look like this:

>
~~~ python
from selenium import webdriver
from browser import Browser
>
def before_all(context):
    context.browser = Browser()
>
def after_all(context):
    context.browser.close()
>
~~~

Now move the functionality that resided in ```pypi_automated_tests/features/steps/search_steps.py``` to two new files, ```pypi_automated_tests/features/pages/home_page.py``` and ```pypi_automated_tests/features/pages/search_results_page.py```.  Firstly, in ```pypi_automated_tests/features/pages/home_page.py``` make the following updates:

>
~~~ python
from selenium.webdriver.common.by import By
from browser import Browser
>
class HomePageLocator(object):
    # Home Page Locators
>
    HEADER_TEXT = (By.XPATH, "//h1")
    SEARCH_FIELD = (By.ID, "term")
    SUBMIT_BUTTON = (By.ID, "submit")
>
>
class HomePage(Browser):
    # Home Page Actions
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

Secondly, in ```pypi_automated_tests/features/pages/search_results.py``` make the following updates:

>
~~~ python
from selenium.webdriver.common.by import By
from browser import Browser
>
class SearchResultsPageLocator(object):
    # Search Results Page Locators
>
    HEADER_TEXT = (By.XPATH, "//h1")
>
>
class SearchResultsPage(Browser):
    # Search Results Page Actions
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

You can now again update ```environment.py``` to initialise the 2 new Page Objects:

>
~~~ python
from selenium import webdriver
from browser import Browser
>
def before_all(context):
    context.browser = Browser()
    context.home_page = HomePage()
    context.search_results_page = SearchResultsPage()
>
def after_all(context):
    context.browser.close()
>
~~~

Now, let's update ```pypi_automated_tests/features/steps/search_steps.py``` to make use of the newly added Page Objects:

>
~~~ python
from nose.tools import assert_equal, assert_true
from selenium.webdriver.common.by import By
>
@step('I navigate to the PyPi Home page')
def step_impl(context):
    context.home_page.navigate("https://pypi.python.org/pypi")
    assert_equal(context.home_page.get_page_title(), "PyPI - the Python Package Index : Python Package Index")
>
@step('I search for "{search_term}"')
def step_impl(context, search_term):
    context.home_page.search(search_term)
>
@step('I am taken to the PyPi Search Results page')
def step_impl(context):
    assert_equal(context.search_results_page.get_page_title(), "Index of Packages Matching 'behave' : Python Package Index")
>
@step('I see a search result "{search_result}"')
def step_impl(context, search_result):
    assert_true(context.search_results_page.find_search_result(search_result))
>
~~~

Finally, in our ```pypi_automated_tests/features/environment.py``` we will need to make these Page Objects avaiable by making the following updates:

>
~~~ python
from selenium import webdriver
from browser import Browser
from pages.home_page import HomePage
from pages.search_results_page import SearchResultsPage
>
def before_all(context):
    context.browser = Browser()
    context.home_page = HomePage()
    context.search_results_page = SearchResultsPage()
>
def after_all(context):
    context.browser.close()
>
~~~

### Execution

You can now run ```behave``` from ```pypi_automated_tests/```, and you should get the following successful results:

>
~~~ shell
bash-3.2$ behave
Feature: Search # features/search.feature:1
>
  Scenario: Search PyPI                             # features/search.feature:3
    Given I navigate to the PyPi Home page          # features/steps/search_steps.py:4 1.807s
    When I search for "behave"                      # features/steps/search_steps.py:9 5.057s
    Then I am taken to the PyPi Search Results page # features/steps/search_steps.py:13 0.014s
    And I see a search result "behave 1.2.5"        # features/steps/search_steps.py:17 0.142s
>
1 feature passed, 0 failed, 0 skipped
1 scenario passed, 0 failed, 0 skipped
4 steps passed, 0 failed, 0 skipped, 0 undefined
Took 0m7.020s
~~~

### Full Example

<https://github.com/the-creative-tester/python-behave-web-browser-automation-example>
