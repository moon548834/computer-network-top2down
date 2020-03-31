# 网络层：控制部分

## 5.1 简介

有两种可以控制转发表/流表，一种是每个router都有自身的控制器，另一种是通过远程的控制器。对于第一种来说，每一个router都有一个路由部分，可以用来和其他的router进行通信，从而确定转发表的数值。对于第二种来说，每个router里面有一个CA(control agent)，不过这个CA主要是用来和远程的控制器通信的，而且并不和其他的router中的CA通信。

## 5.2 路由算法

![](./IMG/chap5/5-2-1_graph.PNG)

算法一：中心化算法——LS算法，需要获得所有节点和所有的边(权值)

算法二：去中心化算法——DV算法，没有一个节点拥有所有的路径信息。

第二种分类方法是根据算法是否是动态/静态，动态算法对路由成环/震荡更加敏感一些。第三种分类方法是根据是否负载敏感划分。

### 5.2.1 LS算法

LS算法需要拥有整个网络的节点和边，这通常是通过节点给所有网络中其他的节点发送信息完成的。实际中，这称作LS广播算法。LS使用的是Dijkstra算法
```
// 从u开始找
N' = {u}
for all nodes
    if v is neighbor of u:
        then D(v) = c(u, v);
        // c(x,y)表示从x到y的距离，如果不存在这样一条路径，则是∞。
    else
        D(v) = INF
Loop
    find w not in N' that D(w) is min
    N' += w
    update D(v) for w's neighbor
        D(v) = min(D(v), D(w) + c(w, v))
until N' = all node

```
**振荡**:

![](./IMG/chap5/5-2-2_oscillations1.PNG) 
![](./IMG/chap5/5-2-3_oscillations2.PNG)


![](./IMG/chap5/5-2-4_oscillations3.PNG) 
![](./IMG/chap5/5-2-5_oscillations4.PNG)

考虑y->w的路径，选择是顺时针方向，因为1 < 1 + e，此时y进过z到w，同理x也选择更小的路径：x->y->z->w。现在都是顺时针了，然后此时逆时针路径空了，就又都变为逆时针了，如此往复。

一个可行的解决方法是并不是所有的路由都同时运行LS算法。不过研究者发现最终router会自己同步，所以即便他们在不同的时刻(有相同的周期)运行这个算法，振荡问题并不能避免。一个解决的方法是(随机化通知的时间周期)大概是这样，就是(让周期不一样个人理解)。

### 5.2.2 DV算法

DV算法，每个节点仅维护有限的信息:当前节点和所有邻居的cost以及，从邻居发送过来的信息
```
// 
if y is neighbor of x: Dx(y) = c(x,y) 
else Dx(y) = INF
Boardcast neight
loop
    wait until one path available
    for each y in N:
        Dx(y) = min(c(x,v) + D(v,y), Dx(y))
    if change boardcast; 
```

![](./IMG/chap5/5-2-6_DV.PNG)

对于DV算法来说，cost变小，在网络中会被很快地传播，但是cost变大的传播速度会非常慢。一种解决的方法是posioned reverse，如果z通过y到达x，那么z会告诉y，z到x的距离是无穷大

**LS和DV的比较**

空间：LS需要O(NE)的信息，而DV只需要neighbor的。
时间：LS是O(N^2)的, DV是不确定的因为可能存在循环的问题，也可能会有count-to-infinity的问题。
健壮: LS只会算自己节点的信息，而不会影响其他的，所以还可以；而DV则会扩散错误，因为每个周期节点都会向他的邻居广播新的cost。