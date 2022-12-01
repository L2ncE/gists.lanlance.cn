 ### 2. HTTP 版本详解：0.9、1.0、1.1、2.0

#### 不同版本的特性差异

![image-20221129165452162](https://picture.lanlance.cn/i/2022/11/29/6385c8dc96de7.png)

#### keep-alive 长连接

减少 TCP 连接与释放的成本。

![image-20221130163241956](https://picture.lanlance.cn/i/2022/11/30/6387152a67391.png)

#### HTTP/2.0

##### 多路复用

![image-20221130163635668](https://picture.lanlance.cn/i/2022/11/30/6387161414a00.png)

##### 头部压缩

用一个相同的静态字典进行映射来维护，用哈夫曼编码进行内容压缩。

![image-20221130163754466](https://picture.lanlance.cn/i/2022/11/30/63871662e4aa2.png)

##### 服务端推送

![image-20221130164012956](https://picture.lanlance.cn/i/2022/11/30/638716ed5d2bc.png)

#### 学习目标

![image-20221130164032225](https://picture.lanlance.cn/i/2022/11/30/6387170085f39.png)

#### 面试题

![image-20221130164059754](https://picture.lanlance.cn/i/2022/11/30/6387171c1129e.png)