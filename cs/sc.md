## 安全

### 1. 什么是 JWT？主要用来做什么？

JWT：Json Web Token，是基于 Json 的一个公开规范，这个规范允许我们使用 JWT 在用户和服务器之间传递安全可靠的信息，他的两大使用场景是：认证和数据交换

### 2. JWT 由几部分组成？分别是什么？

一个 JWT 实际上就是一个字符串，它由三部分组成：头部、载荷与签名

### 3. JWT 校验 token 时，怎么保证数据并没有被黑客拦截并篡改？

signature 中有私钥来进行签名，可以保证安全性

### 4. Session + Cookie 实现

1. 客户端登录成功后，服务器会给每一台主机分配一个唯一的 session_id，用来区分他们，除了存入服务器的缓存，数据库或者内存中，还会把这个 session_id 返回给相应的主机，

2. 主机收到 session_id 后会存入 cookie 中，以后主机的每一次发送其他类型的请求的操作都会携带这个 session_id，

3. 服务器会将客户端发来的这个 session_id 和服务端查到的 session_id 进行对比，如果匹配，则返回给对应主机所需要的资源，否则拒绝

缺点

1. 因为服务器缓存 session_id，需要定期清理 session_id 表，

2. 不能识别跨域请求，

3. 在分布式(即多台服务器)的环境下，还得做 session 同步，一般不推荐，

### 5. JWT 的缺陷

1. 不安全的加密算法
   JWT 给开发者提供了很多的加密算法选择，其中就包括了已知的易受攻击的算法
2. 在 header 中包含了签名算法的种类
   攻击者只需要将 header 中的 alg 字段设置为 none 就可以绕过签名验证过程，在知道服务器使用非对称加密算法的情况下，修改 alg 为一个对称加密算法

### 6. Paseto 相比 JWT 的改变

- 不会向用户开放所有的加密算法
- header 中不再含有 alg 字段，也不会有 none 算法
- payload 使用加密算法，而不是简单的编码

### 7. CSRF 攻击

攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击网站发送跨站请求。利用受害者在被攻击网站已经获取的注册凭证，绕过后台的用户验证，达到冒充用户对被攻击的网站执行某项操作的目的。

### 8. 防御 CSRF 攻击

前面讲到 CSRF 的一个特征是，攻击者无法直接窃取到用户的信息（Cookie，Header，网站内容等），仅仅是冒用 Cookie 中的信息。

而 CSRF 攻击之所以能够成功，是因为服务器误把攻击者发送的请求当成了用户自己的请求。那么我们可以要求所有的用户请求都携带一个 CSRF 攻击者无法获取到的 Token。服务器通过校验请求是否携带正确的 Token，来把正常的请求和攻击的请求区分开，也可以防范 CSRF 的攻击。

### 9. 什么是 Sentinel？

Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

### 10. 什么是服务限流、服务熔断、服务降级、服务雪崩？

- 服务限流：在接口访问超过设置的阈值，走服务降级 fallback 方法
- 服务熔断：接口出现异常或者处理时间过长，直接熔断，走服务降级 fallback 方法
- 服务降级：在服务限流或者服务熔断的情况下，走服务降级 fallback 方法，一段时间内不再走业务逻辑方法
- 服务雪崩：默认的情况下，一个服务器只有一个线程池，在高并发情况下，一个接口访问次数过多，把线程池线程全部占用了，导致其他接口不可用，造成服务雪崩。黑客攻击同一个接口

解决方法：线程池隔离或者信号量隔离

- 线程池隔离就是每个接口设置一个线程池，这样占用内存非常大
- 信号量隔离就是每个接口设置一个阈值，超过阈值直接走服务降级

### 11. QPS 和并发线程数的区别？

- QPS：每秒请求数，即在不断向服务器发送请求的情况下，服务器每秒能够处理的请求数量。
- 并发线程数：指的是施压机施加的同时请求的线程数量。
