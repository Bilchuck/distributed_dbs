## 1) Сконфігурувати кластер з 3-х нод:

Let's start 3 cassandra nodes:
```
docker run --name cas1 -p 9042:9042 -e CASSANDRA_CLUSTER_NAME=MyCluster -e CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch -e CASSANDRA_DC=datacenter1 -d cassandra

docker run --name cas2 -e CASSANDRA_SEEDS="$(docker inspect --format='{{ .NetworkSettings.IPAddress }}' cas1)" -e CASSANDRA_CLUSTER_NAME=MyCluster -e CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch -e CASSANDRA_DC=datacenter2 -d cassandra


docker run --name cas3 -e CASSANDRA_SEEDS="$(docker inspect --format='{{ .NetworkSettings.IPAddress }}' cas1)" -e CASSANDRA_CLUSTER_NAME=MyCluster -e CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch -e CASSANDRA_DC=datacenter3 -d cassandra
```

Let's look at running containers
```
docker ps

ad850eb2120b        cassandra           "docker-entrypoint.s…"   4 seconds ago       Up 3 seconds        7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                 cas3
59bbdddb5791        cassandra           "docker-entrypoint.s…"   8 seconds ago       Up 7 seconds        7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                 cas2
3b1dd3900245        cassandra           "docker-entrypoint.s…"   12 seconds ago      Up 11 seconds       7000-7001/tcp, 7199/tcp, 9160/tcp, 0.0.0.0:9042->9042/tcp   cas1
```

## 2) Перевірити правильність конфігурації за допомогою nodetool status


```
docker exec -ti cas1 nodetool status

Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.2  108.62 KiB  256          100.0%            a600d060-7bcc-43ec-a742-0d9ce0c22aae  rack1
Datacenter: datacenter2
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  93.97 KiB  256          100.0%            67d2f875-f033-42db-b5f6-4ff6b1b34ba7  rack1
```

## 3) Викоритовуючи  cqlsh, створити три Keyspace: replication factor 1, 2, 3

```
docker exec -ti cas1 cqlsh

CREATE KEYSPACE replication1
WITH replication = {
	'class' : 'NetworkTopologyStrategy',
	'datacenter1' : 1
};

CREATE KEYSPACE replication2
WITH replication = {
	'class' : 'NetworkTopologyStrategy',
	'datacenter1' : 1,
  'datacenter2' : 1
};

CREATE KEYSPACE replication3
WITH replication = {
	'class' : 'NetworkTopologyStrategy',
	'datacenter1' : 1,
	'datacenter2' : 1,
  'datacenter3' : 1
};
```

## 4) В кожному з кейспейсів створити таблиці 

```
  CREATE TABLE replication1.products (
    id UUID,
    name text,
    PRIMARY KEY (id)
  );

CREATE TABLE replication2.products (
  id UUID,
  name text,
  PRIMARY KEY (id)
);

CREATE TABLE replication3.products (
  id UUID,
  name text,
  PRIMARY KEY (id)
);
```

## 5) Спробуйте писати і читати на / та з різних нод.

Let's write data in every keyspace in every replica:
```
docker exec -ti cas1 cqlsh

INSERT INTO replication1.products
(id,name)
VALUES (now(), 'repl1-product1');

INSERT INTO replication2.products
(id,name)
VALUES (now(), 'repl2-product1');

INSERT INTO replication3.products
(id,name)
VALUES (now(), 'repl3-product1');
```

And read

```
cqlsh> SELECT * FROM replication1.products;

 id                                   | name
--------------------------------------+----------------
 3a31a2b0-4a3e-11e9-a9d9-6d2c86545d91 | repl1-product1

(1 rows)
cqlsh> SELECT * FROM replication3.products;

 id                                   | name
--------------------------------------+----------------
 3e21a190-4a3e-11e9-a9d9-6d2c86545d91 | repl3-product1

(1 rows)
cqlsh> SELECT * FROM replication2.products;

 id                                   | name
--------------------------------------+----------------
 3c4ae5c0-4a3e-11e9-a9d9-6d2c86545d91 | repl2-product1

(1 rows)
```
By the results above we can see that all data is available in from all nodes.

## 6) Вставте дані в створені таблиці і подивіться на їх розподіл по вузлах кластера (для кожного з кейспесов - nodetool status)

