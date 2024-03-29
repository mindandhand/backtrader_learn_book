Cerebro类是backtrader的基石，把他翻译为执行引擎，是一切后续工作的基础，功能包括：

    - 收集输入如Data Feeds、Stratgegies、Observers、Observers和Writers, 把这一切整合到一起运行；

    - 执行回测/实时数据接收/交易

    - 返回结果

    - 提供绘图功能


```python
import backtrader as bt
import backtrader.indicators as btind
import backtrader.feeds as btfeeds
import backtrader.analyzers as btanalyzers  
import akshare as ak
import pandas as pd
import datetime
```

#### 收集输入
##### 从创建引擎开始
```
cerebro = bt.Cerebro(**kwargs)
```
\*\*kwargs是一些控制执行过程的可选参数

##### 添加 Data feeds
最常见的语句就是是cerebro.adddata(data)，其中data是已经实例化的数据源,如读取Yahoo CSV数据或者使用akshare 获取行情数据


```python
# 使用akshare 读取数据
data1 = bt.feeds.YahooFinanceCSVData(
# 指定数据文件
dataname='./GSPC.csv',
# 已经从前到后按日期从历史到当前进行排序，无需反转
reverse=False,
# 指定开始加载的数据日期，包含
fromdate=datetime.datetime(2020, 8, 3),
# 指定结束加载的数据日期，不包含
todate=datetime.datetime(2020, 8, 21)
)

cerebro1 = bt.Cerebro()
# 添加数据到引擎中
cerebro1.adddata(data1)


# 使用akshare 获取行情数据
# 指定日期的前复权数据，只取前7列
sz000001_df = ak.stock_zh_a_hist(symbol="000001", 
    period="daily", 
    start_date="20231201", 
    end_date="20231231", 
    adjust="qfq").iloc[:, :7]
print("akshare data len:",len(sz000001_df))
print(sz000001_df[-2:])
# 处理字段命名，以符合 Backtrader 的要求
sz000001_df.columns = [
    'date',
    'open',
    'close',
    'high',
    'low',
    'volume',
    'amt',
]
# 把 date 作为日期索引，以符合 Backtrader 的要求
sz000001_df.index = pd.to_datetime(sz000001_df['date'])

data2 = bt.feeds.PandasData(dataname=sz000001_df)  # 加载数据
cerebro2 = bt.Cerebro()
# 添加数据到引擎中
cerebro2.adddata(data2)
```

    akshare data len: 21
                日期    开盘    收盘    最高    最低      成交量           成交额
    19  2023-12-28  9.11  9.45  9.47  9.08  1661592  1.550257e+09
    20  2023-12-29  9.42  9.39  9.48  9.35   853853  8.031967e+08





    <backtrader.feeds.pandafeed.PandasData at 0x7fed0bf2a3e0>



执行引擎可以接受任意数量的数据源，包括混合常规数据与重采样和/或重播数据(概念及示例在DataFeeds中进行描述)。有些组合可能是没有意义的，并且为了能够将数据组合起来，有一个限制条件需要满足：时间对齐(这个概念后续专节描述)。


##### 添加 Strategies

与已经是类实例的Data Feeds不同，cerebro 接受 Strategies 类及要传递给策略的参数。在优化场景中，Strategies会被多次实例化并传递不同的参数。

即使不运行优化，也可以只用于传递参数：


```python
# 直接传递策略参数 period
class MyStrategy3(bt.Strategy):
    params = (
        ('shortMaPeriod', 5),
        ('longMaPeriod', 15),
    )
    # 定义均线周期为15
    def __init__(self):
        # 计算 移动平均线
        print(f"self.params :{self.p.shortMaPeriod}")
        print(f"self.params :{self.p.longMaPeriod}")
        
cerebro3 = bt.Cerebro()
cerebro3.adddata(data2)  # 将数据传入回测系统
# 添加策略到引擎
cerebro3.addstrategy(MyStrategy3,shortMaPeriod = 10,longMaPeriod=15)
cerebro3.run()
```

    self.params :10
    self.params :15





    [<__main__.MyStrategy3 at 0x7fed0bf2b3a0>]




```python
#  优化策略参数
class MyStrategy4(bt.Strategy):  
    params = (  
        ('shortMaPeriod', 15),  # 默认值  
    )  
    print(params)
      
    # 策略的其他部分...  
      
    def next(self):  
        # 使用params.myparam1  
        print(p.shortMaPeriod)
        pass  
    
# 创建Cerebro引擎  
cerebro4 = bt.Cerebro()  
# 添加数据（这里省略了添加数据的代码）  
# ...  
  
# 添加策略和优化参数  
cerebro4.optstrategy(MyStrategy4, shortMaPeriod=range(10, 20))  

  
# 运行回测和优化  
results = cerebro4.run()  
```

    (('shortMaPeriod', 15),)


