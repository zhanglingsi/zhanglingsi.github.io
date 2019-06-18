---
title: 我的 Python 笔记（三）
categories: python
copyright: true
---

# 我的 Python 笔记（三）

## Urllib库   是Python 内置的Http请求库

### 包含以下4个模块

- urllib.request   请求模块
- urlib.error            异常处理模块
- urllib.parse      url解析模块
- urllib.robotparser    robots.txt解析模块


## urlopen方法

### urllib.request.urlopen(url, data=None, [timeout, ]*, cafile=None, capath=None, cadefault=False, context=None)

```python
import urllib.request

response = urllib.request.urlopen('http://www.baidu.com')
print(response.read().decode('utf-8'))
```

```python
import urllib.parse
import urllib.request

data = bytes(urllib.parse.urlencode({'word': 'hello'}), encoding='utf8')
response = urllib.request.urlopen('http://httpbin.org/post', data=data)
print(response.read())
```

    b'{\n  "args": {}, \n  "data": "", \n  "files": {}, \n  "form": {\n    "word": "hello"\n  }, \n  "headers": {\n    "Accept-Encoding": "identity", \n    "Connect-Time": "1", \n    "Connection": "close", \n    "Content-Length": "10", \n    "Content-Type": "application/x-www-form-urlencoded", \n    "Host": "httpbin.org", \n    "Total-Route-Time": "0", \n    "User-Agent": "Python-urllib/3.5", \n    "Via": "1.1 vegur", \n    "X-Request-Id": "89667a57-c909-475a-9870-f01181e8c85d"\n  }, \n  "json": null, \n  "origin": "219.238.82.169", \n  "url": "http://httpbin.org/post"\n}\n'
    


```python
import urllib.request

response = urllib.request.urlopen('http://httpbin.org/get', timeout=1)
print(response.read())
```

    b'{\n  "args": {}, \n  "headers": {\n    "Accept-Encoding": "identity", \n    "Host": "httpbin.org", \n    "User-Agent": "Python-urllib/3.6"\n  }, \n  "origin": "211.138.20.169, 211.138.20.169", \n  "url": "https://httpbin.org/get"\n}\n'
    
<!-- more -->

```python
import socket
import urllib.request
import urllib.error

try:
    response = urllib.request.urlopen('http://httpbin.org/get', timeout=0.1)
except urllib.error.URLError as e:
    if isinstance(e.reason, socket.timeout):
        print('TIME OUT')
```

    TIME OUT
    

## 响应

### 响应类型


```python
import urllib.request

response = urllib.request.urlopen('https://www.python.org')
print(type(response))
```

    <class 'http.client.HTTPResponse'>

### 状态码、响应头

```python
import urllib.request

response = urllib.request.urlopen('https://www.python.org')
print(response.status)
print(response.getheaders())
print(response.getheader('Server'))
```

    200
    [('Server', 'nginx'), ('Content-Type', 'text/html; charset=utf-8'), ('X-Frame-Options', 'SAMEORIGIN'), ('x-xss-protection', '1; mode=block'), ('X-Clacks-Overhead', 'GNU Terry Pratchett'), ('Content-Length', '49283'), ('Accept-Ranges', 'bytes'), ('Date', 'Mon, 18 Feb 2019 06:31:31 GMT'), ('Via', '1.1 varnish'), ('Age', '834'), ('Connection', 'close'), ('X-Served-By', 'cache-lax8626-LAX'), ('X-Cache', 'HIT'), ('X-Cache-Hits', '35'), ('X-Timer', 'S1550471492.850098,VS0,VE0'), ('Vary', 'Cookie'), ('Strict-Transport-Security', 'max-age=63072000; includeSubDomains')]
    nginx


```python
import urllib.request

response = urllib.request.urlopen('https://www.python.org')
print(response.read().decode('utf-8'))
```

