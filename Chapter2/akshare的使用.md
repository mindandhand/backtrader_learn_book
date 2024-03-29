# akshare的使用
#### 在功能调试阶段，使用akshre作为行情的主要获取来源，其[官方文档](https://www.akshare.xyz/index.html )介绍十分详细，这里仅简单说明。
##### 安装：pip install akshare -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host=mirrors.aliyun.com  --upgrade


```python
import akshare as ak
import mplfinance as mpf

### 股票部分
##### 使用akshare 获取上证指数历史行情
stock_zh_index_daily_df = ak.stock_zh_index_daily_em(symbol="sh000001", start_date="20220101", end_date="20230101")
print(stock_zh_index_daily_df[:3])

```

             date     open    close     high      low     volume        amount
    0  2022-01-04  3649.15  3632.33  3651.89  3610.09  405027768  5.102511e+11
    1  2022-01-05  3628.26  3595.18  3628.26  3583.47  423902029  5.389636e+11
    2  2022-01-06  3581.22  3586.08  3594.49  3559.88  371540544  4.742843e+11



```python
##### 使用akshare 获取股票基础信息
stock_individual_info_em_df = ak.stock_individual_info_em(symbol="000001")
print("000001 基础信息如下：")
print(stock_individual_info_em_df)
```

    000001 基础信息如下：
       item                value
    0   总市值  217734402181.559998
    1  流通市值       217730236779.0
    2    行业                   银行
    3  上市时间             19910403
    4  股票代码               000001
    5  股票简称                 平安银行
    6   总股本        19405918198.0
    7   流通股        19405546950.0



```python
##### 获取分红配股数据
##### 获取分红,只打印前3条
stock_history_dividend_detail_df = ak.stock_history_dividend_detail(symbol="000001", indicator="分红")
print(stock_history_dividend_detail_df[:3])
##### 获取配股,只打印前3条
stock_history_dividend_detail_df = ak.stock_history_dividend_detail(symbol="000001", indicator="配股")
print(stock_history_dividend_detail_df[:3])
```

             公告日期   送股  转增    派息  进度       除权除息日       股权登记日 红股上市日
    0  2023-06-07  0.0   0  2.85  实施  2023-06-14  2023-06-13   NaT
    1  2022-07-15  0.0   0  2.28  实施  2022-07-22  2022-07-21   NaT
    2  2021-05-07  0.0   0  1.80  实施  2021-05-14  2021-05-13   NaT
             公告日期  配股方案  配股价格        基准股本         除权日       股权登记日       缴款起始日  \
    0  2000-10-23     3     8  1551850000  2000-11-06  2000-11-03  2000-11-07   
    1  1994-01-09     1     5   404127000  1994-07-09  1994-07-08  1994-07-28   
    2  1993-05-07     1    16           0  1993-05-24  1993-05-21  1993-06-14   
    
            缴款终止日       配股上市日  募集资金合计  
    0  2000-11-20  2000-12-08     NaN  
    1  1994-08-10  1994-08-22     NaN  
    2  1993-07-02  1993-07-05     NaN  



```python
##### 使用akshare 获取股票历史行情，adjust	str	默认返回不复权的数据; qfq: 返回前复权后的数据; hfq: 返回后复权后的数据
stock_zh_a_hist_df = ak.stock_zh_a_hist(symbol="000001", period="daily", start_date="20230301", end_date='20230907', adjust="")
print(stock_zh_a_hist_df[:3])
```

               日期     开盘     收盘     最高     最低      成交量           成交额    振幅   涨跌幅  \
    0  2023-03-01  13.80  14.17  14.19  13.74  1223452  1.719711e+09  3.27  2.83   
    1  2023-03-02  14.13  14.24  14.44  14.06  1015877  1.447566e+09  2.68  0.49   
    2  2023-03-03  14.35  14.29  14.37  14.14   690954  9.855521e+08  1.62  0.35   
    
        涨跌额   换手率  
    0  0.39  0.63  
    1  0.07  0.52  
    2  0.05  0.36  



