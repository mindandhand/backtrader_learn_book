# 1.1 快速运行几个demo 
---
使用backtrader进行回测分析包含以下几个步骤：
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

- 先运行引擎看看
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
    输出如下，可以看到, 引擎默认设置资金为10000.00
        Starting Portfolio Value: 10000.00
        Final Portfolio Value: 10000.00

- 自定义初始化资金
    ```python
    # 引用backtrader
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
    输出如下，可以看到, 初始资金修改为1000000.00
        Starting Portfolio Value: 1000000.00
        Final Portfolio Value: 1000000.00

- 尝试添加数据
    ```python
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
        # 3.设置初始化资金为100万
        cerebro.broker.setcash(1000000.0)
        # 4. 打印策略执行前的资金
        print('Starting Portfolio Value: %.2f' % cerebro.broker.getvalue())
        # 5. 执行引擎
        cerebro.run()
        # 6.打印策略执行后的资金
        print('Final Portfolio Value: %.2f' % cerebro.broker.getvalue())
    ```
    和上一节相比，输出没有任何变化，这是因为仅加载了数据，但并未使用它。
        Starting Portfolio Value: 1000000.00
        Final Portfolio Value: 1000000.00 

- 运行第一个策略
    ```py
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
            # 只是记录收盘价数据
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
    可以看到，输出中打印了策略中指定的收盘价信息，策略里添加了数据，下面就是迫不及待的添加交易逻辑了。 
        Starting Portfolio Value: 1000000.00
        2019-08-20, Close, 2900.51
        2019-08-21, Close, 2924.43
        ......
        Final Portfolio Value: 1000000.00 

- 怎么判断买点
    上一个策略只是打印了收盘价，既然价格信息可以获取，那么就可以对数据做一些逻辑处理，本小节以连涨3天就买入为例进行说明。
    ```py
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
            if self.dataclose[0] > self.dataclose[-1] and self.dataclose[-1] > self.dataclose[-2]:

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
    输出结果如下，日志记录了买入日期，买入标识，买入价格-收盘价。但我们不知道的是，发出的订单是否被准确执行了，如果执行了，是什么价格，买了多少，以及如何选择卖出时机呢？
        Starting Portfolio Value: 100000.00
        2019-08-29, BUY CREATE, 2924.58
        2019-08-30, BUY CREATE, 2926.46
        ......
        2020-08-07, BUY CREATE, 3351.28
        2020-08-10, BUY CREATE, 3360.47
        2020-08-18, BUY CREATE, 3389.78
        Final Portfolio Value: 109613.53 
- 获取订单状态，以及卖点的判断
    ```py
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
            # 3.4.1 检查是否有处理中的订单， 不再重复下
            if self.order:
                return

            # 3.4.2 检测是否已经有持仓
            if not self.position:

                # 3.4.3 没有持仓，判断是否满足买的条件
                if self.dataclose[0] > self.dataclose[-1] and self.dataclose[-1] > self.dataclose[-2]:

                    # 3.4.4 3连涨满足，下单，默认使用下一根K线的开盘价；
                    # self.log('BUY CREATE, %.2f' % self.dataclose[0])

                    # 3.4.5 跟踪订单
                    self.order = self.buy()

            else:

                # 3.4.6 已经有持仓，判断是否到达卖的条件
                if len(self) >= (self.bar_executed + 200.0):
                    # 3.4.7 持仓5天就卖
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
        todate=datetime.datetime(2020, 6, 20),
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
    输出结果如下,为了计算方便，我们买入后直接持有了200天，这样只有一个订单被执行，可以看到，系统默认只下了一笔委托,最终盈利为: 3101.64 - 2937.09 = 164.55
        Starting Portfolio Value: 1000000.00
        2019-08-30, BUY EXECUTED, 2937.09
        2020-06-18, SELL EXECUTED, 3101.64
        Final Portfolio Value: 1000164.55
- 利润的侵蚀者：佣金
    ```py
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
                if self.dataclose[0] > self.dataclose[-1] and self.dataclose[-1] > self.dataclose[-2]:

                    # 3.4.4 3连涨满足，下单，默认使用下一根K线的开盘价；
                    # self.log('BUY CREATE, %.2f' % self.dataclose[0])

                    # 3.4.5 跟踪订单
                    self.order = self.buy()

            else:

                # 3.4.6 已经有持仓，判断是否到达卖的条件
                if len(self) >= (self.bar_executed + 200.0):
                    # 3.4.7 持仓5天就卖
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
            
        # 3.6.1 交易完成输出盈利
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
    输出结果如下,相比上一节，增加了佣金，即Comm, 买卖均收取，极大侵蚀了利润。这里还没有添加印花税。
        Starting Portfolio Value: 1000000.00
        2019-08-30, BUY EXECUTED, Price: 2937.09, Cost: 2937.09, Comm 0.88
        2020-06-18, SELL EXECUTED, Price: 3101.64, Cost: 2937.09, Comm 0.93
        2020-06-18, OPERATION PROFIT, GROSS 164.55, NET 162.74
        Final Portfolio Value: 1000162.74
- 添加系统支持的一些指标
- 可视化