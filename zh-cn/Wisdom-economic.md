# 十一、经济模型
## 11.1 总量
&#160;&#160;&#160;&#160;&#160;&#160;总量恒定为5.9亿个WDC

|编号| 类型|说明|额度
|:----:|:----:|:----:|:----:
|1 | 主链共识|通过矿工奖励的形式发行|88000000
|2| 总孵化器初始余额|设置一个地址|35765823
|3|官方初始|30个地址|177000000
|4|用户初始|用户原孵化数据|289234177
|合计|590000000

## 11.2 释放曲线
&#160;&#160;&#160;&#160;&#160;&#160;公链初期出块时间为30秒，每个区块奖励20个WDC,每过2102400（约2年）个区块减少大约52.22%，直到区块奖励调整为6.66666666个WDC,所以除了第一次降息区块高度调整为5736000个区块后，后面的降息高度固定为6307200个区块。

![经济模型](../img/economic.png)

## 11.3  手续费
&#160;&#160;&#160;&#160;&#160;&#160;事务的手续费等于事务gas乘以gasPrice，不同事务类型的gas各不相同，乘以gasPrice等于手续费，手续费默认0.002个WDC，如果想多付手续费，只需要增加gasPrice，事务的gas表如下

|编号| 事务类型|说明
|:----:|:----:|:----:|
|1| coinbase|0Gas
|2| 普通转账|50000Gas
|3|投票|20000Gas
|4|存证|100000Gas
|5|部署合约|100000Gas
|6|调用合约|100000Gas
|7|申请孵化|100000Gas
|8|提取利息|100000Gas
|9|提取分享|100000Gas
|10|提取本金|100000Gas
|11|撤回投票|20000Gas
|12|抵押|20000Gas
|13|撤回抵押|20000Gas