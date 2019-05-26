# Scrapy框架学习笔记

## 安装（略）

## Scrapy基本用法讲解

### 引子

非常强大的 异步 爬虫框架，内置许多组件，可以只关心 爬虫逻辑

本节先介绍简单的项目，走一遍Scrapy的爬虫流程，大体熟悉基本用法

### 目标站点分析

Scrapy官方提供的抓取网站url : <http://quotes.toscrape.com/> 

这个网站主要显示了一些名人名言，作者，Tags等信息，点击底部的next可以翻页，url变更为：<http://quotes.toscrape.com/page/2/>

Get请求：改变url就可以翻页

网页没有任何反爬虫措施

网页结构也比较简单：class、id较为清晰

### 流程框架

#### 抓取第一页

请求url并得到源代码，进行下一步分析

#### 获取内容和下一个链接

提取内容、获取下一页链接等待进一步爬取

#### 翻页爬取

请求下一页信息，分析内容并请求再下一页链接

#### 保存爬取结果

将爬取结果保存为特定格式如文本、数据库

### 实战演练

#### 创建项目、分析并运行

```powershell
Scrapy 1.6.0 - no active project

Usage:
  scrapy <command> [options] [args]

Available commands:
  bench         Run quick benchmark test
  fetch         Fetch a URL using the Scrapy downloader
  genspider     Generate new spider using pre-defined templates
  runspider     Run a self-contained spider (without creating a project)
  settings      Get settings values
  shell         Interactive scraping console
  startproject  Create new project
  version       Print Scrapy version
  view          Open URL in browser, as seen by Scrapy

  [ more ]      More commands available when run from project directory

Use "scrapy <command> -h" to see more info about a command
```

首先创建项目：

```powershell
scrapy startproject quotetutorial
```

显示如下：

```powershell
New Scrapy project 'quotetutorial', using template directory '~\appdata\local\programs\python\python37\lib\site-packages\scrapy\templates\project', created in:
    ...(hide some path)\quotetutorial

You can start your first spider with:
    cd quotetutorial
    scrapy genspider example example.com
```

```powershell
cd quotetutorial
ls
```

scrapy.cfg 配置文件，定义了部署信息

```python
# Automatically created by: scrapy startproject
#
# For more information about the [deploy] section see:
# https://scrapyd.readthedocs.io/en/latest/deploy.html

[settings]
default = quotetutorial.settings

[deploy]
#url = http://localhost:6800/
project = quotetutorial

```

紧接着用以下命令 py文件名称  和 要爬取的域名

```powershell
scrapy genspider quotes quotes.toscrape.com
```

```powershell
Created spider 'quotes' using template 'basic' in module:
  quotetutorial.spiders.quotes
```

生成文件 quotes.py

```python
# -*- coding: utf-8 -*-
import scrapy


class QuotesSpider(scrapy.Spider):
    name = 'quotes'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        pass

```

items.py 用来保存数据结构

```python
# -*- coding: utf-8 -*-

# Define here the models for your scraped items
#
# See documentation in:
# https://doc.scrapy.org/en/latest/topics/items.html

import scrapy


class QuotetutorialItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    pass

```

middlewares 中间件，用来处理res req Exception等操作
piplines 管道 输出一些items
settings 配置信息，

最主要运行的代码在/spiders中
quotes.py中
name: 指定了spider的名称，唯一，用来表示
allowed_domains = ['quotes.toscrape.com'] # 爬取相关限制
start_urls = ['http://quotes.toscrape.com/'] # 开头链接配置
def parse 解析请求

```powershell
scrapy crawl quotes
# 执行 name 为 quotes 的 Spider
```

不像日常爬虫一样，而是提供了额外的输出，主要是一些调试信息