```python
##### 获取股票实时5档
stock_bid_ask_em_df = ak.stock_bid_ask_em(symbol="000001")
print(stock_bid_ask_em_df)
```

              item       value
    0       sell_5       11.27
    1   sell_5_vol   308400.00
    2       sell_4       11.26
    3   sell_4_vol   252600.00
    4       sell_3       11.25
    5   sell_3_vol   535100.00
    6       sell_2       11.24
    7   sell_2_vol   142000.00
    8       sell_1       11.23
    9   sell_1_vol    86400.00
    10       buy_1       11.22
    11   buy_1_vol   812900.00
    12       buy_2       11.21
    13   buy_2_vol  1374300.00
    14       buy_3       11.20
    15   buy_3_vol  2204900.00
    16       buy_4       11.19
    17   buy_4_vol   522600.00
    18       buy_5       11.18
    19   buy_5_vol   573500.00



```python
##### 获取新股板块代码
stock_zh_a_new_em_df = ak.stock_zh_a_new_em()
print(stock_zh_a_new_em_df)
```

          序号      代码     名称    最新价    涨跌幅    涨跌额     成交量           成交额     振幅  \
    0      1  301550    N斯菱  55.08  46.65  17.52  196087  1.119802e+09  21.06   
    1      2  688702  C盛科-U  57.80  -7.52  -4.70  168659  9.880806e+08   9.42   
    2      3  603075    C热威  29.23  -9.22  -2.97  140686  4.280198e+08   9.07   
    3      4  301529    C福赛  55.01 -11.03  -6.82  105565  6.015911e+08  10.08   
    4      5  688549  中巨芯-U  10.53  -6.81  -0.77  656671  7.082470e+08   5.49   
    ..   ...     ...    ...    ...    ...    ...     ...           ...    ...   
    277  278  688137   近岸蛋白  50.55   3.59   1.75    5411  2.728646e+07   5.66   
    278  279  301319    唯特偶  53.62  -1.61  -0.88    6489  3.529969e+07   4.46   
    279  280  301285    鸿日达  14.20  -2.94  -0.43   42960  6.182806e+07   3.69   
    280  281  301176   逸豪新材  16.62  -0.18  -0.03    9036  1.504612e+07   1.74   
    281  282  688252    天德钰  19.42  -0.31  -0.06    6979  1.358915e+07   4.06   
    
            最高     最低     今开     昨收    量比    换手率  市盈率-动态   市净率  
    0    62.88  54.97  55.00  37.56   NaN  75.18   46.02  4.01  
    1    62.00  56.11  58.94  62.50  0.65  45.34  334.17  9.84  
    2    32.14  29.22  32.10  32.20  0.69  36.26   49.42  6.30  
    3    61.20  54.97  60.50  61.83  0.82  55.02   58.15  3.94  
    4    11.14  10.52  11.10  11.30  0.54  25.10  409.01  5.14  
    ..     ...    ...    ...    ...   ...    ...     ...   ...  
    277  51.35  48.59  48.78  48.80  1.90   3.17  143.50  1.63  
    278  55.78  53.35  55.50  54.50  0.57   4.43   30.21  2.90  
    279  14.70  14.16  14.57  14.63  0.43   8.31  349.60  2.82  
    280  16.82  16.53  16.68  16.65  1.09   2.22 -126.56  1.72  
    281  19.94  19.15  19.65  19.48  0.56   1.87   85.17  4.24  
    
    [282 rows x 17 columns]



```python
##### 获取次新股集合
stock_zh_a_new_df = ak.stock_zh_a_new()
print(stock_zh_a_new_df[:3])
```

         symbol    code  name   open   high    low    volume     amount  \
    0  sh603075  603075   C热威  32.10  32.14  29.22  14068649  428019844   
    1  sh603119  603119  浙江荣泰  23.14  23.86  22.83   4915610  115500414   
    2  sh603270  603270  金帝股份  33.90  34.45  32.82   8822563  294836838   
    
             mktcap  turnoverratio  
    0  1.169229e+06       36.25550  
    1  6.608000e+05        7.16611  
    2  7.318163e+05       17.99072  



