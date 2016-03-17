---
layout: post
title: Rabbit MQ Clustering 설정
weight: 50
excerpt: "Rabbit MQ를 여러서버에 설치하여 동시처리 능력을 향상시키는 Clustering 설정에 대해서 알아본다"
tags: [rabbitMQ, oss]
modified: 2016-03-05
comments: true
category : rabbit
icon : wrench.ico
---


RabbitMQ Clustering
===================

LAN환경하에 RabbitMQ를 수평 확장 할 수 있는 가장 기본적인 방법이 `Clustering`이다.
이는 기본적으로 RabbitMQ서버들의 데이터를 replication하여 함께 공유하는 방법이다.


Clustering 방법
-------------------

RabbitMQ는 여러 방법으로 클러스터 가능

1. rabbitmqctl 통해서 수동으로 하기
2. config file에 cluster할 node들을 선언해서 하기
3. rabbitmq-autocluster 플러그인 사용하기
4. rabbitmq-cluster 플러그인 사용하기


이 방법들 중 가장 기본이 되는 RabbitMQ Commandline tool을 이용한 1번 설정방법에 대해서 구체적으로 알아본다. 



### 노드들간의 인증 준비

RabbitMQ는 Erlang Cookie라는 값을 미리 교환하여 서로를 인증한다.

`/var/lib/rabbitmq/.erlang.cookie` or `$HOME/.erlang.cookie`

경로에 파일로 존재하며 rabbitMQ 설치시 자동으로 생기는 값이므로 클러스터링 할 노드 중 1개의 값을 복사하여

다른 노드의 파일에 덮어써서 서로간에 일치시켜주면 된다.


### 여러서버의 노드들간의 클러스터 구성하기

실환경에서는 RabbitMQ를 각 서버에 설치하고 서버를 구성하여 동시처리 성능을 향상시킨다. 앞선 쿠키 교환도 이러한 다른 서버들간의 통신을 위해 필요한 과정이며, 이러한 서버들간의 서로를 인지하기 위한 구분체계의 설정 또한 되어야 한다.

`rabbit@TestServer` 식의 `{node명}@{서버Host명}` 방식으로 서로를 구분하며, 필요에 따라 `{서버Host명}`은 `/etc/hosts`파일 등에 사전교환되어 설정하여야 한다.

필자는 단일 서버환경만 가지고 있으므로 일단은 아래처럼 단일 서버에 여러 노드를 띄워서 클러스터 구성하는 방법으로 간단히 클러스터 구성하고 해제하는 법에 대해서 테스트해보고자 함.


### 단일서버에 다중노드로 클러스터 구성하기

RabbitMQ를 설치한 서버에 노드라는 단위로 여러 프로세스를 실행하여 사용할 수 있다.
이 프로세스끼리 클러스터로 연결하여 구성을 테스트해 볼 수 있다.


단일서버에서 노드를 실행 할 때 같은 config 설정을 바라보므로 포트 등이 충돌 할 수 있다. 이를 방지하기 위해 Runtime의 환경설정값을 사용하여 충돌을 피한다.

#### Node1 실행

~~~
RABBITMQ_NODE_PORT=5672 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15672}]" RABBITMQ_NODENAME=node1 rabbitmq-server -detached
~~~

#### Node2 실행

~~~
RABBITMQ_NODE_PORT=5673 RABBITMQ_SERVER_START_ARGS="-rabbitmq_management listener [{port,15673}]" RABBITMQ_NODENAME=node2 rabbitmq-server -detached
~~~

#### Cluster되지 않은 상태 확인

~~~~~~
$ sudo rabbitmqctl -n node1 cluster_status

Cluster status of node 'node1@tester-VirtualBox' ...
[{nodes,[{disc,['node1@tester-VirtualBox','node3@tester-VirtualBox']}]},
 {running_nodes,['node1@tester-VirtualBox']},
 {cluster_name,<<"node1@tester-VirtualBox">>},
 {partitions,[]},
 {alarms,[{'node1@tester-VirtualBox',[]}]}]

