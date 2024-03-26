##### Line 迭代器
Line 迭代器在一定程度上模仿了Python的迭代器，但实际上与Python迭代器没有什么关联。迭代器的意思就是对进行遍历。
策略和指标都是line 迭代器。

Line 迭代器有3个方法，它们在特定条件下被调用。

- 1 next方法
    - 每次迭代时都会调用该方法。Line 迭代器所拥有的、作为逻辑/计算基础的datas数组在调用该方法之前已经由backtrader平台移动到了下一个索引。这样保证通过data调用到的都是当前周期的数据。
    - 当Line迭代器的最小周期达到时调用。关于这一点，下文将进行更详细的解释。
- 2 prenext
    - 在Line迭代器的最小周期达到之前调用。
- 3 nextstart
    - 当Line迭代器的最小周期正好达到时调用一次。
    - 默认行为是将调用转发给next方法，但如有需要也可以进行重写。
    
    
    
##### 对于指标（Indicators）的 特殊方法
为了加速运行，指标支持“一次性运行”（runonce）的批量操作模式。尽管不是严格必需的（只使用next方法就足够了），但它可以极大地减少时间消耗。

“一次性运行”方法的规则避开了使用索引0的get/set点，而是依赖于直接访问存储数据的底层数组，并为每个状态传递正确的索引。

它和Line 迭代器一样，也有3个方法：
- 1 once(self, start, end)

    - 当最小周期满足时调用。内部数组必须在start和end之间进行处理，这两个参数都是从内部数组的起始位置开始计算的，以0为基准。

- 2 preonce(self, start, end)

    - 在最小周期满足之前调用。

- 3 oncestart(self, start, end)

    - 当最小周期恰好满足时调用一次。

    - 默认行为是将调用转发给once方法，但当然也可以根据需要进行覆盖重写。
    
    
##### 最小周期
前面的2个小结都提到了最小周期这个概念，那究竟什么是最小周期?

看下面的例子：


```python
# 1.1 先构造测试使用的数据
import backtrader as bt
import backtrader.indicators as btind
import backtrader.feeds as btfeeds
import akshare as ak
import pandas as pd
import datetime
# 利用 AKShare 获取股票的前复权数据，这里只获取前 6 列
sz000001_df = ak.stock_zh_a_hist(symbol="000001", 
    period="daily", 
    start_date="20231201", 
    end_date="20231231", 
    adjust="qfq").iloc[:, :7]
print("data len:",len(sz000001_df))
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

start_date = datetime.datetime(2023, 1, 1)  # 回测开始时间
end_date = datetime.datetime(2023, 12, 31)  # 回测结束时间
sz000001_data = bt.feeds.PandasData(dataname=sz000001_df, fromdate=start_date, todate=end_date)  # 加载数据

```

    data len: 21
                日期    开盘    收盘    最高    最低      成交量           成交额
    19  2023-12-28  9.11  9.45  9.47  9.08  1661592  1.550257e+09
    20  2023-12-29  9.42  9.39  9.48  9.35   853853  8.031967e+08


上面的例子中取了一个月的K线，共计21根，并构造成backtrader 可识别的数据，供后续使用。


```python
# 1.2 有 一个指标的情况
class MyStrategy1(bt.Strategy):
    # 定义均线周期为15
    params = dict(period=10)
    

    def __init__(self):
        # 计算 移动平均线
        self.movav = btind.SimpleMovingAverage(self.data, period=self.p.period)
    
    # 在数据可访问之前每个周期调用   
    def prenext(self):
        print('prenext:: current period:', len(self))
    
    # 在next可以被调用的前一个周期调用
    def nextstart(self):
        print('nextstart:: current period:', len(self))
        
    # 指标数据准备好，有数据开始之后被调用
    def next(self):
        print('next:: current period:', len(self))
cerebro1 = bt.Cerebro()
cerebro1.adddata(sz000001_data)  # 将数据传入回测系统
# 添加策略到引擎
cerebro1.addstrategy(MyStrategy1)
cerebro1.run()
```

    prenext:: current period: 1
    prenext:: current period: 2
    prenext:: current period: 3
    prenext:: current period: 4
    prenext:: current period: 5
    prenext:: current period: 6
    prenext:: current period: 7
    prenext:: current period: 8
    prenext:: current period: 9
    nextstart:: current period: 10
    next:: current period: 11
    next:: current period: 12
    next:: current period: 13
    next:: current period: 14
    next:: current period: 15
    next:: current period: 16
    next:: current period: 17
    next:: current period: 18
    next:: current period: 19
    next:: current period: 20
    next:: current period: 21





    [<__main__.MyStrategy1 at 0x7f7e801a3f40>]



