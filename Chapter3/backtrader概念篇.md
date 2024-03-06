为了方便后面的工作，这里列举一些backtrader的常用名词概念。对于其专有名词仍旧使用英语，但做一些中文的解释，并结合例子加深理解。


```python
import backtrader as bt
import backtrader.indicators as btind
import backtrader.feeds as btfeeds
import akshare as ak
import pandas as pd
import datetime
```

##### 1 Data Feeds

backtrader的基本功能是由策略完成，这需要用到行情数据。策略编写人员不需要关心怎么接收他们，Data Feeds通过数组和数组位置索引的方式把成员变量自动加载给策略。一个策略举例实现如下:


```python
class MyStrategy(bt.Strategy):
    # 定义均线周期
    params = dict(period=5)

    def __init__(self):
        # 计算5日均线
        sma = btind.SimpleMovingAverage(self.datas[0], period=self.params.period)
        sma = btind.SimpleMovingAverage(period=self.params.period)
       
        print(self.datas[0]) # 输出：backtrader.feeds.pandafeed.PandasData object at 0x7f9fba215bd0>
        print(self.data) # 输出：backtrader.feeds.pandafeed.PandasData object at 0x7f9fba215bd0>
                        # 这说明以上两个的访问是等价的
# 初始化策略引擎
cerebro = bt.Cerebro()
# 利用 AKShare 获取股票的前复权数据，这里只获取前 6 列
sz000001_df = ak.stock_zh_a_hist(symbol="000001", 
    period="daily", 
    start_date="20230101", 
    end_date="20231231", 
    adjust="qfq").iloc[:, :7]
print(sz000001_df[-5:])
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

start_date = datetime.datetime(2023, 1, 1)  # 回测开始时间
end_date = datetime.datetime(2023, 12, 31)  # 回测结束时间
data = bt.feeds.PandasData(dataname=sz000001_df, fromdate=start_date, todate=end_date)  # 加载数据
cerebro.adddata(data)  # 将数据传入回测系统
# 添加策略到引擎
cerebro.addstrategy(MyStrategy)
cerebro.run()
```

                 日期    开盘    收盘    最高    最低      成交量           成交额
    237  2023-12-25  9.18  9.19  9.20  9.14   413971  3.796382e+08
    238  2023-12-26  9.19  9.10  9.20  9.07   541896  4.937466e+08
    239  2023-12-27  9.10  9.12  9.13  9.02   641534  5.820367e+08
    240  2023-12-28  9.11  9.45  9.47  9.08  1661592  1.550257e+09
    241  2023-12-29  9.42  9.39  9.48  9.35   853853  8.031967e+08
    <backtrader.feeds.pandafeed.PandasData object at 0x7f96aaefe3b0>
    <backtrader.feeds.pandafeed.PandasData object at 0x7f96aaefe3b0>





    [<__main__.MyStrategy at 0x7f96aaefe650>]



在上面的例子中使用akshare获取行情数据，关于akshare的例子前面文章中已经有描述，关于如何在backtrader 中使用akshare 也可以访问其官方页面查看。https://akshare.akfamily.xyz/demo.html#backtrader

需要注意的是：

- 策略的 \_\_init\_\_ 方法没有接收任何 *args 或 \*\*kwargs，这不代表他们不能被使用，确实可以被使用，将在后面的章节中用到。

- MyStrategy中存在一个成员变量 self.datas，它是一个数组/列表/可迭代对象，至少包含一个元素（否则将引发异常）。self.data 和self.datas[0]，self.dataX 和 self.datas[X]的访问是等价的，这个可以从print(self.datas[0])和print(self.data)输出的地址是同一个0x7f9fba215bd0得到验证。self.datas[0]指向的是第一个添加的数据，访问self.datas[1]会引发异常，因为没有添加更多的数据。

- 通过adddata添加到策略的数据 ，将以添加到系统的顺序出现在策略内部。也即是从上面的0 到X 的数据。
- btind.SimpleMovingAverage(self.datas[0], period=self.params.period) 和 btind.SimpleMovingAverage(period=self.params.period)这2者是一样的，btind.SimpleMovingAverage如何不指定数据，则默认使用self.datas[0]。


