---
title: Scrapy学习笔记（一）- 快速入门
date: 2017-06-24 10:41:04
tags: [Python,爬虫]
---

# 1 概览

## 1.1 安装
`pip install scrapy=1.4.0`

## 1.2 测试Demo

`vim quotes_spider.py`

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/tag/humor/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.xpath('span/small/text()').extract_first(),
            }

        next_page = response.css('li.next a::attr("href")').extract_first()
        if next_page is not None:
            yield response.follow(next_page, self.parse)

```

`scrapy runspider quotes_spider.py -o quotes.json`

<!--more-->

查看输出结果


```
2017-06-23 23:23:58 [scrapy.core.engine] INFO: Spider opened
2017-06-23 23:23:58 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2017-06-23 23:23:58 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
2017-06-23 23:23:59 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/tag/humor/> (referer: None)
2017-06-23 23:23:59 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/>
{'text': '“The person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.”', 'author': 'Jane Austen'}
2017-06-23 23:23:59 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/>
{'text': '“A day without sunshine is like, you know, night.”', 'author': 'Steve Martin'}
2017-06-23 23:23:59 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/>
{'text': '“Anyone who thinks sitting in church can make you a Christian must also think that sitting in a garage can make you a car.”', 'author': 'Garrison Keillor'}
2017-06-23 23:23:59 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/>
{'text': '“Beauty is in the eye of the beholder and it may be necessary from time to time to give a stupid or misinformed beholder a black eye.”', 'author': 'Jim Henson'}
2017-06-23 23:23:59 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/>
{'text': "“All you need is love. But a little chocolate now and then doesn't hurt.”", 'author': 'Charles M. Schulz'}
2017-06-23 23:23:59 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/>
{'text': "“Remember, we're madly in love, so it's all right to kiss me anytime you feel like it.”", 'author': 'Suzanne Collins'}
2017-06-23 23:23:59 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/>
{'text': '“Some people never go crazy. What truly horrible lives they must lead.”', 'author': 'Charles Bukowski'}
2017-06-23 23:23:59 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/>
{'text': '“The trouble with having an open mind, of course, is that people will insist on coming along and trying to put things in it.”', 'author': 'Terry Pratchett'}
2017-06-23 23:23:59 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/>
{'text': '“Think left and think right and think low and think high. Oh, the thinks you can think up if only you try!”', 'author': 'Dr. Seuss'}
2017-06-23 23:23:59 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/>
{'text': '“The reason I talk to myself is because I’m the only one whose answers I accept.”', 'author': 'George Carlin'}
2017-06-23 23:24:00 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/tag/humor/page/2/> (referer: http://quotes.toscrape.com/tag/humor/)
2017-06-23 23:24:00 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/page/2/>
{'text': '“I am free of all prejudice. I hate everyone equally. ”', 'author': 'W.C. Fields'}
2017-06-23 23:24:00 [scrapy.core.scraper] DEBUG: Scraped from <200 http://quotes.toscrape.com/tag/humor/page/2/>
{'text': "“A lady's imagination is very rapid; it jumps from admiration to love, from love to matrimony in a moment.”", 'author': 'Jane Austen'}

```


# 2 入门

## 2.1 创建项目

`scrapy startproject tutorial`
结构如下

```
tutorial/
    scrapy.cfg            # deploy configuration file

    tutorial/             # project's Python module, you'll import your code from here
        __init__.py

        items.py          # project items definition file

        pipelines.py      # project pipelines file

        settings.py       # project settings file

        spiders/          # a directory where you'll later put your spiders
            __init__.py
```

## 2.2 第一个爬虫

### 2.3.1 定义爬虫
爬虫类必须是scrapy.Spider的子类，并且定义初始爬取页面
在tutorial/spiders下创建爬虫类


```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"

    def start_requests(self):
        urls = [
            'http://quotes.toscrape.com/page/1/',
            'http://quotes.toscrape.com/page/2/',
        ]
        for url in urls:
            yield scrapy.Request(url=url, callback=self.parse)

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = 'quotes-%s.html' % page
        with open(filename, 'wb') as f:
            f.write(response.body)
        self.log('Saved file %s' % filename)

```

- name: 爬虫的名称，在项目中必须是唯一的。
- start_requests(): 返回可遍历的Requests，爬虫将从这里进行爬取。
- parse(): 对请求的结果数据进行解析。




### 2.3.2 运行爬虫

`scrapy crawl quotes`

结果


```
2017-06-24 00:02:16 [scrapy.utils.log] INFO: Scrapy 1.4.0 started (bot: tutorial)
2017-06-24 00:02:16 [scrapy.utils.log] INFO: Overridden settings: {'NEWSPIDER_MODULE': 'tutorial.spiders', 'SPIDER_MODULES': ['tutorial.spiders'], 'ROBOTSTXT_OBEY': True, 'BOT_NAME': 'tutorial'}
2017-06-24 00:02:16 [scrapy.middleware] INFO: Enabled extensions:
['scrapy.extensions.memusage.MemoryUsage',
 'scrapy.extensions.logstats.LogStats',
 'scrapy.extensions.telnet.TelnetConsole',
 'scrapy.extensions.corestats.CoreStats']
2017-06-24 00:02:16 [scrapy.middleware] INFO: Enabled downloader middlewares:
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
2017-06-24 00:02:16 [scrapy.middleware] INFO: Enabled spider middlewares:
['scrapy.spidermiddlewares.httperror.HttpErrorMiddleware',
 'scrapy.spidermiddlewares.offsite.OffsiteMiddleware',
 'scrapy.spidermiddlewares.referer.RefererMiddleware',
 'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware',
 'scrapy.spidermiddlewares.depth.DepthMiddleware']
2017-06-24 00:02:16 [scrapy.middleware] INFO: Enabled item pipelines:
[]
2017-06-24 00:02:16 [scrapy.core.engine] INFO: Spider opened
2017-06-24 00:02:16 [scrapy.extensions.logstats] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
2017-06-24 00:02:16 [scrapy.extensions.telnet] DEBUG: Telnet console listening on 127.0.0.1:6023
2017-06-24 00:02:17 [scrapy.core.engine] DEBUG: Crawled (404) <GET http://quotes.toscrape.com/robots.txt> (referer: None)
2017-06-24 00:02:17 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/1/> (referer: None)
2017-06-24 00:02:17 [quotes] DEBUG: Saved file quotes-1.html
2017-06-24 00:02:17 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://quotes.toscrape.com/page/2/> (referer: None)
2017-06-24 00:02:17 [quotes] DEBUG: Saved file quotes-2.html
2017-06-24 00:02:17 [scrapy.core.engine] INFO: Closing spider (finished)
2017-06-24 00:02:17 [scrapy.statscollectors] INFO: Dumping Scrapy stats:
{'downloader/request_bytes': 675,
 'downloader/request_count': 3,
 'downloader/request_method_count/GET': 3,
 'downloader/response_bytes': 5976,
 'downloader/response_count': 3,
 'downloader/response_status_count/200': 2,
 'downloader/response_status_count/404': 1,
 'finish_reason': 'finished',
 'finish_time': datetime.datetime(2017, 6, 23, 16, 2, 17, 551927),
 'log_count/DEBUG': 6,
 'log_count/INFO': 7,
 'memusage/max': 51879936,
 'memusage/startup': 51875840,
 'response_received_count': 3,
 'scheduler/dequeued': 2,
 'scheduler/dequeued/memory': 2,
 'scheduler/enqueued': 2,
 'scheduler/enqueued/memory': 2,
 'start_time': datetime.datetime(2017, 6, 23, 16, 2, 16, 322185)}
2017-06-24 00:02:17 [scrapy.core.engine] INFO: Spider closed (finished)

```

现在当前目录下应该出现了两个文件 `quotes-1.html` `quotes-2.html`

### 2.3.3 抽取数据

使用 `scrapy shell 'http://quotes.toscrape.com/page/1/'` 通过shell来学习如何抽取我们想要的数据。

**CSS选择器**

```
>>> response.css('title')
[<Selector xpath=u'descendant-or-self::title' data=u'<title>Quotes to Scrape</title>'>]
>>> response.css('title').extract()
[u'<title>Quotes to Scrape</title>']
>>> response.css('title::text').extract()
[u'Quotes to Scrape']
>>> response.css('title::text').extract_first()
u'Quotes to Scrape'
>>> response.css('title::text')[0].extract()
u'Quotes to Scrape'
>>> response.css('title::text').extract()[0]
u'Quotes to Scrape'
>>> view(response)
True
```

使用extract_first抽取数据可以避免角标越界，抽取不到会返回None。

**XPath**


```
>>> response.xpath("//title")
[<Selector xpath='//title' data=u'<title>Quotes to Scrape</title>'>]
>>> response.xpath("//title/text()")
[<Selector xpath='//title/text()' data=u'Quotes to Scrape'>]
>>> response.xpath("//title/text()").extract_first()
u'Quotes to Scrape'
```

XPath表达式非常强大，并且它是Scrapy Selectors的基础。
起始CSS选择器在底层也是被转换为XPath去执行的。

[Selectors](https://docs.scrapy.org/en/latest/topics/selectors.html#topics-selectors)


**举例 抽取title和author**

- HTML原文

```html
<div class="quote">
    <span class="text">“The world as we have created it is a process of our
    thinking. It cannot be changed without changing our thinking.”</span>
    <span>
        by <small class="author">Albert Einstein</small>
        <a href="/author/Albert-Einstein">(about)</a>
    </span>
    <div class="tags">
        Tags:
        <a class="tag" href="/tag/change/page/1/">change</a>
        <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
        <a class="tag" href="/tag/thinking/page/1/">thinking</a>
        <a class="tag" href="/tag/world/page/1/">world</a>
    </div>
</div>
```

- 进入抽取shell`scrapy shell 'http://quotes.toscrape.com'`
- 获取quote `quote = response.css('div.quote')[0]`
- 抽取title `title = quote.css('span.text::text').extract_first()`
- 抽取author `author = quote.css('small.author::text').extract_first()`
- 抽取tags `tags = quote.css('div.tags a.tag::text').extract()`

我们学会了如何抽取指定信息后，将抽取结果存储到python的字段中


```python
>>> for quote in response.css("div.quote"):
...     text = quote.css("span.text::text").extract_first()
...     author = quote.css("small.author::text").extract_first()
...     tags = quote.css("div.tags a.tag::text").extract()
...     print(dict(text=text, author=author, tags=tags))
{'tags': ['change', 'deep-thoughts', 'thinking', 'world'], 'author': 'Albert Einstein', 'text': '“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”'}
{'tags': ['abilities', 'choices'], 'author': 'J.K. Rowling', 'text': '“It is our choices, Harry, that show what we truly are, far more than our abilities.”'}
    ... a few more of these, omitted for brevity
```

### 2.3.4 存储数据

`scrapy crawl quotes -o quotes.json`
注意 重复执行爬虫 输出的文件不会覆盖 而是追加，从而导致了错误的json格式
`scrapy crawl quotes -o quotes.jl`


### 2.3.5 获取新的连接

**HTML**


```
<ul class="pager">
    <li class="next">
        <a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
    </li>
</ul>
```

我们虽然可以通过 `response.css('li.next a').extract_first()` 拿到
```html
<a href="/page/2/">Next <span aria-hidden="true">→</span></a>
```

但是我们需要的仅仅是href，可是这样操作 `response.css('li.next a::attr(href)').extract_first()`


```
'/page/2/'
```

现在我们的爬虫是这样的


```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        'http://quotes.toscrape.com/page/1/',
    ]

    def parse(self, response):
        for quote in response.css('div.quote'):
            yield {
                'text': quote.css('span.text::text').extract_first(),
                'author': quote.css('small.author::text').extract_first(),
                'tags': quote.css('div.tags a.tag::text').extract(),
            }

        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            next_page = response.urljoin(next_page)
            yield scrapy.Request(next_page, callback=self.parse)
