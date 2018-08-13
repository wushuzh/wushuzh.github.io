+++
date = "2017-11-20T21:02:10+08:00"
title = "Bokeh 服务器"
showonlyimage = false
image = "/img/blog/try-bokeh-server/wired-money.png"
draft = false
weight = 110
tags = ["bokeh", "visualization"]
+++

用 Bokeh 服务器对可视化进行控制
<!--more-->

## Bokeh 服务器应用

### 应用验证

Bokeh 架构是用 python 生成 Model 对象，然后 JSON 序列化后发送到 JS 客户端。这样的设计使得我们可以
- 将用户界面的事件通过 python 的一系列操作完成;
- 自动将服务器端的更新发送到客户端;
- 使用间歇、超时或是异步回调驱动流式更新;

这里我们先生成一个显示框架(无数据)，然后启动 bokeh 服务器验证环境。

{{< highlight python >}}
from bokeh.plotting import figure, curdoc
from bokeh.models import Range1d


p = figure(title="STOCKSTREAMER v0.0",
           plot_width=1000, plot_height=680,
           x_range=Range1d(0, 1), y_range=Range1d(-50, 1200),
           toolbar_location='below', toolbar_sticky=False)

curdoc().add_root(p)
{{< /highlight >}}

{{< highlight console >}}
(stockstreamer-HdJ_HpJT) $ bokeh serve stockstreamer.py 
ts Starting Bokeh server version 0.12.10 (running on Tornado
ts Bokeh app running at: http://localhost:5006/stockstreamer
ts Starting Bokeh server with process id: 16387
{{< /highlight >}}

### 读取已有数据展示

从数据库中获取静态和动态数据并清洗转化数据为图像元素

数据转化主要借助强大的 pandas 库

{{< highlight pycon >}}
>>> import psycopg2
>>> dburl = "postgres://postgres:postgres@192.168.99.100:5432/stocks"
>>> conn = psycopg2.conn(dburl)

>>> import pandas as pd
>>> image_urls = pd.read_sql("select * from stock_image_urls", conn)
>>> image_urls
  stock_name                                          image_url
0      GOOGL  https://storage.googleapis.com/iex/api/logos/G...
1         FB  https://storage.googleapis.com/iex/api/logos/F...
2       AAPL  https://storage.googleapis.com/iex/api/logos/A...
3       AMZN  https://storage.googleapis.com/iex/api/logos/A...
4       BABA  https://storage.googleapis.com/iex/api/logos/B...

>>> image_urls.set_index('stock_name', inplace=True)
>>> image_urls
                                                    image_url
stock_name
GOOGL       https://storage.googleapis.com/iex/api/logos/G...
FB          https://storage.googleapis.com/iex/api/logos/F...
AAPL        https://storage.googleapis.com/iex/api/logos/A...
AMZN        https://storage.googleapis.com/iex/api/logos/A...
BABA        https://storage.googleapis.com/iex/api/logos/B...

>>> df = pd.read_sql("SELECT * FROM stock_prices", conn)
>>> df.groupby('stock_name').groups
{
'AAPL': Int64Index([4,8,11,...619], dtype='int64', length=124),
'AMZN': Int64Index([0,5,12,...615], dtype='int64', length=124),
'BABA': Int64Index([2,9,13,...617], dtype='int64', length=124),
'FB': Int64Index([1,7,14,...618], dtype='int64', length=124),
'GOOGL': Int64Index([3,6,10,...616], dtype='int64', length=124)
}

{{< /highlight >}}

股价数据的清洗稍微复杂一点，主要思路是只保留有变化的采样数据，清洗掉一直持续不变的记录: 比如那些休市期间的采集的记录:

保留的点存储于 important_indices 

- 将相同股票的数据聚集 (df.groupby)在一起，并选出那些和上一条价格不同的记录 (df.diff)
- 基于这些记录，再加入这些记录的上一个点 (df.shift(-1))

{{< highlight python>}}
df['important_indices'] = 
  df.groupby('stock_name')['price'].diff() != 0
df['important_indices'] = 
  (df['important_indices'] != 0) \
  | (df.groupby('stock_name')['important_indices'].shift(-1))

df = df.loc[df.important_indices, ]
{{< /highlight >}}

这样清洗后得到的 Dataframe 其记录数量至少减少 2/3 ，这种从数据本身出发的优化使得不需要再针对[不同市场开闭市时间](https://en.wikipedia.org/wiki/List_of_stock_exchange_trading_hours) 以及时区、节假日等做任何特殊处理。

> - TODO: 数据表是否设定一个 partition 规则，然后定期删除或是压减记录
> - TODO: 如何将 Presgresql 的容器的数据保存在外部，以便以后可以随时挂载上进行测试

显示每个股票的方法是用最大最小值和时间周期画出一个长方形，对于各个时间点的股价结合折线和圆圈标注:

- ColumnDataSource 是 bokeh.models.sources 最重要的从列名到序列/数组的映射，可从 dict 或 df 转化而来
- 线、点、多边形、椭圆、各种弧都属于 Glyphs ，各自端点/顶点都可借助 ColumnDataSource 指定
- 时间横轴的起始点适当放大——放置各股的图标

### 用最新数据更新图表

更新图表更新相应的 glyph 的数据源即可:

{{< highlight python>}}
def update_figure():
  xs, ys, max_ys, unique_names = get_data()
  for i, (x, y, max_y, name) in enumerate(zip(xs, ys, max_ys, unique_names)):
    lines[i].data_source.data.update(
      x=x,
      y=y,
      stock_name=[name_mapper[name]] * len(x),
      timestamp=[a.strftime('%Y-%m-%d %H-%M-%S') for a in x])


update_figure()
curdoc().add_periodic_callback(update_figure, 5000)

{{< /highlight >}}

因为中美时区相差较大，如果你在美股闭市时开发测试脚本，就无法获得当下动态的股价，因此增加一个 pestgresql function (存储过程) 用来模拟向数据库中插入几条数据，然后测试 bokeh 服务器的动态加载数据的功能:

- 插入数据的时间戳要保持从老到新，不然画图时会出现横向折返(尤其是 data_fetcher.py 同时运行时)
- 因为使用了容器数据库，所以其内部的时区和外部不同
- 因为 bokeh 服务器会不断通过数据库链接读取数据，不要妄图做表的重建、改名等操作
- Postgresql 的存储过程不仅可用 plpgsql，也可以用 python

{{< highlight plpgsql >}}
CREATE OR REPLACE FUNCTION add_records() RETURNS VOID AS $$
  BEGIN
    SET TIMEZONE='Asia/Shanghai';
    INSERT INTO stock_prices
      SELECT 
        stock_name,
        max(price) + round(random()*100+1) as price,
        max(time) + '1 second'::interval
      FROM stock_prices group by stock_name;
  END;
$$ LANGUAGE plpgsql;

/*
DO $$ BEGIN
    PERFORM add_records();
END $$;

UPDATE stock_highlow AS hl
SET high_val52wk = updhl.max
FROM ( SELECT stock_name, MAX(price), MIN(price) FROM stock_prices GROUP BY stock_name) AS updhl
WHERE updhl.stock_name = hl.stock_name and updhl.max > hl.high_val52wk

UPDATE stock_highlow AS hl
SET low_val52wk = updhl.min
FROM ( SELECT stock_name, MAX(price), MIN(price) FROM stock_prices GROUP BY stock_name) AS updhl
WHERE updhl.stock_name = hl.stock_name and updhl.min < hl.low_val52wk
*/

{{< /highlight >}}

至此，所有关键部分都搞定了，下面就是针对图表显示的各种微调了。

### 可视化的微调

- 工具栏和横纵轴标识
- 将近一年最大最小价格以长方形形式呈现
- 添加图例和股票图片


### 容器封装

这部分是跟当下容器的风，将上述的各个组件封装到容器中，通过 docker-compose 定义服务整体启动执行。

按照基本功能的划分，将抓取程序、展示程序和数据库作为 3 个容器，前两个共享一个镜像， 后一个数据库镜像则加入初始化库表及存储过程的sql脚本。

注意: 以前数据库链接串是硬编码在源程序中，现在则需要通过环境变量读入，并将 host 替换为服务容器名

{{< highlight Dockerfile >}}
FROM python:3

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

ADD ./requirements.txt /usr/src/app/requirements.txt

RUN pip install -r requirements.txt

ADD ./project/data_fetcher.py /usr/src/app
ADD ./project/stockstreamer.py /usr/src/app

CMD python data_fetcher.py
{{< /highlight >}}

{{< highlight console >}}
docker build -t bokeh-service --build-arg https_proxy=ip:port .
{{< /highlight >}}

除了引入 wait-for-it 等待数据库，chart-service 也应该等待 fetch-service 插入数据后再启动比较合适，通过在程序内部增加检测等待实现。

{{< highlight yaml >}}
version: "3.2"

services:

    stock-db:
        container_name: stock-db
        # build: ./project/db
        image: stock-db
        #ports:
        #    - 5435:5432 # expose ports - HOST:CONTAINER
        environment:
            - POSTGRES_USER=postgres
            - POSTGRES_PASSWORD=postgres
        healthcheck:
            test: "pg_isready -U postgres"

    fetch-service:
        container_name: fetch-service
        image: bokeh-service
        environment:
            - DATABASE_URL=postgres://postgres:postgres@stock-db:5432/stocks
            - HTTPS_PROXY=proxyip:port
        depends_on:
            - stock-db
        command: ["./wait-for-it.sh", "stock-db:5432", "-t", "90", "--", "python", "data_fetcher.py"]
        links:
            - stock-db

    chart-service:
        container_name: chart-service
        image: bokeh-service
        ports:
            - 5006:5006 # expose ports - HOST:CONTAINER
        environment:
            - DATABASE_URL=postgres://postgres:postgres@stock-db:5432/stocks
        depends_on:
            - fetch-service
        command: ["./wait-for-it.sh", "stock-db:5432", "-t", "90", "--", "bokeh", "serve", "stockstreamer.py", "--allow-websocket-origin=192.168.99.100:5006"]
        links:
            - stock-db


{{< /highlight >}}

参考文档

> - Bokeh User Guide [Running a Bokeh Server](https://bokeh.pydata.org/en/latest/docs/user_guide/server.html)
> - Tutorialspoint [PostgreSQL - Functions](https://www.tutorialspoint.com/postgresql/postgresql_functions.htm)

封面图片来自 [Wired Money](https://dribbble.com/shots/993632-Wired-Money) <a href="https://dribbble.com/goranfactory"><i class="fa fa-dribbble" aria-hidden="true"></i> Marco Goran Romano</a>