当调用cerebro.optstrategy(MyStrategy, shortMaPeriod=range(10, 20))时，你正在告诉Cerebro优化MyStrategy策略中的shortMaPeriod参数。shortMaPeriod的参数值将从10到19（包括10但不包括20）进行遍历，对于每个值，都会运行一次策略回测。

这里的range(10, 20)是一个Python的内置函数，它生成一个从10开始到19结束的整数序列。

##### 添加其他输入
这些内容都会在指定章节展开，这里只是说明，例子中可以暂时不使用。

    - addwriter 日志

    - addanalyzer 分析器

    - addobserver (or addobservermulti) 观察者

##### 自定义broker
如果有自己实现broker，则可以覆盖默认设置，通过如下方式：
```
broker = MyBroker()
cerebro.broker = broker  
```

##### 接收通知

如果 datafeeds 或broker发送通知，它们将通过Cerebro的notify_store方法接收。处理这些通知有三种方式：


- 通过addnotifycallback(callback)调用向Cerebro实例添加一个回调。回调函数的原型为：
    ```
    callback(msg, *args, **kwargs)
    ```
    msg消息体, *args和\*\*kwargs由实现定义，但通常应该期望它们是可打印的，以便接收和测试。
    
- 在Strategy子类中重写notify_store方法。
    函数原型：notify_store(self, msg, *args, **kwargs)
    

- 创建Cerebro的子类，并重写notify_store，参数与上面相同。


#################################

####但是datafeeds 如何发送通知？###

#################################

#### 执行回测

执行回测最简单的命令是：
```
result = cerebro.run(**kwargs)
```
kwargs是要传递给引擎的参数。

Cerebro（除非另有指定）会自动实例化三个Observer：

- Broker observer，用于跟踪现金和价值（投资组合）

- Trades observer，用于显示每笔交易的成效如何

- Buy/Sell observer，用于记录执行操作的时间

将stdstats设置为False即可禁用它们。

#### 返回结果

Cerebro在回测过程中会返回其创建的策略实例。这使得能够分析这些策略的表现，因为策略中的所有元素都是可以访问的。

```
result = cerebro.run(**kwargs)
```
run 方法返回的 result 的格式会根据是否使用了优化（即是否通过 optstrategy 添加了策略）而有所不同：

- 仅通过 addstrategy 添加策略：result 将是一个列表，包含了回测期间运行的所有策略实例。

- 通过 optstrategy 添加了1个或多个策略：result 将是一个列表的列表（即二维列表）。每个内部列表都包含了每次优化运行后的策略。在优化过程中，Cerebro会多次运行回测，每次尝试不同的参数组合。每次优化运行的结果（即策略实例列表）都会被保存在 result 的一个内部列表中。

这样，通过检查 result，可以了解每个策略的表现，以及（如果使用优化）不同参数组合下策略的表现。这对于分析和比较不同策略以及它们在不同参数下的表现很有用。

#### 绘图
如果安装了 matplotlib，可以通过下面的命令进行绘图。

```
cerebro.plot()
```

#### 回测逻辑
  

##### 发送所有的通知
##### 请求data feeds 提供下一组tick/K线数据
Data feeds 尝试根据下一个时间点预取数据，如果在新的时间段内没有交易的数据就提供当前周期的数据，如果有新数据可用，就提供新周期的数据。
##### 通过notify_order回调，通知策略 订单的状态/成交/资金信息。
#####  通知broker 接收新订单，用新数据模拟成交待撮合订单
##### 调用策略的next方法以让策略进行逻辑调用
这取决于数据指标的完成度，可能是prenext或nextstart，在策略/指标的最小周期要求满足之前，相关概念见最小周期部分。

在内部，策略还将触发observers, indicators, analyzers和其他活动元素
##### 写入器写入相关数据


在上述步骤1中，当Data feeds 提供新的K线时，K线已经收盘,这意味着数据已经发生,如果使用此时的收盘价则属于用了未来数据。因此，在步骤4中由策略发出的订单无法使用步骤1中的数据来执行。这意味着订单的执行将遵循x + 1的概念。其中x是订单执行的K线时刻，而x + 1则是下一个K线时刻，即订单可能执行的最早时间点。

Ref：
https://www.backtrader.com/docu/cerebro/#addtztz
    


```python

```