```python
##### 获取股票实时行情数据
##### ？怎么长期获取少数几个，或者指定集合股票？
stock_zh_a_spot_em_df = ak.stock_zh_a_spot_em()
print(stock_zh_a_spot_em_df[:5])
```

       序号      代码    名称    最新价    涨跌幅    涨跌额       成交量           成交额     振幅  \
    0   1  301550   N斯菱  55.08  46.65  17.52  196087.0  1.119802e+09  21.06   
    1   2  300006  莱美药业   4.18  20.11   0.70  933184.0  3.748708e+08  18.97   
    2   3  300909   汇创达  30.05  20.01   5.01  221093.0  6.503694e+08  13.38   
    3   4  300667  必创科技  19.20  20.00   3.20  220237.0  3.966179e+08  20.50   
    4   5  832651  天罡股份  15.79  16.70   2.26   21812.0  3.273053e+07  20.77   
    
          最高  ...    量比    换手率  市盈率-动态   市净率           总市值          流通市值    涨速  \
    0  62.88  ...   NaN  75.18   46.02  4.01  6.058800e+09  1.436590e+09 -0.16   
    1   4.18  ...  8.20  11.49  -40.82  2.13  4.413709e+09  3.394799e+09  0.00   
    2  30.05  ...  3.63  36.49  104.02  2.97  5.000054e+09  1.820833e+09  0.00   
    3  19.20  ...  6.68  13.52  135.93  3.11  3.893086e+09  3.127296e+09  0.00   
    4  16.49  ...  2.22  15.20   19.06  2.26  9.631900e+08  2.265131e+08 -0.32   
    
       5分钟涨跌  60日涨跌幅  年初至今涨跌幅  
    0  -0.97   46.65    46.65  
    1   0.00   24.04     9.71  
    2   0.00   17.47     0.84  
    3   0.00   -7.51    26.57  
    4   0.57   12.79    12.79  
    
    [5 rows x 23 columns]



```python
### 期货合约

##### 使用akshare 获取shfe 全部主力合约
print(ak.match_main_contract(symbol="shfe"))
```

    FU2311
    SC2310
    AL2310
    RU2401
    ZN2310
    CU2310
    AU2312
    RB2401
    WR2310
    PB2310
    AG2312
    BU2311
    HC2401
    SN2310
    NI2310
    SP2401
    NR2311
    SS2311
    LU2312
    BC2401
    AO2311
    BR2401
    EC2404
    shfe主力合约获取成功
    FU2311,SC2310,AL2310,RU2401,ZN2310,CU2310,AU2312,RB2401,WR2310,PB2310,AG2312,BU2311,HC2401,SN2310,NI2310,SP2401,NR2311,SS2311,LU2312,BC2401,AO2311,BR2401,EC2404



```python
##### 使用akshare 获取螺纹钢主力合约历史行情
futures_zh_daily_sina_df = ak.futures_zh_daily_sina(symbol="RB2401")
print(futures_zh_daily_sina_df[:3])
```

             date    open    high     low   close  volume  hold  settle
    0  2023-01-17  4001.0  4027.0  3973.0  4011.0    1038   573  3996.0
    1  2023-01-18  4027.0  4051.0  4014.0  4037.0     314   713  4036.0
    2  2023-01-19  4043.0  4085.0  4043.0  4080.0     352   821  4066.0



```python
##### 使用akshare 获取螺纹钢连续合约历史行情
futures_main_sina_hist = ak.futures_main_sina(symbol="RB0", start_date="20230101", end_date="20230901")
print(futures_main_sina_hist[:3])
```

               日期   开盘价   最高价   最低价   收盘价      成交量      持仓量  动态结算价
    0  2023-01-03  4100  4100  4016  4063  1245821  1882723   4044
    1  2023-01-04  4061  4084  4010  4027  1383006  1874643   4045
    2  2023-01-05  4015  4024  3970  4017  1522301  1825776   4001


akshare的数据非常丰富，不一一列举，用到的时候再进行补充。