```python
# 指标和运算结果也可以是Data Feeds
class MyStrategy(bt.Strategy):
    params = dict(period1=20, period2=25, period3=10, period4=5)
    def __init__(self):

        sma1 = btind.SimpleMovingAverage(self.datas[0], period=self.p.period1)

        # 基于第一个sma指标计算得到sma2
        sma2 = btind.SimpleMovingAverage(sma1, period=self.p.period2)

        # 通过数据计算得到新的计算结果
        something = sma2 - sma1 + self.data.close

        # 通过以上结果计算得到第三个sma结果
        sma3 = btind.SimpleMovingAverage(something, period=self.p.period3)

        # Comparison operators work too ...
        greater = sma3 > sma1

        # 使用比较运算符计算sma
        sma3 = btind.SimpleMovingAverage(greater, period=self.p.period4)

```

##### 2 Parameters


```python
参数和默认值被声明为类属性（元组的元组或类似字典的对象），如果 kwargs有和类属性重复的参数，则使用kwargs中的内容。
可以通过访问成员变量self.params（缩写：self.p）在类的实例lines中使用参数。
```


```python
class MyStrategy2(bt.Strategy):
    params = dict(period=20)

    def __init__(self):
        print(f"set params by dict,self.p.period: {self.p.period}")
        
class MyStrategy3(bt.Strategy):
    params = (('period', 20),)

    def __init__(self):
        print(f"set params by tupple,self.p.period: {self.p.period}")
        
# 添加策略到引擎
cerebro2 = bt.Cerebro()
cerebro2.adddata(data)

cerebro2.addstrategy(MyStrategy2) # set params by dict,self.p.period: 20
cerebro2.addstrategy(MyStrategy3) # set params by tupple,self.p.period: 20
cerebro2.addstrategy(MyStrategy3,period = 40) # set params by tupple,self.p.period: 40

cerebro2.run()
```

    set params by dict,self.p.period: 20
    set params by tupple,self.p.period: 20
    set params by tupple,self.p.period: 40





    [<__main__.MyStrategy2 at 0x7fbc13b8faf0>,
     <__main__.MyStrategy3 at 0x7fbc13d4f3a0>,
     <__main__.MyStrategy3 at 0x7fbc13db8190>]



##### 3 Lines

这个概念非常重要，几乎所有的组件都使用了它。它可以包含一个或多个线系列，线系列是一组值，当这些值在图表中组合在一起时，它们将形成一条线。
线的一个好例子（或线系列）是由股票收盘价形成的线，如self.datas[0].close 就是一条线。


```python
class MyStrategy4(bt.Strategy):
    # 定义均线周期
    params = dict(period=3)

    def __init__(self):
        print(self.data.lines)
        # 计算移动平均线
        self.movav = btind.SimpleMovingAverage(self.data, period=self.p.period)
        print(f"self.data.lines.close[0]:{self.data.lines.close[0]}, self.data.l.close[0]:{self.data.l.close[0]}")
        print(f"self.data.lines.close[0]:{self.data.lines.close[0]}, self.data_close[0]:{self.data_close[0]}")
    
    # 注：next 是在指标计算完成后被调用的，比如sma 5， next 在 第5根，即产生sma 5的值的时候，才开始调用next。
    def next(self):
        # 在Data Feeds中访问lines,通过self.data.lines
#         print("--------")
#         print(self.data.lines.close[0])
#         print(self.data.lines.close[-1])
        # 如果当前周期的sma 大于dangdang
        if self.movav.lines.sma[0] > self.data.lines.close[0]:
            
            pass
            #print('Simple Moving Average is greater than the closing price')

cerebro4 = bt.Cerebro()
cerebro4.adddata(data)  # 将数据传入回测系统
#print(sz000001_df['close'][0:4])
# 添加策略到引擎
cerebro4.addstrategy(MyStrategy4)
cerebro4.run()
```

    <backtrader.lineseries.Lines_LineSeries_DataSeries_OHLC_OHLCDateTime_AbstractDataBase_DataBase_PandasData object at 0x7fbc13b6ff70>
    self.data.lines.close[0]:9.39, self.data.l.close[0]:9.39
    self.data.lines.close[0]:9.39, self.data_close[0]:9.39





    [<__main__.MyStrategy4 at 0x7fbc13dbb280>]



