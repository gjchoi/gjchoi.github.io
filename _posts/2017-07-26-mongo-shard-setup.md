---
layout: post
title: MongoDB - 몽고DB 샤드 구성
weight: 100
excerpt: "몽고DB를 여러 클러스터를 이용해서 샤드구성하는 방법을 소개한다."
tags: [mongoDB, oss]
modified: 2017-07-26
comments: true
category : mongo
icon : sticky-note.ico
---

몽고DB 샤드모드로 구동하기
========================================

서버2대를 몽고 클러스터로하여 샤드 모드를 구성함

0) 몽고DB 프로그램 다운로드

몽고DB 공식사이트에서 환경에 맞는 stable버젼의 프로그램을 다운로드 받는다(wget)


1) Config 서버 구성 (replication)
-------------------------------

### config.conf 파일 생성

~~~
storage:
  dbPath: "/home/test/db/data/configdb"
sharding:
  clusterRole: configsvr
replication:
  replSetName: conf_rep
security:
  keyFile: "/home/test/db/config/mongodb-keyfile"
~~~  

sharding > clusterRole을 "configsvr"로 지정해야 config서버로서 기동된다.
security > keyFile 을 설정하면 config서버-mongos-mongodb 간에 통신간 보안성이 좋아진다. 


### Config서버 기동

~~~
mongod --config config.conf
~~~

기본포트인 27019로 Config서버가 기동된다.


### 2번째 Config 서버 구성

위와 동일한 방법으로 다른서버에 Config 서버(Replication #2)를 구성한다.

### Config서버 replication 연결

첫번째 Config서버가 설치/기동된 노드에서 아래 명령어로 접근한다.

~~~
mongo --host localhost --port 27019
~~~

replication 구성명령을 실행한다.

~~~
rs.initiate(
  {
    _id: "conf_rep",
    configsvr: true,
    members: [
      { _id : 0, host : "cfg1.example.net:27019" },
      { _id : 1, host : "cfg2.example.net:27019" }
    ]
  }
)
~~~

Replication 상태확인

~~~
rs.status()
~~~


2) Shard MongoDB 구성
-------------------------------

### mongo.conf 설정파일 생성

~~~
systemLog:
   destination: file
   path: "/home/test/db/log/log.txt"
   logAppend: true
   logRotate: rename
storage:
   engine: wiredTiger
   wiredTiger:
      engineConfig:
         journalCompressor: snappy
      collectionConfig:
         blockCompressor: snappy
      indexConfig:
         prefixCompression: true
   dbPath: "/home/test/db/data"
   journal:
      enabled: true
      commitIntervalMs: 300
processManagement:
   fork: true
   pidFilePath: "/tmp/mongod.pid"
net:
   port: 20011
   bindIp:
   maxIncomingConnections: 1000
   unixDomainSocket:
      enabled: false
sharding:
  clusterRole: shardsvr
security:
  keyFile: /home/test/db/config/mongodb-keyfile
  authorization: enabled
~~~


### mongoDB no-auth 구동 및 계정생성

~~~
mongod --port 20011 --dbpath /home/test/db/data
~~~

mongo config를 사용하지 않고 단일 노드로 띄운다.
(주의! : security key나 옵션을 주지 않고 띄워야 계정을 만들 수 있다.)

#### mongodb 접근 및 admin계정생성

~~~
mongo --port 20011
~~~

~~~
use admin
db.createUser(
   {
     user: "admin",
     pwd: "passwd",
     roles:
       [
         { role: "root", db: "admin" }
       ]
   }
)
~~~

### mongodb 기동

~~~
mongod --config mongo.conf
~~~

mongo.conf에 설정된 20011 포트로 구동된다.

### N번째 Config 서버 구성

위와 동일한 방법으로 다른 샤드 mongodb도 설정한다. (계정추가도 각 샤드마다 해줘야 함)



3) mongos 구성
-------------------------------


### mongos.conf 생성

~~~
sharding:
  configDB: conf_rep/cfg1.example.net:27019,cfg1.example.net:27019
security:
  keyFile: "/home/test/db/config/mongodb-keyfile"
~~~  

주의 : config서버에 security keyFile과 동일한 key를 사용해야 접근 에러가 안난다. (키설정자체도 config서버와 mongos서버 둘다없거나 둘다 있거나 해야한다.)

### mongos no-auth모드 수행 및 계정생성

mongodb에서 생성해준 계정(shard별 계정)과 다르게 mongos에서도 계정을 생성해주어야 한다.

1) 위의 Config서버를 security > keyFile설정을 재거하고 다시 기동한다. (replication 모두)

2) mongos에 conf파일에서도 security > keyFile설정을 제거하고 다시 기동한다.


#### mongos기동

~~~
mongos --config mongos.conf
~~~

기본포트는 27017


#### mongos 접근

~~~
mongo --port 27017
~~~

~~~
use admin
db.createUser(
   {
     user: "admin",
     pwd: "passwd",
     roles:
       [
         { role: "root", db: "admin" }
       ]
   }
)
~~~

#### mongos 기동

위에 no-auth모드를 재거하고 security key 설정을 붙여서 config서버 및 mongos를 재기동한다


4) Shard 연결
-------------------------------

### mongos 접근

~~~
mongo --port 27017
~~~

### shard 등록

~~~
use admin

#no-auth 모드일때는 안해줘도됨
db.auth("admin","passwd");
~~~

~~~
sh.addShard( "s1-mongo1.example.net:20011")
sh.addShard( "s2-mongo1.example.net:20011")
...

# 각 샤드 mongoDB를 replication한 경우 replSet이름을 앞에 붙여서 등록
sh.addShard( "<replSetName>/s1-mongo1.example.net:20011")
sh.addShard( "<replSetName>/s2-mongo1.example.net:20011")
...
~~~

~~~
sh.enableSharding("test_database")
~~~

데이터베이스 샤드 활성화


5) 접속확인 및 샤드테스트
-------------------------------

### client로 접속

Robomongo나 mongo client등을 이용해서
mongos 주소에 접근한다.
(위의 과정에서 각 mongodb에 생성한 계정으로 mongodb주소로 직접 붙을 수 있지만 부분데이터만 확인가능하다)

샤드 특성에 맞게 라우팅 key값을 설정하여 collection 샤드설정을한다.

~~~
sh.shardCollection("<database>.<collection>", { <key> : "hashed" } )
~~~
  
mongos에서 데이터를 insert하여 key값에 따라서 잘들어가지는지 확인한다.

~~~
db.test_database.shard_collection.insert({ "key" : "test1" });
db.test_database.shard_collection.insert({ "key" : "test2" });
~~~
