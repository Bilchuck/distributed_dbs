## 1) Налаштувати реплікацію в конфігурації: Primary with Two Secondary Members (всі ноди можуть бути запущені як окремі процеси або у Docker контейнерах) 

Create a network: `docker network create mongo-replica-set`.

Create mongo instances indside the network:
```
docker run --net mongo-replica-set -d -p 4001:27017 --name mongo-1 mongo --replSet "rs0"
docker run --net mongo-replica-set -d -p 4002:27017 --name mongo-2 mongo --replSet "rs0"
docker run --net mongo-replica-set -d -p 4003:27017 --name mongo-3 mongo --replSet "rs0"
```

Connect to one of instances: `docker exec -it mongo-1 mongo`

Set replica config:
```
config = {
	"_id" : "rs0",
	"members" : [
		{
			"_id" : 0,
			"host" : "mongo-1:27017"
		},
		{
			"_id" : 1,
			"host" : "mongo-2:27017"
		},
		{
			"_id" : 2,
			"host" : "mongo-3:27017"
		}
	]
}
```
Initialize config: `rs.initiate(config)`

Look at mongo replica set status:

```
rs.status()

{
	"set" : "rs0",
	"date" : ISODate("2019-03-16T19:42:11.477Z"),
	"myState" : 1,
	"term" : NumberLong(1),
	"syncingTo" : "",
	"syncSourceHost" : "",
	"syncSourceId" : -1,
	"heartbeatIntervalMillis" : NumberLong(2000),
	"optimes" : {
		"lastCommittedOpTime" : {
			"ts" : Timestamp(1552765328, 1),
			"t" : NumberLong(1)
		},
		"readConcernMajorityOpTime" : {
			"ts" : Timestamp(1552765328, 1),
			"t" : NumberLong(1)
		},
		"appliedOpTime" : {
			"ts" : Timestamp(1552765328, 1),
			"t" : NumberLong(1)
		},
		"durableOpTime" : {
			"ts" : Timestamp(1552765328, 1),
			"t" : NumberLong(1)
		}
	},
	"lastStableCheckpointTimestamp" : Timestamp(1552765288, 1),
	"members" : [
		{
			"_id" : 0,
			"name" : "mongo-1:27017",
			"health" : 1,
			"state" : 1,
			"stateStr" : "PRIMARY",
			"uptime" : 388,
			"optime" : {
				"ts" : Timestamp(1552765328, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2019-03-16T19:42:08Z"),
			"syncingTo" : "",
			"syncSourceHost" : "",
			"syncSourceId" : -1,
			"infoMessage" : "could not find member to sync from",
			"electionTime" : Timestamp(1552765287, 1),
			"electionDate" : ISODate("2019-03-16T19:41:27Z"),
			"configVersion" : 1,
			"self" : true,
			"lastHeartbeatMessage" : ""
		},
		{
			"_id" : 1,
			"name" : "mongo-2:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 54,
			"optime" : {
				"ts" : Timestamp(1552765328, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1552765328, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2019-03-16T19:42:08Z"),
			"optimeDurableDate" : ISODate("2019-03-16T19:42:08Z"),
			"lastHeartbeat" : ISODate("2019-03-16T19:42:11.204Z"),
			"lastHeartbeatRecv" : ISODate("2019-03-16T19:42:09.726Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "mongo-1:27017",
			"syncSourceHost" : "mongo-1:27017",
			"syncSourceId" : 0,
			"infoMessage" : "",
			"configVersion" : 1
		},
		{
			"_id" : 2,
			"name" : "mongo-3:27017",
			"health" : 1,
			"state" : 2,
			"stateStr" : "SECONDARY",
			"uptime" : 54,
			"optime" : {
				"ts" : Timestamp(1552765328, 1),
				"t" : NumberLong(1)
			},
			"optimeDurable" : {
				"ts" : Timestamp(1552765328, 1),
				"t" : NumberLong(1)
			},
			"optimeDate" : ISODate("2019-03-16T19:42:08Z"),
			"optimeDurableDate" : ISODate("2019-03-16T19:42:08Z"),
			"lastHeartbeat" : ISODate("2019-03-16T19:42:11.204Z"),
			"lastHeartbeatRecv" : ISODate("2019-03-16T19:42:09.726Z"),
			"pingMs" : NumberLong(0),
			"lastHeartbeatMessage" : "",
			"syncingTo" : "mongo-1:27017",
			"syncSourceHost" : "mongo-1:27017",
			"syncSourceId" : 0,
			"infoMessage" : "",
			"configVersion" : 1
		}
	],
	"ok" : 1,
	"operationTime" : Timestamp(1552765328, 1),
	"$clusterTime" : {
		"clusterTime" : Timestamp(1552765328, 1),
		"signature" : {
			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
			"keyId" : NumberLong(0)
		}
	}
}
```
Here we see that 1st replica become PRIMARY and all other SECONDATY instances.

