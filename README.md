# 介绍
backtrader, 是一个功能丰富的Python框架，可用于回测和交易,代码开源在github上([点击此处访问](https://github.com/mementum/backtrader))。按官网的说法，可以使交易者更专注于编写可重用的交易策略，指标和分析器，而不必花费时间来构建基础结构。但是考虑到国内的种种限制，很多基础工作还是要自己来做，后续文章会记录具体的问题及尝试的解决方案。

python开源回测框架还有zipline、vnpy等，量化平台有Quantopian、聚宽等。回测方法千千万，适合自己的才是最好的。自己之所以选择backtrader, 是因为它安装简单，很方便集成机器学习、神经网络, 也可以兼容Qlib，很适合个人进行研究，而且国际上也有个人和机构在生产中使用。但分析出可行策略，真枪实弹部署个人还是偏向cpp来实现，而把backtrader作为思考分析工具[^1]。

资料主要来源：<br>
[backtrader官网](https://www.backtrader.com/)<br>
[successful-algorithmic-trading-ebook](https://www.quantstart.com/successful-algorithmic-trading-ebook/)<br>
<br>

![png](qrcode.png)

[^1]: 内心真正想实现的，其实还是属于算法交易范畴，通过对历史数据或者某些经典书籍等等的研习, 希望找到一些似有似无的可循踪迹，亦或说是模式，利用历史数据建立模型，进行优化，在实盘中验证完善，并不断循环这个过程。盯盘无疑是浪费精力的，而且未必取得良好结果，也是对时间的一种浪费。算法交易不仅可以释放人力，让我们把有限的精力放到更有价值的事情上，而且可以智能盯盘，自动下单，减少心里因素不稳定造成的非理性决策。 臆想总是美好的，且行且珍惜吧。