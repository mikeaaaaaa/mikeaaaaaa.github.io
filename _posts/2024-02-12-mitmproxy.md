---

title: mitmproxy使用
date: 2024-01-12 11:00:00 +0800
categories: [抓包]
tags: [Mitmproxy,Extension]

---
### 选择原因

`mitmproxy`是一种高度可定制化的抓包工具，因为其可以支持`python` API而大受本人的喜爱，当然如果是一般的抓包分析工作的话，我肯定会交给 `charles`（UI操作，快速，操作简单，可视化比较好）。

### 安装：

+ `pip install -i https://pypi.douban.com/simple mitmproxy`

### 工具介绍

安装成功后，可以直接在命令行使用以下三种工具：

mitmproxy：命令行界面,允许交互式检查和修改http数据流.不支持windows
mitmweb： Web界面，用户可以实时看到发生的请求，过滤请求，查看请求数据
mitmdump： 一个命令行工具，没有界面，不能交互，但是可以通过启动参数并结合自定义脚本进行定制化功能的实现，是我们运行的环境

上述也提到了，我们使用Mitmproxy的最大原因就是其高度可定制化，支持`python`脚本对`http`消息做处理。



### 使用模板

使用前提得安装证书，这里不再赘述。

#### 命令行调用（不推荐）

（1）`mitmweb`

`mitmweb -s .\dewu.py`

（2）`mitmdump`

`mitm -s .\dewu.py`

#### python直接启动（推荐）

这里直接给出一个模板：

```python
# -*- coding: utf-8 -*-
# Name:         mitm.py
# Author:       小菜
# Date:         2023/11/03 11:30
# Description:

import atexit
import json
from collections import defaultdict

import mitmproxy.http
from mitmproxy.tools.main import mitmdump
from winproxy import ProxySetting

ps = ProxySetting()


def set_proxy():
    """设置系统代理"""
    ps.enable = True
    ps.server = '127.0.0.1:9527'
    ps.registry_write()
    print('代理已经打开!')


def close_proxy():
    """关闭系统代理"""
    ps.enable = False
    ps.registry_write()
    print('代理已经关闭!')


class ListenComment:
    def __init__(self):
        self.map = {
            'liveObjectId': str(),
            'jsons': dict()
        }
        self.set = set()

    # 定义一个函数，用于处理每一个响应
    def response(self, flow: mitmproxy.http.HTTPFlow) -> None:
        # 判断响应的URL是否是公众号留言的URL
        if "https://mp.weixin.qq.com/mp/appmsg_comment?action=getcomment&scene=0" in flow.request.url:
            # 获取响应的数据包
            response = flow.response
            # 打印出响应的状态码和内容
            print(f"Status: {response.status_code}")
            print(f"Content: {response.content}")
            print(self.parse(data=response.text))

    def parse(self, data: str):
        """解析留言流量包"""
        _data = defaultdict(list)
        try:
            for item in json.loads(data)['elected_comment']:
                _data['nick_name'].append(item['nick_name'])
                _data['content'].append(item['content'])
                _data['like_num'].append(item['like_num'])
                _data['province_name'].append(item['ip_wording']['province_name'])
        except (KeyError, json.decoder.JSONDecodeError):
            ...
        finally:
            return _data


addons = [ListenComment()]

if __name__ == '__main__':
    # 是否需要把当前本机流量转发到mitmproxy上
    system_proxy = Fales
    if(system_proxy):
        # 打开代理
        set_proxy()
        # 注册清理函数
        atexit.register(close_proxy)
    mitmdump(['-s', __file__, '-p 8080', '-q'])


```



### 实战脚本

+ (flow.request.host) file.ljcdn.com 
+ (flow.request.url) https://file.ljcdn.com/shadow2-patch/77910FFE285DC63A36B83E705907AFA6 
+ (flow.request.path) /shadow2-patch/77910FFE285DC63A36B83E705907AFA6 

#### 修改热更新相应包