从上面的内容可以看到，self.data 有一个lines属性，lines包含一个close属性；
self.movav 是一个简单移动平均指标，它有一个lines属性，lines属性包含一个sma属性
- xxx.lines 可以简化为 xxx.l, 
- xxx.lines.name 可以简化为 xxx.lines_name
- self.data.lines.name 可以简化为 self.data_name

开发一个指标时，必须声明该指标具有lines。就像Parameters一样，也作为类属性进行声明，仅支持元组，不支持字典，因为它无法按插入排序。


Lines有一组点，在执行过程中动态增长，可以通过调用Python len函数来随时计算长度， 还有一个方法是buflen，2者的区别在于：
- len表示已处理的数量
- buflen表示加载的数据总量，即datas[x].lines 的长度

如果两者返回相同的值，则表示没有预加载数据，或者已经循环处理了所有的数据（除非系统连接到实时数据源，否则这意味着处理结束），参考下面的例


```python
class MyStrategy5(bt.Strategy):
    # 定义均线周期
    params = dict(period=3)

    def __init__(self):
        pass
        
    def next(self):
        # 只打印前3条
        if len(self.data.lines) <= 3:
            print('-------------------')
            print("当前处理长度为:",len(self.data.lines)) # 随着数据处理，值在增大
            print("总  长  度 为:",self.data.lines.buflen()) # 始终输出相同的值
cerebro5 = bt.Cerebro()
cerebro5.adddata(data)  # 将数据传入回测系统
# 添加策略到引擎
cerebro5.addstrategy(MyStrategy5)
cerebro5.run()
```

    -------------------
    当前处理长度为: 1
    总  长  度 为: 242
    -------------------
    当前处理长度为: 2
    总  长  度 为: 242
    -------------------
    当前处理长度为: 3
    总  长  度 为: 242





    [<__main__.MyStrategy5 at 0x7fbc13b8d3f0>]



##### 4 Lines和Params 中的继承
__Params继承:__
- 支持多重继承
- 继承自基类的Params被继承
- 如果多个基类定义了相同的Params，则使用继承列表中最后一个类的默认值
- 如果子类中重新定义了相同的Params，则新的默认值将覆盖基类的默认值

__Lines继承__
- 支持多重继承
- 从所有基类继承Lines。由于命名为Lines，如果基类中多次使用相同的名称，则只会有一个版本的Lines

#####  5 索引 0 和 -1
Lines是多个Line的组合，Line具有多个点，这些点会构成一条线，为了在常规代码中访问这些点，选择使用基于0的方法来获取/设置当前值。策略（Strategies）只获取值，指标（Indicators）则会修改或者设置值(比如生成新的指标)。修改通常在开发一些指标（Indicator）时使用，因为新的指标是在原始数据上的计算结果，要在策略总使用，必须保存， 比如上个例子中计算移动平均线。

索引0 和-1 的说明：
- 比如访问当前周期的数据，first_point = line[0]
- 访问前一周期的数据，last_point = line[-1]
- 不能使用>0 的数据，因为那意味着在当前时刻使用了未来才产生的数据



##### 6 Slicing 切片
backtrader 不支持切片的方式获取数据，比如python 中array[0:] 表示获取第一个到最后一个数据，但backtrader 中0 已经表示访问当前周期的数据，因此无法再通过切片方式获取更多历史数据。那可以通过什么方式获取多条历史数据呢？backtrader 有自己的写法：

如 ：self.movav.get(ago=0, size=3)
- size 是获取获取的多少根
- ago 是截止到倒数第几个周期，比如0，就是截止到当前周期的size 根，比如-1，就是跳过当前周期的size 根，具体见下面的例子


