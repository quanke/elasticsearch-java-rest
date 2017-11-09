# Java 低级 REST 客户端

## 低级客户端的功能：

- 最小依赖
- 所有可用节点间的负载均衡
- 节点故障和特定响应代码时的故障切换
- 故障连接策略 (是否重新连接故障节点取决于连续失败多少次；失败次数越多，在再次尝试同一个节点之前，客户端等待的时间越长）
- 持久化连接
- 跟踪记录请求和响应
- 自动[发现集群节点](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/sniffer.html) 选项