```powershell
# 已经启动
2019-05-22 20:50:36 [scrapy.utils.log] INFO: Scrapy 1.6.0 started (bot: quotetutorial)
# 版本信息
2019-05-22 20:50:36 [scrapy.utils.log] INFO: Versions: lxml 4.3.3.0, libxml2 2.9.7, cssselect 1.0.3, parsel 1.5.1, w3lib 1.20.0, Twisted 19.2.0, Python 3.7.2 (tags/v3.7.2:9a3ffc0492, Dec 23 2018, 23:09:28) [MSC v.1916 64 bit (AMD64)], pyOpenSSL 19.0.0 (OpenSSL 1.1.1b  26 Feb 2019), cryptography 2.6.1, Platform Windows-10-10.0.17134-SP0
# 当前的设置
2019-05-22 20:50:36 [scrapy.crawler] INFO: Overridden settings: {'BOT_NAME': 'quotetutorial', 'NEWSPIDER_MODULE': 'quotetutorial.spiders', 'ROBOTSTXT_OBEY': True, 'SPIDER_MODULES': ['quotetutorial.spiders']}
2019-05-22 20:50:36 [scrapy.extensions.telnet] INFO: Telnet Password: c76fa9022b4e01b1
2019-05-22 20:50:36 [scrapy.middleware] INFO: Enabled extensions:
['scrapy.extensions.corestats.CoreStats',
 'scrapy.extensions.telnet.TelnetConsole',
 'scrapy.extensions.logstats.LogStats']
 
 # 使用的中间件
2019-05-22 20:50:37 [scrapy.middleware] INFO: Enabled downloader middlewares:
['scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware',
 'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware',
 'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware',
 'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware',
 'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware',
 'scrapy.downloadermiddlewares.retry.RetryMiddleware',
 'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware',
 'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware',
 'scrapy.downloadermiddlewares.redirect.RedirectMiddleware',
 'scrapy.downloadermiddlewares.cookies.CookiesMiddleware',
 'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware',
 'scrapy.downloadermiddlewares.stats.DownloaderStats']
2019-05-22 20:50:37 [scrapy.middleware] INFO: Enabled spider middlewares:
['scrapy.spidermiddlewares.httperror.HttpErrorMiddleware',
 'scrapy.spidermiddlewares.offsite.OffsiteMiddleware',
 'scrapy.spidermiddlewares.referer.RefererMiddleware',
 'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware',
 'scrapy.spidermiddlewares.depth.DepthMiddleware']
2019-05-22 20:50:37 [scrapy.middleware] INFO: Enabled item pipelines:
[]

# 调用了Spider开始抓取……
2019-05-22 20:50:37 [scrapy.core.engine] INFO: Spider opened
2019-05-22 20:50:37 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2019-05-22 20:50:37 [scrapy.extensions.telnet] INFO: Telnet console listening on 127.0.0.1:6023

# 没有找到robots.txt
2019-05-22 20:50:37 [scrapy.core.engine] DEBUG: Crawled (404) <GET http://quotes.toscrape.com/robots.txt> (referer: None)
# 从 start_urls 开始抓取
2019-05-22 20:50:37 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/> (referer: None)

# 这里会输出爬取到的信息(此处无)

2019-05-22 20:50:37 [scrapy.core.engine] INFO: Closing spider (finished)

# 输出一些相关的爬取信息
2019-05-22 20:50:37 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 446,
 'downloader/request_count': 2,
 'downloader/request_method_count/GET': 2,
 'downloader/response_bytes': 2701,
 'downloader/response_count': 2,
 'downloader/response_status_count/200': 1,
 'downloader/response_status_count/404': 1,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2019, 5, 22, 12, 50, 37, 940727),
 'log_count/DEBUG': 2,
 'log_count/INFO': 9,
 'response_received_count': 2,
 'robotstxt/request_count': 1,
 'robotstxt/response_count': 1,
 'robotstxt/response_status_count/404': 1,
 'scheduler/dequeued': 1,
 'scheduler/dequeued/memory': 1,
 'scheduler/enqueued': 1,
 'scheduler/enqueued/memory': 1,
 'start_time': datetime.datetime(2019, 5, 22, 12, 50, 37, 173611)}
2019-05-22 20:50:37 [scrapy.core.engine] INFO: Spider closed (finished)
```

默认情况下只爬取了start_urls里边的信息，请求完后就默认回调 parse方法

+ response.text | 直接输出网页源代码

#### 分析站点

每个区块是 一个 div.quote

```html
<div class="quote" itemscope="" itemtype="http://schema.org/CreativeWork">
        <span class="text" itemprop="text">“It takes a great deal of bravery to stand up to our enemies, but just as much to stand up to our friends.”</span>
        <span>by <small class="author" itemprop="author">J.K. Rowling</small>
        <a href="/author/J-K-Rowling">(about)</a>
        </span>
        <div class="tags">
            Tags:
            <meta class="keywords" itemprop="keywords" content="courage,friends"> 
            
            <a class="tag" href="/tag/courage/page/1/">courage</a>
            
            <a class="tag" href="/tag/friends/page/1/">friends</a>
            
        </div>
    </div>
```

