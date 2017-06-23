---
layout: post
title: Mongo DB Tip
excerpt: "개발하며 필요한 MongoDB 명렁어나 설정등에 대한 tip 모음"
tags: [tip, db]
modified: 2016-05-15
comments: true
category : etc
---

{% include _toc.html %}


데이터 관련 명령
==============================

~~~~~
db.getCollection('ContentInstance').find({ con : "data1" })

db.getCollection('ContentInstance').find({ con : "data1" }).count()

db.getCollection('ContentInstance').find( {con : "data1", path : {$regex : /^.*dir-20.*/ } })

db.getCollection('ContentInstance').find( {con : "data1", path : {$regex : /^dir-1\/content-20.*$/ } })
~~~~~


다중 update여러건을 한꺼번에
~~~
db.user_trace.updateMany(
   { logTimestamp : { $lt : new Date("2016-12-17T00:00:00Z") } },
   { $unset: { data: "" } }
)
~~~

다중 삭제

~~~
db.user_trace.remove({ logTimestamp : { $lt : new Date("2016-12-13T00:00:00Z") } });
~~~


권한설정 명령
==============================

!주의 : 반드시 db안에 들어가서 생성해야 그 db의 계정생성가능 ( use {DB} )

admin계정

~~~~~
db.createUser(
  {
    user: "admin",
    pwd: "!mgadmin2015",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
~~~~~


사용자 계정

~~~~~
db.createUser(
   {
     user: "test1",
     pwd: "test1",
     roles : [{ role: "dbOwner", db: "db1" }]
   }
)

db.createUser(
   {
     user: "user1",
     pwd: "user1",
     roles : [ { role: "userAdmin", db: "db1" },
               { role: "readAnyDatabase", db: "db1" },
                 "readWrite"
             ]
   }
)
~~~~~


db에 권한 부여

~~~~~
db.auth( {
   user: "user3"
   ,pwd: "user3"
} )
~~~~~


user 삭제

`db.dropUser("iotp");`


cli 접근

`mongo -u admin -p '!mgadmin2015'`


replacation 상태확인

`rs.status()`


function call권한 부여

~~~
db.createRole( { role: "executeFunctions", privileges: [ { resource: { anyResource: true }, actions: [ "anyAction" ] } ], roles: [] } ) 

db.grantRolesToUser("bi", [ { role: "executeFunctions", db: "admin" } ])
~~~


