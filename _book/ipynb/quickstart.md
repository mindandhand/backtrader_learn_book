
# 1.1 快速运行几个demo 
---
#### 使用backtrader进行回测分析包含以下几个步骤：
    - 创建策略
        - 确定要调优的参数：可以分析确定是否有效的，或者通过回测要确定最优值的
        - 初始化策略中用到的指标数据：通过引擎传入的数据实现
        - 判断是否达到了买卖点
    - 执行引擎
        - 初始化引擎
        - 注入创建的策略
        - 加载和注入数据Feed
        - 运行引擎
        - 进行可视化分析


#### 1.1.1 先运行引擎看看
第一步是让程序跑起来，而不是考虑策略，因此首先初始化引擎运行看看。


```python
# 引用backtrader
import backtrader as bt
if __name__ == '__main__':
    # 1. 初始化引擎
    cerebro = bt.Cerebro()
    # 2. 打印策略执行前的资金
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 3. 执行引擎
    cerebro.run()
    # 4.打印策略执行后的资金
    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
```

    Starting Portfolio Value: 10000.00
    Final Portfolio Value: 10000.00


可以看到, 引擎默认初始资金为10000.00；下一步尝试自定义初始资金为1000000.0。


```python
import backtrader as bt
if __name__ == '__main__':
    # 1. 初始化引擎
    cerebro = bt.Cerebro()
    # 2.设置初始化资金为100万
    cerebro.broker.setcash(1000000.0)
    # 3. 打印策略执行前的资金
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 4. 执行引擎
    cerebro.run()
    # 5.打印策略执行后的资金
    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
```

    Starting Portfolio Value: 1000000.00
    Final Portfolio Value: 1000000.00


初始资金被成功修改为1000000.00。

##### 1.1.2 尝试添加数据
尝试执行完引擎后，下一步要做的就是尝试添加数据，和上一节相比，输出不会有任何变化，这是因为仅加载了数据，但并未使用它。


```python
# 引用backtrader
import backtrader as bt
import datetime
if __name__ == '__main__':
    # 1. 初始化引擎
    cerebro = bt.Cerebro()
    # 2.  引用一个数据源, 读取雅虎数据格式的本地文件
    # 这里暂时不用关心数据的具体格式，自定义数据加载及其它源加载，后续章节会讲到
    data = bt.feeds.YahooFinanceCSVData(
    dataname='./GSPC.csv',
    fromdate=datetime.datetime(2019, 8, 20),
    todate=datetime.datetime(2020, 8, 20),
    reverse=False)

    # 添加数据到引擎中
    cerebro.adddata(data)
    # 3.设置初始化资金为100万
    cerebro.broker.setcash(1000000.0)
    # 4. 打印策略执行前的资金
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 5. 执行引擎
    cerebro.run()
    # 6.打印策略执行后的资金
    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())

```

    Starting Portfolio Value: 1000000.00
    Final Portfolio Value: 1000000.00


#### 1.1.3 运行第一个策略
设置了资金和行情数据后，就可以创建策略，训练模型。策略必须继承bt策略类，接下来执行一个例子，如果价格下降到2500点以下，就认为价格到达了低点，记录收盘的价格。


```python
# 3.1 创建的策略必须继承bt策略基类
class TestStrategy(bt.Strategy):
    # 3.2 数据和策略添加到同一引擎后，在策略中可通过datas访问数据，这里仅获取了传送数据的收盘价信息
    def __init__(self):
        self.dataclose = self.datas[0].close

    # 3.3 定义本策略的日志文件
    def log(self, txt, dt=None):
        dt = dt or self.datas[0].datetime.date(0)
        print('%s, %s' % (dt.isoformat(), txt))

    # 3.4 添加策略逻辑的地方，添加数据后，遍历每个日期进行调用，当前在的时间点坐标为0，其前面一根或者说之前一个交易日的索引为-1，依次类推
    def next(self):
        # 如果价格低于2500点，就记录下收盘价
        if self.dataclose[0] < 2500:
            self.log('Close, %.2f' % self.dataclose[0])

 # 引用backtrader
import backtrader as bt
if __name__ == '__main__':
    # 1. 初始化引擎
    cerebro = bt.Cerebro()
    # 2.  引用一个数据源, 读取雅虎数据格式的本地文件
    # 这里暂时不用关心数据的具体格式，自定义数据加载及其它源加载，后续章节会讲到
    data = bt.feeds.YahooFinanceCSVData(
    dataname='./GSPC.csv',
    fromdate=datetime.datetime(2019, 8, 20),
    todate=datetime.datetime(2020, 8, 20),
    reverse=False)

    # 添加数据到引擎中
    cerebro.adddata(data)
    # 3. 添加策略类到引擎中
    cerebro.addstrategy(TestStrategy)
    # 4.设置初始化资金为100万
    cerebro.broker.setcash(1000000.0)
    # 5. 打印策略执行前的资金
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 6. 执行引擎
    cerebro.run()
    # 7.打印策略执行后的资金
    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())

```

    Starting Portfolio Value: 1000000.00
    2020-03-12, Close, 2480.64
    2020-03-16, Close, 2386.13
    2020-03-18, Close, 2398.10
    2020-03-19, Close, 2409.39
    2020-03-20, Close, 2304.92
    2020-03-23, Close, 2237.40
    2020-03-24, Close, 2447.33
    2020-03-25, Close, 2475.56
    2020-04-01, Close, 2470.50
    2020-04-03, Close, 2488.65
    Final Portfolio Value: 1000000.00