```python
import json
import os
from mitmproxy import http, ctx
# from datetime import datetime
# from dateutil import parser, tz
# import cv2
md5_val = "f35acc765f760f45021bcdb140a58736"
download_url = "http://192.168.31.91:2222/967452e53292563d688dfda664b4eb6c.jar"
""" dictBody = {
    "code": 200,
    "msg": "success",
    "data": {
        "patchId": "847",
        "buildVersion": "5.36.0.10",
        "targetVersion": "5.36.0.10.1",
        "url": "https://cdn.dewu.com/efe/duapp-wireless-service/10264142/967452e53292563d688dfda664b4eb6c.jar",
        "md5": ,
        "md5Encrypt": ,
        "isAsync": "1",
        "hotAvailable": "1",
        "available": [],
        "immediate": "1",
        "timestamp": 1717666491739,
        "content": "",
        "rollback": false
    },
    "status": 200
} """

def response(flow: http.HTTPFlow) -> None:
    # 获取响应对象

    # if "native-plugins" in flow.request.url:
    #     flow.response.content = flow.response.content.replace(b"0e7185459c2920e95d705f99cb04a584", md5.encode("utf-8"))
    #     print(flow.response.content)

    if  flow.request.url.startswith("https://app.dewu.com/api/v1/app/wireless-platform/client/cold"):
        print("[*] find response")
        ctx.log.info("find response")
        json_body = flow.response.get_text()
        dict_body = json.loads(json_body)
        if(dict_body.get("data","")!=""):
            if(dict_body["data"].get("url","")!=""):
                print("[*] update url")
                ctx.log.info("update url")
                dict_body["data"]["url"] = download_url
            if(dict_body["data"].get("md5","")!=""):
                print("[*] update md5")
                ctx.log.info("update md5")
                dict_body["data"]["md5"] = md5_val
                dict_body["data"]["md5Encrypt"] = md5_val
        flow.response.set_text(json.dumps(dict_body))
        
```

#### 华为商店抓取应用列表

有证书绑定，但是使用一下`objection`自带的解除证书绑定脚本即可。

```python
import json
import os
from mitmproxy import http, ctx
import pandas as pd
from openpyxl import load_workbook


excel_path = os.path.join(os.path.dirname(os.path.abspath(__file__)),'apps_data.xlsx')

def response(flow: http.HTTPFlow) -> None:
    # 获取响应对象

    # if "native-plugins" in flow.request.url:
    #     flow.response.content = flow.response.content.replace(b"0e7185459c2920e95d705f99cb04a584", md5.encode("utf-8"))
    #     print(flow.response.content)

    if  flow.request.url.startswith("https://store-drcn.hispace.dbankcloud.com/hwmarket/api/clientApi"):
        json_body = flow.response.get_text()
        dict_body = json.loads(json_body)
        if(dict_body.get("layoutData","")!=""):
            if (dict_body["layoutData"][0].get("name","")=="热门榜"):
                print("[*] =================================================================")
                app_list = dict_body["layoutData"][0]["dataList"]
                data = {
                    "app_name":[],
                    "app_version":[],
                    "is_web_app":[],
                    "is_cost":[],
                    "download_url":[]
                }
                for i in app_list:
                    name = i["name"]
                    app_version = i["appVersionName"]
                    is_webApp = i["webApp"]
                    down_url = i["downurl"]
                    is_cost = 1 if down_url=="" else 0
                    data["app_name"].append(name)
                    data["app_version"].append(app_version)
                    data["is_web_app"].append(is_webApp)
                    data["is_cost"].append(is_cost)
                    data["download_url"].append(down_url)
                print(data)
                if not os.path.exists(excel_path):
                    pd.DataFrame(data).to_excel(excel_path,sheet_name="app",index=False)
                else:
                    # 加载现有的 Excel 文件
                    df = pd.read_excel(excel_path)
                    new_df = pd.DataFrame(data)
                    pd.concat([df,new_df],ignore_index=True).to_excel(excel_path, sheet_name="app",index=False)
```

