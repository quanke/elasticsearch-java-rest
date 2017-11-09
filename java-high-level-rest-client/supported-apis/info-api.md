
# Info API

## 执行

集群信息可以通过 info() 方法被获取到：

```
MainResponse response = client.info();
```
## 响应

返回的 MainResponse 提供了有关集群的各种信息：

```
ClusterName clusterName = response.getClusterName(); // 获取包含集群名称信息的 ClusterName 对象
String clusterUuid = response.getClusterUuid(); // 获取集群的唯一标识符
String nodeName = response.getNodeName(); // 获取执行请求的节点的名称
Version version = response.getVersion(); // 获取已执行请求的节点版本
Build build = response.getBuild(); // 获取已执行请求的节点的构建信息
```