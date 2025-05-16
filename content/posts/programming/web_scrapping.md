---
date: 2025-05-12T10:50:29+02:00
# description: ""
# image: ""
lastmod: 2025-05-12
showTableOfContents: true
tags: ["python", "web", "bs4", "selenium", "notes"]
title: "Web Scrapping"
type: "post"
weight: 1
---

## Introduction

*Web scrapping* is a term used for a program to download and process content from
the web. Python uses several modules to achive this:

* `webbrowser`: comes with Python, opens a browser to a specific page
* `requests`: downloads files and web pages from the internet
* `bs4`: parses HTML
* `selenium`: launches and controls a web browser. It is able to fill in forms
and simulate clicks on the browser.

## Requests

To use the `requests` module, the function `requests.get(url)` is used to get a
`Response` object, which has some attributes that allow to work with the results
from the call to the url.

Some of the attributes and methods for a `Response` object *res* object are the
following:

* `res.status_code`: status of the request. The codes are available in
[this link](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes).
    * 200 means "ok"
    * 404 means "not found"
* `res.text`: if the request succeeded, this attribute will hold the **HTML**
* `res.raise_for_status()`: raises an exception if there was an error
downloading the file and will do nothing if the download succeeded.
* `res.iter_content(100000)`: iterate over the content by chunks of given size
(100,000 bytes in the example).

```py
import requests
res = requests.get('https://google.com')  # get a Result

# try except to identify if there was an error
try:
    res.raise_for_status()
except Exception as e:
    print('There was a problem: %s' % (e))

# writing to a text file
file = open('some_file.txt', 'wb')
for chunk in res.iter_content(100000):
    file.write(chunk)
file.close()
```

## Beautiful Soup (bs4)

Beautiful Soup is a module for extracting/parsing information from an HTML page.

Steps to use:

1. Getting the html (with requests or from a local file)
2. Create a `bs4.BeautifulSoup(html, parser)` object
3. Use the `select(css_selector)` method to get a list of Tag objects that match
the css_selector.

```py
>>> import requests, bs4
>>> res = requests.get('https://nostarch.com')  # get html
>>> # create bs4 object for res
>>> exampleSoup = bs4.BeautifulSoup(res.text, 'html.parser')
>>> elems = exampleSoup.select('#author')
>>> type(elems) # elems is a list of Tag objects.
<class 'list'>
>>> type(elems[0])
<class 'bs4.element.Tag'>
>>> str(elems[0]) # The Tag object as a string.
'<span id="author">Al Sweigart</span>'
>>> elems[0].getText()
'Al Sweigart'
>>> elems[0].attrs  # dictionary with attributes and it's values
{'id': 'author'}
>>> spanElem.get('id')  # get value
'author'
>>> spanElem.get('some_nonexistent_addr') == None  # if exists
True
```

### Most common selector patterns

| Selector | Match |
| -------- | ----- |
| `soup.select('div')` | All elements named `<div>` |
| `soup.select('#author')` | The element with an `id` attribute of `author` |
| `soup.select('.notice')` | All elements with class attribute named `notice` |
| `soup.select('div span')` | All elements named `<span>` that are within an element named `<div>` |
| `soup.select('div > span')` | All elements named `<span>` that are *directly within* an element named `<div>` (no other elements in between) |
| `soup.select('input[name]')` | All elements named `<input>` that have a `name` attribute with any value |
| `soup.select('input[type=button]')` | All elements named `<input>` that have an attribute named `type` with value `button` |

The various selector patterns can be combined to make sophisticated matches.

## Selenium

This module lets Python directly control de browser by clicking links and
filling in login information. It is more advanced than `requests` and `bs4`
because it launches a web browser.

