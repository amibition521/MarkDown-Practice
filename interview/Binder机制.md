### Binder机制

1.IPC 机制

2.Binder原理

* Stub

  * Binder类

    当客户端和服务端位于同一个进程时，方法调用不会走跨进程的transact过程，当两者位于不同进程时，方法调用走transact过程，这个逻辑由Stub的内部代理类Proxy来完成。

* Stub内部代理类

  ![1490349096](E:\MarkDown-Practice\interview\ic\1490349096.jpg)

  ​