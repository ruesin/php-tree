# 负载均衡

负载均衡（Load Balance）其意思就是分摊到多个操作单元上进行执行，从而共同完成工作任务。通过核心调度者，保证所有后端服务器都将性能充分发挥，从而保持服务器集群的整体性能最优。


## 算法
1. [加权]随机算法

    通过系统的随机算法，根据后端服务器的列表大小值来随机选取其中的一台服务器进行访问。可以按后端机器的配置设置随机概率的权重。调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。

2. [加权]轮询算法

    将请求按顺序轮流地分配到后端服务器上，它均衡地对待后端的每一台服务器，而不关心服务器实际的连接数和当前的系统负载。可以按后端机器的配置为轮询中的服务器附加一定权重的算法。比如服务器1权重1，服务器2权重2，服务器3权重3，则顺序为1-2-2-3-3-3-1-2-2-3-3-3- ......
    
    当服务器群中各服务器的处理能力相同时，且每笔业务处理量差异不大时，最适合使用这种算法。 

3. [加权]最小连接算法

    在多个服务器中，与处理连接数(会话数)最少的服务器进行通信的算法。由于后端服务器的配置不尽相同，对于请求的处理有快有慢，最小连接数法根据后端服务器当前的连接情况，动态地选取其中当前积压连接数最少的一台服务器来处理当前的请求，尽可能地提高后端服务的利用效率，将负责合理地分流到每一台服务器。

    可以事先为每台服务器分配处理连接的数量，并将客户端请求转至连接数最少的服务器上。

4. 源地址哈希法

    根据获取客户端的IP地址，通过哈希函数计算得到一个数值，用该数值对服务器列表的大小进行取模运算，得到的结果便是客服端要访问服务器的序号。采用源地址哈希法进行负载均衡，相同的IP客户端，如果服务器列表不变，将映射到同一个后台服务器进行访问。

    当客户端有一系列业务需要处理而必须和一个服务器反复通信时，该算法能够以流(会话)为单位，保证来自相同客户端的通信能够一直在同一服务器中进行处理。

## 配置

### 简单配置

```
# server1
server {
    listen 8080;
    server_name local.load.com;
    index index.html;
    root /home/www;
}

# Load Balance
upstream load.com.conf {
    server 192.168.1.101:80;
    server 192.168.1.102:80;
    server 127.0.0.1:8080;
}
 
# web server
server {
    listen 80;
    server_name local.load.com;
    location / {
        proxy_pass              http://load.com.conf;
        #proxy_set_header        Host    $host;
        #proxy_set_header        X-Real-IP       $remotr_addr;
        #proxy_set_header        X-Forwarde-For  $proxy_add_x_forwarded_for;
    }
}

# server2 192.168.1.101
server {
    listen 80;
    server_name local.load.com;
    root /home/www;
    location / {
        index index.html;
    }
}

# server3 192.168.1.102
server {
    listen 80;
    server_name local.load.com;
    root /home/www;
    location / {
        index index.html;
    }
}
```

### 详细配置

轮询模式：
```
upstream load.com.conf {
   server 192.168.0.1;
   server 192.168.0.2;
}
```

加权轮询模式：
```
upstream load.com.conf {
   server 192.168.0.1 weight=3;
   server 192.168.0.2 weight=2;
}
```

源地址哈希法：
```
upstream load.com.conf {
   ip_hash;
   server 192.168.0.1 weight=3;
   server 192.168.0.2 weight=2;
}
```

fair（第三方）：按后端服务器的响应时间来分配请求，响应时间短的优先分配。
```
upstream load.com.conf {  
  server server1;  
  server server2;  
  fair;  
}
```

url_hash（第三方）：按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

例：在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法。
```
upstream load.com.conf {  
  server squid1:3128;  
  server squid2:3128;  
  hash $request_uri;  
  hash_method crc32;  
}
```

提示：
```
upstream bakend {
  ip_hash;  
  server 127.0.0.1:9090 down;  
  server 127.0.0.1:8080 weight=2;  
  server 127.0.0.1:6060;  
  server 127.0.0.1:7070 backup;  
}
```
在需要使用负载均衡的server中增加
```
proxy_pass http://bakend/;
```
每个设备的状态设置为：
- down 表示单前的server暂时不参与负载 
- weight 默认为1.weight越大，负载的权重就越大。 
- max_fails ：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误 
- fail_timeout:max_fails次失败后，暂停的时间。 
- backup： 其它所有的非backup机器down或者忙的时候，请求backup
