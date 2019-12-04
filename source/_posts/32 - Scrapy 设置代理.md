---
title: Scrapy 设置代理
abbrlink: a6216d9
date: 2019-08-19 14:20:18
tags:
categories:
thumbnail: https://piccdn.freejishu.com/images/2019/08/19/PnfPcp.jpg
---

*这是六等星的第33篇文章*

首先获取到一些可用代理 ，可以在 [快代理](https://www.kuaidaili.com/free/) [66代理](http://www.66ip.cn/) 获取 ，不过免费代理质量差 ，只能临时用一下 。



在 setting.py 文件中加入代理如下 ，当然你现在 看到这篇文章的时候代理都已经失效了 ，需要再次去获取 。 这里我们放纯净的 ip 地址 ，对地址的处理我们放到中间件做 。

```
IPPOOL=[
    "101.132.164.113:8118", "203.195.162.56:3128", "120.83.98.255:9999" "117.90.1.194:9000" ]
```



修改 middlewares.py 文件 ：

```
# 添加两项导入选项
import random
from weather.settings import IPPOOL

# 新增方法
class MyproxiesSpiderMiddleware(object):
    def __init__(self, ip):
        self.ip = ip

    @classmethod
    def from_crawler(cls, crawler):
        return cls(ip = crawler.settings.get('PROXIES'))

    def process_request(self, request, spider):
        ip = random.choice(IPPOOL) # random choice ip from ippool
        print("this is ip: " + ip) # print ip
        request.meta["proxy"] = "http://" + ip
```



将自定义类 添加到下载器中间件位置 ：

```
DOWNLOADER_MIDDLEWARES = {
    # 'weather.middlewares.WeatherDownloaderMiddleware': 543,
    'weather.middlewares.MyproxiesSpiderMiddleware':543
}
```



再次运行爬虫 ，查看是否使用了代理 ：

```
2019-08-19 14:06:59 [scrapy.core.engine] INFO: Spider opened
2019-08-19 14:06:59 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2019-08-19 14:06:59 [scrapy.extensions.telnet] INFO: Telnet console listening on 127.0.0.1:6023
this is ip: 101.132.164.113:8118 # 使用代理
2019-08-19 14:06:59 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://www.weather.com.cn/textFC/beijing.shtml> (referer: None)
```