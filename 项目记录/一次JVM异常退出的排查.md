## 一次进程异常退出排查流程

### 背景
某个新功能上线过程中，刚发布两台机器，在观察的时候，收到机器load过高告警，登录到机器后，就发现jvm进程直接崩溃退出了，于是先回滚发布了。

### 排查步骤：
1. 观察监控，异常机器在发布之后内存从2G使用升高到4G，机器设置的堆内存为5G，怀疑有内存泄漏。但是在jvm参数设置的进程oom时dump内存目录下没有找到内存溢出文件。

2. 尝试在预发环境复现该问题，启动多个线程并发请求问题接口，始终没有复现问题。考虑到该接口为读服务，部分请求查询失败对业务影响不大，于是将新接口增加开关之后，发布到线上一台beta机器上观察，当负载上升时，关闭开关，同时dump内存。
灰度发布后，一边使用jstat -gcutil pid interval持续观察进程队列以及内存，gc的信息，一边观察系统负载是否在持续升高。后面发现接口功能正常，负载是在一瞬间突然升高，于是jstack打印对战并jmap dump内存后关闭开关，并重启机器。

3. 本以为关闭开关之后，问题便不会发生，可是，重启机器后，问题还是出现了。于是直接将这台机器摘除服务。

4. 网上查找jvm异常退出相关资料的时候看到，jvm在异常退出前会保留异常原因，线程以及内存信息。于是找到这个文件，发现有这么一个提示：
```text
Current CompileTask:
C1: 126957 14641       3       com.alibaba.fastjson.parser.deserializer.FastjsonASMDeserializer_21_ShopFooterRespon
se::deserialze (4032 bytes)
```

ShopFooterRespon这个类是一个pojo，并且这次修改只是修改了其父类，新增了一个字段。但是这里却出现异常，令人费解。google后了解到低版本的fastjson反序列化的类中属性个数正好等于32或者是64的时候就会出错。于是数了一下ShopFooterRespon类的字段以及其父类的字段数，大梦初醒。升级fastjson版本后问题解决。

### 相关文章
![fastjson反序列化的时候报错](https://blog.csdn.net/u013008179/article/details/78904395)
![JVM致命错误日志(hs_err_pid.log)分析](https://blog.csdn.net/github_32521685/article/details/50355661)
