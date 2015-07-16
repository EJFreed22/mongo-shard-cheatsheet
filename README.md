#Sharded Cluster Shd_set1
```
docker run -d --name svr1_mond1 --expose=27017 dev/mongod --replSet shd_set1
docker run -d --name svr1_mond2 --expose=27017 dev/mongod --replSet shd_set1
docker run -d --name svr1_mond3 --expose=27017 dev/mongod --replSet shd_set1
```

#Sharded Cluster Shd_set2
```
docker run -d --name svr2_mond1 --expose=27017 dev/mongod --replSet shd_set2
docker run -d --name svr2_mond2 --expose=27017 dev/mongod --replSet shd_set2
docker run -d --name svr2_mond3 --expose=27017 dev/mongod --replSet shd_set2
```

#Config Servers
```
docker run --name cfg1 -P -d dev/mongod --configsvr --dbpath /data/db --port 27017
docker run --name cfg2 -P -d dev/mongod --configsvr --dbpath /data/db --port 27017
docker run --name cfg3 -P -d dev/mongod --configsvr --dbpath /data/db --port 27017
```

#Mongo Router(s)
```
docker run --name mongos1 -P -d dev/mongos --configdb 172.17.0.70:27017,172.17.0.71:27017,172.17.0.72:27017 --port 27017
docker run --name mongos2 -P -d dev/mongos --configdb 172.17.0.70:27017,172.17.0.71:27017,172.17.0.72:27017 --port 27017
```

#Initialize Shd_set1
```
rs.initiate()
rs.add("172.17.0.65:27017")
rs.add("172.17.0.66:27017")
rs.status()
```

#Modify Primary Host
```
cfg = rs.conf()
cfg.members[0].host = "172.17.0.64:27017"
rs.reconfig(cfg)
rs.status()
```

#Initialize Shd_set2
```
rs.initiate()
rs.add("172.17.0.68:27017")
rs.add("172.17.0.69:27017")
rs.status()
```

#Modify Primary Host
```
cfg = rs.conf()
cfg.members[0].host = "172.17.0.67:27017"
rs.reconfig(cfg)
rs.status()
```

#Add Replica Sets as Shards
```
sh.addShard("shd_set1/172.17.0.64:27017")
sh.addShard("shd_set2/172.17.0.67:27017")
sh.status()
```

#HAProxy Load Balancing
https://github.com/dockerfile/haproxy.git
```
docker build -t dev/haproxy .
docker run --name haproxy1 -d -p 27017:27017 dev/haproxy
```