Tags数量不固定

#### 编写items.py

爬取的时候需要存储这些数据，定义统一的数据结构，指定一些字段，把这些数据作为整体进行存储

这里修改了类名称

```python
class QuoteItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    text = scrapy.Field()
    author = scrapy.Field()
    tags = scrapy.Field()
    pass
```

全部定义为Field，可以定义Item，在爬取完进行赋值

#### 编写parse方法

response 是爬取后得到的，只需要负责 解析操作，无需关心如何请求

```python
from quotetutorial.items import QuoteItem

# 利用CSS选择器选取区块
quotes = response.css('.quote')
for quote in quotes:
    # ::text是Scrapy特有的语法，输出里边的文本内容
    # 仅有一个内容的时候 用 extract_first() 提取
    # 多个内容 用 extract() 提取全部内容
    item = QuoteItem()
    text = quote.css('.text::text').extract_first()
    author = quote.css('.author::text').extract_first()
    tags = quote.css('.tags .tag::text').extract()
    item['text'] = text
    item['author'] = author
    item['tags'] = tags
    yield item
```

调用 yield 把 item 生成出来，Scrapy会识别 item 作下一步存储等操作

```powershell
scrapy crawl quotes
```

```powershell
2019-05-22 21:40:44 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'Albert Einstein',
 'tags': ['change', 'deep-thoughts', 'thinking', 'world'],
 'text': '“The world as we have created it is a process of our thinking. It '
         'cannot be changed without changing our thinking.”'}
2019-05-22 21:40:44 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'J.K. Rowling',
 'tags': ['abilities', 'choices'],
 'text': '“It is our choices, Harry, that show what we truly are, far more '
         'than our abilities.”'}
2019-05-22 21:40:44 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'Albert Einstein',
 'tags': ['inspirational', 'life', 'live', 'miracle', 'miracles'],
 'text': '“There are only two ways to live your life. One is as though nothing '
         'is a miracle. The other is as though everything is a miracle.”'}
2019-05-22 21:40:44 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'Jane Austen',
 'tags': ['aliteracy', 'books', 'classic', 'humor'],
 'text': '“The person, be it gentleman or lady, who has not pleasure in a good '
         'novel, must be intolerably stupid.”'}
2019-05-22 21:40:44 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'Marilyn Monroe',
 'tags': ['be-yourself', 'inspirational'],
 'text': "“Imperfection is beauty, madness is genius and it's better to be "
         'absolutely ridiculous than absolutely boring.”'}
2019-05-22 21:40:44 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'Albert Einstein',
 'tags': ['adulthood', 'success', 'value'],
 'text': '“Try not to become a man of success. Rather become a man of value.”'}
2019-05-22 21:40:44 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'André Gide',
 'tags': ['life', 'love'],
 'text': '“It is better to be hated for what you are than to be loved for what '
         'you are not.”'}
2019-05-22 21:40:44 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'Thomas A. Edison',
 'tags': ['edison', 'failure', 'inspirational', 'paraphrased'],
 'text': "“I have not failed. I've just found 10,000 ways that won't work.”"}
2019-05-22 21:40:44 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'Eleanor Roosevelt',
 'tags': ['misattributed-eleanor-roosevelt'],
 'text': '“A woman is like a tea bag; you never know how strong it is until '
         "it's in hot water.”"}
2019-05-22 21:40:44 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'Steve Martin',
 'tags': ['humor', 'obvious', 'simile'],
 'text': '“A day without sunshine is like, you know, night.”'}
```

至此，单页名言的简单爬取完成

#### 补：Scrapy shell

```powershell
# scrapy shell [url] 进入命令行交互模式
scrapy shell quotes.toscrape.com
```

在命令行交互模式进行 response 相关调试

