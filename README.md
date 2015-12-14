# psadelasticsearclu



#### cp4
- 2 client node
```
service elasticsearch start
update-rc.d elasticsearch defaults 95 10
vim /etc/elasticsearch/elasticsearch.yml
```
change:
```
cluster.name: xx
node.name: client-1
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

- 3 master node
change:```vim /etc/elasticsearch/elasticsearch.yml```
```
cluster.name: xx (same as previous cluster name)
node.name: master-1
node.master: true
node.data: false

network.host: should be remoteaddress(xx.xx.xx.xx)

discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["client1","client2","master1","master2","master3","data1","data2"]
```

Open ```vim /etc/elasticsearch/elasticsearch.yml``` in client machine,
Edit
```
network.host=xx.xx.xx.xx
discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["client1","client2","master1","master2","master3","data1","data2"]
```
To test:
``` curl http://nodename:9200/_cluster/stats```

Install plugin:
```
cd usr/share/elasticsearch/bin
./plugin install mobz/elasticsearch-head
```
Then open the plugin in browser:
```
nodename:9200/_plugin/head/
```
- 4 data node
change:```vim /etc/elasticsearch/elasticsearch.yml```
```
cluster.name: xx (same as previous cluster name)
node.name: data-1
node.master: false
node.data: true

network.host: xx.xx.xx.xx

discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["client1","client2","master1","master2","master3","data1","data2"]
```
Then
```
service elasticsearch restart
```