```
docker exec cas1 nodetool status replication1
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.2  109.13 KiB  256          100.0%            a600d060-7bcc-43ec-a742-0d9ce0c22aae  rack1
Datacenter: datacenter2
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  92.42 KiB  256          0.0%              67d2f875-f033-42db-b5f6-4ff6b1b34ba7  rack1
Datacenter: datacenter3
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.4  94.99 KiB  256          0.0%              4c8fcedb-6b47-4dc5-8a80-33bac89bdd64  rack1
```

```
docker exec cas1 nodetool status replication2
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.2  109.13 KiB  256          100.0%            a600d060-7bcc-43ec-a742-0d9ce0c22aae  rack1
Datacenter: datacenter2
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  102.25 KiB  256          100.0%            67d2f875-f033-42db-b5f6-4ff6b1b34ba7  rack1
Datacenter: datacenter3
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.4  94.99 KiB  256          0.0%              4c8fcedb-6b47-4dc5-8a80-33bac89bdd64  rack1
```

```
docker exec cas1 nodetool status replication3
Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.2  109.13 KiB  256          100.0%            a600d060-7bcc-43ec-a742-0d9ce0c22aae  rack1
Datacenter: datacenter2
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.3  102.25 KiB  256          100.0%            67d2f875-f033-42db-b5f6-4ff6b1b34ba7  rack1
Datacenter: datacenter3
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.17.0.4  94.99 KiB  256          100.0%            4c8fcedb-6b47-4dc5-8a80-33bac89bdd64  rack1
```

## 7) Для якогось запису з кожного з кейспейсу виведіть ноди на яких зберігаються дані

```
 docker exec cas1 nodetool getendpoints replication1 products  3a31a2b0-4a3e-11e9-a9d9-6d2c86545d91

172.17.0.2
```

```
docker exec cas1 nodetool getendpoints replication2 products 3c4ae5c0-4a3e-11e9-a9d9-6d2c86545d91

172.17.0.3
172.17.0.2
```

```
docker exec cas1 nodetool getendpoints replication3 products 8a95f7f0-4a3f-11e9-a9d9-6d2c86545d91

172.17.0.3
172.17.0.2
172.17.0.4
```
From above we see that data are distributed as we expected.

## 8) Для кожного з кейспейсів відключивши одну з нод визначить чи можемо гарантувати strong consistency, для читання та запису, змінюючи рівень consistency

Before killing nodes
```
cqlsh> CONSISTENCY
Current consistency level is ONE.
```
That is default confistency value

Let's kill 3rd node
```
docker kill a360cb17e606
```

And test how consistency work in that case
```
cqlsh> INSERT INTO replication3.products (id, name) VALUES (now(), 'product-3');
OK
```
But after changing consistency to three:
```
cqlsh> CONSISTENCY THREE
Consistency level set to THREE.

cqlsh> INSERT INTO replication3.products (id, name) VALUES (now(), 'product-3');

NoHostAvailable: 
```
It shows that data can not be writte as 3rd node is not in network.

Otherwise it wtill works woth TWO because two nodes are still working
```
cqlsh> CONSISTENCY TWO  
Consistency level set to TWO.
cqlsh> INSERT INTO replication3.products (id, name) VALUES (now(), 'product-3');
```

## 9) Зробить так щоб три ноди працювали, але не бачили одна одну по мережі (відключити зв'язок між ними)

Some `iptable` magic..

## 10) Для кейспейсу з replication factor 3 задайте рівень consistency рівним 1. Виконайте запис одного й того самого значення, з рівним primary key, але різними іншими значенням на кожну з нод (тобто створіть конфлікт)

```
CONSISTENCY ONE 
```

```
docker exect -it cas1 cqlsh

INSERT INTO replication3.products
(id, name)
VALUES ( 52eb76c0-4a46-11e9-bb05-6d2c86545d92, 'test1')
```

```
docker exect -it cas2 cqlsh

INSERT INTO replication3.products
(id, name)
VALUES ( 52eb76c0-4a46-11e9-bb05-6d2c86545d92, 'test2')
```
This is conflict situation, but no exception was raised because of replicas issolation

## 11) Об’єднайте ноди в кластер і визначте яке значення було прийнято кластером та за яким принципом

```
SELECT * FROM replication3.products

 id                                   | name
--------------------------------------+----------------
 56d58230-4a46-11e9-bb05-6d2c86545d91 | repl3-product1
 52eb76c0-4a46-11e9-bb05-6d2c86545d92 | test2
```
test2 object was decided to be true one. It is because test2's timestamp is bigger then test1 and by this rule Cassandra resolve the conflict.