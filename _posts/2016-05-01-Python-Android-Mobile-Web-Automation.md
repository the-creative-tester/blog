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

We are going to write our first automated test against [Google](http://www.google.com).  By using the ```unittest``` framework, we can make use of the setUp() and tearDown() methods to define the initialization and cleanup for the fixture.  You can run ```adb devices -l``` to get the model of your connected device, which should be used in the ```deviceName``` variable.  If you are running with an emulated device, you may wish to change your ```browserName``` variable to use ```'Browser'```, as emulated devices are not shippied with Chrome.  With this information, you can now create a file, ```appium_example.py``` with the following contents:

>
~~~ python
import unittest
from appium import webdriver
>
class AndroidMobileWebTest(unittest.TestCase):
    def setUp(self):
        desired_capabilities = {
            'platformName': 'Android',
            'platformVersion': '6.0',
            'deviceName': 'Nexus_5',
            'browserName': 'Chrome'
        }
        self.driver = webdriver.Remote('http://localhost:4723/wd/hub', desired_capabilities)
>
    def test_mobileweb(self):
        self.driver.get('http://www.google.com')
        self.driver.find_element_by_name('q').clear()
        self.driver.find_element_by_name('q').send_keys('Appium')
        self.driver.find_element_by_name('q').submit()
>
    def tearDown(self):
        self.driver.quit()
>
~~~

### Execution

Start Appium, and ensure your device is connected with USB Debugging enabled and is not locked.  You can now run ```nosetests appium-example.py``` and you should get the following successful results after your test is run in your device:

>
~~~ shell
bash-3.2$ nosetests appium-example.py
.
----------------------------------------------------------------------
Ran 1 test in 19.118s
>
OK
~~~

### Full Example

<https://github.com/the-creative-tester/python-android-mobile-web-automation-example>
