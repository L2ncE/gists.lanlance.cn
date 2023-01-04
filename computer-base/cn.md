## 计算机网络

### 1. OSI 七层模型详解

#### 为什么要分层

1. 保证数据通路顺畅
2. 确定目标计算机状态
3. 识别目标计算机
4. 数据错误勘误

#### OSI 七层模型

![image-20221128171833073](https://picture.lanlance.cn/i/2022/11/28/63847ceaa7cf4.png)

并不是很重要，现在基本都以 TCP/IP 四层模型替代。

#### TCP/IP 四层模型

##### 与 OSI 七层模型的映射关系

![image-20221128172044342](https://picture.lanlance.cn/i/2022/11/28/63847d6cc7d6b.png)

##### 报文结构

![image-20221128172304224](https://picture.lanlance.cn/i/2022/11/28/63847df8878b5.png)

#### 网络层

![image-20221128172524547](https://picture.lanlance.cn/i/2022/11/28/63847e84cb278.png)

#### 传输层

![image-20221128172656165](https://picture.lanlance.cn/i/2022/11/28/63847ee06f04f.png)

#### 应用层

![image-20221128172954399](https://picture.lanlance.cn/i/2022/11/28/63847f92a961c.png)

#### 总结

![image-20221128173025434](https://picture.lanlance.cn/i/2022/11/28/63847fb1b3c6f.png)

#### 学习目标

![image-20221128173132178](https://picture.lanlance.cn/i/2022/11/28/63847ff474d10.png)

#### 面试题

![image-20221128173149696](https://picture.lanlance.cn/i/2022/11/28/63848005ee3de.png)

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

### 3. HTTP 报文结构、请求方法与状态码详解

#### HTTP 报文结构

![image-20221130164830564](https://picture.lanlance.cn/i/2022/11/30/638718df09748.png)

![image-20221130164923312](https://picture.lanlance.cn/i/2022/11/30/63871913a8024.png)

![image-20221130165023700](https://picture.lanlance.cn/i/2022/11/30/638719500f613.png)

#### HTTP 请求方法

![image-20221130165147847](https://picture.lanlance.cn/i/2022/11/30/638719a435bf7.png)

![image-20221130165157438](https://picture.lanlance.cn/i/2022/11/30/638719adc7cfb.png)

![image-20221130165341616](https://picture.lanlance.cn/i/2022/11/30/63871a15e9d06.png)

![image-20221130165437125](https://picture.lanlance.cn/i/2022/11/30/63871a4d70b2d.png)

##### 幂等

![image-20221130165509819](https://picture.lanlance.cn/i/2022/11/30/63871a6e1eac4.png)

#### HTTP 状态码

![image-20221130165830258](https://picture.lanlance.cn/i/2022/11/30/63871b3688c16.png)

![image-20221130165853303](https://picture.lanlance.cn/i/2022/11/30/63871b4d9db78.png)

![image-20221130165949751](https://picture.lanlance.cn/i/2022/11/30/63871b860c4e9.png)

![image-20221130170008698](https://picture.lanlance.cn/i/2022/11/30/63871b9908760.png)

![image-20221130170057893](https://picture.lanlance.cn/i/2022/11/30/63871bca45f66.png)

![image-20221130170559247](https://picture.lanlance.cn/i/2022/11/30/63871cf78bc1e.png)

#### 学习目标

![image-20221130171210010](https://picture.lanlance.cn/i/2022/11/30/63871e6a48f4f.png)

#### 面试题

![image-20221130171134235](https://picture.lanlance.cn/i/2022/11/30/63871e4698d98.png)

### 4. 安全传输的基础：加密

#### 安全传输模型

![image-20221201204427637](https://picture.lanlance.cn/i/2022/12/01/6388a1ac05369.png)

![image-20221201210158013](https://picture.lanlance.cn/i/2022/12/01/6388a5c6507db.png)

#### 对称加密

![image-20221201210552593](https://picture.lanlance.cn/i/2022/12/01/6388a6b0d81f4.png)

![image-20221201210618052](https://picture.lanlance.cn/i/2022/12/01/6388a6ca4805f.png)

#### 非对称加密

![image-20221201210648108](https://picture.lanlance.cn/i/2022/12/01/6388a6e85a5dd.png)

![image-20221201210703217](https://picture.lanlance.cn/i/2022/12/01/6388a6f77282d.png)

![image-20221201211018126](https://picture.lanlance.cn/i/2022/12/01/6388a7ba629d4.png)

#### 对比

![image-20221201211941617](https://picture.lanlance.cn/i/2022/12/01/6388a9ee02c37.png)

#### 散列算法

![image-20221201212055006](https://picture.lanlance.cn/i/2022/12/01/6388aa3762403.png)

#### 散列算法安全性

![image-20221201212537085](https://picture.lanlance.cn/i/2022/12/01/6388ab516452e.png)

#### 学习目标

![](https://picture.lanlance.cn/i/2022/12/01/6388ab6848dfc.png)

#### 面试题

![image-20221201212614508](https://picture.lanlance.cn/i/2022/12/01/6388ab76b84c0.png)

### 5. HTTPS 安全基础 —— TLS 技术详解

#### HTTP 与 HTTPS

![image-20221202142122374](https://picture.lanlance.cn/i/2022/12/02/63899962f0f4d.png)

![image-20221202142155595](https://picture.lanlance.cn/i/2022/12/02/63899983eb144.png)

#### TLS

![image-20221202142230514](https://picture.lanlance.cn/i/2022/12/02/638999a6d8c04.png)

#### 数字证书

![image-20221202142316376](https://picture.lanlance.cn/i/2022/12/02/638999d4b2b64.png)

![image-20221202142351126](https://picture.lanlance.cn/i/2022/12/02/638999f77acef.png)

> 使用非对称加密算法来生成对称秘钥

#### HTTPS 建立连接的过程

![image-20221202142510078](https://picture.lanlance.cn/i/2022/12/02/63899a46685fb.png)

#### SSL 安全参数握手过程

![image-20221202142705993](https://picture.lanlance.cn/i/2022/12/02/63899aba568f3.png)

![image-20221202142746734](https://picture.lanlance.cn/i/2022/12/02/63899ae31820b.png)

#### 学习目标

![image-20221202142902023](https://picture.lanlance.cn/i/2022/12/02/63899b2e5b092.png)

#### 面试题

![image-20221202142912600](https://picture.lanlance.cn/i/2022/12/02/63899b38e4a7c.png)

### 6. DNS 服务详解

#### DNS 是什么

![image-20221202143412991](https://picture.lanlance.cn/i/2022/12/02/63899c656714b.png)

#### DNS 工作原理

##### 域名

![image-20221202143523112](https://picture.lanlance.cn/i/2022/12/02/63899cab7ba44.png)

![image-20221202143612192](https://picture.lanlance.cn/i/2022/12/02/63899cdc9a682.png)

![image-20221202143713159](https://picture.lanlance.cn/i/2022/12/02/63899d197cd82.png)

##### 原理

![image-20221202143946084](https://picture.lanlance.cn/i/2022/12/02/63899db279418.png)

#### 学习目标

![image-20221202144040173](https://picture.lanlance.cn/i/2022/12/02/63899de87464c.png)

#### 面试题

![image-20221202144052938](https://picture.lanlance.cn/i/2022/12/02/63899df544543.png)

### 7. DNS 安全

#### 现象

![image-20221202144400534](https://picture.lanlance.cn/i/2022/12/02/63899eb0cf01a.png)

![image-20221202144433234](https://picture.lanlance.cn/i/2022/12/02/63899ed18c7f4.png)

#### DNS 劫持

![image-20221202144809043](https://picture.lanlance.cn/i/2022/12/02/63899fa96d9ea.png)

#### DNS 欺骗

![image-20221202145119504](https://picture.lanlance.cn/i/2022/12/02/6389a067dd8a9.png)

#### DDos 攻击

![image-20221202145337367](https://picture.lanlance.cn/i/2022/12/02/6389a0f1bbbf7.png)

#### 面试题

![image-20221202145434667](https://picture.lanlance.cn/i/2022/12/02/6389a12b10671.png)

### 8. TCP/UDP 协议详解

#### 端口

![image-20221206084303840](https://picture.lanlance.cn/i/2022/12/06/638e9017f0e49.png)

#### UDP 协议

![image-20221206084453311](https://picture.lanlance.cn/i/2022/12/06/638e908547768.png)

#### TCP 协议

![image-20221206084720861](https://picture.lanlance.cn/i/2022/12/06/638e9118c7138.png)

![image-20221206084848519](https://picture.lanlance.cn/i/2022/12/06/638e91708303b.png)

![image-20221206084910245](https://picture.lanlance.cn/i/2022/12/06/638e91863c3bd.png)

![image-20221206085128686](https://picture.lanlance.cn/i/2022/12/06/638e9210cec2a.png)

![image-20221206085240489](https://picture.lanlance.cn/i/2022/12/06/638e925877479.png)

#### UDP 与 TCP

![image-20221206085326651](https://picture.lanlance.cn/i/2022/12/06/638e9286a2224.png)

![image-20221206085520038](https://picture.lanlance.cn/i/2022/12/06/638e92f7e8967.png)

![image-20221206085707512](https://picture.lanlance.cn/i/2022/12/06/638e93638320d.png)

![image-20221206085826906](https://picture.lanlance.cn/i/2022/12/06/638e93b333f8d.png)

#### 学习目标

![image-20221206085904618](https://picture.lanlance.cn/i/2022/12/06/638e93d87929f.png)

#### 面试题

![image-20221206085919793](https://picture.lanlance.cn/i/2022/12/06/638e93e7a6050.png)

### 9. TCP 三次握手（建立过程）

#### 过程

![image-20221206090447092](https://picture.lanlance.cn/i/2022/12/06/638e952f1c116.png)

![image-20221206090516152](https://picture.lanlance.cn/i/2022/12/06/638e954c127f7.png)

#### 异常情况

![image-20221206091338041](https://picture.lanlance.cn/i/2022/12/06/638e97420b235.png)

#### 学习目标

![image-20221206091442737](https://picture.lanlance.cn/i/2022/12/06/638e978296be8.png)

#### 面试题

![image-20221206091425995](https://picture.lanlance.cn/i/2022/12/06/638e9771d7e00.png)

### 10. TCP 四次挥手（释放过程）

#### 过程

![image-20221206091735783](https://picture.lanlance.cn/i/2022/12/06/638e982fc8317.png)

![image-20221206092436262](https://picture.lanlance.cn/i/2022/12/06/638e99d445a53.png)

#### TIME-WAIT

![image-20221206092538369](https://picture.lanlance.cn/i/2022/12/06/638e9a1243666.png)

![image-20221206092747220](https://picture.lanlance.cn/i/2022/12/06/638e9a9380022.png)

#### 学习目标

![image-20221206092817219](https://picture.lanlance.cn/i/2022/12/06/638e9ab119162.png)

#### 面试题

![image-20221206092831373](https://picture.lanlance.cn/i/2022/12/06/638e9abf3e985.png)

### 11. TCP 的可靠传输（滑动窗口）

#### 停止-等待协议

![image-20221206093253224](https://picture.lanlance.cn/i/2022/12/06/638e9bc57b133.png)

#### 滑动窗口

![image-20221206093627316](https://picture.lanlance.cn/i/2022/12/06/638e9c9b4b3f2.png)

![image-20221206094557163](https://picture.lanlance.cn/i/2022/12/06/638e9ed516903.png)

![image-20221206094649627](https://picture.lanlance.cn/i/2022/12/06/638e9f0983587.png)

#### 学习目标

![image-20221206094820205](https://picture.lanlance.cn/i/2022/12/06/638e9f640e871.png)

#### 面试题

![image-20221206094803238](https://picture.lanlance.cn/i/2022/12/06/638e9f5322cbc.png)

### 12. 拥塞避免算法

#### 网络拥塞

![image-20221206094948439](https://picture.lanlance.cn/i/2022/12/06/638e9fbc5f6ff.png)

![image-20221206095011302](https://picture.lanlance.cn/i/2022/12/06/638e9fd33b3b7.png)

![image-20221206095124790](https://picture.lanlance.cn/i/2022/12/06/638ea01cb767d.png)

#### 慢开始和拥塞避免

![image-20221206095228390](https://picture.lanlance.cn/i/2022/12/06/638ea05c5a13d.png)

![image-20221206095325502](https://picture.lanlance.cn/i/2022/12/06/638ea09574956.png)

#### 快重传与快恢复

![image-20221206095540732](https://picture.lanlance.cn/i/2022/12/06/638ea11cb2174.png)

![image-20221206095728302](https://picture.lanlance.cn/i/2022/12/06/638ea188b7620.png)

#### 学习目标

![image-20221206095924783](https://picture.lanlance.cn/i/2022/12/06/638ea1fc9f53b.png)

#### 面试题

![image-20221206095917418](https://picture.lanlance.cn/i/2022/12/06/638ea1f54a957.png)

### 13. TCP 粘包

#### TCP 协议与应用层协议

![image-20221206150005636](https://picture.lanlance.cn/i/2022/12/06/638ee875734f6.png)

#### 应用层的数据拆分

![image-20221206150726600](https://picture.lanlance.cn/i/2022/12/06/638eea2e5ce58.png)

#### Nagle 算法

![image-20221206150935447](https://picture.lanlance.cn/i/2022/12/06/638eeaaf4779e.png)

- 缓冲区中数据超过最大数据段（MSS）
- 上一个数据段被确认（ACK）后

#### 总结

![image-20221206151245855](https://picture.lanlance.cn/i/2022/12/06/638eeb6da0e70.png)

#### 学习目标

![image-20221206151321566](https://picture.lanlance.cn/i/2022/12/06/638eeb914d97f.png)

#### 面试题

![image-20221206151311494](https://picture.lanlance.cn/i/2022/12/06/638eeb8735347.png)

### 14. SYN flood 攻击

#### SYN flood 攻击原理

![image-20221206155200618](https://picture.lanlance.cn/i/2022/12/06/638ef4a07afdb.png)

#### 资源耗尽类攻击

![image-20221206155442014](https://picture.lanlance.cn/i/2022/12/06/638ef541ca308.png)

#### 协议特性漏洞攻击

![image-20221206155625602](https://picture.lanlance.cn/i/2022/12/06/638ef5a95c2a5.png)

#### 防范手段

![image-20221206155811034](https://picture.lanlance.cn/i/2022/12/06/638ef612c4774.png)

![image-20221206155856380](https://picture.lanlance.cn/i/2022/12/06/638ef6402f048.png)

#### 学习目标

![image-20221206155928641](https://picture.lanlance.cn/i/2022/12/06/638ef6604efa9.png)

#### 面试题

![image-20221206155940228](https://picture.lanlance.cn/i/2022/12/06/638ef66be4660.png)

### 15. 虚拟专用网（VPN)

#### 产生 VPN 的背景

![image-20221206160221561](https://picture.lanlance.cn/i/2022/12/06/638ef70d6680d.png)

#### 专用 IP 地址

![image-20221206160511486](https://picture.lanlance.cn/i/2022/12/06/638ef7b743f5b.png)

#### VPN 工作原理

![image-20221206160757737](https://picture.lanlance.cn/i/2022/12/06/638ef85dd1c45.png)

#### 学习目标

![image-20221206160958382](https://picture.lanlance.cn/i/2022/12/06/638ef8d613221.png)

#### 面试题

![image-20221206161005140](https://picture.lanlance.cn/i/2022/12/06/638ef8dcc73e7.png)