价格到达低点，触发了我们的条件，记录了收盘价，那干嘛不直接执行买入呢？


```python
# 3.1 创建的策略必须继承bt策略基类
class TestStrategy(bt.Strategy):
    # 3.2 数据和策略添加到同一引擎后，在策略中可通过datas访问数据，这里仅获取了传送数据的收盘价信息
    def __init__(self):
        self.dataclose = self.datas[0].close

    # 3.3 定义本策略的日志文件
    def log(self, txt, dt=None):
        dt = dt or self.datas[0].datetime.date(0)
        print('%s, %s' % (dt.isoformat(), txt))

    # 3.4 添加策略逻辑的地方，添加数据后，遍历每个日期进行调用，当前在的时间点坐标为0，其前面一根K线或者说之前一个交易节点的索引为-1，依次类推
    def next(self):
        # 3.5 0 当前交易节点的索引，-1 上一个交易节点的索引
        if self.dataclose[0] < 2500:

            # 3.6 条件满足,则发出下单指令并log记录，默认使用下一根K线的开盘价；如果未指定买入标的，则买入self.datas[0]的标的
            self.buy()
            self.log('BUY CREATE, %.2f' % self.dataclose[0])

 # 引用backtrader
import backtrader as bt
if __name__ == '__main__':
    # 1. 初始化引擎
    cerebro = bt.Cerebro()
    # 2.  引用一个数据源, 读取雅虎数据格式的本地文件
    # 这里暂时不用关心数据的具体格式，自定义数据加载及其它源加载，后续章节会讲到
    data = bt.feeds.YahooFinanceCSVData(
    dataname='./GSPC.csv',
    fromdate=datetime.datetime(2019, 8, 20),
    todate=datetime.datetime(2020, 8, 20),
    reverse=False)

    # 添加数据到引擎中
    cerebro.adddata(data)
    # 3. 添加策略类到引擎中
    cerebro.addstrategy(TestStrategy)
    # 4.设置初始化资金为100万
    cerebro.broker.setcash(1000000.0)
    # 5. 打印策略执行前的资金
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 6. 执行引擎
    cerebro.run()
    # 7.打印策略执行后的资金
    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
```

    Starting Portfolio Value: 1000000.00
    2020-03-12, BUY CREATE, 2480.64
    2020-03-16, BUY CREATE, 2386.13
    2020-03-18, BUY CREATE, 2398.10
    2020-03-19, BUY CREATE, 2409.39
    2020-03-20, BUY CREATE, 2304.92
    2020-03-23, BUY CREATE, 2237.40
    2020-03-24, BUY CREATE, 2447.33
    2020-03-25, BUY CREATE, 2475.56
    2020-04-01, BUY CREATE, 2470.50
    2020-04-03, BUY CREATE, 2488.65
    Final Portfolio Value: 1009296.40


上面的例子中，日志记录了触发点的日期，买入标识，以及收盘价。但我们不知道的是，发出的订单是否被执行了，如果执行了，是什么价格，买了多少，以及如何选择卖出时机呢？下面我们要获取订单的状态，以及寻找卖点。


