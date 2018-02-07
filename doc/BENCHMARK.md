# Benchmark

常见的压力测试工具有ab，[wrk](https://github.com/wg/wrk)。HTTP/1.1的长连接已经很普及，wrk默认支持长连接，ab需要加上`-k`选项， 否则ab的压力测试会默认采用HTTP/1.0，即每一个请求建立一个TCP连接。

## 测试环境

- 测试环境为本地，配置4核心`Intel(R) Core(TM) i5-2320 CPU @ 3.00GHz`
- 测试工具为wrk和ab，测试时间为60s，worker数设置为4
- nginx的worker_processes配置为4
- 标准1KB静态页面测试

## 测试内容

### 1.使用wrk测试长连接Req/Sec

并发数  | lotos  | nginx
---- | ------ | ------
10   | 21.94k | 15.42k
100  | 28.21k | 16.95k
1000 | 26.01k | 16.62k

### 2.使用ab测试长连接Req/Sec

并发数  | lotos   | nginx
---- | ------- | ------
10   | 129.11k | 81.89k
100  | 136.20k | 77.56k
1000 | 97.56k  | 53.69k

### 3.使用ab测试短连接Req/Sec

并发数  | lotos  | nginx
---- | ------ | ------
10   | 25.96k | 25.92k
100  | 26.52k | 26.63k
1000 | 22.53k | 23.29k

## 测试总结

我不知道ab的结果为什么比wrk大很多，但起码Lotos确实表现比nginx要好(毕竟功能简单)。

并发的瓶颈不在于内存的malloc和free，现代的内存分配器性能已经很强，但是如果可以用内存池在初始化阶段就分配一大片内存，对于短连接的测试，性能应该还是有些许提升的。

从ab短连接和长连接的测试结果来看，连接的建立和关闭才是真的瓶颈所在， TCP的三次握手和四次挥手比较耗费时间。