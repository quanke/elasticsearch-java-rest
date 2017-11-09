# 日志记录

Java REST 客户端使用和`Apache Async Http`客户端使用的相同日志记录库：`Apache Commons Logging`，它支持许多流行的日志记录实现。 用于启用日志记录的java包是客户端本身的`org.elasticsearch.client`，嗅探器是`org.elasticsearch.client.sniffer`。

请求 `tracer` 日志记录可以开启以curl格式记录。 这在调试时非常方便，例如在需要手动执行请求的情况下，检查是否仍然产生相同的响应。  请注意，这种类型的日志记录非常消耗资源，不应一直在生产环境中启用，而只是在需要时暂时使用。

