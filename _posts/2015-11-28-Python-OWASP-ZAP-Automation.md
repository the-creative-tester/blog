---
layout: post
title: Python OWASP ZAP Automation
desc: Using Selenium to Drive OWASP ZAP
proj-num: 03
colour: 
---



## Selenium and OWASP ZAP in Python

### Introduction

In this post, we will have a look at using [Selenium WebDriver](http://www.seleniumhq.org/projects/webdriver/) with [Lettuce](http://lettuce.it/), in a Python context to create tests to drive the browser.  We will then integrate these tests with [OWASP ZAP](https://github.com/zaproxy/zaproxy/), which is a penetration testing tool for discovering vulnerabilities in browser-based applications.

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

You can now use the following commands to install the Selenium, OWASP ZAP, Lettuce and Nose packages:

>
~~~
pip install selenium
pip install python-owasp-zap-v2.4
pip install lettuce
pip install nose
~~~

##### Sublime Text

Install [Sublime Text 3](http://www.sublimetext.com/3).

##### Firefox

Install [Firefox](https://www.mozilla.org/en-US/firefox/all/).

### Gruyere

[Gruyere](https://google-gruyere.appspot.com/) is a small web application that has purposely exposed multiple security vulnerabilities.  An instance of Gruyere can be accessed [here](https://google-gruyere.appspot.com/start).  We will make use of Lettuce to start up OWASP ZAP server on a given port.  We will then trigger a simple Selenium test against Gruyere through the OWASP ZAP server and port, which allows OWASP ZAP to intercept and save the requests sent to the application server by the Selenium test. We will then finally make use of Lettuce to trigger a Active Scan through OWASP ZAP and produce a report.

### Initial Setup

Clone the repository located [here](https://github.com/the-creative-tester/python-zap-example):

>
~~~
git clone https://github.com/the-creative-tester/python-zap-example.git
~~~

You will notice the latest version of ZAP, 2.4.2 is contained in /bin/

### ZAP Configuration in Lettuce

First, let's start the OWASP ZAP server on a specified port of 8090.  Note, we are start ZAP in daemon or headless mode, and we are also disabling the [API key](https://github.com/zaproxy/zaproxy/wiki/FAQapikey) through ```'bin/zap_2.4.2/zap.sh','-daemon', '-config api.disablekey=true'```.  We will also create a Firefox profile that is automatically configured for the OWASP ZAP server and port, which allows all traffic from Firefox to be sent through the started OWASP ZAP server:

>
~~~
import os
import subprocess
from lettuce import before, world, after
from selenium import webdriver
from selenium.webdriver.common.proxy import *
from selenium.webdriver.firefox.firefox_binary import FirefoxBinary
from time import sleep
from zapv2 import ZAPv2
from pprint import pprint
>
@before.all
def open_shop():
    start_zap_server()
    firefox_profile = prepare_firefox_profile()
    open_drivers(firefox_profile)
>
def start_zap_server():
    subprocess.Popen(['bin/zap_2.4.2/zap.sh','-daemon', '-config api.disablekey=true'],stdout=open(os.devnull,'w'))
    world.zap = ZAPv2(proxies={'http': 'http://127.0.0.1:8090', 'https': 'https://127.0.0.1:8090'})
    sleep(5)
>
def prepare_firefox_profile():
    zap_proxy_host = "127.0.0.1"
    zap_proxy_port = 8090
    firefox_profile = webdriver.FirefoxProfile()
    firefox_profile.set_preference("network.proxy.type", 1)
    firefox_profile.set_preference("network.proxy.http", zap_proxy_host)
    firefox_profile.set_preference("network.proxy.http_port", int(zap_proxy_port))
    firefox_profile.set_preference("network.proxy.ssl",zap_proxy_host)
    firefox_profile.set_preference("network.proxy.ssl_port", int(zap_proxy_port))
    firefox_profile.set_preference("browser.startup.homepage", "about:blank")
    firefox_profile.set_preference("startup.homepage_welcome_url", "about:blank")
    firefox_profile.set_preference("startup.homepage_welcome_url.additional", "about:blank")
    firefox_profile.set_preference("webdriver_assume_untrusted_issuer", False)
    firefox_profile.set_preference("accept_untrusted_certs", True)
    firefox_profile.update_preferences()
    return firefox_profile
>
def open_drivers(firefox_profile):
    world.driver = get_firefox(firefox_profile)
    world.driver.set_page_load_timeout(20)
    world.driver.implicitly_wait(20)
    world.driver.maximize_window()
>
def get_firefox(firefox_profile):
    # Locate Firefox from the default directory otherwise use FIREFOX_BIN #
    try:
        driver = webdriver.Firefox(firefox_profile=firefox_profile)
    except Exception:
        my_local_firefox_bin = os.environ.get('FIREFOX_BIN')
        firefox_binary = FirefoxBinary(my_local_firefox_bin)
        driver = webdriver.Firefox(firefox_binary=firefox_binary)
    return driver
>
~~~

### Selenium + Lettuce Setup

Our test that we are executing against Gruyere is defined in ```features/gruyere.feature```:

>
~~~
Feature: Gruyere
>
  Scenario: Gruyere
    Given I navigate to Gruyere
    When I choose to Agree & Start
    Then I am taken to "Gruyere: Home"
>
    Given I choose to Sign Up
    When I choose to Create Account with user name "blue" and password "cheese"
    Then I am taken to "Gruyere: Error"
>
    Given I choose to Sign In
    When I choose to Login with user name "blue" and password "cheese"
    Then I am taken to "Gruyere: Home"
~~~

The corresponding steps for ```gruyere.feature``` are defined in ```features/steps/gruyere.py```:

>
~~~
from lettuce import step, world
from nose.tools import assert_equal, assert_true
from selenium.webdriver.common.by import By
>
@step('I navigate to Gruyere')
def step_impl(step):
    world.driver.get("http://google-gruyere.appspot.com/start")
>
@step('I choose to Agree & Start')
def step_impl(step):
    world.driver.find_element(By.LINK_TEXT, "Agree & Start").click()
>
@step('I am taken to "([^"]*)"')
def step_impl(step, page_title):
    assert_equal(world.driver.title, page_title)
>
@step('I choose to Sign Up')
def step_impl(step):
    world.driver.find_element(By.LINK_TEXT, "Sign up").click()
>
@step('I choose to Create Account with user name "([^"]*)" and password "([^"]*)"')
def step_impl(step, user_name, password):
    world.driver.find_element(By.NAME, "uid").send_keys(user_name)
    world.driver.find_element(By.NAME, "pw").send_keys(password)
    world.driver.find_element(By.XPATH, "//input[@value='Create account']").click()
>
@step('I choose to Sign In')
def step_impl(step):
    world.driver.find_element(By.LINK_TEXT, "Sign in").click()
>
@step('I choose to Login with user name "([^"]*)" and password "([^"]*)"')
def step_impl(step, user_name, password):
    world.driver.find_element(By.NAME, "uid").send_keys(user_name)
    world.driver.find_element(By.NAME, "pw").send_keys(password)
    world.driver.find_element(By.XPATH, "//input[@value='Login']").click()
>
~~~

### ZAP Execution in Lettuce

After we have finished the execution of the Selenium test, we will then instruct ZAP to run a Spider Scan, an Active Scan and finally produce an XML report:

>
~~~
@after.all
def close_shop(total):
    print "Total %d of %d scenarios passed!" % (total.scenarios_passed, total.scenarios_ran)
    close_drivers()
    do_some_zap_stuff()
>
def close_drivers():
    if world.driver:
        world.driver.quit()
>
def do_some_zap_stuff():
    target = "http://google-gruyere.appspot.com"
    print "opening target: " + target
    world.zap.urlopen(target)
    sleep(2.5)
    print "starting spider scan" 
    world.zap.spider.scan(target)
    while (int(world.zap.spider.status()) < 100):
        print "spider scan progress %: " + world.zap.spider.status()
        sleep(1)
    print "starting active scan"
    world.zap.ascan.scan(target)
    sleep(2.5)
    while (int(world.zap.ascan.status()) < 100):
        print "active scan progress %: " + world.zap.ascan.status()
        sleep(1)
    pprint(world.zap.core.alerts())
    report_type = 'xml'
    report_file = 'sample_report.xml'
    with open(report_file, 'a') as f:
        xml = world.zap.core.xmlreport()
        f.write(xml)
        print('Success: {1} report saved to {0}'.format(report_file, report_type.upper()))
    world.zap.core.shutdown()
>
~~~

### Execution

You can now run ```lettuce```, and you should see something similar to the following results:

>
~~~
Total 1 of 1 scenarios passed!
opening target: http://google-gruyere.appspot.com
starting spider scan
spider scan progress %: 0
spider scan progress %: 14
spider scan progress %: 20
spider scan progress %: 40
spider scan progress %: 53
spider scan progress %: 65
spider scan progress %: 74
spider scan progress %: 82
spider scan progress %: 94
[{u'alert': u'Cookie set without HttpOnly flag',
  u'attack': u'943720935142; path=/',
  u'confidence': u'Medium',
  u'cweid': u'0',
  u'description': u'A cookie has been set without the HttpOnly flag, which means that the cookie can be accessed by JavaScript. If a malicious script can be run on this page then the cookie will be accessible and can be transmitted to another site. If this is a session cookie then session hijacking may be possible.',
  u'evidence': u'GRUYERE_ID=943720935142; path=/',
  u'id': u'0',
  u'messageId': u'1',
  u'other': u'',
  u'param': u'GRUYERE_ID',
  u'reference': u'www.owasp.org/index.php/HttpOnly',
  u'risk': u'Low',
  u'solution': u'Ensure that the HttpOnly flag is set for all cookies.',
  u'url': u'http://google-gruyere.appspot.com/start',
  u'wascid': u'13'},
  ..
  ..
]
~~~

### Full Example

<https://github.com/the-creative-tester/python-zap-example>