```powershell
>>> quotes = response.css('.quote')
>>> quotes
[<Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>, <Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>, <Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>, <Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>, <Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>, <Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>, <Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>, <Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>, <Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>, <Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>]

>>> quotes[0]
<Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype="h'>

>>> quotes[0].css('.text')
# 输出结果为 list,只有1个结果
# data 属性为 HTML标签内容
[<Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' text ')]" data='<span class="text" itemprop="text">“The '>]

>>> quotes[0].css('.text::text')
# 加了::text 后不同之处：data属性为字符串
[<Selector xpath="descendant-or-self::*[@class and contains(concat(' ', normalize-space(@class), ' '), ' text ')]/text()" data='“The world as we have created it is a pr'>]

>>> quotes[0].css('.text::text').extract()
# 返回 list
['“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”']

>>> quotes[0].css('.text').extract()
['<span class="text" itemprop="text">“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”</span>']

>>> quotes[0].css('.text::text').extract_first()
'“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'

>>> quotes[0].css('.tags .tag::text').extract()
['change', 'deep-thoughts', 'thinking', 'world']

>>> quotes[0].css('.tags .tag::text').extract_first()
'change'
```

通过上述操作 体会到 extract() 和 extract_first() 的不同之处

+ extract()  |  返回列表
+ extract_first()  |  返回第1条结果

#### 翻页的实现

+ url 在变化，可以直接通过请求url 得到网页的下一页
+ next 对应下一页的 超链接，也可以提取出下一页的 url

在def parse中继续添加如下代码：

```python
next = response.css('.pager .next a::attr(href)').extract_first()
# 获取绝对路径
url = response.urljoin(next)
# 生成Request
# callback 回调函数，得到的res的解析函数，这里递归调用自己
yield scrapy.Request(url=url, callback=self.parse)
```

第1页解析后，重新发起一次请求，指定url、callback，也就是说parse生成了下一页的链接再回调自己，再下一页还会生成下一页的链接继续回调自己……这样就完成了 递归 的调用，用这种机制实现翻页循环

```powershell
scrapy crawl quotes
```

```powershell
2019-05-22 21:56:44 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/page/9/>
# data
2019-05-22 21:56:44 [scrapy.dupefilters] DEBUG: Filtered duplicate request: <GET http://quotes.toscrape.com/page/10/> - no more duplicates will be shown (see DUPEFILTER_DEBUG to show all duplicates)
2019-05-22 21:56:44 [scrapy.core.engine] INFO: Closing spider (finished)
2019-05-22 21:56:44 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 2870,
 'downloader/request_count': 11,
 'downloader/request_method_count/GET': 11,
 'downloader/response_bytes': 24812,
 'downloader/response_count': 11,
 'downloader/response_status_count/200': 10,
 'downloader/response_status_count/404': 1,
 'dupefilter/filtered': 1,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2019, 5, 22, 13, 56, 44, 734927),
 'item_scraped_count': 100,
 'log_count/DEBUG': 112,
 'log_count/INFO': 9,
 'request_depth_max': 10,
 'response_received_count': 11,
 'robotstxt/request_count': 1,
 'robotstxt/response_count': 1,
 'robotstxt/response_status_count/404': 1,
 'scheduler/dequeued': 10,
 'scheduler/dequeued/memory': 10,
 'scheduler/enqueued': 10,
 'scheduler/enqueued/memory': 10,
 'start_time': datetime.datetime(2019, 5, 22, 13, 56, 39, 409048)}
2019-05-22 21:56:44 [scrapy.core.engine] INFO: Spider closed (finished)
```

再次运行，爬到第10页就停止了

至此，通过非常简单的代码完成了整个站点的抓取，翻页和生成item就是一些简单的配置，体会两种机制：

+ yield item  |  字典类型的数据
+ yield scrapy.Request()  |  重新发送一次请求

#### 保存数据

-o 可以指定输出文件的格式和名称，自动判断后缀；保存很方便，也不需要额外写方法

```powershell
scrapy crawl quotes -o quotes.json
# 在目录处 生成了json格式的数据文件，可以用 json.loads进行解析
```

```powershell
scrapy crawl quotes -o quotes.jl
# 结果存成了单独一行的数据，没有 []
```

```powershell
scrapy crawl quotes -o quotes.csv
```

```powershell
scrapy crawl quotes -o quotes.xml
```

另外还有pickle. marshal等格式

还可以用 远程 FTP的保存方式
```powershell
scrapy crawl quotes -o ftp://user:pass@ftp.example.com/path/quotes.csv
```

#### piplines方式

在保存之前，还需要对item进行处理，比如解析后，有一些item不是想要的，或者想存到数据库中，上述方式就不能提供服务了，需要借助piplines工具，加入后注意看温馨提示：

```
Don't forget to add your pipeline to the ITEM_PIPELINES setting
```

