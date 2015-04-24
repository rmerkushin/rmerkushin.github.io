title: Использование jQuery на любой web-странице
date: 2015-04-23 23:12:00
tags:
   - jquery
   - javascript
   - webdriver
   - python
category:
   - automation testing
---

![](/images/logo-jquery.jpg "jQuery injection")
Простой пример jQuery injection для любой web-страницы.

<!-- more -->

Пример написан с использованием Python и Selenium, но аналогичное решение можно применить как руками из консоли разработчика, так и при помощи других ЯП для которых есть Selenium библиотеки (Java, C#, Ruby, PHP и etc.).

```python
from selenium import webdriver

driver = webdriver.Firefox()
driver.get("http://example.org/")
driver.execute_script(
    'var elem = document.createElement("script");'
    'elem.src = "//code.jquery.com/jquery-2.1.3.min.js";'
    'elem.type="text/javascript";'
    'document.getElementsByTagName("head")[0].appendChild(elem);')
header_text = driver.execute_script('return $("h1").text()')
driver.close()
assert "Example Domain" in header_text
```