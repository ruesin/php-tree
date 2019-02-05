# 管道

## 查进程
`ps aux | grep php`

## 从目录下所有文件中查找匹配字符、统计次数

`find . -name 'test.php' | xargs grep 'ruesin'`

`find . -name 'test.php' | xargs grep 'ruesin' | wc -l`