```python
class TestStrategy(bt.Strategy):
    # 3.2 数据和策略添加到同一引擎后，在策略中可通过datas访问数据，这里仅获取了传送数据的收盘价信息
    def __init__(self):
        self.dataclose = self.datas[0].close
        self.order = None

    # 3.3 定义本策略的日志文件
    def log(self, txt, dt=None):
        dt = dt or self.datas[0].datetime.date(0)
        print('%s, %s' % (dt.isoformat(), txt))

    ## 3.4 添加策略逻辑的地方，添加数据后，遍历每个日期进行调用，当前在的时间点坐标为0，其前面一根或者说之前一个交易节点的索引为-1，依次类推
    def next(self):
        # 3.4.1 为了简单示例，检查是否有处理中的订单， 不再重复下
        if self.order:
            return

        # 3.4.2 检测是否已经有持仓
        if not self.position:

            # 3.4.3 没有持仓，判断是否满足买的条件
            if self.dataclose[0] < 2500:

                # 3.4.4 价格低于2500，到达买点，下单，默认使用下一根K线的开盘价；
                # self.log('BUY CREATE, %.2f' % self.dataclose[0])

                # 3.4.5 跟踪订单
                self.order = self.buy()

        else:

            # 3.4.6 已经有持仓，判断是否到达卖的条件
            if self.dataclose[0] > 3000:
                # 3.4.7 价格高于3000，市场被高估，卖出
                # self.log('SELL CREATE, %.2f' % self.dataclose[0])

                # 3.4.8 跟踪订单
                self.order = self.sell()

    # 3.5 订单状态回调
    def notify_order(self, order):
        if order.status in [order.Submitted, order.Accepted]:
            # 3.5.1 状态变更信息，直接返回
            return

        # 3.5.2 检查订单是否已完成，有可能回测拒单：因为剩余资金可能不足
        if order.status in [order.Completed]:
            # 3.5.3 订单已执行，无论买卖都打印
            if order.isbuy(): 
                self.log('BUY EXECUTED, %.2f' % order.executed.price)
            elif order.issell():
                self.log('SELL EXECUTED, %.2f' % order.executed.price)
            # 3.5.4 记录执行的日期
            self.bar_executed = len(self)

        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
            # 3.5.5 订单未能执行
            self.log('Order Canceled/Margin/Rejected')

        # 3.5.6 订单均已处理完成
        self.order = None

# 引用backtrader
import backtrader as bt
if __name__ == '__main__':
    # 1. 初始化引擎
    cerebro = bt.Cerebro()
    # 2.  引用一个数据源, 读取雅虎数据格式的本地文件
    # 这里暂时不用关心数据的具体格式，自定义数据加载及其它源加载，后续章节会讲到
    data = bt.feeds.YahooFinanceCSVData(
    dataname='./GSPC.csv',
    fromdate=datetime.datetime(2019, 8, 20),
    todate=datetime.datetime(2020, 8, 20),
    reverse=False)

    # 添加数据到引擎中
    cerebro.adddata(data)
    # 3. 添加策略类到引擎中
    cerebro.addstrategy(TestStrategy)
    # 4.设置初始化资金为100万
    cerebro.broker.setcash(1000000.0)
    # 5. 打印策略执行前的资金
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 6. 执行引擎
    cerebro.run()
    # 7.打印策略执行后的资金
    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
```

    Starting Portfolio Value: 1000000.00
    2020-03-13, BUY EXECUTED, 2569.99
    2020-05-28, SELL EXECUTED, 3046.61
    Final Portfolio Value: 1000476.62


可以看到，订单的买入、卖出及执行价格都被记录了下来。在我们未指定下单数量的时候，系统默认为1（买入价格为下一个K线的开盘价）。系统最终盈利: 3046.61 - 2569.99 = 476.62。如果最后一天是有持仓的，则在计算最终组合价值时，使用持有股票最后的收盘价来进行计算。

#### 1.1.4 利润的侵蚀者：佣金
无论在什么时候，佣金都不可忽视。


```python
class TestStrategy(bt.Strategy):
    # 3.2 数据和策略添加到同一引擎后，在策略中可通过datas访问数据，这里仅获取了传送数据的收盘价信息
    def __init__(self):
        self.dataclose = self.datas[0].close
        self.order = None
        self.buyprice = None
        self.buycomm = None

    # 3.3 定义本策略的日志文件
    def log(self, txt, dt=None):
        dt = dt or self.datas[0].datetime.date(0)
        print('%s, %s' % (dt.isoformat(), txt))

    ## 3.4 添加策略逻辑的地方，添加数据后，遍历每个日期进行调用，当前在的时间点坐标为0，其前面一根或者说之前一个交易节点的索引为-1，依次类推
    def next(self):
        # 3.4.1 检查是否有处理中的订单， 不再重复下
        if self.order:
            return

        # 3.4.2 检测是否已经有持仓
        if not self.position:

            # 3.4.3 没有持仓，判断是否满足买的条件
            if self.dataclose[0] < 2500:

                # 3.4.4 条件满足，下单，默认使用下一根K线的开盘价；
                # self.log('BUY CREATE, %.2f' % self.dataclose[0])

                # 3.4.5 跟踪订单
                self.order = self.buy()

        else:

            # 3.4.6 已经有持仓，判断是否到达卖的条件
            if self.dataclose[0] > 3000:
                # 3.4.7 条件满足，卖出
                # self.log('SELL CREATE, %.2f' % self.dataclose[0])

                # 3.4.8 跟踪订单
                self.order = self.sell()

    # 3.5 订单状态回调
    def notify_order(self, order):
        if order.status in [order.Submitted, order.Accepted]:
            # 3.5.1 状态变更信息，直接返回
            return

        # 3.5.2 检查订单是否已完成，有可能回测拒单：因为剩余资金可能不足
        if order.status in [order.Completed]:
            # 3.5.3 订单已执行，无论买卖都打印
            if order.isbuy(): 
                self.log(
                    'BUY EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
                    (order.executed.price,
                    order.executed.value,
                    order.executed.comm))

                self.buyprice = order.executed.price
                self.buycomm = order.executed.comm
            elif order.issell():
                self.log('SELL EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
                        (order.executed.price,
                        order.executed.value,
                        order.executed.comm))
            # 3.5.4 记录执行的日期
            self.bar_executed = len(self)

        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
            # 3.5.5 订单未能执行
            self.log('Order Canceled/Margin/Rejected')

        # 3.5.6 订单均已处理完成
        self.order = None

    # 3.6.1 交易结果回调
    def notify_trade(self, trade):
        if not trade.isclosed:
            return
        self.log('OPERATION PROFIT, GROSS %.2f, NET %.2f' %
                (trade.pnl, trade.pnlcomm))
# 引用backtrader
import backtrader as bt
if __name__ == '__main__':
    # 1. 初始化引擎
    cerebro = bt.Cerebro()
    # 2.  引用一个数据源, 读取雅虎数据格式的本地文件
    # 这里暂时不用关心数据的具体格式，自定义数据加载及其它源加载，后续章节会讲到
    data = bt.feeds.YahooFinanceCSVData(
    dataname='./GSPC.csv',
    fromdate=datetime.datetime(2019, 8, 20),
    todate=datetime.datetime(2020, 6, 20),
    reverse=False)

    # 添加数据到引擎中
    cerebro.adddata(data)
    # 3. 添加策略类到引擎中
    cerebro.addstrategy(TestStrategy)
    # 4.设置初始化资金为100万
    cerebro.broker.setcash(1000000.0)
    # 4.1 佣金设置为万3
    cerebro.broker.setcommission(commission=0.0003)
    # 5. 打印策略执行前的资金
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 6. 执行引擎
    cerebro.run()
    # 7.打印策略执行后的资金
    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
```

    Starting Portfolio Value: 1000000.00
    2020-03-13, BUY EXECUTED, Price: 2569.99, Cost: 2569.99, Comm 0.77
    2020-05-28, SELL EXECUTED, Price: 3046.61, Cost: 2569.99, Comm 0.91
    2020-05-28, OPERATION PROFIT, GROSS 476.62, NET 474.94
    Final Portfolio Value: 1000474.94