## 2) Продемонструвати запис даних на primary node з різними Write Concern Levels

### a) Unacknowledged
As I understand "Unacknowledged" mean that we require 0 nodes acknowledge from replicas and memory level from primary.
```
db.getCollection('products').save({
    "task" : 1.0,
    "sub" : "a",
    "try" : 1
}, { writeConcern: { w: 0, j: false } })
```

### b) Acknowledged
I used majority write level that requires majority replicas acknowledge.
```
db.getCollection('products').save({
    "task" : 1.0,
    "sub" : "b",
    "try" : 1
}, { writeConcern: { w: 'majority' } })
```

### c) Journaled
Journaled mean that we require from replica not memory level acknowledge but `disk` level.

```
db.getCollection('products').save({
    "task" : 1.0,
    "sub" : "c",
    "try" : 1
}, { writeConcern: { w: 'majority', j: true } })
```

### d) AcknowledgedReplica


## 3) Продемонструвати Read Preference Modes: читання з primary і secondary node

### a) primary
 All operations read from the current replica set primary.

`db.getCollection('products').find({}).readPref('primary')`

### b) primaryPreferred
 Operations read from the primary but if it is unavailable, operations read from secondary members

`db.getCollection('products').find({}).readPref('primaryPreferred')`

### c) secondary
 All operations read from the secondary members of the replica set.

`db.getCollection('products').find({}).readPref('secondary')`

### d) secondaryPreferred
In most situations, operations read from secondary members but if no secondary members are available, operations read from the primary.

`db.getCollection('products').find({}).readPref('secondaryPreferred')`

### e) nearest
Operations read from member of the replica set with the least network latency, irrespective of the member’s type.

`db.getCollection('products').find({}).readPref('nearest')`

## 4) Спробувати зробити запис з однією відключеною нодою та write concern рівнім 3 та нескінченім таймаутом. Спробувати під час таймаута включити відключену ноду 
Stop mongo-3 instance:
```
docker em cb3d1a6e797a
```
Make write command with `replica 3` and `timeout none`:
```
db.getCollection('products').save({
    "task" : 4,
    "try" : 1
}, { writeConcern: { w: 3, wtimeout: 0 } })
```
Now inserting is on `waiting` stage and I start new mongo-3 instance.
```
docker run --net mongo-replica-set -d -p 4003:27017 --name mongo-3 mongo --replSet "rs0"
```
After this insert is finished successfully.

## 5) Продемонстрував перевибори primary node в разі виходу з ладу поточної primary (Replica Set Elections)

Вбиваємо праймері ноду
```
docker kill df3ec14591ea
```
Підключаюсь до 2 ноди і перевіряю статус мережі
```
docker exec -it mongo-2 mongo
rs.status()
```
По результату бачу що 2 перпліка стала PRIMARY

## 6) Привести кластер до неконсистентного стану користуючись моментом часу коли primary node не відразу помічає відсутність secondary node 

