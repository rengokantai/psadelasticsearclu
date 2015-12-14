# psadelasticsearclu



### 4
- 2
```js
service elasticsearch start
update-rc.d elasticsearch defaults 95 10
vim /etc/elasticsearch/elasticsearch.yml
```
change:
```js
cluster.name: xx
node.name: xx
node.client: true
node.data: false
```
restart
```js
service elasticsearch restart
ulimit -n

```
Open```js vim /etc/security/limits.conf```Add the following line:
```js
*    soft   nofile   64000
*    hard   nofile   64000
root soft   nofile   64000
root hard   nofile   64000
```


