# 1.1 快速运行一个demo 
---
- 先跑起来
    ```
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