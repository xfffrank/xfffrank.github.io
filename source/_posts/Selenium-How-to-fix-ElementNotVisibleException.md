---
title: '[Selenium]How to fix ElementNotVisibleException'
date: 2018-06-27 12:34:35
categories:
- 技术笔记
tags:
---
* I was crawling a website dynamically loaded by js and I continually got this error everytime I want to click an element with Selenium. I was stuck on this issue for a whole day. I tried different kinds of ways to handle with it and they didn't work! When I almost gave it up, I found the problem. 
* In Selenium, if anlement satisfies all the following conditions, it is visible:
  Referring to [this answer on stackoverflow](https://stackoverflow.com/questions/6101461/how-to-force-selenium-webdriver-to-click-on-element-which-is-not-currently-visib)
  1. visibility != hidden
  2. display != none (is also checked against every parent element)
  3. opacity != 0 (this is not checked for clicking an element)
  4. height and width are both > 0
  5. for an input, the attribute type != hidden
  
  So check these out when you encountered the issue of ElementNotVisibleException. In my case, I used `xxx.find_element_by_xpath('//xpath...').click()` to locate the element, which is not accurate enough. Because there are multiple elements with similar xpath expressions. When I check it with `xxx.find_elements_by_xpath('//xpath...')`, I did get a list with multiple elements.The next step is as simple as getting the specific element I want by using the index of the list.

* Keep learning and don't give up easily.

* [UPDATE 2018.06.29]解决过程
  1. Google搜索该错误产生的可能原因
  2. 将原因逐个排查，仍没有解决
  3. 发现google上关于该问题的相关资料不多，说明不是一个普通遇到或难以解决的问题，很可能只是一个简单但没有注意到的问题
  4. 重新调试，发现是因为使用的xpath路径可以定位到不止一个元素而造成的。该网站由多个具有相似结构的部分构成，元素的xpath和css表达比较相似，造成这个错误比较隐蔽。
