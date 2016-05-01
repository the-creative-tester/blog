---
layout: post
title: Python Android Mobile Web Automation
desc: Android SDK, Appium, Selenium and Nose
proj-num: 09
colour: 
---



## Android SDK, Appium, Selenium (Appium Python Client) and Nose in Python

### Introduction

In this post, we will write a simple Selenium test in Python for Android Mobile Web.  We will make use of the [Android SDK](http://developer.android.com/sdk/index.html) command line tools, [Appium](http://appium.io/) and the Selenium functionality made available by the [Appium Python Client](https://github.com/appium/python-client).

### Installation

##### Android SDK

Download the [Android SDK](http://developer.android.com/sdk/index.html).  You do not need to install Android Studio, so feel free to just download the command line tools.  Unzip the Android SDK to a directory of your choosing.  Set your ```ANDROID_HOME``` to the location of your Android SDK, e.g., ```/Users/jasonthye/Library/Android/sdk```.  Add ```$ANDROID_HOME/platform-tools``` and ```$ANDROID_HOME/tools``` to your ```PATH```.

Once this is done, we need to run the Android SDK Manager to download the relevant packages to create an Android Virtual Device.  To do this, run the following command:

>
~~~ shell
bash-3.2$ android sdk  
~~~

To allow the creation of a Android Virtual Device, ensure for a chosen Android version (e.g., Android 6.0), that you have both the SDK Platform and a System Image (e.g., Intel x86 Atom System Image) packages installed.  Once installed, run the following command to to start Android Virtual Device Manager:

>
~~~ shell
bash-3.2$ android avd  
~~~

Create and start an Android Virtual Device.  When started, you should be able to see that the device can be detected via the Android Device Bridge by running the following command:

>
~~~ shell
bash-3.2$ adb devices
List of devices attached
emulator-5554   device
>  
~~~

You have now successfully installed the Android SDK.  For the rest of the guide we will not use the recently created emulatated device, instead, we will make use of a real device.  Ensure that your device has USB Debugging enabled which can be done after Developer Options has been enabled on your device.  Once this is done, connect your device and run ```adb devices``` to ensure it has been connected.

##### Appium

Appium can easily be downloaded as a desktop application from [here](http://appium.io/downloads.html).  Alternatively, you can install and run Appium as a Node.js application.  To do this, first install [Node.js](https://nodejs.org/en/).  As part of the installation, you will also install [npm](https://docs.npmjs.com/getting-started/what-is-npm), which is the package manager for Node.js, allowing for easy access, installation and management of packages in the Node.js repository.

Ensure that you have successfully installed Node.js:  

>
~~~ shell
bash-3.2$ node --version
v4.2.4
~~~

Ensure that you have successfully installed npm: 

>
~~~ shell
bash-3.2$ npm --version
2.14.12
~~~

You can now use the following npm commands to install the Appium package:

>
~~~ shell
bash-3.2$ sudo npm install appium -g
~~~

Ensure that Appium has been succesfully installed by running the following command:

>
~~~ shell
bash-3.2$ appium
[Appium] Welcome to Appium v1.5.1 (REV 083ae3ce3fd7f5111d5214462a2c7644ea4f5253)
[Appium] Appium REST http interface listener started on 0.0.0.0:4723
~~~

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

You can now use the following commands to install the Appium Python Client and Nose packages:

>
~~~ shell
bash-3.2$ pip install Appium-Python-Client
bash-3.2$ pip install nose
~~~

### Initial Setup

We are going to write our first automated test against [PyPI](https://pypi.python.org/pypi).  Create a new directory for your test automation project, and open that directory in Sublime Text 3.  Now create a folder structure similar to this:

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
