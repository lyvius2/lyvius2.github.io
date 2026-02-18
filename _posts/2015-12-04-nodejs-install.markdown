---
layout: post
title:  "Node.js 설치"
date:   2015-12-04 14:40:00
categories: Node.js
---

[https://nodejs.org/en/download/][1]

위 사이트로 접속하여 Node.js를 다운받는다. 프로젝트에서 사용하는 버전은 ~~4.2.3~~이다.  
→ 지금 구버전 0.12로 변경했다. 오라클 접속 문제 때문에... [오라클에서는 이제야 4.2와 5.0도 지원한다고 밝혔지만][2],
별 짓을 다 해봐도 안붙음...;  
Live 서버의 OS를 업그레이드 해야하는 큰 문제로 발전할지 몰라 구 버전으로 돌아가기로 했다.  
(그래도 구 버전이라 해봐야 올해 2월에 릴리즈 된 것이다)
![screenshot](./../../../../../images/20151204/1.png)
설치는 간단한데 다운 받은 것을 서버에 올려놓고 `tar -xvf #다운받은 파일명#` 명령어로 압축을 풀기만 하면 된다.  
(안풀고 그냥 tar파일 상태로 돌려도 상관없음. 오히려 이쪽이 더 나은 방법 같기도 하고.)

Node.js 설치한 디렉터리로 들어가서 간단한 웹서버를 구동하는 js 파일 하나를 작성한다.
![linux screenshot](./../../../../../images/20151204/2.png)

js파일 하나 만들고 node 명령어로 js파일 구동.
![linux screenshot](./../../../../../images/20151204/3.png)

웹 브라우저에서 서버 IP와 포트번호 등 주소치고 들어가면 Hello World~
![screenshot](./../../../../../images/20151204/4.png)

[1]: https://nodejs.org/en/download/
[2]: https://blogs.oracle.com/opal/entry/node_oracledb_1_4_0