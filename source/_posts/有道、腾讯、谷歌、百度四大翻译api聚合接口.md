---
title: 有道、腾讯、谷歌、百度四大翻译api聚合接口[python实现]
date: 2018-07-09 17:26:30
tags:
categories:
- 技术笔记
---
> 如需转载，请注明作者。

### api分析
1. 谷歌
使用的是一个[非官方的api](https://github.com/ssut/py-googletrans)，优点是免费且调用非常方便，缺点是不稳定，需要翻墙才能正常使用，且后续可能会被屏蔽。
2. 有道、百度
* 在[有道智云](http://ai.youdao.com/)和[百度翻译开发平台](http://api.fanyi.baidu.com/api/trans/product/index)申请一下账号即可试用服务，大规模使用 api 则需要付费。
* 获得账号以后在各自的接口文档里根据步骤编写代码调用 api 即可，将这两个 api 写在一起是因为两者的调用参数非常相似，签名的生成方法也基本相同。
3. 腾讯
* 同样是在[腾讯云](https://cloud.tencent.com/document/product/551/15611)注册账号以后即可调用接口，暂时是免费的，但有请求限制。
* 腾讯的接口调用比较严格，主要是签名的生成过程比较繁琐，很容易出现错误且难以检测，我用python折腾了几天都不知道错在哪里，最后发现是调用`requests`库的`get`方法时，`params`参数修改了签名的编码导致了错误。

<!-- more -->
### 关键代码
*完整代码请移步 [Github](https://gist.github.com/xfffrank/bcce9cc038f2569769b7f4e1cfd3a65a)*
1. 谷歌
```python
def google_trans(self, query_text):
    lang_to = 'zh-CN' if self.lang_to == 'zh' else self.lang_to # 谷歌api对中文的缩写是zh-CN，在统一调用时需单独处理
    google_trans = Translator()
    result = google_trans.translate(query_text, dest=lang_to).text
    # print('---google---')
    # print(result)
    return result
```
2. 有道
```python
def youdao_get_url_encoded_params(self, query_text):
    """按api调用要求拼接url
    
    Args:
        query_text: 待翻译的文本
        
    Returns:
        符合调用接口要求的参数dict
    """
    app_key = '你的app_key'
    app_secret = '你的app_secret'
    salt = str(round(time.time() * 1000))
    sign_raw = app_key + query_text + salt + app_secret
    sign = hashlib.md5(sign_raw.encode('utf8')).hexdigest()
    params = {
        'q': query_text,
        'from': self.lang_from,
        'to': self.lang_to,
        'appKey': app_key,
        'salt': salt,
        'sign': sign
    }
    return params

def youdao_parse(self, query_text):
    """解析有道api返回的 json 数据
    
    Args:
        query_text: 待翻译的字符串文本
        
    Returns:
        翻译好的文本
    """
    base_url = 'https://openapi.youdao.com/api'
    params = self.youdao_get_url_encoded_params(query_text)
    response = requests.get(base_url, headers=self.headers, params=params).text
    json_data = json.loads(response)
    trans_text = json_data['translation'][0]
    # print('---youdao---')
    # print(trans_text)
    return trans_text
```

3. 百度
```python
def baidu_get_url_encoded_params(self, query_text):
    """按api调用要求拼接url
    
    Args:
        query_text: 待翻译的文本
        
    Returns:
        符合调用接口要求的参数dict
    """
    app_id = '你的app_id'
    app_secret = '你的app_secret'
    salt = str(round(time.time() * 1000))
    sign_raw = app_id + query_text + salt + app_secret
    sign = hashlib.md5(sign_raw.encode('utf8')).hexdigest()
    params = {
        'q': query_text,
        'from': self.lang_from,
        'to': self.lang_to,
        'appid': app_id,
        'salt': salt,
        'sign': sign
    }
    return params

def baidu_parse(self, query_text):
    """解析有道api返回的 json 数据
    
    Args:
        query_text: 待翻译的字符串文本
        
    Returns:
        翻译好的文本
    """
    base_url = 'https://fanyi-api.baidu.com/api/trans/vip/translate'
    params = self.baidu_get_url_encoded_params(query_text)
    response = requests.get(base_url, headers=self.headers, params=params).text
    json_data = json.loads(response)
    res = [i['dst'] for i in json_data['trans_result']]
    trans_text = '\n\n'.join(res)
    # print('---baidu---')
    # print(trans_text)
    return trans_text
```

4. 腾讯
```python
def tencent_get_url_encoded_params(self, query_text):
    action = 'TextTranslate'
    region = 'ap-guangzhou'
    timestamp = int(time.time())
    nonce = random.randint(1, 1e6)
    secret_id = '你的secret_id'
    secret_key = '你的secret_key'  # my secret_key
    version = '2018-03-21'
    lang_from = self.lang_from
    lang_to = self.lang_to

    params_dict = {
        # 公共参数
        'Action': action,
        'Region': region,
        'Timestamp': timestamp,
        'Nonce': nonce,
        'SecretId': secret_id,
        'Version': version,
        # 接口参数
        'ProjectId': 0,
        'Source': lang_from,
        'Target': lang_to,
        'SourceText': query_text
    }
    # 对参数排序，并拼接请求字符串
    params_str = ''
    for key in sorted(params_dict.keys()):
        pair = '='.join([key, str(params_dict[key])])
        params_str += pair + '&'
    params_str = params_str[:-1]
    # 拼接签名原文字符串
    signature_raw = 'GETtmt.tencentcloudapi.com/?' + params_str
    # 生成签名串，并进行url编码
    hmac_code = hmac.new(bytes(secret_key, 'utf8'), signature_raw.encode('utf8'), hashlib.sha1).digest()
    sign = quote(base64.b64encode(hmac_code))
    # 添加签名请求参数
    params_dict['Signature'] = sign
    # 将 dict 转换为 list 并拼接为字符串
    temp_list = []
    for k, v in params_dict.items():
        temp_list.append(str(k) + '=' + str(v))
    params_data = '&'.join(temp_list)
    return params_data

def tencent_parse(self, query_text):
    url_with_args = 'https://tmt.tencentcloudapi.com/?' + self.tencent_get_url_encoded_params(query_text)
    res = requests.get(url_with_args, headers=self.headers)
    json_res = json.loads(res.text)
    trans_text = json_res['Response']['TargetText']
    return trans_text
```


