# 概览

Java REST 客户端分为两种:

- [Java 低级 REST ](java-low-level-rest-client.md)

Elasticsearch 官方低级客户端： 通过 `http`协议 与Elasticsearch服务进行通信。请求编码和响应解码保留给用户实现。与所有 Elasticsearch 版本兼容。

- [Java 高级 REST ](java-high-level-rest-client.md)

Elasticsearch 官方高级客户端： 基于低级客户端，提供特定的方法的API，并处理请求编码和响应解码。