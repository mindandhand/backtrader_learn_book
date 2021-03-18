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

        # 3.4 添加策略逻辑的地方，添加数据后，遍历每个日期进行调用，当前在的时间点坐标为0，其前面一根或者之前一个交易日的索引为-1，依次类推
        def next(self):
            # Simply log the closing price of the series from the reference
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
        ……
        Final Portfolio Value: 1000000.00 

    