---
layout: post  
title:  "SpringBoot 버전 2.7 이상에서 MariaDB(RDS)의 Aurora 옵션 미지원"  
date:   2022-09-04 15:00:00  
categories: AWS SpringCloudAWS RDS MariaDB
---

AWS RDS에서 MariaDB 기반의 Aurora 클러스터를 사용한다면, FailOver를 위해 보통 DB Connection URL에 __aurora__ 옵션을 넣어서 사용한다.

![application.yml](./../../../../../../../images/20220904/1.png)

이 URL의 의미는 아래와 같다.

| Mode           | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| __sequential__ | This mode supports connection failover in a multi-master environment, such as MariaDB Galera Cluster. This mode does not support load-balancing reads on slaves. <br> The connector will try to connect to hosts in the order in which they were declared in the connection URL, so the first available host is used for all queries. For example, let's say that the connection URL is the following: `jdbc:mariadb:sequential:host1,host2,host3/testdb`<br>When the connector tries to connect, it will always try host1 first. If that host is not available, then it will try host2. etc. When a host fails, the connector will try to reconnect to hosts in the same order.<br>This mode has been available since [MariaDB Connector/J 1.3.0](https://mariadb.com/kb/en/mariadb-connector-j-130-release-notes/) |
| __replication__ | This mode supports connection failover in a master-slave environment, such as a MariaDB Replication cluster. The mode supports environments with one or more masters. This mode does support load-balancing reads on slaves if the connection is set to read-only before executing the read. The connector performs load-balancing by randomly picking a slave from the connection URL to execute read queries for a connection.<br>This mode has been available since [MariaDB Connector/J 1.3.0](https://mariadb.com/kb/en/mariadb-connector-j-130-release-notes/) |
| __loadbalance__ | This mode permits load-balancing connection in a multi-master environment, such as MariaDB Galera Cluster. This mode does not support load-balancing reads on slaves. The connector performs load-balancing for all queries by randomly picking a host from the connection URL for each connection, so queries will be load-balanced as a result of the connections getting randomly distributed across all hosts.<br>This mode has been available since [MariaDB Connector/J 1.3.0](https://mariadb.com/kb/en/mariadb-connector-j-130-release-notes/)|
| __aurora__ | This mode supports connection failover in an Amazon Aurora cluster. This mode does support load-balancing reads on slave instances if the connection is set to read-only before executing the read. The connector performs load-balancing by randomly picking a slave instance to execute read queries for a connection.<br>This mode has been available since [MariaDB Connector/J 1.3.0](https://mariadb.com/kb/en/mariadb-connector-j-130-release-notes/)|

그러나,

![waslog](./../../../../../../../images/20220904/2.png)

스프링부트 최신 버전 (2.7.x 이상)에서는 이 옵션을 인식하지 못해서 Application이 실행되지 않는다.  
이유인즉슨 aurora 옵션은 Maria DB Driver 1.2까지만 지원하는데, 스프링부트 2.7 부터는 Maria DB Driver 3.0 버전을 사용하기 때문이다.  
FailOver 기능 유지를 위해서 aurora 옵션을 sequential, replication, loadbalance 옵션 등으로 변경하여 사용할 수 있다.