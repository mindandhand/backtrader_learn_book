Empyrical是一个金融风险指标库，可以用于计算和评估各种金融风险指标。它能够用于计算年平均回报、最大回撤、Alpha值、Beta值、卡尔马率、Omega率、夏普率等关键指标。 Empyrical接口简洁，计算效率非常高，能够快速处理大量数据并返回准确的结果。


下面结合akshare 对常见方法做说明。

##### 安装：
```
pip install empyrical

```


```python
from __future__ import print_function
from abc import ABCMeta, abstractmethod 
import datetime
import threading  
import os, os.path
import numpy as np 
import pandas as pd
import akshare as ak
import empyrical as em  
```

##### 准备数据


```python
## 使用Empyrical计算 
# 使用AkShare获取股票历史价格数据,以深圳平安银行为例  
stock_data = ak.stock_zh_a_hist(symbol="000001", period="daily", start_date="20230101", end_date='20231231', adjust="qfq")  
print(stock_data[:3])
# 提取收盘价数据  
close_prices = stock_data['收盘']  

index_data = ak.stock_zh_index_daily_em(symbol="sh000001", start_date="20230101", end_date="20231231")
print(index_data[:3])
index_close_prices = index_data['close']
# 计算日收益率  
returns = close_prices.pct_change().dropna()  
index_returns = index_close_prices.pct_change().dropna() 
```

               日期     开盘     收盘     最高     最低      成交量           成交额    振幅   涨跌幅  \
    0  2023-01-03  12.92  13.49  13.57  12.77  2194128  2.971547e+09  6.21  4.74   
    1  2023-01-04  13.43  14.04  14.14  13.35  2189683  3.110729e+09  5.86  4.08   
    2  2023-01-05  14.12  14.20  14.46  14.09  1665425  2.417272e+09  2.64  1.14   
    
        涨跌额   换手率  
    0  0.61  1.13  
    1  0.55  1.13  
    2  0.16  0.86  
             date     open    close     high      low     volume        amount
    0  2023-01-03  3087.51  3116.51  3119.86  3073.05  281370362  3.313921e+11
    1  2023-01-04  3117.57  3123.52  3129.09  3109.45  273313626  3.163912e+11
    2  2023-01-05  3132.76  3155.22  3159.43  3130.23  257003018  3.356359e+11


##### Empyrical 示例


```python
# jisauuuuuuuuuuuuuuuu
volatility = em.annual_volatility(returns)  

print(f"em计算股票 的历史波动率为: {volatility:.4f}")
```

    em计算股票 的历史波动率为: 0.2135



```python
max_drawdown = em.max_drawdown(returns) 
print(f"em计算股票 的最大回撤为: {max_drawdown:.4f}")
```

    em计算股票 的最大回撤为: -0.3927



```python
sharpe_ratio = em.sharpe_ratio(returns, risk_free=0, period='daily', annualization=None)
print(f"em计算股票 的夏普比为: {sharpe_ratio:.4f}")
```

    em计算股票 的夏普比为: -1.6672



```python
#em.beta?
beta = em.beta(returns, index_returns)
print(f"em计算股票 的beta为: {beta:.4f}")
```

    em计算股票 的beta为: 1.2533