```python
# piplines.py
import pymongo
from scrapy.exceptions import DropItem


class TextPipeline(object):

    def __init__(self):
        self.limit = 50

    def process_item(self, item, spider):
        # 对 item 进行处理后返回
        if item['text']:
            if len(item['text']) > self.limit:
                item['text'] = item['text'][0: self.limit].rstrip() + '...'
                # item 是函数必须返回的类型之一
                return item
        else:
            # 如果不存在则“抛出异常” item被舍弃
            return DropItem('Missing Text')
        return item

# 如果想保存到MongoDB，则定义一个piplines
class MongoPipeline(object):

    def __init__(self, mongo_url, mongo_db):
        self.mongo_url = mongo_url
        self.mongo_db = mongo_db

    # pipline 的类方法，
    # 从settings 取出配置信息,进行初始化操作
    # 返回 这个类的对象
    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            # 可以取到 settings.py中的配置信息
            mongo_url= crawler.settings.get('MONGO_URL'),
            mongo_db = crawler.settings.get('MONGO_DB')
        )

    # 实现open_spider方法
    # 爬虫刚要启动的时候执行的操作
    # 这里进行pymongo相关的初始化操作
    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_url)
        self.db = self.client[self.mongo_db]

    # 复写最重要的 process_item方法
    # 参数固定item, spider
    # 这里实现存到 mongo_db
    def process_item(self, item, spider):
        # item 类型 和 MongoDB 的Collections 名称对应，这样相对较灵活
        name = item.__class__.__name__
        # 存入到MongoDB的 判断
        self.db[name].insert(dict(item))
        return item
        pass

    # 爬虫关闭的时候 关闭连接信息，释放内容
    def close_spider(self, spider):
        self.client.close()
```

settings.py 部分，序号表示优先级顺序，越小表示优先调用

```python
ITEM_PIPELINES = {
   'quotetutorial.pipelines.TextPipeline': 300,
   'quotetutorial.pipelines.MongoPipeline': 400,
}
```

再次运行程序：

```powershell
...
2019-05-23 10:58:34 [scrapy.middleware] INFO: Enabled item pipelines:
['quotetutorial.pipelines.TextPipeline',
 'quotetutorial.pipelines.MongoPipeline']
2019-05-23 10:58:34 [scrapy.core.engine] INFO: Spider opened
...
2019-05-23 10:58:35 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/> (referer: None)
2019-05-23 10:58:35 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'Albert Einstein',
 'tags': ['change', 'deep-thoughts', 'thinking', 'world'],
 'text': '“The world as we have created it is a process of o...'}
2019-05-23 10:58:35 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/>
{'author': 'J.K. Rowling',
 'tags': ['abilities', 'choices'],
 'text': '“It is our choices, Harry, that show what we truly...'}
 ...
```

可以看到结果都被 结果 已经被 TextPipeline 处理了
同时MongoDB中也存取了处理后的结果，MongoPipeline已经调用成功

如果 TextPipeline 不启动或在MongoPipeline之前，则 MongoDB 中存储的是原始 text



## Scrapy 命令行详解

### 官方文档

中文0.24： <https://scrapy-chs.readthedocs.io/zh_CN/0.24/index.html>
中文1.0 ： <https://scrapy-chs.readthedocs.io/zh_CN/1.0/index.html>
英文1.6 (the latest version )：https://docs.scrapy.org/en/latest/

命令行在 Command line tool 下



## Scrapy中选择器用法

Scrapy中的主要操作是 对数据进行 提取，在一些 callback 中，主要操作是 对 代码 进行解析，用到最多的是 选择器，分为 xpath、CSS、re 等

选择器实际上是 Scrapy 内置的对象，直接利用这个对象调用方法进行提取

```html
<html>
 <head>
  <base href='http://example.com/' />
  <title>Example website</title>
 </head>
 <body>
  <div id='images'>
   <a href='image1.html'>Name: My image 1 <br /><img src='image1_thumb.jpg' /></a>
   <a href='image2.html'>Name: My image 2 <br /><img src='image2_thumb.jpg' /></a>
   <a href='image3.html'>Name: My image 3 <br /><img src='image3_thumb.jpg' /></a>
   <a href='image4.html'>Name: My image 4 <br /><img src='image4_thumb.jpg' /></a>
   <a href='image5.html'>Name: My image 5 <br /><img src='image5_thumb.jpg' /></a>
  </div>
 </body>
</html>
```