$ sudo rabbitmqctl -n node2 cluster_status
Cluster status of node 'node2@tester-VirtualBox' ...
[{nodes,[{disc,['node2@tester-VirtualBox']}]},
 {running_nodes,['node2@tester-VirtualBox']},
 {cluster_name,<<"node2@tester-VirtualBox">>},
 {partitions,[]},
 {alarms,[{'node2@tester-VirtualBox',[]}]}]

~~~~~~


### Node2를 Node1에 Cluster로 설정

node2를 stop_app으로 running상태를 멈추고 join_cluster명령으로 cluster 설정해야 한다.

~~~~~~
sudo rabbitmqctl -n node2 stop_app

sudo rabbitmqctl -n node2 join_cluster node1@tester-VirtualBox

sudo rabbitmqctl -n node2 start_app

~~~~~~


#### Cluster된 상태 확인

node1과 node2의 cluster_status 명령을 통해 Cluster된 상태를 확인 할 수 있다.

~~~~~~

$ sudo rabbitmqctl -n node1 cluster_status
Cluster status of node 'node1@tester-VirtualBox' ...
[{nodes,[{disc,['node1@tester-VirtualBox','node2@tester-VirtualBox']}]},
 {running_nodes,['node2@tester-VirtualBox','node1@tester-VirtualBox']},
 {cluster_name,<<"node1@tester-VirtualBox">>},
 {partitions,[]},
 {alarms,[{'node2@tester-VirtualBox',[]},{'node1@tester-VirtualBox',[]}]}]
 
$ sudo rabbitmqctl -n node2 cluster_status
Cluster status of node 'node2@tester-VirtualBox' ...
[{nodes,[{disc,['node1@tester-VirtualBox','node2@tester-VirtualBox']}]},
 {running_nodes,['node1@tester-VirtualBox','node2@tester-VirtualBox']},
 {cluster_name,<<"node1@tester-VirtualBox">>},
 {partitions,[]},
 {alarms,[{'node1@tester-VirtualBox',[]},{'node2@tester-VirtualBox',[]}]}]

~~~~~~


#### Cluster 서버중 1번서버 중지 & 재시작

Cluster에 포함된 node1이 중지되어도 node2에 의해서 동작이 가능하다.
running상태가 node2하나 뿐인 것으로 표시된다.

~~~~~~~~
$ sudo rabbitmqctl -n node1 stop

$ sudo rabbitmqctl -n node2 cluster_status
Cluster status of node 'node2@tester-VirtualBox' ...
[{nodes,[{disc,['node1@tester-VirtualBox','node2@tester-VirtualBox']}]},
 {running_nodes,['node2@tester-VirtualBox']},
 {cluster_name,<<"node1@tester-VirtualBox">>},
 {partitions,[]},
 {alarms,[{'node2@tester-VirtualBox',[]}]}]
~~~~~~~~

node1을 다시 시작하면 같은 Cluster인 node2는 자동적으로 node1이 살아났음을 감지한다.


##### Cluster 서버 중지시 주의사항

1. 재기동 순서
cluster서버를 순차적으로 모두 중지하고 재기동 할 때, 재기동 순서에 주의해야한다.
가장 마지막에 stop된 노드가 가장먼저 재기동해야 한다. (그렇지 않은 노드를 먼저 기동하려고 하면 30초뒤에 에러를 내며 기동되지않는다.)

2. 동시에 죽었을 때
동시에 죽어서 모든 노드가 자기가 마지막 노드라고 생각하지 않을때, force_boot를 사용하여 


#### 노드를 Cluster에서 제외하기


##### 일반적인 방법

제거하고자하는 노드에서 작업을 수행하는 방법.

node2를 stop_app, reset하면 node2가 같은 Cluster에서 제외되었음을 다른 노드들이 감지한다.

~~~~~~~~
$ sudo rabbitmqctl -n node2 stop_app
Stopping node 'node2@tester-VirtualBox' ...

$ sudo rabbitmqctl -n node2 reset
Resetting node 'node2@tester-VirtualBox' ...