```python
class MyStrategy6(bt.Strategy):
    # 定义均线周期
    params = dict(period=3)

    def __init__(self):
        # 计算 移动平均线
        self.movav = btind.SimpleMovingAverage(self.data, period=self.p.period)
    def next(self):
        # 在Data Feeds中访问lines,通过self.data.lines
        # 为观察方便，打印倒数5天的均线
        # 依次输出： [9.186666666666666, 9.163333333333332, 9.136666666666665, 9.223333333333333, 9.32]  
        if len(self.data.lines) >  self.data.lines.buflen() - 5:
            print(">>>>>>>")
            print("movav ",self.movav[0]) #print("close ",self.datas[0].lines.close[0])
        # 为了观察方便，在最后一次获取并打印
        if len(self.data.lines) == self.data.lines.buflen():
            print("--------")
            # 包含当前的最新3个周期数据
            print(self.movav.get(ago=0, size=3))
            # 不包含当前周期的的最新3个周期数据
            print(self.movav.get(ago=-1, size=3))
            # 不包含当前周期和上个周期的最新3个周期数据
            print(self.movav.get(ago=-2, size=3))
        
cerebro6 = bt.Cerebro()
cerebro6.adddata(data)  # 将数据传入回测系统
# 添加策略到引擎
cerebro6.addstrategy(MyStrategy6)
cerebro6.run()
```

    >>>>>>>
    movav  9.186666666666666
    >>>>>>>
    movav  9.163333333333332
    >>>>>>>
    movav  9.136666666666665
    >>>>>>>
    movav  9.223333333333333
    >>>>>>>
    movav  9.32
    --------
    array('d', [9.136666666666665, 9.223333333333333, 9.32])
    array('d', [9.163333333333332, 9.136666666666665, 9.223333333333333])
    array('d', [9.186666666666666, 9.163333333333332, 9.136666666666665])





    [<__main__.MyStrategy6 at 0x7fbc13db9cc0>]



使用[] 只能提取单个值。Lines 支持通过(delay)进行批量操作，比如要比较前一天的收盘价与当前的移动平均值，有 2 种方式：
- 在每个next迭代中手动进行此操作
- 生成一个预定义的 lines 对象


```python
class MyStrategy7(bt.Strategy):
    # 定义均线周期
    params = dict(period=5)
    

    def __init__(self):
        # 计算 移动平均线
        self.movav = btind.SimpleMovingAverage(self.data, period=self.p.period)
        # (delay) 是一种语法，表示比较前一天的收盘价与当前的移动平均值
        self.cmpval = self.data.close(-1) > self.movav
        self.count=0
        
    def next(self):
        # 只打印3次
        if(self.count < 3):
            print("-----------------------------")
            # 以下 2种判断是等效的，一种是在next 函数每次计算，一种是在__init__函数就计算完成
            if(self.data.close[-1] > self.movav[0]):
                print('V1:Previous close is higher than the moving average')
                self.count= self.count + 1
            if self.cmpval[0]:
                print('V2:Previous close is higher than the moving average')
cerebro7 = bt.Cerebro()
cerebro7.adddata(data)  # 将数据传入回测系统
# 添加策略到引擎
cerebro7.addstrategy(MyStrategy7)
cerebro7.run()
```

    -----------------------------
    V1:Previous close is higher than the moving average
    V2:Previous close is higher than the moving average
    -----------------------------
    V1:Previous close is higher than the moving average
    V2:Previous close is higher than the moving average
    -----------------------------
    -----------------------------
    V1:Previous close is higher than the moving average
    V2:Previous close is higher than the moving average





    [<__main__.MyStrategy7 at 0x7fbc13dbe5f0>]



按照官方的文档：
当() 内没有任何参数时，返回一个LinesCoupler对象，具体是什么，暂时不用关心，只需要知道怎么用。
比如比较两个不同周期的简单移动平均值，直接对比250个交易日（每日数据）和52周（每周数据）的数据点是没有意义的，因为在数量上不匹配。

使用LinesCoupler或类似的机制可以解决这个问题。LinesCoupler并不是简单地将数据点进行一对一的匹配，而是提供了一个方式来在内部处理不同时间周期之间的对应关系。
这通常是通过时间戳或数据点的索引来实现，即使这些索引在表面上可能看起来不同（比如交易日和交易周的计数方式不同），但在内部，它们可以根据日期时间进行对齐。

但问题在于，此时执行self.buysig = self.sma0 > self.sma1() 是直接抛出异常的，不知道哪里有问题，还需后续排查。。
可以看下面的例子：


