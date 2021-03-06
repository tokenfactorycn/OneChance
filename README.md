# OneChance
基于以太坊的一元夺宝实现
该项目由三个智能合约组成


## OneChanceCoin-活动代币合约

OneChanceCoin合约实现了以太坊的数字货币规范，用户可以使用以太坊钱包添加代币合约，实现对活动代币的管理

余额查询、用户间转账等功能都可以在图形界面的Mist应用中轻易实现

设计中OneChanceCoin代币与人民币是1:1兑换，没有汇率波动，代币的发行不依赖于自动的代码，而是由活动举办方负责发行代币

举办方应该提供一个服务网站提供代币兑换，用户在网站中填写自己的以太坊账号地址，并支付人民币，举办方网站后台收到支付结果后向用户的以太坊账户发送对应数额代币

代币是可以被消耗的，OneChance合约是唯一有权扣减用户代币(用户之间转移代币不算扣减)的实体

用户在OneChance合约中提交购买请求，OneChane合约自动扣减用户相应金额的代币(即用户用活动代币兑换了奖品的中奖Chance)

之所以要使用代币而不是人民币直接购买Chance，是因为合约要处理人民币的支付结果太过复杂，用ChanceCoin代替人民币兑换Chance，简化了OneChance合约的业务逻辑，而ChanceCoin和人民币的兑换则交由主办方的服务网站负责


## OneChance-一元夺宝活动合约

OneChance合约实现了一元夺宝活动的奖品发布、用户购买奖品中奖Chance、自动计算中奖用户功能

只有主办方有权发布奖品信息，奖品信息发布后，用户可以查询奖品信息，在兑换活动代币后，可以用代币兑换奖品的中奖Chance

用户提交购买Chance请求后，合约自动调用OneChanceCoin合约扣减用户代币，然后将用户加入奖品购买用户列表中

在所有Chance都卖完后，合约计算中奖用户，用户可以注册event来接收中奖通知，也可以主动查询中奖用户结果

合约并不会自动给用户发放奖品，用户信息中也没有中奖者的收货人姓名地址联系方式等隐私数据

设计中用户在中奖后，可以用私钥对用户的个人信息签名后提交到主办方服务网站，主办方验证用户签名后负责发货，可以保证用户个人隐私

奖品id从1开始递增，如果最新奖品id是0说明主办方还没有发布过奖品

购买用户id从1开始递增，如果中奖用户id值为0说明中奖用户还没有计算出

中奖用户的计算，在区块链上获得一个公平的真随机数是一个很大的难题，开始考虑的randao项目

但是对于Randao项目有一个疑问

如果是个人使用Randao项目生成一个随机数，sha3(随机数种子)在提交到Randao合约时对应的随机数可以在链下保存，等到种子披露阶段再上链，这没有问题

但是如果是合约调用Randao项目，在提交sha3(随机数种子)到Randao合约时，种子明文如何保存，保存在链上的话，不等种子披露阶段就已经暴露了

本合约参考Randao项目，在用户购买Chance时需要一同提交一个sha3(随机数种子)，在奖品Chance售罄后，合约通知所有购买用户提交种子明文

等所有用户的种子明文提交后，使用 sum(所有用户随机数种子)%amt+1 算法计算中奖用户

目前随机数种子限制为整数，而且大小限制在9223372036854775807以内(受限于web3.js的toHex方法精度)

后续优化可以考虑引入慢hash算法、增大随机数范围(更换为更大长度的字符串或者byte数组)，以避免快速查表攻击等

合约不记录用户已购买过的商品(遍历所有商品的购买用户列表也可以查到)，请用户自行记录自己的购买记录(可以考虑在提供给用户使用的本地应用或web页面中实现)


## AddressCompress-地址压缩合约

该合约并不仅限于为OneChance合约服务，可以作为基础设施为全网用户(包括合约)提供20字节address转为4字节uid的地址压缩服务

类似于一元夺宝合约，合约场景需要存储大量重复地址数据时(每个奖品都要存储一个商品价格长度的地址列表，而且重复地址的频率非常高，高价值奖品一般用户一次买多个Chance)

使用地址压缩，对于重复地址数据，只需要存储一份20字节address与4字节uid的双向映射，然后存储4字节uid即可，可以节省相当的存储空间


## 合约演示

请参见html-test和geth-test目录下的README文件
