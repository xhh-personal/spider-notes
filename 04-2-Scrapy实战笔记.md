# Scrapy实战笔记

## Scrapy抓取知乎用户信息

### 前言

知乎的用户信息量很大，本节演示如何抓取所有知乎用户信息，后续还会涉及 分布式 的概念，该实战也会为后续分布式做个铺垫

### 分析爬取逻辑

选定某一个用户，会有 关注列表 粉丝列表，依照粉丝依次递归找粉丝…… 

通过一个用户蔓延到所有用户，只要不是 零关注、零粉丝即可。

#### 选定起始人

选定一位关注数或粉丝数 多的大V 作为爬取起始点

#### 获取粉丝和关注列表

通过知乎接口获得 该大V的粉丝列表和关注列表

#### 获取列表用户信息

通过知乎接口 获得列表中每位用户的详细信息

#### 获取每位用户粉丝和关注

进一步对列表中的每一个用户，获取他们的粉丝和关注列表，实现递归爬取



粉丝列表 和 用户信息 都有接口，传入include 字段 请求 url 可获得用户信息

### 实战演练

#### 设置中进行修改：

```python
ROBOTSTXT_OBEY = False

DEFAULT_REQUEST_HEADERS = {
   'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.131 Safari/537.36',
   'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
   'Accept-Language': 'zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2',
}
```

#### 测试接口是否可用

> 视频中需要在请求头加入 authorization 字段才可以

##### 关注列表的接口

```url
https://www.zhihu.com/api/v4/members/excited-vczh/followees?include=data[*].answer_count,articles_count,gender,follower_count,is_followed,is_following,badge[?(type=best_answerer)].topics&offset=0&limit=20
```

+ url_token  |  用户唯一标识
+ offset  |  偏移值
+ limit  |  限制
+ include  |  

##### 粉丝列表的接口

```
https://www.zhihu.com/api/v4/members/excited-vczh/followers?include=data[*].answer_count,articles_count,gender,follower_count,is_followed,is_following,badge[?(type=best_answerer)].topics&offset=0&limit=20
```



##### 用户详细信息的接口

```
https://www.zhihu.com/api/v4/members/gui-yan-xie-78?include=allow_message,is_followed,is_following,is_org,is_blocking,employments,answer_count,follower_count,articles_count,gender,badge[?(type=best_answerer)].topics
```

+ url_token
+ include  |  不需要改

![1558697677238](./img/user_query.png)

![1558699857287](./img/user_info1.png)

![1558699972237](./img/user_info2.png)

![1558700021182](./img/user_info3.png)

##### 构造接口





> 注：在5.24测试时候发现，翻页效果的请求出现404错误，发现是next那的接口错误，因此用正则把信息提取再构造url提取

```powershell
2019-05-24 22:06:29 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 2614656,
 'downloader/request_count': 3204,
 'downloader/request_method_count/GET': 3204,
 'downloader/response_bytes': 3385799,
 'downloader/response_count': 3204,
 'downloader/response_status_count/200': 3074,
 'downloader/response_status_count/410': 130,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2019, 5, 24, 14, 6, 29, 261205),
 'httperror/response_ignored_count': 130,
 'httperror/response_ignored_status_count/410': 130,
 'item_scraped_count': 2921,
 'log_count/DEBUG': 9199,
 'log_count/INFO': 140,
 'request_depth_max': 153,
 'response_received_count': 3204,
 'scheduler/dequeued': 3204,
 'scheduler/dequeued/memory': 3204,
 'scheduler/enqueued': 3204,
 'scheduler/enqueued/memory': 3204,
 'start_time': datetime.datetime(2019, 5, 24, 14, 4, 54, 112891)}
2019-05-24 22:06:29 [scrapy.core.engine] INFO: Spider closed (finished)
```

### 思考：还可以提高效率吗？

现在是用一台主机，后续会将如何用多台主机进行爬取，也就是分布式爬取。Redis请求队列



## Scrapy分布式结构搭建抓取知乎

settings加入：

```python
# Enables scheduling storing requests queue in redis.
SCHEDULER = "scrapy_redis.scheduler.Scheduler"

# Ensure all spiders share same duplicates filter through redis.
DUPEFILTER_CLASS = "scrapy_redis.dupefilter.RFPDupeFilter"

REDIS_URL = 'redis://73654:zyh19970720@127.0.0.1:6379'
```

