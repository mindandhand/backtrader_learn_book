# 1.1 快速运行几个demo 
---
##### 使用backtrader进行回测分析包含以下几个步骤：
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


```python

```