```python
from urllib import request, parse

url = 'http://httpbin.org/post'
headers = {
    'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)',
    'Host': 'httpbin.org'
}
dict = {
    'name': 'Germey'
}
data = bytes(parse.urlencode(dict), encoding='utf8')
req = request.Request(url=url, data=data, headers=headers, method='POST')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```

    {
      "args": {}, 
      "data": "", 
      "files": {}, 
      "form": {
        "name": "Germey"
      }, 
      "headers": {
        "Accept-Encoding": "identity", 
        "Connect-Time": "1", 
        "Connection": "close", 
        "Content-Length": "11", 
        "Content-Type": "application/x-www-form-urlencoded", 
        "Host": "httpbin.org", 
        "Total-Route-Time": "0", 
        "User-Agent": "Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)", 
        "Via": "1.1 vegur", 
        "X-Request-Id": "f96e736e-0b8a-4ab4-9dcc-a970fcd2fbbf"
      }, 
      "json": null, 
      "origin": "219.238.82.169", 
      "url": "http://httpbin.org/post"
    }

```python
from urllib import request, parse

url = 'http://httpbin.org/post'
dict = {
    'name': 'Germey'
}
data = bytes(parse.urlencode(dict), encoding='utf8')
req = request.Request(url=url, data=data, method='POST')
req.add_header('User-Agent', 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```

    {
      "args": {}, 
      "data": "", 
      "files": {}, 
      "form": {
        "name": "Germey"
      }, 
      "headers": {
        "Accept-Encoding": "identity", 
        "Connect-Time": "0", 
        "Connection": "close", 
        "Content-Length": "11", 
        "Content-Type": "application/x-www-form-urlencoded", 
        "Host": "httpbin.org", 
        "Total-Route-Time": "0", 
        "User-Agent": "Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)", 
        "Via": "1.1 vegur", 
        "X-Request-Id": "a624bcaa-3581-4b93-84b0-037940338e71"
      }, 
      "json": null, 
      "origin": "219.238.82.169", 
      "url": "http://httpbin.org/post"
    }

## Handler

### 代理

```python
import urllib.request

proxy_handler = urllib.request.ProxyHandler({
    'http': 'http://127.0.0.1:9743',
    'https': 'https://127.0.0.1:9743'
})
opener = urllib.request.build_opener(proxy_handler)
response = opener.open('http://httpbin.org/get')
print(response.read())
```

    b'{\n  "args": {}, \n  "headers": {\n    "Accept-Encoding": "identity", \n    "Connect-Time": "2", \n    "Connection": "close", \n    "Host": "httpbin.org", \n    "Total-Route-Time": "0", \n    "User-Agent": "Python-urllib/3.5", \n    "Via": "1.1 vegur", \n    "X-Request-Id": "b0e2272d-1663-4192-ac45-eb958279afd8"\n  }, \n  "origin": "110.10.176.224", \n  "url": "http://httpbin.org/get"\n}\n'

### Cookie

```python
import http.cookiejar, urllib.request

cookie = http.cookiejar.CookieJar()
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
for item in cookie:
    print(item.name+"="+item.value)
```

    BAIDUID=E77BF84491E332F6F8F1D451AD0063D3:FG=1
    BIDUPSID=E77BF84491E332F6F8F1D451AD0063D3
    H_PS_PSSID=1466_21127_22075
    PSTM=1490198051
    BDSVRTM=0
    BD_HOME=0

```python
import http.cookiejar, urllib.request
filename = "cookie.txt"
cookie = http.cookiejar.MozillaCookieJar(filename)
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
cookie.save(ignore_discard=True, ignore_expires=True)
```

```python
import http.cookiejar, urllib.request
filename = 'cookie.txt'
cookie = http.cookiejar.LWPCookieJar(filename)
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
cookie.save(ignore_discard=True, ignore_expires=True)
```

```python
import http.cookiejar, urllib.request
cookie = http.cookiejar.LWPCookieJar()
cookie.load('cookie.txt', ignore_discard=True, ignore_expires=True)
handler = urllib.request.HTTPCookieProcessor(cookie)
opener = urllib.request.build_opener(handler)
response = opener.open('http://www.baidu.com')
print(response.read().decode('utf-8'))
```

## 异常处理

```python
from urllib import request, error
try:
    response = request.urlopen('http://cuiqingcai.com/index.htm')
except error.URLError as e:
    print(e.reason)
```

    Not Found

