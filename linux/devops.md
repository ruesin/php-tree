# 运维

## 查端口号
查看端口号被哪个进程占用：`lsof -i:9000`

## 查看内存情况
`free -m`、`free -h`、`watch -n 1 free`

## 查看进程情况
`top`， C m

## 查看文件使用
`df -h`、`du -sh /*`

## kill
发送指定的信号到相应进程。不指定型号将发送SIGTERM（15）终止指定进程。如果任无法终止该程序可用“-KILL” 参数，其发送的信号为SIGKILL(9) ，将强制结束进程，使用ps命令或者jobs 命令可以查看进程号。root用户将影响用户的进程，非root用户只能影响自己的进程。

## nginx 日志
uniq -c 命令输出统计词频 

sort -k1nr 解释： -k指定以那个列排序 1表示第一列 n表示使用数字而非文本排序 r表示倒序

假设Nginx日志的格式为：
```
[] 100.116.108.148 - - [13/Jul/2017:00:05:19 +0800] "POST /message/check HTTP/1.0" 200 89 "https://www.example.com/message/add" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36" "36.57.161.201"
[] 100.109.253.3 - - [13/Jul/2017:00:12:16 +0800] "GET /statisticDaily/index HTTP/1.0" 200 37374 "https://www.example.com/statisticDaily/index" "Mozilla/5.0 (Linux; Android 5.1.1; vivo Xplay5A Build/LMY47V; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/53.0.2785.49 Mobile MQQBrowser/6.2 TBS/043305 Safari/537.36 MicroMessenger/6.5.10.1080 NetType/WIFI Language/zh_CN" "223.87.234.226"
```

```shell
# 统计nginx访问量最多的前100个url和频次 #输出：频次 请求路径
grep -E "POST|GET" /data/logs/nginx/2017/07/13/manage.access.log | awk -F '"' '{print $2,$3}' | awk '{print $2}'| sort | uniq -c | sort -k1nr | head -100

# 统计nginx访问状态码非200的前100个url和频次 #输出：频次 状态 请求方法 请求路径
grep -E "POST|GET" /data/logs/nginx/2017/07/13/manage.access.log | awk -F '"' '{print $2,$3}' | awk '{if ($4!=200) {print $4,$1,$2}}' | sort | uniq -c | sort -k1nr | head -100

# 统计nginx访问不正常（状态码400+）的前100个url和频次 #输出：频次 状态码 请求方法 请求路径
grep -E "POST|GET" /data/logs/nginx/2017/07/13/manage.access.log | awk -F '"' '{print $2,$3}' | awk '{if ($4>="400") {print $4,$1,$2}}' | sort | uniq -c | sort -k1nr | head -100

# 统计nginx访问频次最高的100个Ip #输出: 频次 ip
grep -E "POST|GET" /data/logs/nginx/2017/07/13/manage.access.log | awk -F '"' '{print $(NF-1)}' | sort | uniq -c | sort -k1nr | head -100
```