# 计算机网络 自顶向下方法(原书第6版)
## 第一章
### 1.4 延时，损失，吞吐量
延时由四部分组成，processing, queuing, transmission, propagation组成，一般而言，processing的延时比较少，可以忽略不计。

- processing   里面主要是确定某个host的请求格式是否有效，然后根据地址确定发送给哪个接下来的router。
- queuing      是由于router可能先前的请求还没有结束，这个时间取决于线路的繁忙程度
- transmission 这个时间定义为 L/R 其中L代表这个包的长度，R代表传输速率，比如R = 100Mbps之类的
- propagation  这个延时定义为 d/s 其中d代表distance， s代表speed， 一般而言s比光速慢一点

<div align=center>  

![](./IMG/1-4-2-average_queuing_delay.PNG)

</div>

对于queuing delay,需要注意的是，La/R < 1，其中L是每个包的bit数，a是每秒中包的个数，R是路由传输数据的速率，可想而知，如果