**IMPORTANT NOTE**: All of the functions below are used for Firefox. Here's the
[API reference for Chrome](https://www.selenium.dev/selenium/docs/api/py/webdriver_chrome/selenium.webdriver.chrome.webdriver.html).

### Creating a selenium-controlled browser

1. Import webdriver from selenium
2. Create (and open) a webdriver. It can be Chrome, Firefox or any other browser.
3. Get method to go to a web page.
4. Find elements and interact

```py
>>> from selenium import webdriver  # has to be imported this way
>>> browser = webdriver.Firefox()  # starting a selenium-controlled browser
>>> browser.get('https://inventwithpython.com')  # go to page
```

### Finding elements on the page

`Webdriver` objects methods for finding elements on a page are divided into the
`find_element_*` and `find_elements_*` methods.

* `find_element_*`: returns a single `WebElement` object for the first match
* `find_elements_*`: returns a list of `WebElement` objects for every match

#### Selenium's methods for finding elements

For the methods below, only change `*element*` to `*elements*` to go from the
first object to the list of every object.

|  Method | Object/List returned |
| ------- | -------------------- |
| `browser.find_element_by_class_name(name)` | Elements that use the CSS `class name` |
| `browser.find_element_by_css_selector(selector)` | Elements that match the CSS `selector` |
| `browser.find_element_by_id(id)` | Elements matching `id` attribute value |
| `browser.find_element_by_link_text(text)` | `<a>` elements that completely match the `text` provided |
| `browser.find_element_by_partial_link_text(text)` | `<a>` elements that contain `text` provided |
| `browser.find_element_by_name(name)` | Elements with a matching `name` attribute value |
| `browser.find_element_by_tag_name(name)` | Elements with a matching tag `name` (case-insensitive; an `<a>` element is matched by `'a'` and `'A'`)

Except for the last one, all methods are case-sensitive. If nothing is found,
`selenium` raises a `NoSuchElement` exception.

#### WebElement methods and attributes

After having the `WebElement` object(s), the attributes and methods for it can
be used to know more about it.


| Attribute or method | Description                                                                                             |
|---------------------|---------------------------------------------------------------------------------------------------------|
| `tag_name`            | The tag name, such as `'a'` for an `<a>` element |
| `get_attribute(name)` | The value for the element’s `name` attribute |
| `text`                | The text within the element, such as `'hello'` in `<span>hello </span>` |
| `clear()`             | For text field or text area elements, clears the text typed into it |
| `is_displayed()`      | Returns `True` if the element is visible; otherwise returns `False` |
| `is_enabled()`        | For input elements, returns `True` if the element is enabled; otherwise returns `False` |
| `is_selected()`       | For checkbox or radio button elements, returns `True` if the element is selected; otherwise returns `False` |
| `location`            | A dictionary with keys `'x'` and `'y'` for the position of the element in the page |

#### Actions

`WebElement` objects have methods to interact with the page:

**Clicking on an elmenet**

```py
>>> from selenium import webdriver
>>> browser = webdriver.Firefox()
>>> browser.get('https://inventwithpython.com')
>>> linkElem = browser.find_element_by_link_text('Read Online for Free')
>>> type(linkElem)
<class 'selenium.webdriver.remote.webelement.FirefoxWebElement'>
>>> linkElem.click() # follows the "Read Online for Free" link
```

**Filling out and submitting forms**

```py
>>> from selenium import webdriver
>>> browser = webdriver.Firefox()
>>> browser.get('https://login.metafilter.com')
>>> userElem = browser.find_element_by_id('user_name')
>>> userElem.send_keys('your_real_username_here')


>>> passwordElem = browser.find_element_by_id('user_pass')
>>> passwordElem.send_keys('your_real_password_here')
>>> passwordElem.submit()
```

**Clicking browser buttons**

* `browser.back()`.- Clicks the Back button.
* `browser.forward()`.- Clicks the Forward button.
* `browser.refresh()`.- Clicks the Refresh/Reload button.
* `browser.quit()`.- Clicks the Close Window button.

#### Sending special keys

There are several functions for keys that are impossible to type into a string
value under the `selenium.webdriver.common.keys` module.

| Atrributes                                          | Meanings                                      | 
| --------------------------------------------------- | --------------------------------------------- |
| `Keys.DOWN, Keys.UP, Keys.LEFT, Keys.RIGHT`         | The keyboard arrow keys                       |
| `Keys.ENTER, Keys.RETURN`                           | The ENTER and RETURN keys                     |
| `Keys.HOME, Keys.END, Keys.PAGE_DOWN, Keys.PAGE_UP` | The HOME, END, PAGEDOWN, and PAGEUP keys      |
| `Keys.ESCAPE, Keys.BACK_SPACE, Keys.DELETE`         | The ESC, BACKSPACE, and DELETE keys           |
| `Keys.F1, Keys.F2, . . . , Keys.F12`                | The F1 to F12 keys at the top of the keyboard |
| `Keys.TAB`                                          | The TAB key                                   |

*Example*: moving to the bottom and the top of a page

```py
>>> from selenium import webdriver
>>> from selenium.webdriver.common.keys import Keys
>>> browser = webdriver.Firefox()
>>> browser.get('https://nostarch.com')
>>> htmlElem = browser.find_element_by_tag_name('html')
>>> htmlElem.send_keys(Keys.END)     # scrolls to bottom
>>> htmlElem.send_keys(Keys.HOME)    # scrolls to top
```

NOTE: the `<html></html>` tag refer to the full content of the HTML.

## Summary

| Function | Description |
| -------- | ----------- |
| `webbrowser.open('https://google.com')` | Open the page in the default web browser. |
| `res.status_code` | status of the request |
| `res.text` | this attribute will hold the **HTML** |
| `res.raise_for_status()` | raises an exception if there was an error and nothing if it succeeded |
| `res.iter_content(n)` |x| iterate over the content by chunks of *n* size |