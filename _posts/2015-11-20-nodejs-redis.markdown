---
layout: post
title:  "Node.js 프로그래밍 과정 5일차"
date:   2015-11-20 17:30:00
categories: Node.js
---

**2015.11.20 T아카데미 Node.js 프로그래밍 강좌를 수강하면서 필기.**

소스자료 : [http://nashorn.tistory.com/entry/TNODE][1]  
Node.js 프로그래밍 샘플 예제
 
##### Redis (NoSQL)
 
[https://github.com/dmajkic/redis/downloads][2]  
설치파일 다운 받아 압축 풀고 실행
![redis](./../../../../../images/20151120/01.png)
Redis 접속. Redis는 Key:Value 형식으로 데이터가 구성되는 NoSQL이다.  
`set foo bar` → `{foo:'bar'}` 데이터 생성  
`get foo` → `key:'foo'`인 데이터 조회
`subscribe chat` → subcribe 객체를 생성(publish되는 메시지를 받는다)
![redis](./../../../../../images/20151120/02.png)
다른 프롬프트 창으로 Redis에 접속하여 publish 객체를 생성하고 hello 메시지 전송
![redis](./../../../../../images/20151120/03.png)
메시지가 subcribe객체로 수신된 것을 알 수 있다.
![redis](./../../../../../images/20151120/04.png)
Mongoose : 다른 방식으로 MongoDB와 연결, 스키마를 만들어서 쓰기/읽기를 한다.
![js](./../../../../../images/20151120/05.png)

[1]: http://nashorn.tistory.com/entry/TNODE
[2]: https://github.com/dmajkic/redis/downloads