### a) відключити останню secondary node та протягом 5 сек. на мастері записати значення і перевірити, що воно записалось

Let's kill 3rd node:
```
docker kill 641e88d0a238
```
Make insert request to 2nd node
```
db.getCollection('products').save({
    id: 123
})
```

### b) спробувати зчитати це значення з різними рівнями read concern

```
db.getCollection('products').find({}).readPref('primary')
Error: error: {
	"operationTime" : Timestamp(1552771421, 2),
	"ok" : 0,
	"errmsg" : "not master and slaveOk=false",
	"code" : 13435,
	"codeName" : "NotMasterNoSlaveOk"
}
```

```
db.getCollection('products').find({}).readPref('primaryPreferred')
/* 1 */
{
    "_id" : ObjectId("5c8d695d5beb31c7ac23b5af"),
    "id" : 123.0
}
```

```
db.getCollection('products').find({}).readPref('secondary')
/* 1 */
{
    "_id" : ObjectId("5c8d695d5beb31c7ac23b5af"),
    "id" : 123.0
}
```

```
db.getCollection('products').find({}).readPref('secondaryPreferred')
/* 1 */
{
    "_id" : ObjectId("5c8d695d5beb31c7ac23b5af"),
    "id" : 123.0
}
```

```
db.getCollection('products').find({}).readPref('nearest')
/* 1 */
{
    "_id" : ObjectId("5c8d695d5beb31c7ac23b5af"),
    "id" : 123.0
}
```

### c) включити дві інші ноди таким чином, щоб вони не бачили попереднього мастера (його можна відключити) і дочекатись поки вони оберуть нового мастера 

kill current node
```
docker kill 5ea34ae7a185
```
And start 1st and 3rd nodes
```
docker run --net mongo-replica-set -d -p 4001:27017 --name mongo-1 mongo --replSet "rs0"

docker run --net mongo-replica-set -d -p 4003:27017 --name mongo-3 mongo --replSet "rs0"
```

### d) підключити (включити) третю ноду до кластеру і подивитись, що сталось зі значенням яке було на неї записано

```
docker run --net mongo-replica-set -d -p 4002:27017 --name mongo-2 mongo --replSet "rs0"
```

1st node become a PRIMARY

## 7) Показати відмінності в поведінці між рівнями readConcern: {level: <"majority"|"local"| "linearizable">}

```
db.getCollection('products').find({}).readConcern('majority')
```
This will return data that has been acknowledged by a majority of the replica set members. It means that `123` id prouct would not be returned as it was saved only in one node.


```
db.getCollection('products').find({}).readConcern('local')
```
It will return different data depending to which node you do request. It return data from this node with no guarantee that the data has been written to a majority of the replica set members.


```
db.getCollection('products').find({}).readConcern('linearizable')
```
This request is similar ro "majority" but it wait for write operation that are in progress in moment of read request. It guarantee more consistency but take more time as well.

## 8) Земулювати eventual consistency за допомогою установки затримки реплікації для репліки

As 1st node is primary one, let's add some deplay to 2nd node.

Connect to 1st node.
```
cfg = rs.conf()
cfg.members[1].priority = 0
cfg.members[1].hidden = true
cfg.members[1].slaveDelay = 20
rs.reconfig(cfg)
```
Added deplay of 20 seconds

Insert to PRIMARY
```
db.getCollection('products').save({
    id: 2
})
```

2nd node would return this on find:
```
db.getCollection('products').find({})

/* 1 */
{
    "_id" : ObjectId("5c8d6e195beb31c7ac23b5b1"),
    "id" : 123.0
}
```

And after 20 seconds..

```
db.getCollection('products').find({})


/* 1 */
{
    "_id" : ObjectId("5c8d6e195beb31c7ac23b5b1"),
    "id" : 123.0
}

/* 2 */
{
    "_id" : ObjectId("5c8d6eb05beb31c7ac23b5b2"),
    "id" : 2.0
}
```