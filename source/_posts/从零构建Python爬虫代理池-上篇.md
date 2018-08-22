---
title: 从零构建Python爬虫代理池--上篇
date: 2018-08-22 10:33:42
tags:
categories:
- 技术笔记
---

我将通过两篇文章，介绍一下如何构建一个简单可用的代理池，以及如何定制一个 get 函数来发送网页请求。对于小规模爬虫来说，这样的工具就足够了。这里说的小规模对应的场景是数据量在十万以下，且爬虫程序不需要一直运行或定时执行的。

我比较懒，就直接讲代码啦。我会把代码结构和全部代码都放出来，也会尽量把注释写详细，把结构写得简洁易懂。这样就省得再放一个 GitHub 链接了。虽然我可能还是会放一下，意思意思哈哈。
<!-- more -->

### 代码结构
我是从[西刺首页](http://www.xicidaili.com/)获取代理的，其实也可以只获取一些[高匿的代理](http://www.xicidaili.com/nn)。改一下html解析部分的代码就好了。

话不多说，上代码。

```python
# 用于发送 HTTP 请求的库
import requests
# 用于解析 HTML 并提取信息
from bs4 import BeautifulSoup
# time 这个库主要用来记录一下时间
import time
# 在验证 ip 有效性的时候我会开多线程
from threading import Thread
'''
队列用于线程间通信。需要注意的是，得从 queue 导入 Queue。
不能从 multiprocessing 导入 Queue，这是用于进程间通信的。
虽然两者用法基本一致，但底层实现不同。
'''
from queue import Queue
```

然后是把获取、验证代理ip并持久化到文件的操作封装成一个类。先摆出类的结构，让各位有个整体的认识，再讲每一个方法。（哈哈其实是因为如果我单独讲每个函数的话整体性比较差，也不利于你们复制粘贴进行测试。）

```python
class GetIp:

    def __init__(self, verify_site):
        '''
        类初始化。

        :param str verify_site: 用于对代理ip进行测试的网站，可以填入要爬的网站链接。
        '''
        # proxy_list 保存从西刺爬下来的代理
        self.proxy_list = []
        # success_list 保存验证通过的代理
        self.success_list = []
        self.site = verify_site
    
    def get_proxies(self):
        '''
        从西刺首页获取 http 和 https 代理地址并存入实例变量 proxy_list。
        '''
        pass
    
    def run(self):
        '''
        负责获取代理，开启多线程任务对代理地址进行验证。
        '''
        pass

    def verify_proxies(self, proxy_q):
        '''
        顾名思义，验证代理的有效性。
        '''
        pass
```

### 获取代理

整体思路是先对西刺首页的 HTML 进行解析，找到 http 和 https 类型的代理，再把类型(http/https)、ip地址、端口号拼接在一起。用 time 模块记录一下运行时间。

```python
def get_proxies(self):
    # 记录开始时间
    start = time.time()
    print('start crawling ip...')
    try:
        # 发送http请求，加一个 User-Agent 的请求头让服务器觉得我们不像爬虫脚本，
        # 不加 headers 也可以。
        html = requests.get('http://www.xicidaili.com/', headers={
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.79 Safari/537.36'
        })
    except:
        # 若请求失败则抛出异常，中断程序，因为获取不到网页内容，继续下去也没意义了。
        raise
    # 以下的解析部分的注释省去，各位看一下 BeautifulSoup 的文档就知道了，不是本文的重点。
    soup = BeautifulSoup(html.text, 'lxml')
    trs = soup.find('table', id='ip_list').find_all('tr', class_=['', 'odd'])[2:]
    for tr in trs:
        scheme = tr.findAll('td', class_="")[-3].get_text().lower()
        if scheme in ['http', 'https']:
            ip = tr.findAll('td', class_="")[0].get_text()
            port = tr.findAll('td', class_="")[1].get_text()
            proxy = scheme + '://' + ip + ':' + port
            self.proxy_list.append(proxy)
    # 打印运行时间
    print('[Get Proxies]total time: %.2f' % (time.time() - start))
```

### 开启多线程任务

思路：开 15 个线程用于验证 ip 有效性，并用队列实现线程间的通信。这里使用多线程的原因是：虽然 Python 中存在 GIL 全局解释器锁，不能完成真正的并行任务，但是在等待时间较长的 IO 操作（网络IO，磁盘IO）时 GIL 会被释放，允许其他线程执行。在完成爬虫任务时，多线程的执行效率会比多进程更高。

撸代码啦。
```python
def run(self):
    # 先执行获取代理的函数，将爬下来的代理暂时保存至实例变量 proxy_list
    self.get_proxies()
    start = time.time()
    # 实例化一个队列对象
    proxy_q = Queue()
    print('Verifying proxies...')
    # workers 列表保存线程
    workers = []
    # 把爬下来的代理地址逐个放进队列
    for p in self.proxy_list:
        proxy_q.put(p)
    # 开启15个线程并放入workers
    # 最后往队列放入数量与线程一样的数字0，作为每个线程结束的标记
    for _ in range(15):
        workers.append(Thread(target=self.verify_proxies, args=(proxy_q,)))
        proxy_q.put(0)
    for w in workers:
        # 启动线程
        w.start()
    for w in workers:
        # 等待线程终止
        w.join()
    print('[verification] total time: %.2f s' % (time.time() - start))
    print('proxies verified !')
```

### 验证代理有效性

思路：对爬下来的代理逐个进行验证，检查返回的状态码是否为200，即请求是否成功。

```python
def verify_proxies(self, proxy_q):
    '''
    :param list proxy_q: 待验证的代理队列
    '''
    # 写一个死循环不断从传入队列中获取代理地址
    while 1:
        proxy = proxy_q.get()
        # 若获取到的值是 0，即退出循环，线程终止
        if proxy == 0:  break
        protocol = 'https' if 'https' in proxy else 'http'
        proxies = {
            protocol: proxy
        }
        try:
            # 若返回状态码不等于200、或在HTTP连接过程中发生其他错误，则抛出异常
            # 设置连接超时时间为3秒
            # self.site为初始化时设置的测试链接
            assert requests.get(self.site, timeout=3, proxies=proxies).status_code == 200
        except:
            print('[fail] %s' % proxy)
        else:
            print('[success] %s' % proxy)
            # 保存验证通过的代理
            self.success_list.append(proxy)
```

### 测试代码

这个类就写完了，来测试一下代码吧。
随便用一个测试网站，把验证通过的代理存入 txt 文件。
```python
if __name__ == '__main__':
    print('Test Site: {}'.format('http://www.hiyd.com/dongzuo/'))
    g = GetIp('http://www.hiyd.com/dongzuo/')
    g.run()
    with open('proxies.txt', 'w') as f:
        for proxy in g.success_list:
            f.write(proxy + '\n')
```

### 总结
本文主要讲了如何构建一个Python爬虫代理池，并详细解释了代码，代码应该还算简洁易懂吧。

如果只想知道如何建立代理池，看到这里就差不多了，下一篇我主要讲如何定制一个url请求函数，除了使用代理ip之外，加入随机 User-Agent 等等，增强爬虫程序的隐蔽性和健壮性。

本人水平有限，如有纰漏，请不吝告知。

> 转载请联系作者
