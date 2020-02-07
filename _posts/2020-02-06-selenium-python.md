---
published: true
layout: single
title: Web Automation. Selenium WebDriver and Python
excerpt: >-
  Selenium is an open-source web-based automation tool. It can be used for many purposes: to test designed web applications, to scrapping content from web pages, etc. This post will explain some basic implementation of Selenium with Python.
categories: dev
tags: python selenium testing scrapping QA
toc: true
header:
  teaser: /assets/images/selenium/selenium_python_logo.png
  og_image: /assets/images/selenium/selenium_python_logo.png
---

[Selenium][selenium] is a Web Browser Automation Tool.
Primarily, it is for automating web applications for testing purposes, but is certainly not limited to just that.
It allows you to open a browser of your choice & perform tasks as a human being would, such as:
* Clicking buttons
* Entering information in forms
* Searching for specific information on the web pages

## Setup and tools

I’m assuming you know how to use pip and virtual environments. If not, start here.

### install selenium using pip

```
pip install selenium
```

### install chrome driver

You will also need the Google Chrome Driver found [here][chromedriver].
If you run into an error: `chromedriver` executable needs to be in `PATH`, you have a few options.
First, add `chromedriver` to your system path variables.
Second, add `chromedriver.exe` to your Scripts folder of the virtual environment.
Third, explicitly give the location of the driver.

### initiate the driver

I will use MacOS and the last option that explicitly point to `chromedriver`.
So, for using webdriver open Python console or create py-file and insert initialization commands:

```python
from selenium import webdriver
driver = webdriver.Chrome(executable_path='./chromedriver')
```

## Let’s Automate

### First Script

Our first example will be simple.
The script will open a browser, input some URL (`https://www.google.com`) and get title of the loaded site.
Returned title will print into console.

```python
from selenium import webdriver

chrome = webdriver.Chrome(executable_path='./chromedriver')
chrome.get('https://www.google.com')
print(chrome.title)
chrome.quit()
```

### More action

You can find proper documentation on selenium [here][selenium-docs].

Following methods will help to find elements in a webpage (these methods will return a list):
* find_elements_by_name
* find_elements_by_xpath
* find_elements_by_link_text
* find_elements_by_partial_link_text
* find_elements_by_tag_name
* find_elements_by_class_name
* find_elements_by_css_selector

#### search in google

Once we make a request and it is successful we need to get a response.
With response we can make some actions for example input a search string.

```python
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

chrome = webdriver.Chrome(executable_path='./chromedriver')

chrome.get("https://www.google.com")
element = chrome.find_element_by_xpath('//input[@name="q"]')
element.click()
element.send_keys('site:gainanov.pro Web Automation: Selenium WebDriver and Python')
element.send_keys(Keys.RETURN)

chrome.quit()
```

XPaths allow the script to determine the exact web element you want.
In the XPath, double forward slash (`//`) means find an element anywhere on the page.
The star (`*`) means find any element.
The `@` sign specifies the attribute you want.
In this case, I wanted input element with attribute name="q" - `//input[@name="q"]`.

Once you grab a web element, you can act with elements.
For example, I send a search phrase and press return key with `send_keys` method.

Read the [documentation][xpath] and [examples][xpath-examples], you can search by XPath or by other means.

#### waiting

Selenium Webdriver provides two types of waits - **implicit** & **explicit**.
An explicit wait makes WebDriver wait for a certain condition to occur before proceeding further with execution.
An implicit wait makes WebDriver poll the DOM for a certain amount of time when trying to locate an element.

Change the row with `find_element_by_xpath` method to new the method with explicit waits.
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# ...
element = WebDriverWait(chrome, 10).until(
        EC.visibility_of_element_located((By.XPATH, "//input[@name='q']")))
# ...
```
This waits up to 10 seconds before throwing a `TimeoutException` unless it finds the element to return within 10 seconds.


An implicit wait tells WebDriver to poll the DOM for a certain amount of time when trying to find any element (or elements) not immediately available.
The default setting is 0.
Once set, the implicit wait is set for the life of the WebDriver object.
```python
chrome.implicitly_wait(10) # seconds
```

## Conclusion

You now have the foundational skills necessary to scrape websites and automate test a web interface.
But this solution can be run only on local environment.
For the best performance and remote testing read my next [post][selenoid-post].
It will include materials about [Selenoid][selenoid] can help you save time to run parallel scripts remote.

Thank you for reading.

## Additional information

* [What is Selenium? Getting started with Selenium Automation Testing](https://www.edureka.co/blog/what-is-selenium/) - good explanation from Edureka education platform
* [How to Get Started with XPath in Selenium – XPath Tutorial](https://www.edureka.co/blog/xpath-in-selenium/) - Getting started XPath
* [Selenium with Python](https://selenium-python.readthedocs.io/) - Selenium library for Python

[xpath-examples]: https://www.lambdatest.com/blog/complete-guide-for-using-xpath-in-selenium-with-examples
[xpath]: https://selenium-python.readthedocs.io/locating-elements.html#locating-by-xpath
[selenium]: https://selenium.dev/
[selenium-docs]: https://selenium-python.readthedocs.io/
[chromedriver]: https://sites.google.com/a/chromium.org/chromedriver/
[selenoid]: https://aerokube.com/selenoid/latest/
[selenoid-post]: {% post_url 2020-02-07-selenium-in-docker-with-selenoid %}
