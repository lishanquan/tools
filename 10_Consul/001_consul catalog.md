查看Consul中已注册的服务，可以使用Consul的`CLI`工具或者通过`HTTP API`进行查询。

1. `CLI`

   `consul catalog`

   - `consul catalog datacenters`：查询所有数据中心列表

     示例：

     ```shell
     [root@iZ docker]# docker exec -it consul_prod consul catalog datacenters
     dc1
     ```

   - `consul catalog nodes`：查询所有节点列表

     示例：

     ```shell
     [root@iZ docker]# docker exec -it consul_prod consul catalog nodes
     Node         ID        Address     DC
     consul_prod  ff9446b7  172.19.0.5  dc1
     ```

   - `consul catalog services`：查询所有服务列表

     示例：

     ```shell
     [root@iZ docker]# docker exec -it consul_prod consul catalog services
     ac
     ba
     consul
     do
     ```

2. `HTTP API`

   - 查看服务列表（带详细信息）`curl http://localhost:8500/v1/catalog/services`

     ```shell
     [root@iZ docker]# curl http://localhost:8500/v1/catalog/services
     {
     	"ac": ["secure=false"],
     	"ba": ["secure=false"],
     	"consul": [],
     	"do": ["secure=false"]
     }
     ```

   - 查看单个服务详情`curl http://localhost:8500/v1/catalog/service/do`

     ```shell
     [root@iZ docker]# curl http://localhost:8500/v1/catalog/service/do
     [{
     	"ID": "ff9446b7-841a-dd9a-b90b-ac51ba4250ae",
     	"Node": "consul_prod",
     	"Address": "172.19.0.5",
     	"Datacenter": "dc1",
     	"TaggedAddresses": {
     		"lan": "172.19.0.5",
     		"wan": "172.19.0.5"
     	},
     	"NodeMeta": {
     		"consul-network-segment": ""
     	},
     	"ServiceKind": "",
     	"ServiceID": "do-8560",
     	"ServiceName": "do",
     	"ServiceTags": ["secure=false"],
     	"ServiceAddress": "do",
     	"ServiceWeights": {
     		"Passing": 1,
     		"Warning": 1
     	},
     	"ServiceMeta": {},
     	"ServicePort": 8560,
     	"ServiceEnableTagOverride": false,
     	"ServiceProxyDestination": "",
     	"ServiceProxy": {},
     	"ServiceConnect": {},
     	"CreateIndex": 4281389,
     	"ModifyIndex": 4281550
     }]
     ```

3. 