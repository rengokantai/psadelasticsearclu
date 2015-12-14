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
#### cp5
- 4 template, time-based idx
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
            "a":{  
               "type":"string"
            },
            "b":{  
               "type":"string"
            }
            "c":{
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
  "a":"xx",
  "b":"yy"
  "c":"2015-01-01T01:01:01Z"
}
```
####6
- 1 (disable shard auto-allocation when upgrading elasticsearch)
open ```http://client-01:9200/_cluster/settings``` choose 'put'
Note 'transient' means this setting is temp
```
{
   "transient":{
      "cluster.routing.allocation.enable" : "none"
   }
}
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
open```http://client-01:9200/_snapshots/es_repo_01``` choose 'put'
```
{
   "type":"fs",
   "settings":{
      "ocation":"/mnt/esbackup"
   }
}
```
open```http://client-01:9200/_snapshots/es_repo_01/snapshot01``` choose 'put'
should return
```
{
   "accepted":"true"
}
```
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