相比上一节，增加了佣金设置，即Comm, 双向收取，如果买卖频率过高，将极大的侵蚀利润。这里还没有添加印花税。476.62-0.77-0.91= 474.94, 474.94才是暂时属于我们的利润。

#### 1.1.5  自定义策略参数
策略应该尽量是模板式的，这样环境变了才能更方便的修改优化，这也契合backtrader的设计思想。那怎么自定义策略参数呢？


```python
class TestStrategy(bt.Strategy):
    # 3.2 数据和策略添加到同一引擎后，在策略中可通过datas访问数据，这里仅获取了传送数据的收盘价信息
    def __init__(self, sellprice):
        self.dataclose = self.datas[0].close
        self.order = None
        self.buyprice = None
        self.buycomm = None
        self.sellprice = sellprice

    # 3.3 定义本策略的日志文件
    def log(self, txt, dt=None):
        dt = dt or self.datas[0].datetime.date(0)
        print('%s, %s' % (dt.isoformat(), txt))

    ## 3.4 添加策略逻辑的地方，添加数据后，遍历每个日期进行调用，当前在的时间点坐标为0，其前面一根或者说之前一个交易节点的索引为-1，依次类推
    def next(self):
        # 3.4.1 检查是否有处理中的订单， 不再重复下
        if self.order:
            return

        # 3.4.2 检测是否已经有持仓
        if not self.position:

            # 3.4.3 没有持仓，判断是否满足买的条件
            if self.dataclose[0] < 2500:

                # 3.4.4 条件满足，下单，默认使用下一根K线的开盘价；
                # self.log('BUY CREATE, %.2f' % self.dataclose[0])

                # 3.4.5 跟踪订单
                self.order = self.buy()

        else:

            # 3.4.6 已经有持仓，判断是否到达卖的条件
            if self.dataclose[0] > self.sellprice:
                # 3.4.7 条件满足，卖出
                # self.log('SELL CREATE, %.2f' % self.dataclose[0])

                # 3.4.8 跟踪订单
                self.order = self.sell()

    # 3.5 订单状态回调
    def notify_order(self, order):
        if order.status in [order.Submitted, order.Accepted]:
            # 3.5.1 状态变更信息，直接返回
            return

        # 3.5.2 检查订单是否已完成，有可能回测拒单：因为剩余资金可能不足
        if order.status in [order.Completed]:
            # 3.5.3 订单已执行，无论买卖都打印
            if order.isbuy(): 
                self.log(
                    'BUY EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
                    (order.executed.price,
                    order.executed.value,
                    order.executed.comm))

                self.buyprice = order.executed.price
                self.buycomm = order.executed.comm
            elif order.issell():
                self.log('SELL EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
                        (order.executed.price,
                        order.executed.value,
                        order.executed.comm))
            # 3.5.4 记录执行的日期
            self.bar_executed = len(self)

        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
            # 3.5.5 订单未能执行
            self.log('Order Canceled/Margin/Rejected')

        # 3.5.6 订单均已处理完成
        self.order = None

    # 3.6.1 交易结果回调
    def notify_trade(self, trade):
        if not trade.isclosed:
            return
        self.log('OPERATION PROFIT, GROSS %.2f, NET %.2f' %
                (trade.pnl, trade.pnlcomm))
# 引用backtrader
import backtrader as bt
if __name__ == '__main__':
    # 1. 初始化引擎
    cerebro = bt.Cerebro()
    # 2.  引用一个数据源, 读取雅虎数据格式的本地文件
    # 这里暂时不用关心数据的具体格式，自定义数据加载及其它源加载，后续章节会讲到
    data = bt.feeds.YahooFinanceCSVData(
    dataname='./GSPC.csv',
    fromdate=datetime.datetime(2019, 8, 20),
    todate=datetime.datetime(2020, 6, 20),
    reverse=False)

    # 添加数据到引擎中
    cerebro.adddata(data)
    # 3. 添加策略类到引擎中,自定义卖出参数，到3200才卖
    cerebro.addstrategy(TestStrategy, sellprice=3200)
    # 4.设置初始化资金为100万
    cerebro.broker.setcash(1000000.0)
    # 4.1 佣金设置为万3
    cerebro.broker.setcommission(commission=0.0003)
    # 5. 打印策略执行前的资金
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 6. 执行引擎
    cerebro.run()
    # 7.打印策略执行后的资金
    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
```

    Starting Portfolio Value: 1000000.00
    2020-03-13, BUY EXECUTED, Price: 2569.99, Cost: 2569.99, Comm 0.77
    2020-06-09, SELL EXECUTED, Price: 3213.32, Cost: 2569.99, Comm 0.96
    2020-06-09, OPERATION PROFIT, GROSS 643.33, NET 641.60
    Final Portfolio Value: 1000641.60


