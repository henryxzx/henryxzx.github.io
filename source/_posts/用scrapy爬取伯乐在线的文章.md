---
title: 用scrapy爬取伯乐在线的文章
date: 2018-10-20 00:38:47
tags:
	- "Python"
	- "Scrapy"
	- "Spider"	
categories:
	- "Python"
---

> 简单记录一下自己在linux环境下使用scrapy爬取伯乐在线并提取结构性数据到数据库中的过程

---

### 项目环境

​	deepin15.7 + python 3.6 + mysql 5.5 + pycharm

---

<!--more-->

### 搭建环境

- 安装python环境
- 安装pip
- 安装mysql
- 安装scrapy，virtualenv

  - pip源推荐使用清华大学镜像源 https://pypi.tuna.tsinghua.edu.cn/simple

---

### 创建scrapy项目

在pycharm中创建一个虚拟环境，安装好环境后，在目录下面创建scrapy项目

```python
scrapy startproject ArticleSpider
```
项目的结构如下

```
├── ArticleSpider
│   ├── items.py
│   ├── middlewares.py
│   ├── pipelines.py
│   ├── __pycache__
│   ├── settings.py
│   ├── spiders
│   │   ├── __init__.py
│   │   ├── jobbole.py
│   │   └── __pycache__
└── scrapy.cfg
```

---

### 分析网站中文章页面结构

先进入伯乐在线网站` http://www.jobbole.com/ `，然后随便点开一篇文章

例如` http://blog.jobbole.com/114297/`

