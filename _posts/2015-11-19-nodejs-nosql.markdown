---
layout: post
title:  "Node.js 프로그래밍 과정 4일차"
date:   2015-11-19 17:54:00
categories: Node.js
---

**2015.11.19 T아카데미 Node.js 프로그래밍 강좌를 수강하면서 필기.**

##### NoSQL
+ 데이터 비정규화
+ 어그리게이션
+ 어플리케이션 사이드 조인 → 연산을 단순화하여 속도를 빠르게 한다.
 
##### RDBMS(MySql) 연결
MariaDB 실행 후 테스트 데이터 삽입
![mariadb](./../../../../../images/20151119/12.png)
Node.js를 통한 INSERT 문 수행시 결과값인 results 객체에 아래와 같이 primary key값이 나온다.
![js](./../../../../../images/20151119/11.png)
 
##### MongoDB
1. MongoDB 설치 후 저장소로 쓸 디렉터리를 생성한다.
![mongodb](./../../../../../images/20151119/01.png)
2. 저장소 디렉터리를 지명하고 MongoDB를 실행한다.
![mongodb](./../../../../../images/20151119/02.png)
3. 아래와 같이 실행된다.(프롬프트창 유지)
![mongodb](./../../../../../images/20151119/03.png)
4. MongoDB Shell 실행  
위에서부터  
  1 - `show dbs;` - DB용량 체크  
  2 - `use moviest;` - 'moviest'라는 이름의 데이터베이스를 사용하거나 없으면 새로 만든다.  
  3 - `db.users.save({a:99});` - 'users'라는 이름의 Collection에 {a:99}라는 값을 갖는 객체를 저장한다.('users' Collection이 없으면 새로 만든다.)  
  4 - `db.users.find();` = 'users' Collection의 데이터를 조회.
![mongodb](./../../../../../images/20151119/04.png)
5. 명령어는 자바스크립트 문법과 유사하다. for 순환문을 이용하여 10개의 데이터를 생성하여 'users' Collection에 INSERT하는 예제.
![mongodb](./../../../../../images/20151119/05.png)
6. 데이터를 찾는 예제이다.(RDBMS의 SELECT에 해당)  
위에서부터  
  1 - a = 2 조건에 해당하는 데이터  
  2 - a의 값이 2보다 크고 8보다 작은 데이터 (AND)  
  3 - a의 값이 [3,4,5,6,7] 범위에 해당되는 데이터 (IN조건)  
  4 - a의 값이 3보다 작거나 7보다 큰 데이터 (OR)  
  5 - a의 값이 [3,4,5,6,7] 범위에 해당하지 않는 데이터 (NOT IN조건)
![mongodb](./../../../../../images/20151119/06.png)
 7. 데이터를 갱신하는 예제.  
'a = 3' 조건에 해당하는 데이터의 temp 항목 값을 10으로 업데이트. 
![mongodb](./../../../../../images/20151119/07.png)
스키마에 구애받지 않기 때문에, 아예 항목 명을 바꿔버리는 것도 가능하다. ObjectId의 값은 그대로 유지된다.
![mongodb](./../../../../../images/20151119/08.png)
8. 데이터 삭제 예제.  
위에서부터  
  1 - a의 값이 5보다 크면 삭제  
  2 - 전체 데이터를 삭제  
  3 - Collection 자체를 삭제
![mongodb](./../../../../../images/20151119/09.png)
9. MongoDB 종료 - 정상종료가 아닐 때 데이터가 깨질 위험이 있다.
![mongodb](./../../../../../images/20151119/10.png)
 
##### MongoDB의 ObjectId
 
데이터에는 저마다의 고유한 ObjectId가 있는데 이 값을 프런트엔드 <-> 백엔드 사이에 전송할 경우 String객체로 전송이 되며,
이럴 때는 createFromHexString 메서드를 이용해 다시 객체화 해줘야 쿼리에 사용할 때 제대로 적용이 된다.
![mongodb](./../../../../../images/20151119/13.png)
Node.js 소스코드에서 ObjectId 객체를 생성하여 createFromHexString 메서드로 객체화 해주는 예제.
![js](./../../../../../images/20151119/14.png)