这里仅简单的自定义了卖出价格，随着模型的逐步建立，个性化的参数也会不断增加，最好把参数保存到文件或者缓存redis集群中。

#### 1.1.6 添加技术指标
除了价格信息，很多时候技术指标更能反应当前的行情趋势，而无需关心现在价位，因为无论价格多高，只要还会上涨，就可以通过做多盈利。幸运的是，backtrader内置了常用的金融分析库talib，可以很方便的添加技术指标；这里需要提前说明的是，talib中的部分技术指标计算与中国国内券商采用的部分指标计算方式略有不同，这点在后面会进行说明。这里构造一个双均线策略，即MA5上穿MA15则买入，MA5下穿MA15则卖出。


```python
class TestStrategy(bt.Strategy):
    # 3.2 数据和策略添加到同一引擎后，在策略中可通过datas访问数据，这里仅获取了传送数据的收盘价信息
    def __init__(self):
        self.dataclose = self.datas[0].close
        self.order = None
        self.buyprice = None
        self.buycomm = None
        # 3.2.1 数据初始化的时候就添加技术指标的计算
        self.sma5 = bt.indicators.SimpleMovingAverage(
            self.datas[0], period=5)
        self.sma15 = bt.indicators.SimpleMovingAverage(
            self.datas[0], period=15)

    # 3.3 定义本策略的日志文件
    def log(self, txt, dt=None):
        dt = dt or self.datas[0].datetime.date(0)
        print('%s, %s' % (dt.isoformat(), txt))

    ## 3.4 添加策略逻辑的地方，添加数据后，遍历每个日期进行调用，当前在的时间点坐标为0，其前面一根或者说之前一个交易节点的索引为-1，依次类推
    def next(self):
        # 3.4.1 检查是否有处理中的订单， 不再重复下
        if self.order:
            return

        # 3.4.2 检测是否已经有持仓
        if not self.position:

            # 3.4.3 没有持仓，判断是否满足买的条件
            if self.sma5[-1] < self.sma15[-1] and self.sma5[0] > self.sma15[0]:

                # 3.4.4 前一天条件满足，下单，默认使用下一根K线的开盘价；
                self.log('BUY CREATE, %.2f' % self.dataclose[0])

                # 3.4.5 跟踪订单
                self.order = self.buy()

        else:

            # 3.4.6 已经有持仓，判断是否到达卖的条件
            if self.sma5[-1] > self.sma15[-1] and self.sma5[0] < self.sma15[0]:
                # 3.4.7 条件满足，卖出
                # self.log('SELL CREATE, %.2f' % self.dataclose[0])

                # 3.4.8 跟踪订单
                self.order = self.sell()

    # 3.5 订单状态回调
    def notify_order(self, order):
        if order.status in [order.Submitted, order.Accepted]:
            # 3.5.1 状态变更信息，直接返回
            return

        # 3.5.2 检查订单是否已完成，有可能回测拒单：因为剩余资金可能不足
        if order.status in [order.Completed]:
            # 3.5.3 订单已执行，无论买卖都打印
            if order.isbuy(): 
                self.log(
                    'BUY EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
                    (order.executed.price,
                    order.executed.value,
                    order.executed.comm))

                self.buyprice = order.executed.price
                self.buycomm = order.executed.comm
            elif order.issell():
                self.log('SELL EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
                        (order.executed.price,
                        order.executed.value,
                        order.executed.comm))
            # 3.5.4 记录执行的日期
            self.bar_executed = len(self)

        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
            # 3.5.5 订单未能执行
            self.log('Order Canceled/Margin/Rejected')

        # 3.5.6 订单均已处理完成
        self.order = None

    # 3.6.1 交易结果回调
    def notify_trade(self, trade):
        if not trade.isclosed:
            return
        self.log('OPERATION PROFIT, GROSS %.2f, NET %.2f' %
                (trade.pnl, trade.pnlcomm))
# 引用backtrader
import backtrader as bt
if __name__ == '__main__':
    # 1. 初始化引擎
    cerebro = bt.Cerebro()
    # 2.  引用一个数据源, 读取雅虎数据格式的本地文件
    # 这里暂时不用关心数据的具体格式，自定义数据加载及其它源加载，后续章节会讲到
    data = bt.feeds.YahooFinanceCSVData(
    dataname='./GSPC.csv',
    fromdate=datetime.datetime(2019, 8, 20),
    todate=datetime.datetime(2020, 6, 20),
    reverse=False)

    # 添加数据到引擎中
    cerebro.adddata(data)
    # 3. 添加策略类到引擎中
    cerebro.addstrategy(TestStrategy)
    # 4.设置初始化资金为100万
    cerebro.broker.setcash(1000000.0)
    # 4.1 佣金设置为万3
    cerebro.broker.setcommission(commission=0.0003)
    # 5. 打印策略执行前的资金
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 6. 执行引擎
    cerebro.run()
    # 7.打印策略执行后的资金
    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
```

    Starting Portfolio Value: 1000000.00
    2019-10-15, BUY CREATE, 2995.68
    2019-10-16, BUY EXECUTED, Price: 2989.68, Cost: 2989.68, Comm 0.90
    2019-12-06, SELL EXECUTED, Price: 3134.62, Cost: 2989.68, Comm 0.94
    2019-12-06, OPERATION PROFIT, GROSS 144.94, NET 143.10
    2019-12-10, BUY CREATE, 3132.52
    2019-12-11, BUY EXECUTED, Price: 3135.75, Cost: 3135.75, Comm 0.94
    2020-01-30, SELL EXECUTED, Price: 3256.45, Cost: 3135.75, Comm 0.98
    2020-01-30, OPERATION PROFIT, GROSS 120.70, NET 118.78
    2020-02-07, BUY CREATE, 3327.71
    2020-02-10, BUY EXECUTED, Price: 3318.28, Cost: 3318.28, Comm 1.00
    2020-02-25, SELL EXECUTED, Price: 3238.94, Cost: 3318.28, Comm 0.97
    2020-02-25, OPERATION PROFIT, GROSS -79.34, NET -81.31
    2020-03-30, BUY CREATE, 2626.65
    2020-03-31, BUY EXECUTED, Price: 2614.69, Cost: 2614.69, Comm 0.78
    2020-05-18, SELL EXECUTED, Price: 2913.86, Cost: 2614.69, Comm 0.87
    2020-05-18, OPERATION PROFIT, GROSS 299.17, NET 297.51
    2020-05-20, BUY CREATE, 2971.61
    2020-05-21, BUY EXECUTED, Price: 2969.95, Cost: 2969.95, Comm 0.89
    2020-06-17, SELL EXECUTED, Price: 3136.13, Cost: 2969.95, Comm 0.94
    2020-06-17, OPERATION PROFIT, GROSS 166.18, NET 164.35
    Final Portfolio Value: 1000642.44