![2.png](https://i.loli.net/2018/10/19/5bc9df9ba3f5f.png)
![11.png](https://i.loli.net/2018/10/19/5bc9e058e13a9.png)

对于某一篇文章，目前只需要获得文章的评论数，收藏数，点赞数（默认为1），分类，标题，正文，文章封面图和文章链接。

在Chrome中按F12查看个元素的标签，例如文章的收藏数

![3.png](https://i.loli.net/2018/10/19/5bc9e1c561821.png)

#### 在scrapy shell中获取网页数据

scrapy提供了一个shell环境方便我们调试，并查看获取的元素是否正确

在venv目录的bin目录下

```python
source activate
```

进入venv环境后输入

```python
scrapy shell http://blog.jobbole.com/114297/
```

进入到shell后用css选择器或xpath选择器选取刚才要获得的元素

```shell
>>> response.css('.bookmark-btn::text').extract()
[' 18 收藏']
```

获得的是一个列表，因为class为bookmark-btn的元素只有一个，所以将上面的extract()改为extract_first('')

```shell
>>> response.css('.bookmark-btn::text').extract_first()
' 18 收藏'
```

得到一个字符串，这时将其中的18用正则表达式提取出来后就可以获取到收藏数了，其他的也是如此

---

### 编写代码

#### 从pycharm中启动项目

先在项目根目录创建一个main.py文件

```python
from scrapy.cmdline import execute

import sys
import os

sys.path.append(os.path.dirname(os.path.abspath(__file__)))

execute(['scrapy', 'crawl', 'jobbole']) 
```

在main文件中只有两行执行的代码，

```python
sys.path.append(os.path.dirname(os.path.abspath(__file__)))
```

是将当前的main文件所在目录，即根目录将入环境中，就可以调用别的py文件的方法和类

```python
execute(['scrapy', 'crawl', 'jobbole']) 
```

在第一行导入了scrapy.cmdline即scrapy命令行工具的execute方法，因为Scrapy是通过 `scrapy` 命令行工具进行控制的。 这里我们称之为 “Scrapy tool” 以用来和子命令进行区分。 对于子命令，我们称为 “command” 或者 “Scrapy commands”。上面代码即相当于执行了`scrapy crawl jobbole`

#### 获取url

在jobbole.py中，其中的parse函数用来获取当前页中的所有文章url和下一页的url

```python
class JobboleSpider(scrapy.Spider):	
	name = 'jobbole'
    allowed_domains = ['blog.jobbole.com']
    start_urls = ['http://blog.jobbole.com/all-posts/']
    '''
    提取当前页所有文章的url 一页20个 并获取下一页的url
    '''
    def parse(self, response):
        res_nodes = response.css('.post.floated-thumb .post-thumb a')
        for post_node in res_nodes:
            image_url = post_node.css('img::attr(src)').extract_first('')
            post_url = post_node.css('::attr(href)').extract_first('')
            yield Request(url=parse.urljoin(response.url, post_url), meta={'front_image_url':image_url}, callback=self.parse_detail)

        next_url = response.css('.next.page-numbers::attr(href)').extract_first('')
        if next_url:
            yield Request(url=parse.urljoin(response.url, next_url), callback=self.parse)

```

#### 通过itemloader装载items

在scrapy中提供`Item`类，`Item`对象是种简单的容器，保存了爬取到得数据。 其提供了类似于词典(dictionary-like)的API以及用于声明可用字段的简单语法。

如果直接用Item类，如果后期要对代码进行维护就会变得很繁琐，使用`ItemLoader`类就可以自己编写函数来处理item中的数据，在jobbole.py中加入Item

```python
        res_image_url = response.meta.get('front_image_url', '')
        item_loader = ArticleItemLoader(item=JobBoleArticleItem(), response=response)
        item_loader.add_css('title', '.entry-header h1::text')
        item_loader.add_value('url', response.url)
        item_loader.add_value('url_md5', get_md5(response.url))
        item_loader.add_css('create_date', 'p.entry-meta-hide-on-mobile::text')
        item_loader.add_value('front_image_url', [res_image_url])
        item_loader.add_css('comments_nums', 'a[href="#article-comment"] span::text')
        item_loader.add_css('tags', '.entry-meta-hide-on-mobile a::text')
        item_loader.add_css('content', 'div.entry')
        item_loader.add_css('collection_nums', '.bookmark-btn::text')
        item_loader.add_css('praise_nums', '.vote-post-up h10::text')
```

##### Input and Output processors

Item Loader在每个(Item)字段中都包含了一个输入处理器和一个输出处理器｡ 输入处理器收到数据时立刻提取数据 (通过 [`add_xpath()`](https://scrapy-chs.readthedocs.io/zh_CN/1.0/topics/loaders.html#scrapy.loader.ItemLoader.add_xpath), [`add_css()`](https://scrapy-chs.readthedocs.io/zh_CN/1.0/topics/loaders.html#scrapy.loader.ItemLoader.add_css) 或者 [`add_value()`](https://scrapy-chs.readthedocs.io/zh_CN/1.0/topics/loaders.html#scrapy.loader.ItemLoader.add_value) 方法) 之后输入处理器的结果被收集起来并且保存在ItemLoader内. 收集到所有的数据后, 调用 [`ItemLoader.load_item()`](https://scrapy-chs.readthedocs.io/zh_CN/1.0/topics/loaders.html#scrapy.loader.ItemLoader.load_item) 方法来填充,并得到填充后的 [`Item`](https://scrapy-chs.readthedocs.io/zh_CN/1.0/topics/items.html#scrapy.item.Item) 对象. 这是当输出处理器被和之前收集到的数据(和用输入处理器处理的)被调用.输出处理器的结果是被分配到Item的最终值｡

之后在Item.py中加入

```python
class JobBoleArticleItem(scrapy.Item):
	title = scrapy.Field()
    create_date = scrapy.Field(
        input_processor=MapCompose(date_convert)
    )
    url = scrapy.Field()
    url_md5 = scrapy.Field()
    front_image_url = scrapy.Field(
        output_processor=MapCompose(return_value)
    )
    front_image_path = scrapy.Field()
    praise_nums = scrapy.Field(
        input_processor=MapCompose(get_nums)
    )
    comments_nums = scrapy.Field(
        input_processor=MapCompose(get_nums)
    )
    tags = scrapy.Field(
        output_processor=Join(',')
    )
    content = scrapy.Field()
    collection_nums = scrapy.Field(
        input_processor=MapCompose(get_nums)
    )
```

然后在settings.py中，将robots.txt规则设置为`False`

```python
ROBOTSTXT_OBEY = False
```

这时爬虫项目大致完成，接下来配置数据库和pipelines

---

#### 写入数据库

首先需要安装python的mysql-dev，在这里记录一下我遇到的坑，在linux下直接`pip install mysql-python`会出现错误，这时需要在外面打开命令行，执行`sudo apt-get install libmysqlclient-dev`后就可以安装了。

> 写入数据库有同步和异步的方法，这里只记录异步方法，因为同步方法出错让我卡了两个小时，爷吐了

```python
class MysqlTwistedPipline(object):
    def __init__(self, dbpool):
        self.dbpool = dbpool

    @classmethod
    def from_settings(cls, settings):
        dbparms = dict(
            host = settings["MYSQL_HOST"],
            db = settings["MYSQL_DBNAME"],
            user = settings["MYSQL_USER"],
            passwd = settings["MYSQL_PASSWORD"],
            charset='utf8',
            cursorclass=MySQLdb.cursors.DictCursor,
            use_unicode=True,
        )
        dbpool = adbapi.ConnectionPool("MySQLdb", **dbparms)

        return cls(dbpool)

    def process_item(self, item, spider):
        # 使用twisted将mysql插入变成异步执行
        query = self.dbpool.runInteraction(self.do_insert, item)
        query.addErrback(self.handle_error, item, spider) # 处理异常

    def handle_error(self, failure, item, spider):
        # 处理异步插入的异常
        print(failure)

    def do_insert(self, cursor, item):
        # 执行具体的插入
        # 根据不同的item 构建不同的sql语句并插入到mysql中
        insert_sql, params = item.get_insert_sql()
        print(insert_sql, params)
        cursor.execute(insert_sql, params)
```

在pipelines.py中如上配置，然后在settings.py的最后面加上数据库帐号密码之类的信息，再在items.py中编写get_insert_sql函数就可以将数据写入到数据库中了，而且不同的爬虫项目还可以复用这个结构，只需要改写get_insert_sql中的sql语句就可以了。

---

#### 将图片和爬取的数据导出为Json文件

懒得写了，放在github上。