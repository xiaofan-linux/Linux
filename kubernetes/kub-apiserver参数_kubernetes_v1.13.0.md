

###### 通用参数

参数名|含义|	默认值
-|-|-
advertise-address|用于向集群通告 apiserver 的 IP 地址。该地址必须可由集群的其余组件访问。|如果为空，则使用--bind-address。 如果未指定--bind-address，将使用主机的默认接口。
cloud-provider-gce-lb-src-cidrs|CIDRs opened in GCE firewall for LB traffic proxy & health checks|default 130.211.0.0/22,209.85.152.0/22,209.85.204.0/22,35.191.0.0/16
cors-allowed-origins|List of allowed origins for CORS, comma separated.  An allowed origin can be a regular expression to support subdomain matching.


default-not-ready-toleration-seconds
notReady 容忍时间，NoExecute that is added by default to every pod that does not already have such a toleration.
300


default-unreachable-toleration-seconds
unreachable 容忍时间，NoExecute that is added by default to every pod that does not already have such a toleration.
300


external-hostname
The hostname to use when generating externalized URLs for this master (e.g. Swagger API Docs).


feature-gates
一组key = value对，用于描述特征处于alpha / experimental状态


master-service-namespace
废弃：the namespace from which the kubernetes master services should be injected into pods
default


max-mutating-requests-inflight
在给定时间内的最大 mutating 请求数。 当服务器超过此值时，它会拒绝请求。 0表示无限制。
200


max-requests-inflight
在给定时间内的最大 non-mutating 请求数。 当服务器超过此值时，它会拒绝请求。 0表示无限制。
400


min-request-timeout
表示处理程序在请求超时之前必须至少链接多久。 目前只有监视请求处理程序能处理该值，会选择高于此数字的随机值作为连接超时，以分散负载。
1800


request-timeout
表示处理程序在请求超时之前必须至少链接多久。这是默认请求超时，但可能会被其他参数覆盖，比如 min-request-timeout
1m


target-ram-mb
apiserver的内存限制（MB）用于配置缓存大小等

作者：耳机在哪里
链接：https://www.jianshu.com/p/c201724a4ec6
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
