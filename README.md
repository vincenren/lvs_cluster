# 说明

此系统是基于 LVS + OSPF + FULLNAT + Nginx 架构的, 几点说明：

    1. 集群有万兆和千兆, 万兆集群的 LVS 机器是万兆网卡, 千兆集群的 LVS 机器是普通千兆网卡;

    2. 集群类型分为外网和内网, 通过 type 指定, extra 为外网, internal 为内网;

    3. 一个域名通过 DNS 解析到的 IP 即是 VIP, VIP 以 lo 的方式起在每台 LVS 上, VIP 下面可以挂多台 Nginx, Nginx 端口也可以多个;

    4. 为每个 VIP 增加了一个属性: wstype, 用于定义 Nginx 的业务属性;

    5. 操作 LVS 机器还是通过 SSH 信任来做的, SSH 用户还要有 sudo 权限;

    6. LVS 机器重编内核和配置工具(funcs.py里面)可能需要重写, 编写合适的脚本即可.


    功能有:

    1. 建立 LVS + OSPF + FULLNAT + Nginx 集群, 并且把配置文件发布到所有 LVS 机器, 但不直接 reload keepalived, 需要手动操作(LVS 机器需要事先用「装机系统」安装好);

    2. 删除 LVS + OSPF + FULLNAT + Nginx 集群, 只是从数据库中删除集群信息, 集群的机器只需要重装即可;

    3. 增加和删除 VIP;

    4. 增加和删除 LB(LVS 机器);

    5. 增加和删除 WS(Nginx 机器);

    6. 增加、修改和删除某一个 VIP 的 port;

    7. 修改某一个 VIP 的 wstype;

    8. 同步配置到 LB(LVS 机器).



# 集群信息存储格式

集群信息存在 redis 里面, 下面是一个集群信息的例子:

```
{
name: "hy-outer-global-proxy-test",
vip2ws: [
	{
		wstype: "test",
		vip: "xx.xx.208.1",
		wss: [
			"test-extngtest0.nosa01"
		],
		ports: [
			{
				dport: 80,
				synproxy: 1,
				sport: 80,
				persistence_timeout: 50
			},
			{
				dport: 443,
				synproxy: 1,
				sport: 443,
				persistence_timeout: 50
			}
		]
	},
	{
		wstype: "app",
		vip: "xx.xx.208.2",
		wss: [
			"app-extngtest0.nosa01"
		],
		ports: [
			{
				dport: 80,
				synproxy: 1,
				sport: 80,
				persistence_timeout: 50
			},
			{
				dport: 443,
				synproxy: 1,
				sport: 443,
				persistence_timeout: 50
			}
		]
	}
],
lbinfos: [
{
	internalgateway: "10.xx.19.225",
	routerid: "xx.xx.246.186",
	internalip: "10.xx.19.226",
	internalnetmask: "255.255.255.224",
	extragateway: "xx.xx.246.185",
	ospfnet: "xx.xx.246.184/30",
	extraip: "xx.xx.246.186",
	hostname: "sa-extlbtest0.nosa01",
	extranetmask: "255.255.255.252"
	}
],
vipnets: [
	"xx.xx.208.0/25",
	"xx.xx.178.128/25",
	"xx.xx.93.48/28",
	"xx.xx.216.0/25"
],
device: "em1",
type: "extra"
}

```



# 依赖

```
ujson
futures
redis 2.10.3
```