$ sudo rabbitmqctl -n node1 cluster_status
Cluster status of node 'node1@tester-VirtualBox' ...
[{nodes,[{disc,['node1@tester-VirtualBox']}]},
 {running_nodes,['node1@tester-VirtualBox']},
 {cluster_name,<<"node1@tester-VirtualBox">>},
 {partitions,[]},
 {alarms,[{'node1@tester-VirtualBox',[]}]}]
~~~~~~~~


##### Remote에서 수행하는 방법

제거하고자하는 노드를 살아있는 노드에서 제거하는 방법

node2에서 node1과의 cluster를 끊는다. node1이 stop_app된 상태가 아닌 상태에서 forget_cluster_node node1@`hostname -s` 명령어로
node1을 제거하려고하면 에러가 발생한다. (아직 running중이라고)

forget_cluster_node로 원격의 node1을 제거하면 node2에는 갱신된 Cluster의 상태를 알고 있지만 node1은 이 사실을 모른다.
이 때문에 node1이 start_app하려고 할 때 에러가 발생하므로 node1을 reset해주고 다시 기동해야 한다.


~~~~~~~~
$ sudo rabbitmqctl -n node1 stop_app
Stopping node 'node1@tester-VirtualBox' ...

$ sudo rabbitmqctl -n node2 forget_cluster_node node1@`hostname -s`
Removing node 'node1@tester-VirtualBox' from cluster ...

$ sudo rabbitmqctl -n node1 cluster_status
Cluster status of node 'node1@tester-VirtualBox' ...
[{nodes,[{disc,['node1@tester-VirtualBox','node2@tester-VirtualBox']}]},
 {alarms,[{'node2@tester-VirtualBox',[]}]}]
tester@tester-VirtualBox:~$ sudo rabbitmqctl -n node2 cluster_status
Cluster status of node 'node2@tester-VirtualBox' ...
[{nodes,[{disc,['node2@tester-VirtualBox']}]},
 {running_nodes,['node2@tester-VirtualBox']},
 {cluster_name,<<"node1@tester-VirtualBox">>},
 {partitions,[]},
 {alarms,[{'node2@tester-VirtualBox',[]}]}]

$ sudo rabbitmqctl -n node1 reset
Resetting node 'node1@tester-VirtualBox' ...

$ sudo rabbitmqctl -n node1 start_app

$ sudo rabbitmqctl -n node1 cluster_status
Cluster status of node 'node1@tester-VirtualBox' ...
[{nodes,[{disc,['node1@tester-VirtualBox']}]},
 {running_nodes,['node1@tester-VirtualBox']},
 {cluster_name,<<"node1@tester-VirtualBox">>},
 {partitions,[]},
 {alarms,[{'node1@tester-VirtualBox',[]}]}]
~~~~~~~~




#### DISC와 RAM 노드 타입 설정

Cluster로 구성할 때 2가지 Type으로 구성할 수 있다.

**disc**
: 기본type으로 disk에 쓰는 모드.(persistent queue경우)

**ram**
: 메타정보를 오직 메모리에만 보관하는 모드. 성능 향상에 도움이되지만 서버문제시 데이터가 날아갈 우려가 있다.


##### join_cluster시 타입 설정

~~~~~~~~
rabbitmqctl join_cluster --ram rabbit@rabbit1
~~~~~~~~


##### 이미 cluster된 노드의 타입 변경

~~~~~~~~
$ sudo rabbitmqctl -n node2 stop_app
Stopping node 'node2@tester-VirtualBox' ...

$ sudo rabbitmqctl -n node2 change_cluster_node_type ram
Turning 'node2@tester-VirtualBox' into a ram node ...

$ sudo rabbitmqctl -n node2 start_app

$ sudo rabbitmqctl -n node2 cluster_status
Cluster status of node 'node2@tester-VirtualBox' ...
[{nodes,[{disc,['node1@tester-VirtualBox']},
         {ram,['node2@tester-VirtualBox']}]},
 {running_nodes,['node1@tester-VirtualBox','node2@tester-VirtualBox']},
 {cluster_name,<<"node1@tester-VirtualBox">>},
 {partitions,[]},
 {alarms,[{'node1@tester-VirtualBox',[]},{'node2@tester-VirtualBox',[]}]}]
~~~~~~~~
