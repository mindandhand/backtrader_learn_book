<!--
 * @Author: mindandhand 1639545667@qq.com
 * @Date: 2023-09-07 16:57:08
 * @LastEditors: mindandhand 1639545667@qq.com
 * @LastEditTime: 2023-09-14 16:35:05
 * @FilePath: /backtrader_learn_book/Chapter2/README.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
# Introduction

- 说明如何获取行情及backtrader使用行情的方式。
- 行情的获取：
    - 行情的获取有历史行情和实时行情两种，这里先对其进行初步的介绍，仅以跑通示例为目的，对于生产可用版本，是一门学问，逐步进行完善和补充。
    - 历史行情的获取。如腾讯、新浪等都提供了免费的行情接口，但是如果逐个去对接，不免耗费大量宝贵的精力，好在有[akshare](https://www.akshare.xyz/tutorial.html)、[baostock](http://baostock.com/baostock/index.php) 等开源项目对其做了汇总，不仅免费，对于学习探索也足够了，这里简单去调用即可。tushare[^1]、淘宝也提供了收费版本的行情，如果免费的不能满足需求，则可以考虑付费。如果上面的还无法满足需求，则可以考虑专业的平台，如通联、wind、Rice Quant等。
    - 实时行情。实时行情也可以通过以上方式获取，追求速度的话，CTP可以直接对接交易所，股票行情则要对接券商。这里用到的时候再详细讨论。

[^1]: 目前tushare的积分制度让人吐槽，还不如直接收年费，简单直接，这么乖乖绕绕，不还是为了收费。