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

You can now use ```pip install <package>``` to install the required packages.

##### Selenium  

>
~~~
pip install selenium
~~~

##### Lettuce  

>
~~~
pip install lettuce
~~~

##### Nose  

>
~~~
pip install nose
~~~

##### Sublime Text

Install [Sublime Text 3](http://www.sublimetext.com/3).

### BDD

### Selenium

### Page Objects

### Execution
