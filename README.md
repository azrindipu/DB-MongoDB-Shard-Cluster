### HI THERE!!! 👋

This docker-compose file can create a shared mongoDB ENV with 2 clusters and a single MySQL. Go to the docker-compose.yml location and run the following commands.

```
sudo docker-compose up -d

sudo docker exec -it mongo-config-01 mongosh
rs.initiate({
    _id : "rs-config-server",
    members: [
        { _id: 0, host: "mongo-config-01:27017", "priority": 3 },
        { _id: 1, host: "mongo-config-02:27017", "priority": 2 },
        { _id: 2, host: "mongo-config-03:27017", "priority": 1 }
    ]
});
rs.status();

sudo docker exec -it shard-01-node-a mongosh
rs.initiate({
    _id : "rs-shard-01",
    members: [
        { _id: 0, host: "shard-01-node-a:27017", "priority": 2 },
        { _id: 1, host: "shard-01-node-b:27017", "priority": 1 }
    ]
});

rs.addArb("shard-01-node-c:27017");
rs.status();

sudo docker exec -it shard-02-node-a mongosh
rs.initiate({
    _id : "rs-shard-02",
    members: [
        { _id: 0, host: "shard-02-node-a:27017", "priority": 2 },
        { _id: 1, host: "shard-02-node-b:27017", "priority": 1 }
    ]
});
rs.addArb("shard-02-node-c");
rs.status();

sudo docker exec -it router-01 mongosh
use admin;
db.adminCommand({
    setDefaultRWConcern : 1,
    defaultWriteConcern: {w: "majority"}
  }
);
sh.addShard("rs-shard-01/shard-01-node-a:27017,shard-01-node-b:27017,shard-01-node-c:27017");
sh.addShard("rs-shard-02/shard-02-node-a:27017,shard-02-node-b:27017,shard-02-node-c:27017");

use db;
db.studentModelMongo.createIndex( {"institute" : "hashed" } );
sh.shardCollection("db.studentModelMongo", {"institute" : "hashed" });
```



Get the status of the DBs run the following command

```
sudo docker ps
```

