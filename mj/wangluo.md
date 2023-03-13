# 计算机网络

## TCP/IP 网络模型有哪几层？

TCP/IP 网络通常是由上到下分成 4 层，分别是应用层，传输层，网络层和网络接口层。

- 应用层只需要专注于为用户提供应用功能，比如 HTTP、FTP、Telnet、DNS、SMTP 等。
- 在传输层会有两个传输协议，分别是 TCP 和 UDP。
- 网络层最常使用的是 IP 协议（Internet Protocol）。

网络接口层的传输单位是帧（frame），IP 层的传输单位是包（packet），TCP 层的传输单位是段（segment），HTTP 的传输单位则是消息或报文（message）。但这些名词并没有什么本质的区分，可以统称为数据包。
