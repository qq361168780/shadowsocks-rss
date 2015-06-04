# ShadowsocksR 协议插件文档 #
----

## 概要 ##

用于方便地产生各种协议接口

## 接口 ##
以下以C#语言为例做说明

interface IObfs

成员函数

`InitData()`  
参数：无  
返回：一个自定义类型变量，通常用于保存此接口的全局信息，不应返回null，c语言中返回void*  
说明：第一次创建实例前调用，同一服务端配置不会重复调用，服务端在建立监听时调用，客户端在第一次连接时调用。  

`SetServerInfo(ServerInfo serverInfo)`  
参数：ServerInfo结构，包含成员变量：  

- host: 字符串类型，服务端ip，客户端需要把域名转换为ip，如有前置代理，则直接使用配置时所用的域名也可，服务端需要获取监听ip
- port: 整数类型，服务端监听端口
- tcp_mss: 整数类型，TCP分包大小，设置为1440
- param: 用户设置的参数，字符串类型
- data: 由InitData返回的结果，为object类型（c语言中使用void*）

返回：无  
说明：实例构造的时候（每个连接建立时）调用  

`Dispose()`  
参数：无  
返回：无  
说明：实例析构时（每个连接关闭时）调用

`byte[] ClientPreEncrypt(byte[] plaindata, int datalength, out int outlength)`  
参数：需要处理的字节数组及其长度  
返回：处理后的字节数组及其长度  
说明：客户端发送到服务端数据**加密前**调用此接口

`byte[] ClientEncode(byte[] encryptdata, int datalength, out int outlength)`  
参数：需要编码的字节数组及其长度  
返回：编码后的字节数组及其长度  
说明：客户端发送到服务端数据**加密后**调用此接口

`byte[] ClientDecode(byte[] encryptdata, int datalength, out int outlength, out bool needsendback)`  
参数：需要解码的字节数组及其长度  
返回：解码后的字节数组及其长度，以及needsendback标记是否立即回发服务端数据。如needsendback为true，则会立即调用ClientEncode，调用时参数是一个长度为0的字节数组  
说明：客户端收到服务端数据在**解密前**调用此接口

`byte[] ClientPostDecrypt(byte[] plaindata, int datalength, out int outlength)`  
参数：需要处理的字节数组及其长度  
返回：处理后的字节数组及其长度  
说明：客户端收到服务端数据在**解密后**调用此接口

`byte[] ServerPreEncrypt(byte[] plaindata, int datalength, out int outlength)`  
参数：需要处理的字节数组及其长度  
返回：处理后的字节数组及其长度  
说明：服务端发送到客户端数据**加密前**调用此接口

`byte[] ServerEncode(byte[] encryptdata, int datalength, out int outlength)`  
参数：需要编码的字节数组及其长度  
返回：编码后的字节数组及其长度  
说明：服务端发送到客户端数据**加密后**调用此接口

`byte[] ServerDecode(byte[] encryptdata, int datalength, out int outlength, out bool needdecrypt, out bool needsendback)`  
参数：需要解码的字节数组及其长度  
返回：解码后的字节数组及其长度，以及needdecrypt标记数据是否需要解密（一般都应该为true），以及needsendback标记是否立即回发客户端数据。如needsendback为true，则会立即调用ServerEncode并发送其返回结果，调用时参数是一个长度为0的字节数组  
说明：服务端收到客户端数据在**解密前**调用此接口

`byte[] ServerPostDecrypt(byte[] plaindata, int datalength, out int outlength)`  
参数：需要处理的字节数组及其长度  
返回：处理后的字节数组及其长度  
说明：服务端收到客户端数据在**解密后**调用此接口

## 插件编写 ##

有两类插件，一类是协议插件，一类是混淆插件

其中接口InitData, SetServerInfo, Dispose接口必须实现，其它的接口为通讯接口

编写协议插件的话，需要重写ClientPreEncrypt, ClientPostDecrypt, ServerPreEncrypt, ServerPostDecrypt，其它的按原样返回，needdecrypt必须为true，needsendback必须为false

编写混淆插件的话，需要重写ClientEncode, ClientDecode, ServerEncode, ServerDecode，其它的按原样返回。

如果编写的部分仅含客户端部分，那么只需要编写Client为前缀的两个接口，服务端同理。

目前支持此插件接口的，有 [ShadowsocksR C#](https://github.com/breakwa11/shadowsocks-csharp/releases) 和 [ShadowsocksR Python](https://github.com/breakwa11/shadowsocks/tree/manyuser)