## Set up Sharding using Docker Containers

### Config servers
Start config servers (3 member replica set)
```
docker-compose -f config-server/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://192.168.1.67:40001
```
```
rs.initiate(
  {
    _id: "cfgrs",
    configsvr: true,
    members: [
      { _id : 0, host : "192.168.1.67:40001" },
      { _id : 1, host : "192.168.1.67:40002" },
      { _id : 2, host : "192.168.1.67:40003" }
    ]
  }
)
rs.status()
```

### Shard 1 servers
Start shard 1 servers (3 member replicas set)
```
docker-compose -f shard1/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://192.168.1.67:50001
```
```
rs.initiate(
  {
    _id: "shard1rs",
    members: [
      { _id : 0, host : "192.168.1.67:50001" },
      { _id : 1, host : "192.168.1.67:50002" },
      { _id : 2, host : "192.168.1.67:50003" }
    ]
  }
)
rs.status()
```

### Mongos Router
Start mongos query router
```
docker-compose -f mongos/docker-compose.yaml up -d
```

### Add shard to the cluster
Connect to mongos
```
mongo mongodb://192.168.1.67:60000
```
Add shard
```
mongos> sh.addShard("shard1rs/192.168.1.67:50001,192.168.1.67:50002,192.168.1.67:50003")
mongos> sh.status()
```
## Adding another shard
### Shard 2 servers
Start shard 2 servers (3 member replicas set)
```
docker-compose -f shard2/docker-compose.yaml up -d
```
Initiate replica set
```
mongo mongodb://192.168.1.67:50004
```
```
rs.initiate(
  {
    _id: "shard2rs",
    members: [
      { _id : 0, host : "192.168.1.67:50004" },
      { _id : 1, host : "192.168.1.67:50005" },
      { _id : 2, host : "192.168.1.67:50006" }
    ]
  }
)
rs.status()
```
### Add shard to the cluster
Connect to mongos
```
mongo mongodb://192.168.1.67:60000
```
Add shard
```
mongos> sh.addShard("shard2rs/192.168.1.67:50004,192.168.1.67:50005,192.168.1.67:50006")
mongos> sh.status()
```
docker ps

### Example 
### Collection Sharding 

make a database 
```
use sharddemo
```

check db and Collections
```
db

```

Create Collections
```
db.createCollection("movies")

show Collections
```

db.movies.getShardDistribution()

sh.enableSharding("sharddemo")

sh.shardCollection("sharddemo.movies",{"title":"hashed"})

db.movies.getShardDistribution()
```

inserting data into mongodb collection 
```

db.movies.insertMany([
    {
        "title": "The Shawshank Redemption",
        "rank": "1",
        "id": "tt0111161"
    },
    {
        "title": "The Godfather",
        "rank": "2",
        "id": "tt0068646"
    },
    {
        "title": "The Godfather: Part II",
        "rank": "3",
        "id": "tt0071562"
    },
    {
        "title": "Pulp Fiction",
        "rank": "4",
        "id": "tt0110912"
    },
    {
        "title": "The Good, the Bad and the Ugly",
        "rank": "5",
        "id": "tt0060196"
    },
    {
        "title": "The Dark Knight",
        "rank": "6",
        "id": "tt0468569"
    },
    {
        "title": "12 Angry Men",
        "rank": "7",
        "id": "tt0050083"
    },
    {
        "title": "Schindler's List",
        "rank": "8",
        "id": "tt0108052"
    },
    {
        "title": "The Lord of the Rings: The Return of the King",
        "rank": "9",
        "id": "tt0167260"
    },
    {
        "title": "Fight Club",
        "rank": "10",
        "id": "tt0137523"
    }
])

db.movies.find() 
db.movies.count()

db.movies.getShardDistribution()