```python
from urllib import request, error

try:
    response = request.urlopen('http://cuiqingcai.com/index.htm')
except error.HTTPError as e:
    print(e.reason, e.code, e.headers, sep='\n')
except error.URLError as e:
    print(e.reason)
else:
    print('Request Successfully')
```

    Not Found
    404
    Server: nginx/1.10.1
    Date: Wed, 22 Mar 2017 15:59:55 GMT
    Content-Type: text/html; charset=UTF-8
    Transfer-Encoding: chunked
    Connection: close
    Vary: Cookie
    Expires: Wed, 11 Jan 1984 05:00:00 GMT
    Cache-Control: no-cache, must-revalidate, max-age=0
    Link: <http://cuiqingcai.com/wp-json/>; rel="https://api.w.org/"

```python
import socket
import urllib.request
import urllib.error

try:
    response = urllib.request.urlopen('https://www.baidu.com', timeout=0.01)
except urllib.error.URLError as e:
    print(type(e.reason))
    if isinstance(e.reason, socket.timeout):
        print('TIME OUT')
```

    <class 'socket.timeout'>
    TIME OUT

## URL解析

### urlparse方法

#### urllib.parse.urlparse(urlstring, scheme='', allow_fragments=True)

```python
from urllib.parse import urlparse

result = urlparse('http://www.baidu.com/index.html;user?id=5#comment')
print(type(result), result)
```

    <class 'urllib.parse.ParseResult'> ParseResult(scheme='http', netloc='www.baidu.com', path='/index.html', params='user', query='id=5', fragment='comment')

```python
from urllib.parse import urlparse

result = urlparse('www.baidu.com/index.html;user?id=5#comment', scheme='https')
print(result)
```

    ParseResult(scheme='https', netloc='', path='www.baidu.com/index.html', params='user', query='id=5', fragment='comment')

```python
from urllib.parse import urlparse

result = urlparse('http://www.baidu.com/index.html;user?id=5#comment', scheme='https')
print(result)
```

    ParseResult(scheme='http', netloc='www.baidu.com', path='/index.html', params='user', query='id=5', fragment='comment')

```python
from urllib.parse import urlparse

result = urlparse('http://www.baidu.com/index.html;user?id=5#comment', allow_fragments=False)
print(result)
```

    ParseResult(scheme='http', netloc='www.baidu.com', path='/index.html', params='user', query='id=5#comment', fragment='')

```python
from urllib.parse import urlparse

result = urlparse('http://www.baidu.com/index.html#comment', allow_fragments=False)
print(result)
```

    ParseResult(scheme='http', netloc='www.baidu.com', path='/index.html#comment', params='', query='', fragment='')

### urlunparse

```python
from urllib.parse import urlunparse

data = ['http', 'www.baidu.com', 'index.html', 'user', 'a=6', 'comment']
print(urlunparse(data))
```

    http://www.baidu.com/index.html;user?a=6#comment

### urljoin

```python
from urllib.parse import urljoin

print(urljoin('http://www.baidu.com', 'FAQ.html'))
print(urljoin('http://www.baidu.com', 'https://cuiqingcai.com/FAQ.html'))
print(urljoin('http://www.baidu.com/about.html', 'https://cuiqingcai.com/FAQ.html'))
print(urljoin('http://www.baidu.com/about.html', 'https://cuiqingcai.com/FAQ.html?question=2'))
print(urljoin('http://www.baidu.com?wd=abc', 'https://cuiqingcai.com/index.php'))
print(urljoin('http://www.baidu.com', '?category=2#comment'))
print(urljoin('www.baidu.com', '?category=2#comment'))
print(urljoin('www.baidu.com#comment', '?category=2'))
```

    http://www.baidu.com/FAQ.html
    https://cuiqingcai.com/FAQ.html
    https://cuiqingcai.com/FAQ.html
    https://cuiqingcai.com/FAQ.html?question=2
    https://cuiqingcai.com/index.php
    http://www.baidu.com?category=2#comment
    www.baidu.com?category=2#comment
    www.baidu.com?category=2

### urlencode

```python
from urllib.parse import urlencode

params = {
    'name': 'germey',
    'age': 22
}
base_url = 'http://www.baidu.com?'
url = base_url + urlencode(params)
print(url)
```

    http://www.baidu.com?name=germey&age=22