```

加入一些简化操作


follow支持相对路径
```python
 next_page = response.css('li.next a::attr(href)').extract_first()
 if next_page is not None:
    yield response.follow(next_page, callback=self.parse)
```

follow可以自动识别href


```python
for a in response.css('li.next a'):
    yield response.follow(a, callback=self.parse)
```


# 3 Demo

帮同学爬一个健康网站的数据，网站也没有反爬手段，结构又很简单，所以现学现卖用scrapy（代码写的比较随意，仅供参考），
其实向这种简单的爬虫，直接用requests更省事儿。
project最终结构
```
└── boohee
    ├── boohee
    │   ├── __init__.py
    │   ├── items.py
    │   ├── middlewares.py
    │   ├── pipelines.py
    │   ├── settings.py
    │   └── spiders
    │       ├── __init__.py
    │       ├── food_detail.py
    │       └── food_name_spider.py
    │       
    ├── food_detail.json
    ├── food_name.json
    └── scrapy.cfg


```

核心代码

**food_name.py**


```python
import scrapy
import time

from flask import json

file_res = json.loads(open('/Users/lixiwei-mac/Documents/IdeaProjects/rhinotech_spider/boohee_spider/boohee/food_name.json').read())
cookie = 'gwdang_brwext_share=0; from_device=default; gwdang_brwext_more_force=0; history=1479334-223%2C2365144-3%2C1462046326-3%2C12596075384-3%2C2365148-3%2C2365158-3%2C4830462-3%2C3498623-3%2C2504829-3%2C11905178-3; gwdang_permanent_id=9b26f59a3e854641cafe23c267ea728b; gwdang_brwext_is_open=0; gwdang_brwext_first=1; gwdang_brwext_position=0; gwdang_brwext_close_update=0; gwdang_brwext_close_update_hour=0; gwdang_brwext_close_install=0; gwdang_brwext_style=top; gwdang_brwext_notice=0; gwdang_brwext_fold=0; gwdang_brwext_show_tip=1; gwdang_brwext_imageAd=1; gwdang_brwext_show_popup=1; gwdang_brwext_show_ljfqrcode=1; gwdang_brwext_hide_shoptip=0; gwdang_brwext_apptg_close=0; gwdang_brwext_show_lowpri=1; gwdang_brwext_show_guessfavor=1; gwdang_brwext_show_lowpri_right=1; gwdang_brwext_show_guessfavor_right=1; gwdang_brwext_show_vips=1; gwdang_brwext_show_wishlist=1; gwdang_brwext_show_guess=1; gwdang_brwext_show_promo=1; gwdang_search_way=0'
headers = {'Cookie':cookie}

