---
title: "Tcp笔记"
date: 2020-03-31T10:10:22+08:00
draft: false
categories: ["计算机网络"]
---



1. 以前使用telnet的时候, 经常是输入字符但是它不显示, 感觉输入无效. 今天看自顶向下, 才知道原来在telnet(基于TCP)客户端上, 用户输入了一个字符, 将会被传输到服务端, 然后服务端返回这个字符它才会回显. 而不是像ssh那样都会显示. 在linux下开一个`nc -l 6666`就可以测试. 

2. *RTT*(Round-Trip Time)，往返时延: 报文发出到收到确认之间的时间.

3. 超时重传会一直重传吗? 

    不会

4. 定时器的作用? 

    有很多种定时器

    1. 连接建立定时器: 建立连接时如果长时间没有响应, 将会以指数增长的时间重传尝试重连.
    2. 超时重传的定时器
    3. 延迟确认定时器: 延迟ACK的发送
    4. 坚持定时器(persist timer): 接收窗口为0时, 会定时地发送0字节探测包
    5. keepaliver timer: 当tcp两端长时间不发送数据时, 如何判断连接是否失效呢? 使用该定时器
    6. FIN_WAIT_2定时器: 防止对方一直不发送FIN包
    7. TIME_WAIT定时器

5. 防止SYN洪泛攻击(SYV flood attack): SYN cookie. 

6. 流量控制: 如果主机B的接收缓存满了, 接收窗口为0, 那么主机A将会继续发送一个字节数据的报文段. 因为如果不发送, 那么如果B的程序清空缓存, 它不会发送ack, A不会直到有接收空间了. 

7. nagle算法与延时确认的冲突: nagle算法只允许同时存在一个已发送但未确认的小包(小于MSS), 之后的小包会试图合并. 而延迟确认指的是当接收方接收到数据想要发送ACK的时候, 会延迟一段时间(比如40ms), 如果这段时间内有数据需要发送, 那么会一起发送. nagle和延时确认在这种特殊情况下, 会导致网络的延迟过高.

    nagle算法是时代的产物, 它是为了避免网络上有过多的小包而降低吞吐量. 现在很多程序都没有开启nagle算法, 因为现在的设施已经比以前好多了, 而无法接收过高的延迟.

    nagle算法的描述如下

    ```
    	if there is new data to send #有数据要发送
            # 发送窗口缓冲区和队列数据 >=mss，队列数据（available data）为原有的队列数据加上新到来的数据
            # 也就是说缓冲区数据超过mss大小，nagle算法尽可能发送足够大的数据包
            if the window size >= MSS and available data is >= MSS 
                send complete MSS segment now # 立即发送
            else
                if there is unconfirmed data still in the pipe # 前一次发送的包没有收到ack
                    # 将该包数据放入队列中，直到收到一个ack再发送缓冲区数据
                    enqueue data in the buffer until an acknowledge is received 
                else
                    send data immediately # 立即发送
                end if
            end if
        end if　
    ```

    即

    ```
    Nagle算法的实现规则：
    如果包长度达到MSS，则允许发送；
    如果该包含有FIN，则允许发送；
    设置了TCP_NODELAY选项，则允许发送；
    未设置TCP_CORK选项时，若所有发出去的小数据包（包长度小于MSS）均被确认，则允许发送；
    上述条件都未满足，但发生了超时（一般为200ms），则立即发送。
    ```

    