#### 1.1.7 可视化买卖点
尽管已经执行了策略，但结果却不是那么显而易见，在哪个点买入，哪个点卖出，是否符合逻辑，当时的趋势怎么样,，即使日志能找到也很抽象，显然不符合人来的视觉习惯，怎么可以图形化展示呢？backtrader依赖matplotlib提供了将执行结果可视化的功能，这里继续使用上一节的例子，对结果添加可视化，代码里唯一的改动是添加了第八项绘图展示。


```python
class TestStrategy(bt.Strategy):
    # 3.2 数据和策略添加到同一引擎后，在策略中可通过datas访问数据，这里仅获取了传送数据的收盘价信息
    def __init__(self):
        self.dataclose = self.datas[0].close
        self.order = None
        self.buyprice = None
        self.buycomm = None
        # 3.2.1 数据初始化的时候就添加技术指标的计算
        self.sma5 = bt.indicators.SimpleMovingAverage(
            self.datas[0], period=5)
        self.sma15 = bt.indicators.SimpleMovingAverage(
            self.datas[0], period=15)

    # 3.3 定义本策略的日志文件
    def log(self, txt, dt=None):
        dt = dt or self.datas[0].datetime.date(0)
        print('%s, %s' % (dt.isoformat(), txt))

    ## 3.4 添加策略逻辑的地方，添加数据后，遍历每个日期进行调用，当前在的时间点坐标为0，其前面一根或者说之前一个交易节点的索引为-1，依次类推
    def next(self):
        # 3.4.1 检查是否有处理中的订单， 不再重复下
        if self.order:
            return

        # 3.4.2 检测是否已经有持仓
        if not self.position:

            # 3.4.3 没有持仓，判断是否满足买的条件
            if self.sma5[-1] < self.sma15[-1] and self.sma5[0] > self.sma15[0]:

                # 3.4.4 前一天条件满足，下单，默认使用下一根K线的开盘价；
                self.log('BUY CREATE, %.2f' % self.dataclose[0])

                # 3.4.5 跟踪订单
                self.order = self.buy()

        else:

            # 3.4.6 已经有持仓，判断是否到达卖的条件
            if self.sma5[-1] > self.sma15[-1] and self.sma5[0] < self.sma15[0]:
                # 3.4.7 条件满足，卖出
                # self.log('SELL CREATE, %.2f' % self.dataclose[0])

                # 3.4.8 跟踪订单
                self.order = self.sell()

    # 3.5 订单状态回调
    def notify_order(self, order):
        if order.status in [order.Submitted, order.Accepted]:
            # 3.5.1 状态变更信息，直接返回
            return

        # 3.5.2 检查订单是否已完成，有可能回测拒单：因为剩余资金可能不足
        if order.status in [order.Completed]:
            # 3.5.3 订单已执行，无论买卖都打印
            if order.isbuy(): 
                self.log(
                    'BUY EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
                    (order.executed.price,
                    order.executed.value,
                    order.executed.comm))

                self.buyprice = order.executed.price
                self.buycomm = order.executed.comm
            elif order.issell():
                self.log('SELL EXECUTED, Price: %.2f, Cost: %.2f, Comm %.2f' %
                        (order.executed.price,
                        order.executed.value,
                        order.executed.comm))
            # 3.5.4 记录执行的日期
            self.bar_executed = len(self)

        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
            # 3.5.5 订单未能执行
            self.log('Order Canceled/Margin/Rejected')

        # 3.5.6 订单均已处理完成
        self.order = None

    # 3.6.1 交易结果回调
    def notify_trade(self, trade):
        if not trade.isclosed:
            return
        self.log('OPERATION PROFIT, GROSS %.2f, NET %.2f' %
                (trade.pnl, trade.pnlcomm))
# 引用backtrader
import backtrader as bt
if __name__ == '__main__':
    # 1. 初始化引擎
    cerebro = bt.Cerebro()
    # 2.  引用一个数据源, 读取雅虎数据格式的本地文件
    # 这里暂时不用关心数据的具体格式，自定义数据加载及其它源加载，后续章节会讲到
    data = bt.feeds.YahooFinanceCSVData(
    dataname='./GSPC.csv',
    fromdate=datetime.datetime(2019, 8, 20),
    todate=datetime.datetime(2020, 6, 20),
    reverse=False)

    # 添加数据到引擎中
    cerebro.adddata(data)
    # 3. 添加策略类到引擎中
    cerebro.addstrategy(TestStrategy)
    # 4.设置初始化资金为100万
    cerebro.broker.setcash(1000000.0)
    # 4.1 佣金设置为万3
    cerebro.broker.setcommission(commission=0.0003)
    # 5. 打印策略执行前的资金
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 6. 执行引擎
    cerebro.run()
    # 7.打印策略执行后的资金
    print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 8. 调用plot()将结果绘图展示
    cerebro.plot()
```

    Starting Portfolio Value: 1000000.00
    2019-10-15, BUY CREATE, 2995.68
    2019-10-16, BUY EXECUTED, Price: 2989.68, Cost: 2989.68, Comm 0.90
    2019-12-06, SELL EXECUTED, Price: 3134.62, Cost: 2989.68, Comm 0.94
    2019-12-06, OPERATION PROFIT, GROSS 144.94, NET 143.10
    2019-12-10, BUY CREATE, 3132.52
    2019-12-11, BUY EXECUTED, Price: 3135.75, Cost: 3135.75, Comm 0.94
    2020-01-30, SELL EXECUTED, Price: 3256.45, Cost: 3135.75, Comm 0.98
    2020-01-30, OPERATION PROFIT, GROSS 120.70, NET 118.78
    2020-02-07, BUY CREATE, 3327.71
    2020-02-10, BUY EXECUTED, Price: 3318.28, Cost: 3318.28, Comm 1.00
    2020-02-25, SELL EXECUTED, Price: 3238.94, Cost: 3318.28, Comm 0.97
    2020-02-25, OPERATION PROFIT, GROSS -79.34, NET -81.31
    2020-03-30, BUY CREATE, 2626.65
    2020-03-31, BUY EXECUTED, Price: 2614.69, Cost: 2614.69, Comm 0.78
    2020-05-18, SELL EXECUTED, Price: 2913.86, Cost: 2614.69, Comm 0.87
    2020-05-18, OPERATION PROFIT, GROSS 299.17, NET 297.51
    2020-05-20, BUY CREATE, 2971.61
    2020-05-21, BUY EXECUTED, Price: 2969.95, Cost: 2969.95, Comm 0.89
    2020-06-17, SELL EXECUTED, Price: 3136.13, Cost: 2969.95, Comm 0.94
    2020-06-17, OPERATION PROFIT, GROSS 166.18, NET 164.35
    Final Portfolio Value: 1000642.44



    <IPython.core.display.Javascript object>



