---
layout: post
title: Visual Regression Testing
desc: Needle, Selenium, Nose and Python
proj-num: 06
colour: 
---



## Needle, Selenium, Nose and Python

### Introduction

In this post, we will have a look at using [Needle](https://github.com/bfirsh/needle) which allows you to automatically check that your visuals render correctly by taking screenshots of portions of a website and comparing them against known good screenshots.  We can then use [PerceptualDiff](http://pdiff.sourceforge.net/) as an image comparison utility to show the difference between the screenshots.  We will write a simple Selenium test using Needle and PerceptualDiff to automate the checking of the header bar for the [Sydney Morning Herald](http://www.smh.com.au/) home page.

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

You can now use the following commands to install the Selenium, Needle and Nose packages:

>
~~~ shell
bash-3.2$ pip install selenium
bash-3.2$ pip install needle
bash-3.2$ pip install nose
~~~

##### PerceptualDiff

Download the latest version of [PerceptualDiff](https://sourceforge.net/projects/pdiff/files/).  Include the PerceptualDiff folder in your ```PATH``` environment variable.

### Initial Setup

Create a file such as ```sydney-morning-herald-network-strip-test.py``` and write a simple Selenium test:

>
~~~ python
import unittest
from selenium import webdriver
>
class SydneyMorningHeraldNetworkStripTest(unittest.TestCase):
>
    def setUp(self):
        self.driver = webdriver.Firefox()
>
    def test_check_network_strip_of_sydney_morning_herald_home_page(self):
        self.driver.set_page_load_timeout(20)
        self.driver.implicitly_wait(20)
        self.driver.maximize_window()
        self.driver.get('http://www.smh.com.au/')
>
    def tearDown(self):
        self.driver.quit()
>
~~~

Run the test using ```nosetests sydney-morning-herald-network-strip-test.py```:

>
~~~ bash
bash-3.2$ nosetests sydney-morning-herald-network-strip-test.py
.
----------------------------------------------------------------------
Ran 1 test in 15.795s
>
OK
~~~

### Using Needle and PerceptualDiff

To make use of Needle and PerceptualDiff we will have to make a few small changes in ```sydney-morning-herald-network-strip-test.py```.  In this file make the following changes to import and use NeedleTestCase:

>
~~~ python
from needle.cases import NeedleTestCase
from selenium import webdriver
>
class SydneyMorningHeraldNetworkStripTest(NeedleTestCase):
>
    def test_check_network_strip_of_sydney_morning_herald_home_page(self):
        self.driver.set_page_load_timeout(20)
        self.driver.implicitly_wait(20)
        self.driver.maximize_window()
        self.driver.get('http://www.smh.com.au/')
>
~~~

Now, also make the following changes to enable the PerceptualDiff engine, which will allow for a visual difference between captures to be generated (the default engine does not provide this):

>
~~~ python
from needle.cases import NeedleTestCase
from selenium import webdriver
>
class SydneyMorningHeraldNetworkStripTest(NeedleTestCase):
>
    engine_class = 'needle.engines.perceptualdiff_engine.Engine'
~~~

Also make the following changes to set the browser's viewport (this will allow for consistency between capture sizes):

>
~~~ python
from needle.cases import NeedleTestCase
from selenium import webdriver
>
class SydneyMorningHeraldNetworkStripTest(NeedleTestCase):
>
    engine_class = 'needle.engines.perceptualdiff_engine.Engine'
    viewport_width = 1024
    viewport_height = 768
>
~~~

Add the ```assertScreenshot()``` which requires two arguments, a CSS selector for the element that you are capturing and a filename for the captured image.

>
~~~ python
from needle.cases import NeedleTestCase
from selenium import webdriver
>
class SydneyMorningHeraldNetworkStripTest(NeedleTestCase):
>
    engine_class = 'needle.engines.perceptualdiff_engine.Engine'
    viewport_width = 1024
    viewport_height = 768
>
    def test_check_network_strip_of_sydney_morning_herald_home_page(self):
        self.driver.set_page_load_timeout(20)
        self.driver.implicitly_wait(20)
        self.driver.maximize_window()
        self.driver.get('http://www.smh.com.au/')
        self.assertScreenshot('#network-strip', 'network-strip-capture')
>
~~~

### Execution

On first execution of the test, you need to capture a baseline image by using the ```--with-save-baseline``` paramter:

>
~~~ bash
bash-3.2$ nosetests sydney-morning-herald-network-strip-test.py --with-save-baseline
.
----------------------------------------------------------------------
Ran 1 test in 24.711s
>
OK
~~~

This should now create ```screenshots/baseline/network-strip-capture.png```.  Open the file and you should see a timestamp.  Upon a second execution of the test you can remove the ```--with-save-baseline``` parameter, and you should see something similar to this extract:

>
~~~ bash
bash-3.2$ nosetests sydney-morning-herald-network-strip-test.py
..
FAIL: Images are visibly different
87 pixels are different
..
Ran 1 test in 23.810s
>
FAILED (failures=1)
~~~

This should now create ```screenshots/network-strip-capture.png```.  Open the file and you should see a different timestamp.  Also open ```screenshots/network-strip-capture.diff.png``` and you should see the area where the difference was found.

### Full Example

<https://github.com/the-creative-tester/needle-visual-regression-testing-example>