class FoodNameSpider(scrapy.Spider):
    name = 'food_name'
    base_url = 'http://www.boohee.com/food/group/%s'
    start_urls = [base_url % x for x in range(41)] * 10

    def parse(self, response):
        for food_ref in response.css('ul.food-list li h4 a::attr(href)').extract():
            group_id = response.url.split('/')[5].split("?")[0]
            food_name = response.xpath('//a[@href="%s"]/@title' % food_ref).extract_first()
            res = {'group_id': group_id, 'href': food_ref, 'food_name': food_name}
            if res not in file_res:
                yield res
        for href in response.css('a.next_page'):
            time.sleep(0.1)
            yield response.follow(href, self.parse, headers=headers)

```


**food_detail.py**


```python
import scrapy
import time

from flask import json

food_names = json.loads(open('food_name.json').read())


class FoodDetailSpider(scrapy.Spider):
    name = 'food_detail'
    start_urls = ['http://www.boohee.com' + food_name_obj['href'] for food_name_obj in food_names]

    def parse(self, response):
        calories_value = response.css('ul.basic-infor').xpath('//li/span/span/text()').extract_first()
        food_name = response.css('h2.crumb::text').extract()[-1].split("/")[-1].strip()
        evaluation = ''.join(response.css('div.content p::text').extract()).strip()
        food_group_name = response.xpath('//h2/a/text()').extract()[1]

        # nutrition infomation
        nutrition_info = []
        for nutr_dl in response.css('div.nutr-tag dl')[1:]:
            for nutr_dd in nutr_dl.css('dd'):
                nutr_key = nutr_dd.css('span.dt::text').extract_first()
                if nutr_dd.css('span.stress'):
                    nutr_value = nutr_dd.css('span.stress::text').extract_first()
                else:
                    nutr_value = nutr_dd.css('span.dd::text').extract_first()
                # print nutr_key, nutr_value
                nutrition_info.append({nutr_key: nutr_value})

        # widget-unit

        widget_unit_info = []
        for w_u_tr in response.css('div.widget-unit tbody tr'):
            if w_u_tr.css('td a'):
                w_u_name = w_u_tr.css('td a::text').extract()[0]
                w_u_value = w_u_tr.css('td a::text').extract()[1]
            elif w_u_tr.css('td span'):
                w_u_name = w_u_tr.css('td span::text').extract()[0]
                w_u_value = w_u_tr.css('td span::text').extract()[1]
            else:
                w_u_name = w_u_tr.css('td::text').extract()[0]
                w_u_value = w_u_tr.css('td::text').extract()[1]
            # print w_u_name, w_u_value
            widget_unit_info.append({w_u_name: w_u_value})
        time.sleep(0.5)
        yield {'food_group_name': food_group_name, 'food_name': food_name,'nutrition_info': nutrition_info, 'widget_unit_info': widget_unit_info, 'evaluation': evaluation}

```

**必须掌握 调试解析html的方法**
`scrapy shell ' scrapy shell 'http://www.boohee.com/food/group/1''`

**必须掌握导出结果的方法**
输出到控制台 `scrapy crawl author`
输出到文件 `scrapy crawl quotes -o quotes.json`