```python
# 通过接口获取周行情数据
sz000001_week_df = ak.stock_zh_a_hist(symbol="000001", 
    period="weekly", 
    start_date="20230101", 
    end_date="20231231", 
    adjust="qfq").iloc[:, :7]
print(sz000001_week_df[-5:])
# 处理字段命名，以符合 Backtrader 的要求
sz000001_week_df.columns = [
    'date',
    'open',
    'close',
    'high',
    'low',
    'volume',
    'amt',
]
# 把 date 作为日期索引，以符合 Backtrader 的要求
sz000001_week_df.index = pd.to_datetime(sz000001_week_df['date'])

start_date = datetime.datetime(2023, 1, 1)  # 回测开始时间
end_date = datetime.datetime(2023, 12, 29)  # 回测结束时间
data_week = bt.feeds.PandasData(dataname=sz000001_week_df, fromdate=start_date, todate=end_date)  # 加载数据



class MyStrategy8(bt.Strategy):
    params = dict(period=20)

    def __init__(self):
        # 查看下数据的长度，以确保有数据
        print(" name:", self.data0._name) # name: daily
        print(" data len:", self.data0.buflen()) # data len: 242
        #print(type(self.datas))
        print(" name:", self.data1._name)  #name: weekly
        print(" data len:", self.data1.buflen()) # data len:50
        # data0 is a daily data
        self.sma0 = btind.SMA(self.data0, period=15)  # 15 days sma
        # data1 is a weekly data
        self.sma1 = btind.SMA(self.data1, period=5)  # 5 weeks sma
        # sma1() 被扩展到和sma0 一样的长度
        self.buysig = self.sma0 > self.sma1

    def next(self):
        pass
        #  在最后一个位置看之前的数据，进行对比分析
#         if len(self.data.lines) == self.data.lines.buflen():
#             print(self.buysig[0])
#             #print(self.bugsig.buflen())
#             pass
            #print(self.buysig.get(ago=-1, size=3 * 5))
#             print(self.sma1.get(ago=-1, size=5))
#             print(self.buysig.get(ago=-1, size=3 * 5))
            
cerebro8 = bt.Cerebro()
# 查看下添加数据的方式
# cerebro.adddata? 
# 将日线数据传入回测系统，数据按数据被添加，也可以指定名称添加
# cerebro8.adddata(data)  
# cerebro8.adddata(data_week)
# 和上面的方式是等价的，只是有了名字
cerebro8.adddata(data,name = 'daily')  
cerebro8.adddata(data_week,name = 'weekly')
# 添加策略到引擎
cerebro8.addstrategy(MyStrategy8)
#cerebro8.run()
```

                日期     开盘    收盘     最高    最低      成交量           成交额
    45  2023-12-01  10.09  9.66  10.09  9.58  4769781  4.680145e+09
    46  2023-12-08   9.67  9.30   9.68  9.30  3752404  3.558544e+09
    47  2023-12-15   9.22  9.21   9.50  9.13  4781821  4.429172e+09
    48  2023-12-22   9.18  9.20   9.28  8.99  4036822  3.687013e+09
    49  2023-12-29   9.18  9.39   9.48  9.02  4112846  3.808875e+09





    0



##### python 操作符
backtrader 允许在使用python操作符，并分为两种。
- 用于创建对象
- 用户判断真假

先看第一种，实际上前面的例子已经用到了：


```python
class MyStrategy9(bt.Strategy):
    # 定义均线周期
    params = dict(period=3)
    

    def __init__(self):
        # 计算 移动平均线
        self.movav = btind.SimpleMovingAverage(self.data, period=self.p.period)
        # 创建比较对象
        close_over_sma = self.data.close > self.movav
        # 计算最高价和sma 的差值
        sma_dist_to_high = self.data.high - self.movav
        # 得到最小值
        sma_dist_small = sma_dist_to_high < 3.5

        # and 操作符不能被重新，作者提供了方法模拟它
        self.sell_sig = bt.And(close_over_sma, sma_dist_small)
        self.count=0
       
    def next(self):
        # 打印前3笔
        if self.count < 5:
            self.count = self.count + 1
            print(self.sell_sig[0])


cerebro9 = bt.Cerebro()
cerebro9.adddata(data)  # 将数据传入回测系统
# 添加策略到引擎
cerebro9.addstrategy(MyStrategy9)
cerebro9.run()
```

    1.0
    1.0
    1.0
    0.0
    1.0





    [<__main__.MyStrategy9 at 0x7f969a85d030>]



第二种，用户判断真假
对python 不能重写的部分函数进行了单独提供与封装，使其可以类似python原生方法进行功能调用。

Operators:

and -> And

or -> Or

Logic Control:

if -> If

Functions:

any -> Any

all -> All

cmp -> Cmp

max -> Max

min -> Min

sum -> Sum

reduce -> Reduce