运行后：

```python
{'downloader/request_bytes': 2614656,
 'downloader/request_count': 3204,
 'downloader/request_method_count/GET': 3204,
 'downloader/response_bytes': 3385407,
 'downloader/response_count': 3204,
 'downloader/response_status_count/200': 3074,
 'downloader/response_status_count/410': 130,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2019, 5, 25, 8, 55, 15, 266107),
 'httperror/response_ignored_count': 130,
 'httperror/response_ignored_status_count/410': 130,
 'item_scraped_count': 2921,
 'log_count/DEBUG': 9199,
 'log_count/INFO': 141,
 'request_depth_max': 153,
 'response_received_count': 3204,
 'scheduler/dequeued/redis': 3204,
 'scheduler/enqueued/redis': 3204,
 'start_time': datetime.datetime(2019, 5, 25, 8, 52, 49, 517544)}

```

默认上 redis 也存储了一份item，这是因为 'scrapy_redis.pipelines.RedisPipeline': 301，影响了爬取效果

去掉后就实现了分布式存储，每个从机爬取到的数据 存到本地，而不再向 redis 主机传输了，提升了爬取效率

如果Redis有密码，就用 REDIS_URL  配置



## Scrapy+Cookies池抓取新浪微博

### 前言

站点是移动端的微博 weibo.cn，相对于PC端结构更加简单，抓取更加容易

### 站点分析

#### 翻页方式有两种：

+ 直接点击下一页 |  GET请求

```
https://weibo.cn/search/mblog?hideSearchFrame=&keyword=000001&page=2
```

+ 指定页码跳页 | POST请求

```
https://weibo.cn/search/mblog?hideSearchFrame=&keyword=000001

form_data:
mp	100
page	3
```

对比：get 请求没有 mp 参数，不能控制最大页码，post请求可以控制mp为200，比get更多

#### 内容分析

昵称、内容、赞、转发、评论等链接
xx 转发了 xx  原文转发、原文评论

我们要抓取微博的正文内容，如果遇到原创微博就把内容抓下来，如果遇到是转发的微博把转发的原文抓下来
微博呈现并不全，有些微博有“全文”链接，因为原文太长……点击后才能查看详情
如果点击评论，也能跳到原文详情
因此，为了便于逻辑更加清晰，首先请求索引页，然后再通过"评论"请求每一条微博的详情页

### 实战

+ settings改 配置，其中 请求头 可以复制过来用正则表达式批量替换

  > Ctrl + R 调出替换框，分别输入
  >
  > ```python
  > (.*?):(.*)
  > '$1':'$2',
  > ```

  然后把不用的header删掉，比如Referer，Content-Length和Cookies
  其中Cookies可以另外的Middleware里边加

实际测试的时候，即使传入Cookies也没办法访问，不知道为何……

+ parse_index中

  主要是用 xpath 和 正则抓取一些字段

![1558756120455](./img/weibo_field.png)

```python
# 用eval 动态获取 变量名
weibo_item = WeiboItem()
for field in weibo_item.fields:
    try:
        weibo_item[field] = eval(field)
    except NameError:
        # deal Error
        pass
yield weibo_item
```

![1558756059604](./img/weibo_item.png)

+ 对数据进行清洗 (视频1h处开始)

  一些数据在得到的时候，有时效性，比如 ”今天“，需要对这些字段进行改写成 通用时间

  结果发现：

  + 1小时之内，显示xx 分钟前
  + 1小时之外并且在今天，显示 "今天 hh:mm"
  + 其他的，"dddd-mm-dd hh:mm"

+ 存储MongoDB

![1558757156607](./img/weibo_mongo.png)

+ 收尾工作

  微博反爬虫很厉害，可能会跳转到封号的页面……，在中间件中加入 process_response 处理

![1558757077793](./img/weibo_anti-anti.png)



> 通过搜索结果 一部分 股票的搜索结果内容

## Scrapy+Tushare爬取微博股票数据

### Tushare

http://tushare.org/

财经数据接口，获得股票关键字，通过代号获取爬取结果

