# Python 网络编程(笔记)

代码地址：https://github.com/brandon-rhodes/fopnp

## 第一章 客户端/服务器网络简介

- 介绍Python模块：[Python Module of the Week](https://pymotw.com/3/)
- 查找Python模块：[Python Package Index](https://pypi.org/)

### 构建虚拟环境

<!-- - 创建放置虚拟环境的文件夹: `mkdir python-virtual-env && cd python-virtual-env`
- 新建虚拟环境: `python3 -m venv demo`
- 启用虚拟环境: `source demo/bin/activate` -->

- 新建虚拟环境
    - `virtualenv py-env`
    - `virtualenv -p python3 py3env`
    - `virtualenv -p python2 py2env`
- 进入虚拟环境: `source XXX/bin/activate`
- 退出虚拟环境: `deactivate`

测试环境中指定package是否安装：`python -c 'import package-name'`

列出当前安装的所有包：`pip list`

搜索指定的安装包：`pip search package-name`，搜索过程可以为非完全匹配

安装某个package的同时，也会安装其依赖的package，例如安装pygeocoder后，其依赖的certifi、chardet、idna、requests、urllib也被安装。

### request

获取经度和纬度信息本质上是一次web请求，可以通过curl完成：
```sh
$ curl https://nominatim.openstreetmap.org/search?q=207%20N.%20Defiance%20St,%20Archbold,%20OH&format=json
[{"place_id":220270441,"licence":"Data © OpenStreetMap contributors, ODbL 1.0. https://osm.org/copyright","osm_type":"way","osm_id":672338172,"boundingbox":["41.5388408","41.5427373","-84.3067012","-84.306594"],"lat":"41.5405379","lon":"-84.3066398","display_name":"North Defiance Street, Archbold, Fulton County, 俄亥俄州, 43502, 美国","class":"highway","type":"secondary","importance":0.51}]
```
可以看出返回的是只有一个元素的数组，其中的经纬度信息为：`lat:41.5405379`，`lon:-84.3066398`，用request请求的代码为：
```python
#!env python
import requests

def geocode(address):
    base = "https://nominatim.openstreetmap.org/search"
    parameters = {'q':address, 'format':'json'}
    response = requests.get(base, parameters)
    reply = response.json()
    print(reply[0]['lat'], reply[0]['lon'])

if __name__ == '__main__':
    geocode('207 N. Defiance St, Archbold, OH')
```
运行结果为：
```sh
$ python search2.py
41.5405379 -84.3066398
```

获取 request 默认使用的 user-agent：
```python
#!env python
import requests

def getUserAgent():
    resp = requests.get('http://httpbin.org/user-agent')
    return resp.json()['user-agent']

if __name__ == '__main__':
    print(getUserAgent())
```
对比和 curl 的结果：
```sh
$ curl http://httpbin.org/user-agent
{
  "user-agent": "curl/7.55.1"
}

$ python get-agent.py
python-requests/2.23.0

$ python -c "import requests; print(requests.__version__)"
2.23.0
```

例如 `http://httpbin.org/user-agent` 包含了3个部分：协议的名称（http）、主机名（httpbin.org）、文档路径（user-agent），该URL使用的底层协议为HTTP，HTTP几乎是所有现代网络通信的基础，使用比requests跟底层的库进行http请求：
```python
import http.client
import json
from urllib.parse import quote_plus

if __name__ == '__main__':
    # 创建连接对象，并没有真正连接
    connection = http.client.HTTPConnection('httpbin.org')
    # 建立连接，发送GET请求
    connection.request('GET', '/ip')
    rawreply = connection.getresponse().read()
    # rawreply.decode('utf-8') 返回的是字符串
    reply = json.loads(rawreply.decode('utf-8'))
    print(reply['origin'])
```






