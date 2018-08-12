---
title: scrapy建立ip代理池教程
date: 2018-07-10 17:41:22
tags:
categories:
- 技术笔记
---
> 如需转载，请注明作者。

完整代码请移步[GitHub](https://github.com/xfffrank/Scrapy/tree/master/meishijie)。

### 获取ip并验证有效性

* 从[西刺](http://www.xicidaili.com/)首页获取了类型为 HTTP 和 HTTPS 的ip，数量不多，此处主要是说明步骤。
* 获取 ip 后将有效 ip 存入 txt 文件。
* 该部分的代码参考了[此博文](https://blog.csdn.net/u011781521/article/details/70194744)。
<!-- more -->
1. 分析网页，获取ip
```python
def get_proxy_ip(self):
    start = time.time()  
    print('start crawling ip...')
    html = requests.get('http://www.xicidaili.com/', headers={
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.79 Safari/537.36'
    })
    soup = BeautifulSoup(html.text, 'lxml')
    trs = soup.find('table', id='ip_list').find_all('tr', class_=['', 'odd'])[2:]
    for tr in trs:
        scheme = tr.findAll('td', class_="")[-3].get_text().lower()
        if scheme in ['http', 'https']:
            ip = tr.findAll('td', class_="")[0].get_text()
            port = tr.findAll('td', class_="")[1].get_text()
            ip_info = scheme + '://' + ip + ':' + port
            self.proxy_list.append(ip_info)
    # print(self.proxy_list)
    print('total time: %.2f' % (time.time() - start))  # 记录函数运行时间
```

2. 对 ip 逐个验证
利用`requests`库对过滤掉没有用的 ip，主要是检查响应状态码。`old_queue`用于进程间通信，第三步会说明。
```python
def verify_one_ip(self, old_queue, new_queue):
    while 1:
        proxy = old_queue.get()
        if proxy == 0:
            break
        protocol = 'https' if 'https' in proxy else 'http'
        proxies = {
            protocol: proxy
        }
        try:
            if requests.get('https://www.baidu.com', timeout=2, proxies=proxies).status_code == 200:
                print('[success] %s' % proxy)
                new_queue.put(proxy)
        except:
            print('[fail] %s' % proxy)
```

3. 使用多进程对 ip 进行验证
`old_queue`保存未验证的 ip 队列，`new_queue`保存验证成功的 ip 队列。
```python
def verify_proxies(self):
    start = time.time()
    old_queue = Queue()
    new_queue = Queue()
    print('verify proxies...')
    workers = []
    for _ in range(15):  # 开启 15 个进程
        workers.append(Process(target=self.verify_one_ip, args=(old_queue, new_queue)))
    for w in workers:
        w.start()
    for ip in self.proxy_list:
        old_queue.put(ip)
    for w in workers:
        old_queue.put(0)  # 为每个进程设置一个结束点，详见 verify_one_ip 的设计。
    for w in workers:
        w.join()
    self.proxy_list = []
    while(1):
        try:
            self.proxy_list.append(new_queue.get(timeout=1))
        except:
            break
    print('[verification] total time: %.2f s' % (time.time() - start))  # 记录验证时间
    print('proxies verified !')
```

4. 将有效 ip 保存到 txt 文件
```python
with open('proxies.txt', 'w') as f:
    for proxy in g.proxy_list:
        f.write(proxy + '\n')
```

### scrapy 设置
打开项目的`middlewares.py`文件，在`your_project_DownloaderMiddleware`类中新增一个`get_random_ip`和`__init__`方法，并重写`process_request`和`process_exception`方法。

1. 从已建立的 ip 池中随机抽取一个 ip 用于代理
```python
def get_random_ip(self):
    with open('proxies.txt', 'r') as f:
        proxies = f.readlines()  # readlines 方法将一次性读取文件的所有行，返回一个列表
    proxy = random.choice(proxies).strip()  # 从返回的列表中随机选取一个 ip
    return proxy
```

2. 增加一个初始化方法，设置 scrapy 的默认代理ip
```python
def __init__(self):
    self.proxy = self.get_random_ip()
```

3. 重写`process_request`和`process_response`方法。

* 说明：当响应状态码不为 200，即连接不成功时，更换 ip，否则使用默认 ip，这样可以有效利用代理 ip。

```python
def process_request(self, request, spider):
    # Called for each request that goes through the downloader
    # middleware.

    # Must either:
    # - return None: continue processing this request
    # - or return a Response object
    # - or return a Request object
    # - or raise IgnoreRequest: process_exception() methods of
    #   installed downloader middleware will be called
    # print('当前代理为 ', self.proxy)
    '''
    每次发请求的时候将代理修改为自定义 ip
    '''
    request.meta['proxy'] = self.proxy
    # return None

def process_exception(self, request, exception, spider):
    # Called when a download handler or a process_request()
    # (from other downloader middleware) raises an exception.

    # Must either:
    # - return None: continue processing this exception
    # - return a Response object: stops process_exception() chain
    # - return a Request object: stops process_exception() chain
    '''
    我在设置中将请求的超时时间设为 3 秒（见第四步），
    这个函数的作用是当超时异常发生时，更换代理 ip
    '''
    self.proxy = self.get_random_ip()
    spider.logger.info('---更换 ip---')
    print('更换 ip 为：', self.proxy)
    request.meta['proxy'] = self.proxy
    return request
```

4. 在项目`settings.py`中作相关设置

```python
DOWNLOADER_MIDDLEWARES = {
   'your_project.middlewares.your_project_DownloaderMiddleware': 400,
}
DOWNLOAD_TIMEOUT = 3  # 设置连接超时时间
```

