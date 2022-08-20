[TOC]

# 一. TCP流量控制

## <span id="jump">1.1 定义</span>

> 避免分组丢失，控制发送者的发送速度，使得接收者来得及接收



## 1.2 实现

接收方返回的 ACK 中会包含自己的接收窗口的大小，并且利用大小来控制发送方的数据发送(滑动窗口协议)



**避免死锁**：每当发送者收到一个零窗口的应答后就启动该计时器，时间一到便主动发送报文询问接收者的窗口大小。若接收者仍然返回零窗口，则重置该计时器继续等待；若窗口不为0，则表示应答报文丢失了，此时重置发送窗口后开始发送，这样就避免了死锁的产生



# 二. TCP拥塞控制

> 在某段时间，若**对网络中某一资源的需求超过了该资源所能提供的可用部分，网络性能就要变坏**，这种情况就叫做**网络拥塞**



**拥塞避免：**
![TCP的拥塞控制](https://raw.githubusercontent.com/fyyuestc/Images/main/img/202205062118940.png)

**快速重传：**
![](https://raw.githubusercontent.com/fyyuestc/Images/main/img/202205062121065.png)
![](https://raw.githubusercontent.com/fyyuestc/Images/main/img/202205062121537.png)

# 三. 区别

> 拥塞控制是作用于网络的、流量控制是作用于接收者的

























[link](#jump)