提取标题信息：

```python
xpath: '//title/text()'
css: 'title::text'
```

```
http://example.com/
```

查找 id 为image 的 div

```javascript
xpath: '//div[@id="images"]'
css: '#images'
```

进一步提取img的src属性

```python
css: 'img::attr(src)'
```

查找所欲 a 标签的 href属性

```python
xpath: '//a/@href'
css: 'a::attr(href)'
```

获取 a 标签的文本内容

```python
xpath: '//a/text()'
css: 'a::text'
```



查找 属性名称包含 image 的a标签的href值

```python
xpath: '//a[contains(@href, "image")]/@href'
css: 'a[href*=image]::attr(href)'
```

所有属性名称包含 image 的 a 标签中 img 标签中的 src 属性

```python
xpath: '//a[contains(@href, "image")]/img/@src'
css: 'a[href*=image] img::attr(src)'
```



获取a标签中 Name: 后的值

```python
response.css('a::text').re('Name\:(.*)')
# 返回 My image x 的str列表

response.css('a::text').re_first('Name\:(.*)')
# My image 1 
```

> 匹配字符串中特殊内容时，用 re 通过正则表达式提取，很方便



## Spiders 用法

Spiders 完成爬虫逻辑，进行网页数据的解析



## Item Pipeline 用法

item 管道，对item 进一步处理：数据清洗，重复检查，存储到数据库等高级操作

实现方法：定义class，实现如下方法：

### process_item(self, item, spider)

用于处理 item 

+ item是 在 spider 通过 yield 生成的 item 作为参数传递到此，已经被赋值
+ spider 运行的时候 爬虫实例

两种返回结果：

+ 处理后的 item
+ 返回 DropItem('Msg')

### open_spider(self, spider)

在spider开启的时候调用，进行初始化操作

### close_spider(self, spider)

在spider关闭的时候调用，进行结束操作

### from_crawler(cls, crawler)

类方法，用于获取 settings 设置

> 多个pipeline的情况下，根据优先级（0-1000）依次处理，每次处理的数据 是 上次处理后返回的item

一些实例看官方文档，如去重、存储到数据库、形成快照等



## Download Middleware

下载中间件，可以改写请求、 处理异常等

![](https://docs.scrapy.org/en/latest/_images/scrapy_architecture_02.png)

地位：对 request 和 response 进行改写，中间处理

实现方法：定义中间件类，实现方法，在settings启动

###  process_request(request, spider)

需要返回以下几种数据之一：

+ None   对整个框架无影响，会继续把其他的下载中间件 process_request 处理完毕
+ Response  不再调用其他的中间件的request和exception，而是会调用process_response来进行处理
+ Request 将请求重新放回调度队列，循环往复的调度
+ raise IgnoreRequest 被其他的 process_exception捕捉进行处理



###  process_response(request, response, spider)

需要返回以下几种数据之一：

+ Response  继续运行其他中间件的 response 方法
+ Request  不会再调用 response ，而是重新加入到 调度队列
+ raise IgnoreRequest 调用异常处理方法进行处理



###  process_exception(request, exception, spider)

捕捉异常、处理异常

超时异常、连接失败异常等等

+ None  |  不影响，继续执行其他的 process_exception 
+ Response  |  继续执行其他的 process_response 而不会调用 process_exception 
+ Request  |  重新加入调度队列（失败重试，可加代理）



> 有自带的 download middleware，如果不想用，可以设置为None

```python
# scrapy settings --get=DOWNLOADER_MIDDLEWARES_BASE
{"scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware": 100, "scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware": 300, "scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware": 350, "scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware": 400, "scrapy.downloadermiddlewares.useragent.UserAgentMiddleware": 500, "scrapy.downloadermiddlewares.retry.RetryMiddleware": 550, "scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware": 560, "scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware": 580, "scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware": 590, "scrapy.downloadermiddlewares.redirect.RedirectMiddleware": 600, "scrapy.downloadermiddlewares.cookies.CookiesMiddleware": 700, "scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware": 750, "scrapy.downloadermiddlewares.stats.DownloaderStats": 850, "scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware": 900}
```

官方文档还介绍了内置的middle ware 参数和用法，简单介绍，比如 COOKIES_ENABLED相关的，重定向的，meta，retry，参考一下。通过这些中间件 实现 灵活的配置