上面的例子中，把第一步生成的数据添加到策略中，在策略的初始化函数中定义10日均线。
从日志可以看到：
- 首先调用“prenext”方法9次，
- 接着调用“nextstart”方法1次
- 最后调用“next”方法n次，直到数据处理完毕。



```python
# 1.3 有2个指标的情况
class MyStrategy2(bt.Strategy):
    # 定义均线周期为15
    params = dict(period=10)
    

    def __init__(self):
        # 计算 移动平均线
        self.sma1 = btind.SimpleMovingAverage(self.data, period=self.p.period)
        self.sma2 = btind.SimpleMovingAverage(self.sma1, period=5)
    # 在数据可访问之前每个周期调用   
    def prenext(self):
        print('prenext:: current period:', len(self))
    
    # 在next可以被调用的前一个周期调用
    def nextstart(self):
        print('nextstart:: current period:', len(self))
        
    # 指标数据准备好，有数据开始之后被调用
    def next(self):
        print('next:: current period:', len(self))
cerebro2 = bt.Cerebro()
cerebro2.adddata(sz000001_data)  # 将数据传入回测系统
# 添加策略到引擎
cerebro2.addstrategy(MyStrategy2)
cerebro2.run()
```

    prenext:: current period: 1
    prenext:: current period: 2
    prenext:: current period: 3
    prenext:: current period: 4
    prenext:: current period: 5
    prenext:: current period: 6
    prenext:: current period: 7
    prenext:: current period: 8
    prenext:: current period: 9
    prenext:: current period: 10
    prenext:: current period: 11
    prenext:: current period: 12
    prenext:: current period: 13
    nextstart:: current period: 14
    next:: current period: 15
    next:: current period: 16
    next:: current period: 17
    next:: current period: 18
    next:: current period: 19
    next:: current period: 20
    next:: current period: 21





    [<__main__.MyStrategy2 at 0x7f7eb3e899c0>]



上面的例子中，把第一步生成的数据添加到策略中，在策略的初始化函数中定义10日均线，再用10日均线数据计算5日均线，sma1的计算和上一小节一样，主要增加了sma2的计算，从日志可以看到：

- 首先调用“prenext”方法13次，包含上一小节的9次，到第10次产生第一个周期的sma1数据, 加上后面的3个周期，共四周期基于sm2的prenext输出
- 接着调用“nextstart”方法1次，表示下一步数据完备；
- 最后调用“next”方法n次，直到数据处理完毕。

假设需要N个周期的数据才能产生最终的有效数据，那么prenext 会运行N-2 次，nextstart 1次，接下来就是next 到数据结束。

也就是说，只有在自动计算的最小周期达到之后，才会调用next方法（除了最初对nextstart的调用以进行初始化），策略和指标都遵循这一点。

preonce, oncestart 和 once 的关系和以上一样，区别在于用途，once的用法在后面的章节中会展开描述。


##### 启动和运行
要启动和运行一个交易系统，至少需要三个对象：
- Data feed 一份行情数据
- Strategy 一条策略
- Cerebro 一个执行引擎

##### Data Feeds

它负责提供数据，这些数据将添加到引擎后，会自动填充到策略（直接或通过指标）进行回测。

数据支持多种方式：

- 多种CSV格式和通用的CSV读取器

- Yahoo在线数据获取器

- 支持接收Pandas DataFrames和blaze对象

- 与Interactive Brokers、Visual Chart和Oanda等平台的实时数据馈送

- 关于读取多种格式数据的实例，将在Data Feeds 一节展开，这里只说明读取csv格式。

backtrader 对Data Feeds的内容（如时间范围和压缩）不做任何假设。这些值，都可以提供给引擎用于后续操作，如Data Feeds重采样（例如，将5分钟K线转换为日K）。