<img src="data:image/png;base64,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" width="640">


#### 1.1.8 优化策略参数
在双均线的例子中，我们使用的分别是5日和15日K线。不同的标的对应的参与人群、股本、行业景气度等不同，导致每个标的可能都有适合自己的参数，人工输入校验找规律对比显然是不现实的，那使用backtrader如何实现呢？


```python
class TestStrategy(bt.Strategy):
    params = (
        ('shortMaPeriod', 5),
        ('longMaPeriod', 15),
    )
    # 3.2 数据和策略添加到同一引擎后，在策略中可通过datas访问数据，这里仅获取了传送数据的收盘价信息
    def __init__(self):
        self.dataclose = self.datas[0].close
        self.order = None
        self.buyprice = None
        self.buycomm = None
        # 3.2.1 数据初始化的时候就添加技术指标的计算
        self.smashort = bt.indicators.SimpleMovingAverage(
            self.datas[0], period=self.params.shortMaPeriod)
        self.smalong = bt.indicators.SimpleMovingAverage(
            self.datas[0], period=self.params.longMaPeriod)

    # 3.3 定义本策略的日志文件
    def log(self, txt, dt=None):
        dt = dt or self.datas[0].datetime.date(0)
        print('%s, %s' % (dt.isoformat(), txt))

    ## 3.4 添加策略逻辑的地方，添加数据后，遍历每个日期进行调用，当前在的时间点坐标为0，其前面一根或者说之前一个交易节点的索引为-1，依次类推
    def next(self):
        # 3.4.1 检查是否有处理中的订单， 不再重复下
        if self.order:
            return

        # 3.4.2 检测是否已经有持仓
        if not self.position:

            # 3.4.3 没有持仓，判断是否满足买的条件
            if self.smashort[-1] < self.smalong[-1] and self.smashort[0] > self.smalong[0]:

                # 3.4.4 前一天条件满足，下单，默认使用下一根K线的开盘价；
                # self.log('BUY CREATE, %.2f' % self.dataclose[0])

                # 3.4.5 跟踪订单
                self.order = self.buy()

        else:

            # 3.4.6 已经有持仓，判断是否到达卖的条件
            if self.smashort[-1] > self.smalong[-1] and self.smashort[0] < self.smalong[0]:
                # 3.4.7 条件满足，卖出
                # self.log('SELL CREATE, %.2f' % self.dataclose[0])

                # 3.4.8 跟踪订单
                self.order = self.sell()

    # 3.5 订单状态回调
    def notify_order(self, order):
        if order.status in [order.Submitted, order.Accepted]:
            # 3.5.1 状态变更信息，直接返回
            return

        # 3.5.2 检查订单是否已完成，有可能回测拒单：因为剩余资金可能不足
        if order.status in [order.Completed]:
            # 3.5.3 订单已执行，无论买卖都打印
            if order.isbuy(): 

                self.buyprice = order.executed.price
                self.buycomm = order.executed.comm
            
            # 3.5.4 记录执行的日期
            self.bar_executed = len(self)

        elif order.status in [order.Canceled, order.Margin, order.Rejected]:
            # 3.5.5 订单未能执行
            self.log('Order Canceled/Margin/Rejected')

        # 3.5.6 订单均已处理完成
        self.order = None

    # 3.6 交易结果回调
    def notify_trade(self, trade):
        if not trade.isclosed:
            return
    # 3.7 添加策略停止时的价值统计
    def stop(self):
        self.log('(shortMaPeriod %2d) (longMaPeriod %2d) Ending Value %.2f' %
                 (self.params.shortMaPeriod, self.params.longMaPeriod, self.broker.getvalue()))
        
# 引用backtrader
import backtrader as bt
if __name__ == '__main__':
    # 1. 初始化引擎
    cerebro = bt.Cerebro()
    # 2.  引用一个数据源, 读取雅虎数据格式的本地文件
    # 这里暂时不用关心数据的具体格式，自定义数据加载及其它源加载，后续章节会讲到
    data = bt.feeds.YahooFinanceCSVData(
    dataname='./GSPC.csv',
    fromdate=datetime.datetime(2019, 8, 20),
    todate=datetime.datetime(2020, 6, 20),
    reverse=False)

    # 添加数据到引擎中
    cerebro.adddata(data)
    # 3. 参数优化调用optstrategy,设置传入参数范围
    cerebro.optstrategy(TestStrategy, shortMaPeriod=range(5,15, 5), longMaPeriod=[15, 20, 30,60])
    # 4.设置初始化资金为100万
    cerebro.broker.setcash(1000000.0)
    # 4.1 佣金设置为万3
    cerebro.broker.setcommission(commission=0.0003)
    # 5. 打印策略执行前的资金
    print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
    # 6. 执行引擎
    cerebro.run()
```

    Starting Portfolio Value: 1000000.00
    2020-06-19, (shortMaPeriod  5) (longMaPeriod 20) Ending Value 1000577.52
    2020-06-19, (shortMaPeriod  5) (longMaPeriod 60) Ending Value 1000227.79
    2020-06-19, (shortMaPeriod  5) (longMaPeriod 15) Ending Value 1000642.44
    2020-06-19, (shortMaPeriod  5) (longMaPeriod 30) Ending Value 1000340.50
    2020-06-19, (shortMaPeriod 10) (longMaPeriod 20) Ending Value 1000229.32
    2020-06-19, (shortMaPeriod 10) (longMaPeriod 15) Ending Value 1000433.96
    2020-06-19, (shortMaPeriod 10) (longMaPeriod 60) Ending Value 1000213.74
    2020-06-19, (shortMaPeriod 10) (longMaPeriod 30) Ending Value 1000222.38


短均线使用了5\10两种均线类型,长均线使用15\20\30\60四种均线类型，共产生2*4 = 8种结果，可以看到，不同参数的回测结果是不一样的，其短均线为5，长均线为15时获取到了最大收益。

