#### psadelasticsearclu
#####install 
######ubuntu
```
cat /etc/os-release
add-apt-repository ppa:webupd8team/java
apt-get update
apt-get install oracle-java8-installer
java -version
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/deb/elasticsearch/2.3.1/elasticsearch-2.3.1.deb
dpkg -i elasticsearch-2.3.1.deb
service elasticsearch start
```
check config files:
```
ls /etc/elasticsearch
vim elasticsearch.yml
```
edit
```
custer.name: yd
node.name: n1
```
restart
```
service elasticsearch restart
```
######windows
```
es\bin> service install elasticsearch
```
using powershell
```
invoke-webrequest http://localhost:9200
```
#####clustering and config stra
######planning
set in yaml file:
```
manimum_master_nodes:x
```
good rule is no. of master nodes/2+1
######cluster and operating systen setting
for server:
```
munimun_master_nodes: 2
```

```
gateway.recovery_after_nodes:8
gateway.wxpected_nodes:9
gateway.recover_after_time: 3M
```
for master node:
```
node.master: true
node.data: false
```
for client node:
```
node.client: true
node.data:false
```
for data node:
```
node.data: true
node.master: false
node.client: false
```
setthis, we dont want to allow data nodes to be queried
```
http.enabled: false
```
##### installing and configuring the cluster
######first client node
```
service elasticsearch start
update-rc.d elasticsearch defaults 95 10
vim /etc/elasticsearch/elasticsearch.yml
```
change:
```
cluster.name: yd
node.name: c1
node.client: true
node.data: false
```
restart
```
service elasticsearch restart
ulimit -n

```
Then
``` 
vim /etc/security/limits.conf
```
Add the following line:
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
Add```ES_HEAP_SIZE="512m"``

reboot server.`

######first master node
change:```vim /etc/elasticsearch/elasticsearch.yml```
```
cluster.name: yd (same as previous cluster name)
node.name: m1
node.master: true
node.data: false

network.host: 172.1.2.3

discovery.zen.ping.multicast.enabled: false
discovery.zen.ping.unicast.hosts: ["client1","client2","master1","master2","master3","data1","data2"] //if not bind to /etc/host, using ip
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
cd /usr/share/elasticsearch/bin
./plugin install mobz/elasticsearch-head
```
Then open the plugin in browser:
```
nodename:9200/_plugin/head/
```
######first data node
change:```vim /etc/elasticsearch/elasticsearch.yml```
```
cluster.name: yd
node.name: d1
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
##### index stra
######template, time-based idx
Open postman, choose 'POST' and url should be ```http://nodename:9200/_template/log_tamplate```
send data as JSON:
```json
{  
   "template":"log_*",
   "aliases":{  
      "log":{  

      }
   },
   "settings":{  
      "index":{  
         "number_of_shards":5,
         "number_of_replicas":1
      }
   },
   "mappings":{  
      "log_event":{  
         "properties":{  
            "log_event":{  
               "type":"string"
            },
            "log_text":{  
               "type":"string"
            },
            "log_date":{
               "type":"date"
            }
         }
      }
   }
}
```

Try to insert some data:
Type ```http://nodename:9200/log_yyyymmdd/log_event``` choose 'post'
using data
```
{
  "log_name":"xx",
  "log_text":"yy",
  "log_date":"2015-01-01T01:01:01Z"
}
```
Test in head plugin->any request using POST
```
http://54.85.0.0:9200/log/
http://54.85.0.0:9200/log_20150111/
```
##### maintaining es cluster
######1 (disable shard auto-allocation when upgrading elasticsearch)
open ```http://client-01:9200/_cluster/settings``` choose 'put'
Note 'transient' means this setting is temp
```
{
   "transient":{
      "cluster.routing.allocation.enable" : "none"
   }
}
```
When we shut down data node, it will not rebalance.  
Then upgrade:
```
service stop elasticsearch
....upgrade
service start elasticsearch
```
To recover setting:
```
{
   "transient":{
      "cluster.routing.allocation.enable" : "all"
   }
}
```
- 2
Open ```vim /etc/elasticsearch/elasticsearch.yml``` in master machine,
Edit
```
path.repo: ["/mnt/esbackup"]
```
Do this in all machines.  
To create repo:  
open```http://client-01:9200/_snapshots/es_repo_01``` choose 'put'
```
{
   "type":"fs",
   "settings":{
      "location":"/mnt/esbackup"
   }
}
```
To perform first shpshot:
open```http://client-01:9200/_snapshots/es_repo_01/snapshot01``` choose 'put'
should return
```
{
   "accepted":"true"
}
```
get status:  
open```http://client-01:9200/_snapshots/es_repo_01/snapshot01/status``` choose 'get'

should show ```success```

Now create a new snapshot.
open```http://client-01:9200/_snapshots/es_repo_01/snapshot02``` choose 'post'
with following data:
```
{
   "indice":"log_20151213,log_20151214"
}
```
Then open```http://client-01:9200/_snapshots/es_repo_01/snapshot02``` choose 'get'
Check attribute ```indices```

Then delete log_* indices,we can rectore those data"
Open```http://client-01:9200/_snapshots/es_repo_01/snapshot02/_restore``` choose 'post'
Enter the returned json data we get

######Curator: GUI tool for backup
```
wget -qO - https://packages.elastic.co/GPG/KEY-elasticsearch | sudo apt-key add -
```
Then 
``` vim /etc/apt/sources.list.d/curator.list```
Add
```
deb http://packages.elastic.co/curator/3/debian stable main
```
Then
```
apt-get update
apt-get install python-elasticsearch-curator
```

After installation:
```
curator --host client-01 show indices --all-indices //ex
curator --host 54.85.0.0 show indices --all-indices
curator --host 54.85.0.0 show indices --older-than 0 --time-unit days --timestring '%Y%m%d'
curator --host 54.85.0.0 snapshot --repository es_repo_01 --name snapshot01 indices --older-than 0 --time-unit days --prefix log_
```
delete indices:
```
curator --host 54.85.0.0 delete indices --older-than 0 --time-unit days --timestring '%Y%m%d' --prefix log_
```
#####monitoring
######Health API
Ex:
```
http://client-01:9200/_cluster/health
```
get response like this
```
{
  "cluster_name": "yd",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 4,
  "number_of_data_nodes": 3,
  "active_primary_shards": 5,
  "active_shards": 10,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100
}
```
More details:
```
http://client-01:9200/_cluster/health?level=indices
```
return like this
```
{
  "cluster_name": "yd",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 4,
  "number_of_data_nodes": 3,
  "active_primary_shards": 5,
  "active_shards": 10,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100,
  "indices": {
    "log_20151111": {
      "status": "green",
      "number_of_shards": 5,
      "number_of_replicas": 1,
      "active_primary_shards": 5,
      "active_shards": 10,
      "relocating_shards": 0,
      "initializing_shards": 0,
      "unassigned_shards": 0
    }
  }
}
```

Stats:

```
http://client-01:9200/_nodes/stats
```

- 2 Marvel
```
cd /usr/share/elasticsearch/bin
./plugin install license
./plugin install marvel-agent
```
download kibana,
```
wget https://download.elastic.co/kibana/kibana/kibana_4.5.0_amd64.deb
dpkg -i
```
then
```
vim /opt/kibana/config/kibana.yml
```
edit
```
elasticsearch.url: "http://172.31.1.1:9200"  //client node ip
```
Then
```
cd /opt/kibana/bin
./kibana plugin --install elasticsearch/marvel/latest
./kibana &  (port 5601)
```

