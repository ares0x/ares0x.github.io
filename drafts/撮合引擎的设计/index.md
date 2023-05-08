# 撮合引擎的设计


在交易系统中，撮合引擎是一个核心的组件，它需要承接海量的交易
架构设计没有银弹。对于一个交易系统，在交易和撮合这个步骤就已经出现了分歧。
可以选择将交易和撮合耦合在一起，这样用户下单以后就可以直接进入撮合逻辑
将加以和撮合拆分，最大程度上保证功能的独立，后续扩展也会相对更方便一些。
两种方案的逻辑调用如下

![耦合](./../../static/images/engine/split.png)

![拆分](./../../static/images/engine/coupling.png)

![Binance 订单簿](./../../static/images/engine/Binance-orderbook.png)
![OKX 订单簿](./../../static/images/engine/Okx-orderbook.png)



