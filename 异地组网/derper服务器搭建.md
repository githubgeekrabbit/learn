
# DERP
DERP 即 Detoured Encrypted Routing Protocol，这是 Tailscale 自研的一个协议：

它是一个通用目的包中继协议，运行在 HTTP 之上，而大部分网络都是允许 HTTP 通信的。
它根据目的公钥（destination’s public key）来中继加密的流量（encrypted payloads）。

![image.png](https://geekrabbit-1308222610.cos.ap-beijing.myqcloud.com/image_1677829483269.png)

**Tailscale 使用的算法很有趣，所有客户端之间的连接都是先选择 DERP 模式（中继模式），这意味着连接立即就能建立（优先级最低但 100% 能成功的模式），用户不用任何等待。然后开始并行地进行路径发现，通常几秒钟之后，我们就能发现一条更优路径，然后将现有连接透明升级（upgrade）过去，变成点对点连接（直连）。**

**因此，DERP 既是 Tailscale 在 NAT 穿透失败时的保底通信方式（此时的角色与 TURN 类似），也是在其他一些场景下帮助我们完成 NAT 穿透的旁路信道。 换句话说，它既是我们的保底方式，也是有更好的穿透链路时，帮助我们进行连接升级（upgrade to a peer-to-peer connection）的基础设施。**

tailscale以及headscale是当下最好的组网方案，可以搭建私有的vpn网络实现，

用来内网穿透，用来神奇上网，用来组建异地集群，配图

对个人用户来说，所处的网络环境比较复杂，很多时候会无法建立点对点的直连，配图，连接失败

这就需要 derp relay server，derp不仅是一个中转服务器，它也有stun的功能

tailscale官方在许多国家都有derp 服务器，这是列表，如果没有条件自建的可以去挑官方的来使用，

```bash
https://login.tailscale.com/derpmap/default
```

一方面毕竟是异国他乡，一方面公共服务器带宽有限用的人多了就体验很差了。

为了保证穿透的可用性和体验，我们就要付出一些成本，搭建一个私有的中转服务 derp relay server

derp服务有一个特点

节点与节点进行连接时，会首先通过derp服务进行中转连接，以此让穿透立刻实现，因为中转服务是可以保证100%连接成功的，配图

然后，derp服务尝试让两个节点进行点对点的直连，如果直连成功，derp不再中转数据

否则，穿透会一直通过derp进行中转 

所以derp是可以极大的提高穿透体验的，但是，总要有个但是

derp服务搭建完毕后，别人只要知道了你的域名和端口就可以白嫖你的服务了，难受
# 我的物理拓扑图

![[Pasted image 20230330184727.png]]



# 我的网络拓扑图
最终实现把4个不同地区的4条宽带下的设备连到一起，在不同网络下客户端设备实现通过内网ip来访问内网设备资源，实现异地组网，不用通过端口映射到公网再去访问

![[Pasted image 20230330184745.png]]
### 搭建derp服务有两种办法

方案一是通过docker安装，但是需要下依赖包，速度会比较慢，甚至安装失败，最好的办法其实就是要么不停的重试，要么用神奇上网

一种是下载安装二进制文件，在服务器或者vps上配置golang环境，编译安装，但是如果你的服务器在中国国内，那安装依赖和编译都是非常痛苦的

docker安装
--------

确保vps上docker运行正常

```bash
# 安装docker
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
systemctl start docker
systemctl enable docker
docker info
```

部署derper

```bash

  docker run --restart always \
  --name derper -p 1234:1234 -p 3478:3478/udp \
  -e DERP_ADDR=:1234 \
  -e DERP_DOMAIN=域名 \
  -d ghcr.io/yangchuansheng/derper:latest
  
  # nginx会把81反代到443
  
  
  # 如果不是用nginx代理的话就要直接把证书申请好
  docker run --restart always \
  --name derper -p 1234:1234 -p 3478:3478/udp \
  -e DERP_CERT_MODE=manual \
  -e DERP_ADDR=:12345 \
  -e DERP_DOMAIN=域名 \
  -d ghcr.io/yangchuansheng/derper:latest
  
  # 查看容器运行情况
  docker logs -f derper
  
```

编译安装
----

上面是docker安装的过程，如果你不想用docker，也可以直接编译安装

安装golang环境，版本必须大于1.16

```bash
wget https://golang.google.cn/dl/go1.19.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.19.linux-amd64.tar.gz 
# 注意 #
# 如果是升级golang，先删掉原来的go目录，不然可能会有问题#
```

在/etc/profile添加

```bash
export GOROOT=/usr/local/go
export GOPATH=/usr/local/gopath
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOROOT/bin
export PATH=$PATH:$GOPATH/bin
```

刷新

```bash
source /etc/profile
```

安装derp

```bash
go version
# 国内可以尝试代理
#go env -w GOPROXY=https://goproxy.cn,direct
#go env -u GOPROXY
go install tailscale.com/cmd/derper@main
derper -h
```

解析一个域名到这个vps，如果是国内的话域名必须备案才能用，香港的目前还不用备案

域名可以是二级域名，必须是A记录

使用supervisor守护
--------------

```bash
sudo yum install supervisor
```

/etc/supervisord.d/derper.conf

```bash
[program:derper]
command=/usr/local/gopath/bin/derper --hostname=域名  -c $HOME/derper.conf --stun -http-port -1   -a :81
autorestart=true
user=root
redirect_stderr=true
stdout_logfile=/var/log/supervisor/derper.log
stderr_logfile=/var/log/supervisor/derper-err.log
stdout_logfile_maxbytes=5MB
stdout_logfile_backups=1
```

重启supervisord

```bash
sudo systemctl restart supervisord
sudo systemctl enable supervisord
```

申请证书
----

```bash
sudo yum install epel-release -y


sudo pip uninstall urllib3 -y
sudo pip uninstall requests -y
sudo yum remove python-urllib3 -y
sudo yum remove python-requests -y


sudo yum install python-urllib3 -y
sudo yum install python-requests -y
sudo yum install certbot -y



# 申请证书，注意此时vps上的80端口必须是未被占用状态，否则失败#
sudo certbot certonly --standalone -d 域名

# 证书路径为/etc/letsencrypt/live/域名/
# 证书有效期只有3个月，每次更新的时候要先关掉nginx服务，在执行#
certbot renew
systemctl start nginx
```

nginx反向代理
---------

```bash
# centos7 为例
sudo yum install epel-release -y
sudo yum install nginx -y
```

添加/etc/nginx/conf.d/derper.conf，内容如下

```bash

server {
    listen 443 ssl;
    server_name 域名;
    charset utf-8;

    ssl_certificate /etc/letsencrypt/live/域名/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/域名/privkey.pem;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 5m;
    keepalive_timeout 70;


    location / {
        proxy_redirect off;
        proxy_pass http://127.0.0.1:81; # port
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

`systemctl start nginx`

`systemctl enable nginx`

tailscale测试derp
---------------

derper搭建完毕，但是要注意，现在任何人都可以使用我们的derper服务，一会加个认证，在此之前，我们先测试下效果

确保你的vps的443端口与3478端口开放

如果你用的是tailscale，打开tailscale控制台，打开access controls

```bash
// Example/default ACLs for unrestricted connections.
{
  // Declare static groups of users beyond those in the identity service.
  "Groups": {
    "group:example": [ "user1@example.com", "user2@example.com" ],
  },
  // Declare convenient hostname aliases to use in place of IP addresses.
  "Hosts": {
     "example-host-1": "100.100.100.100",
  },
  "ACLs": [
    // Match absolutely everything. Comment out this section if you want
    // to define specific ACL restrictions.
    { "Action": "accept", "Users": ["*"], "Ports": ["*:*"] },
  ],
  "derpMap": {
    "OmitDefaultRegions": true,
    "Regions": { "900": {
      "RegionID": 900,
      "RegionCode": "mangoderp",
      "RegionName": "AliHongkong",
      "Nodes": [{
          "Name": "1",
          "RegionID": 900,
          "HostName":"域名",
          "DERPPort": 443
      }]
    }}
  }
}
```

以下是原内容备份

```bash
// Example/default ACLs for unrestricted connections.
{
	// Declare static groups of users beyond those in the identity service.
	"groups": {
		"group:example": ["user1@example.com", "user2@example.com"],
	},

	// Declare convenient hostname aliases to use in place of IP addresses.
	"hosts": {
		"example-host-1": "100.100.100.100",
	},

	// Access control lists.
	"acls": [
		// Match absolutely everything.
		// Comment this section out if you want to define specific restrictions.
		{"action": "accept", "users": ["*"], "ports": ["*:*"]},
	],
	"ssh": [
		// Allow all users to SSH into their own devices in check mode.
		// Comment this section out if you want to define specific restrictions.
		{
			"action": "check",
			"src":    ["autogroup:members"],
			"dst":    ["autogroup:self"],
			"users":  ["autogroup:nonroot", "root"],
		},
	],
}
```

headscale使用derp
---------------

修改主配置文件config.yaml

```bash
paths:
    - /etc/headscale/derp.yaml
    
```

`/etc/headscale/derp.yaml添加内容`

```bash
# /etc/headscale/derp.yaml
regions:
  900:
    regionid: 900
    regioncode: ahk 
    regionname: AliHongkong 
    nodes:
      - name: 900a
        regionid: 900
        hostname: 域名
        # ipv4: ip
        stunport: 3478
        # stunonly: false
        derpport: 443
```

重启headscale

```bash
# 查看节点#
tailscale netcheck
```

截止到目前，已经能够使用了

![[Pasted image 20230330184827.png]]

开启https验证
---------

刚才说了，现在derp是没有加验证的，谁都可以白嫖我们的服务，这是不允许的

安装tailscale客户端

直接下载。例如：

[https://pkgs.tailscale.com/stable/](https://pkgs.tailscale.com/stable/)

```bash
wget <https://pkgs.tailscale.com/stable/tailscale_1.28.0_amd64.tgz>
```

解压：

```bash
tar zxvf tailscale_1.28.0_amd64.tgz x tailscale_1.28.0_amd64/ x
```

`tailscale_1.28.0_amd64/tailscale x tailscale_1.28.0_amd64/tailscaled x tailscale_1.28.0_amd64/systemd/ x tailscale_1.28.0_amd64/systemd/tailscaled.defaults x tailscale_1.28.0_amd64/systemd/tailscaled.service`

将二进制文件复制到官方软件包默认的路径下：

```bash
cp tailscale_1.28.0_amd64/tailscaled /usr/sbin/tailscaled
cp tailscale_1.28.0_amd64/tailscale /usr/bin/tailscale
chmod +x /usr/sbin/tailscaled
chmod +x /usr/bin/tailscale
```

将 systemD service 配置文件复制到系统路径下：

```bash
cp tailscale_1.28.0_amd64/systemd/tailscaled.service /lib/systemd/system/tailscaled.service
```

将环境变量配置文件复制到系统路径下：

```bash
cp tailscale_1.28.0_amd64/systemd/tailscaled.defaults /etc/default/tailscaled
```

启动 tailscaled.service 并设置开机自启：

`$ systemctl enable --now tailscaled`

查看服务状态：

`$ systemctl status tailscaled`

修改derp启动，加上验证

```bash
command=/usr/local/gopath/bin/derper --hostname=域名  -c $HOME/derper.conf -http-port -1 -a :81 --verify-clients=true --stun 
```

这个验证只是验证是否是允许的域名，并不做身份验证，这意味着，别人只要知道了你的域名和端口，就可以白票你的derper服务，这一点请知晓

相比而言，zerotier的moon服务就没有这个问题了。


# 避坑指南
- 在 tailscale 的实现里，即使没配置好 https 也会在控制页面显示 derp 服务可用，但实际不能用，也没有看到日志输出，难排查错误
- caddy 反向代理 https 但上游用的是 ip 地址时，加上 tls_server_name 选项，这个我找了好久，虽然解决了对最后也没啥帮助，相关讨论。 之前没有折腾过，一直以为 https 的反向代理就是一句配置就完事了，但事实上比想象中要复杂。

# 多节点配置文件
```bash
// Example/default ACLs for unrestricted connections.
{
	// Declare static groups of users beyond those in the identity service.
	"Groups": {
		"group:example": ["user1@example.com", "user2@example.com"],
	},
	// Declare convenient hostname aliases to use in place of IP addresses.
	"Hosts": {
		"example-host-1": "100.100.100.100",
	},
	"ACLs": [
		// Match absolutely everything. Comment out this section if you want
		// to define specific ACL restrictions.
		{"Action": "accept", "Users": ["*"], "Ports": ["*:*"]},
	],
	"derpMap": {
		"OmitDefaultRegions": true,
		"Regions": {
			"900": {
				"RegionID":   900,
				"RegionCode": "hb",
				"RegionName": "xingtai",
				"Nodes": [{
					"Name":     "1",
					"RegionID": 900,
					"HostName": "aaaa.com",
					"DERPPort": 3489,
				}],
			},

			"901": {
				"RegionID":   901,
				"RegionCode": "xg",
				"RegionName": "Hongkong",
				"Nodes": [{
					"Name":     "1",
					"RegionID": 901,
					"HostName": "xxx.com",
					"DERPPort": 443,
				}],
			},
		},
	},
}
```