```python
# 雅虎 CSV 格式

class MyStrategy3(bt.Strategy):
    params = (  
        ('printlogs', True),  
    )  
    def next(self):
        # 除了打印，什么也不做
        # 获取当前周期的时间  
        current_datetime = self.datas[0].datetime.datetime(0)  
        # 获取当前周期的收盘价  
        current_close = self.datas[0].close[0]  
          
        # 如果params中的printlogs为True，则打印信息  
        if self.params.printlogs:  
            print(f"Current DateTime: {current_datetime}, Close Price: {current_close}")  
        
# 引用一个数据源, 读取雅虎数据格式的本地文件
# 这里暂时不用关心数据的具体格式，自定义数据加载及其它源加载，后续章节会讲到
data = bt.feeds.YahooFinanceCSVData(
    # 指定数据文件
    dataname='./GSPC.csv',
    # 已经从前到后按日期从历史到当前进行排序，无需反转
    reverse=False,
    # 指定开始加载的数据日期，包含
    fromdate=datetime.datetime(2020, 8, 3),
    # 指定结束加载的数据日期，不包含
    todate=datetime.datetime(2020, 8, 21)
   )

cerebro3 = bt.Cerebro()
# 添加数据到引擎中
cerebro3.adddata(data)
cerebro3.addstrategy(MyStrategy3)
# 执行引擎
cerebro3.run()


```

    Current DateTime: 2020-08-03 23:59:59.999989, Close Price: 3294.61
    Current DateTime: 2020-08-04 23:59:59.999989, Close Price: 3306.51
    Current DateTime: 2020-08-05 23:59:59.999989, Close Price: 3327.77
    Current DateTime: 2020-08-06 23:59:59.999989, Close Price: 3349.16
    Current DateTime: 2020-08-07 23:59:59.999989, Close Price: 3351.28
    Current DateTime: 2020-08-10 23:59:59.999989, Close Price: 3360.47
    Current DateTime: 2020-08-11 23:59:59.999989, Close Price: 3333.69
    Current DateTime: 2020-08-12 23:59:59.999989, Close Price: 3380.35
    Current DateTime: 2020-08-13 23:59:59.999989, Close Price: 3373.43
    Current DateTime: 2020-08-14 23:59:59.999989, Close Price: 3372.85
    Current DateTime: 2020-08-17 23:59:59.999989, Close Price: 3381.99
    Current DateTime: 2020-08-18 23:59:59.999989, Close Price: 3389.78
    Current DateTime: 2020-08-19 23:59:59.999989, Close Price: 3374.85
    Current DateTime: 2020-08-20 23:59:59.999989, Close Price: 3374.45





    [<__main__.MyStrategy3 at 0x7f7eb41efaf0>]



##### A Strategy (derived) class- 策略类

backtrader主要用于回测，策略逻辑主要在派生类的Strategy部完成。

至少有两个方法继承实现以下2个方法：

- init： 初始化的相关操作，如指标的处理等；

- next：迭代每一个周期的数据，进行逻辑判断


```python
# 一个最简单的策略
class MyStrategy4(bt.Strategy):

    def __init__(self):
        # sma 周期定义为15
        self.sma = btind.SimpleMovingAverage(self.data, period=15)

    def next(self):
        current_datetime = self.datas[0].datetime.datetime(0) 
        if self.sma > self.data.close:
            print(f"{current_datetime} sma {self.sma[0]} > close {self.datas[0].close[0]},执行买操作")
            self.buy()

        elif self.sma < self.data.close:
            print(f"{current_datetime} sma {self.sma[0]} > close {self.datas[0].close[0]  },执行卖操作")
            self.sell()

cerebro4 = bt.Cerebro()
# 添加数据到引擎中
cerebro4.adddata(sz000001_data)
cerebro4.addstrategy(MyStrategy4)
# 执行引擎
cerebro4.run()
```

    2023-12-21 00:00:00 sma 9.315333333333333 > close 9.17,执行买操作
    2023-12-22 00:00:00 sma 9.284666666666668 > close 9.2,执行买操作
    2023-12-25 00:00:00 sma 9.255333333333335 > close 9.19,执行买操作
    2023-12-26 00:00:00 sma 9.229999999999999 > close 9.1,执行买操作
    2023-12-27 00:00:00 sma 9.204666666666666 > close 9.12,执行买操作
    2023-12-28 00:00:00 sma 9.205333333333332 > close 9.45,执行卖操作
    2023-12-29 00:00:00 sma 9.211333333333332 > close 9.39,执行卖操作





    [<__main__.MyStrategy4 at 0x7f7ea1e58850>]




```python
# 策略还有其他的方法可以继承实现
class MyStrategy5(bt.Strategy):

    def __init__(self):

        self.sma = btind.SimpleMovingAverage(self.data, period=15)

    def next(self):

        if self.sma > self.data.close:
            submitted_order = self.buy()

        elif self.sma < self.data.close:
            submitted_order = self.sell()

    def start(self):
        print('Backtesting is about to start')

    def stop(self):
        print('Backtesting is finished')

    def notify_order(self, order):
        print('An order new/changed/executed/canceled has been received')
        
cerebro5 = bt.Cerebro()
# 添加数据到引擎中
cerebro5.adddata(sz000001_data)
cerebro5.addstrategy(MyStrategy5)
# 执行引擎
cerebro5.run()
```

    Backtesting is about to start
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    An order new/changed/executed/canceled has been received
    Backtesting is finished





    [<__main__.MyStrategy5 at 0x7f7ea1e58cd0>]



start 和stop 函数分别在策略开始执行和执行结束的时候被调用.
notify_order 在以下情况有用：
- buy/sell 返回一个订单，这个订单随后会被提交给broker，调用者可以根据返回的订单引用跟踪这个订单。比如，当前订单仍在等待处理时，策略不会产生新的订单。
- 如果订单的状态是Accepted/Executed/Canceled/Changed，broker会通过notify方法将这些状态变化（以及例如执行数量等信息）通知给交易策略。

