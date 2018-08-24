---
title: 从零构建Python爬虫代理池--下篇
date: 2018-08-22 16:39:34
tags:
categories:
- 技术笔记
---
上一篇我们讲了如何构建一个代理池，这一篇来讲讲如何把这个代理池无缝对接到实际应用中。

我们将实现一个下载器，把发送HTTP请求的方法封装成一个类，后续只需要继承这个类就可以直接使用方法了。

### 代码结构
定义一个下载器 Downloader，有一个初始化方法和一个 get 方法，可以理解为定制了 HTTP 中的 GET 方法。如有需要，还可以添加定制化的 POST 等方法。

上代码。
```python
import requests
# 用于从列表中随机选择
import random 
import time
# 导入“上篇”实现好的类
from build_ip_pool import GetIp

class Downloader:

    def __init__(self):
        pass

    def get(self, url, timeout=5):
        '''
        :param str url: 要访问的链接
        :param int timeout: 连接超时时间，默认为 5 秒
        '''
        pass
```

### 定义初始化方法
对于不需要登录的网站，使用随机 User-Agent 和代理 ip 基本上就可以破解爬虫限制了。

在初始化方法中，我们加载存有代理ip的文件，也提供一个 User-Agent 列表供爬虫程序选择。
```python
def __init__(self):
    try:
        with open('proxies.txt') as f:
            self.proxies = f.readlines()
    except:   # 若代理文件不存在，就重新获取代理，并保持至文件以供下次使用
        print('-'*10 + '代理文件不存在，重新获取代理' + '-'*10)
        g = GetIp(verify_site='http://www.hiyd.com/dongzuo/')   # verify_site 替换为你要爬的网站链接
        g.run()
        with open('proxies.txt', 'w') as f:
            for proxy in g.success_list:
                f.write(proxy + '\n')
        self.proxies = g.success_list
    # 随机选择一个代理提供给 requests.get 方法，见下文的 get 函数
    self.proxy = random.choice(self.proxies).strip()
    # 从网上可以搜到 User-Agent 列表，下面的也可以直接复制，用于将爬虫程序伪装成浏览器
    self.user_agent_list = [
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.1 "
        "(KHTML, like Gecko) Chrome/22.0.1207.1 Safari/537.1",
        "Mozilla/5.0 (X11; CrOS i686 2268.111.0) AppleWebKit/536.11 "
        "(KHTML, like Gecko) Chrome/20.0.1132.57 Safari/536.11",
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.6 "
        "(KHTML, like Gecko) Chrome/20.0.1092.0 Safari/536.6",
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.6 "
        "(KHTML, like Gecko) Chrome/20.0.1090.0 Safari/536.6",
        "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/537.1 "
        "(KHTML, like Gecko) Chrome/19.77.34.5 Safari/537.1",
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/536.5 "
        "(KHTML, like Gecko) Chrome/19.0.1084.9 Safari/536.5",
        "Mozilla/5.0 (Windows NT 6.0) AppleWebKit/536.5 "
        "(KHTML, like Gecko) Chrome/19.0.1084.36 Safari/536.5",
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",
        "Mozilla/5.0 (Windows NT 5.1) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",
        "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_8_0) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1063.0 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1062.0 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1062.0 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1061.1 Safari/536.3",
        "Mozilla/5.0 (Windows NT 6.2) AppleWebKit/536.3 "
        "(KHTML, like Gecko) Chrome/19.0.1061.0 Safari/536.3",
        "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/535.24 "
        "(KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24",
        "Mozilla/5.0 (Windows NT 6.2; WOW64) AppleWebKit/535.24 "
        "(KHTML, like Gecko) Chrome/19.0.1055.1 Safari/535.24"
    ]
```

### 定义 get 方法

```python
def get(self, url, timeout=5):
    headers = {
        # 从已有的 User-Agent 列表随机选一个添加到请求头
        # 请求头中还可以添加其他参数，
        # 比如 Referer，用于破解多媒体文件（图片、视频）的防盗链
        'User-Agent': random.choice(self.user_agent_list)
    }
    # 明确代理的协议类型
    protocol = 'https' if 'https' in self.proxy else 'http'
    proxies = {
        protocol: self.proxy
    }
    try:
        response = requests.get(url, headers=headers, proxies=proxies, timeout=timeout)
    except:
        # 若请求出错，则更换代理 ip，并在一秒后重新发送请求，
        # 也就是只要不出错，会一直使用同一个 ip 进行访问，这样可以提高每个有效 ip 的利用率。
        print('[requests]正在更换代理ip...')
        self.proxy = random.choice(self.proxies).strip()
        time.sleep(1)
        return self.get(url)
    else:
        # 请求成功则返回响应文本
        return response.text
```

### What is next?
下载器写完啦，接下来就很简单了，在你的爬虫程序中继承这个类，在访问链接的时候就可以直接调用这个增强版的 get 方法了。

模板如下：
```python
class YourSpider(Downloader):

    def __init__(self):
        # 必须先调用父类(Downloader)的初始化方法
        super().__init__()

    def your_method(self):
        '''
        接下来就可以愉快地编写你的爬虫程序了
        '''
        pass
```

### 总结
通过这两篇文章，大家应该可以学习到如何构建一个可用的代理池并应用到爬虫程序中了。如果爬虫规模比较大，还可以把代理池写成一个服务，封装成api，放到服务器上。可以设置代理池定时进行自我更新，保证每个代理 ip 的可用性。

由于本人水平有限，如有纰漏，请及时告知。