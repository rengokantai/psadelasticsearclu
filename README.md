# psadelasticsearclu



## 4
- 2
```
service elasticsearch start
update-rc.d elasticsearch defaults 95 10
vim /etc/elasticsearch/elasticsearch.yml
```
change:
```
cluster.name: xx
node.name: xx
node.client: true
node.data: false
```
restart
```
service elasticsearch restart
ulimit -n

```
Then``` vim /etc/security/limits.conf```Add the following line:
```
*    soft   nofile   64000
*    hard   nofile   64000
root soft   nofile   64000
root hard   nofile   64000
```
Then``` vim /etc/pam.d/commom-session```Add the following line:
```
session required pam_limits.so
```
Then``` vim /etc/pam.d/commom-session-noninteractive```Add the following line:
```
session required pam_limits.so
```
check again:
```
ulimit -m
```
should show unlimited.
Finally, ```vim /etc/environment```
Add```ES_HEAP_SIZE="512m"```