使用策略类可以做的其他事：

- buy 买入 / sell 卖出 / close 平仓：使用broker和sizer 来发送买入/卖出订单。手动创建一个订单并将其传递给broker也可以达到同样的效果。但是，该平台的目的是让使用者能够更轻松地操作。平仓功能会获取当前的市场头寸并立即关闭它。

- getposition：返回当前持有的市场头寸。

- setsizer/getsizer：这个用于设置发送的订单数量。

策略（Strategy）是一个Lines对象，它支持参数化。这些参数是通过标准的Python关键字参数（kwargs）传递。

##### A Cerebro 执行引擎

一旦数据源可用且策略定义完成，Cerebro实例就将所有内容整合在一起并执行操作，Cerebor是backtrader的核心。创建一个实例很简单：
```
cerebro = bt.Cerebro()
```
如果没有特殊设置，Backtrader将使用默认设置， 包括：

   - 将创建一个默认的经纪人（broker：就是券商）。

   - 设置默认佣金为0

   - 预加载数据源（Data Feeds）

   - 默认的执行模式是runonce（批处理操作），这是最快的方式。所有指标都必须支持runonce模式以实现最大速度。平台中包含的指标均支持该模式。自定义指标不需要实现runonce功能。Cerebro将模拟该功能，这意味着那些不兼容runonce模式的指标将运行得更慢。但是，系统的大部分仍然会以批处理模式运行。

既然已经有一个可用的数据源和一个策略（之前创建的），那么将它们整合在一起并使其运行起来的标准方式是：
```
cerebro.adddata(data)
cerebro.addstrategy(MyStrategy, period=25)
cerebro.run()
```

在上面的步骤中：
   - 添加了“Data Feed”的“实例” data

   - 添加了“MyStrategy”的“类”以及将传递给它的参数（kwargs）。

   - “MyStrategy”的实例化将由cerebro在后台完成，而“addstrategy”中的任何kwargs都将传递给它。

   - 用户可以根据需要添加任意多的strategy和Data Feed。
   
   
cerebro还提供了其他接口，包括：

- 决定预加载和操作模式：具体的函数接口及展开例子在cerebro篇还会进行详细说明。
```
cerebro = bt.Cerebro(runonce=True, preload=True)
```
    这里有一个限制：runonce需要预加载（如果不预加载，则无法执行批量操作）
- setbroker / getbroker 设置交易商
- 绘图。在正常情况下，绘图操作非常简单，如下：
```
cerebro.run()
cerebro.plot()
```
plot()函数接受一些用于自定义的参数。

    - numfigs=1 如果图表过于密集，它可以被分割成多个图表。

    - plotter=None 可以传递一个自定义的绘图器实例，这样cerebro就不会实例化默认的绘图器。

    - **kwargs    标准的关键字参数,这些参数将被传递给绘图器。

请查阅绘图部分以获取更多信息。

- 策略优化

    - 如cerebro5中，Cerebro 接收一个由 Strategy 派生的类（而不是实例）以及实例化时将传递给它的关键字参数，这将在调用“run”时触发。之所以这样做， 是为了实现优化逻辑。同一个 Strategy 类将根据需要被实例化多次，每次使用新的参数。如果传递了一个实例给 cerebro，那么这是不可能的。

请求的优化如下：
```
cerebro.optstrategy(MyStrategy, period=xrange(10, 20))
```

方法 optstrategy 参数与 addstrategy 相同，但会进行额外的内部处理，以确保优化按预期运行。策略优化需要传递期望范围作为策略的正常参数，而 addstrategy默认不需要任何参数。

需要注意的是：optstrategy 会把一个可迭代对象当作一组值，这些值需要按顺序传递给 Strategy 类的每个实例。在这个简单的例子中，这个策略将尝试 10 个值，从 10 到 19（20 是上限）,和python 的范围定义一致。

如果要开发一个具有额外参数的复杂策略，所有这些参数都可以传递给 optstrategy。那些不需要进行优化的参数可以直接传递。例如：
```
cerebro.optstrategy(MyStrategy, period=xrange(10, 15), factor=3.5)
```
这个例子中，period 是可迭代对象，factor 是固定值。optstrategy 方法通过一定方式存储factor的值，并在迭代period的时候把factor=3.5 作为传进去。于是上面的例子相当于执行了5次addstrategy，格式上如下：
```
cerebro5.addstrategy(MyStrategy,period=10, factor=3.5)
cerebro5.addstrategy(MyStrategy,period=11, factor=3.5)
cerebro5.addstrategy(MyStrategy,period=12, factor=3.5)
cerebro5.addstrategy(MyStrategy,period=13, factor=3.5)
cerebro5.addstrategy(MyStrategy,period=14, factor=3.5)

```
