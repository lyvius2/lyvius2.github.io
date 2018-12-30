---
layout: post
title:  "Node.js와 Oracle DB 붙이기"
date:   2015-12-09 16:11:00
categories: Node.js
---

MongoDB나 Redis, 게다가 같은 RDBMS인 MySQL 연결하는 것보다 훨씬 까다롭다.  
회사에서는 RDBMS로 무조건 Oracle을 쓰기 때문에 이 문제를 해결하지 않으면 Node.js로 개발하는 것은 어려워진다.  
MySQL이나 다른 NoSQL DB에 연결하려면 npm으로 드라이버 모듈(이 표현이 맞는지 모르겠지만)을 설치하고 require로 가져다 쓰면 되지만 Oracle의 경우 인스턴트클라이언트를 사전에 설치해놓아야 한다.  
가장 정확한 가이드는 오라클 사에서 GitHub에 만들어놓은 레퍼런스 설치 가이드다.
실제로 구글링해서 나온 내용들이 결국은 다 레퍼런스 가이드의 내용과 대동소이했고 하다가 안되서 결국 레퍼런스 가이드를 보고 천천히 따라해봤다.  
레퍼런스 설치 가이드 : [https://github.com/oracle/node-oracledb/blob/master/INSTALL.md][1]

설치 과정을 압축해서 열거하자면

1. Oracle 인스턴트클라이언트를 설치한다.(Basic, SDK)
[http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html][2]
![linux screenshot](./../../../../../images/20151209/1.png)
개발서버에서는 그냥 rpm으로 전역설치했는데 Live에서는 실제 서비스 하는 서버이니만큼 tar파일 다운받아 압축만 풀었다. 압축을 푼 경로는 **/opt/oracle/instantclient** 이다.

2. 설치한 인스턴트클라이언트의 Path를 설정한다.
![linux screenshot](./../../../../../images/20151209/2.png)
**ORACLE_HOME, OCI_LIB_DIR** : Oracle Instantclient 를 설치한 경로  
**OCI_INC_DIR** : Oracle Instantclient 중 SDK를 설치한 경로  
**LD_LIBRARY_PATH** : 라이브러리 Path에 ORACLE_HOME 디렉터리를 추가한다.  
참고로 라이브러리 Path 추가하는 것을 빼먹고 그냥 DB접속 시도하면 아래와 같이 에러가 난다.  
![linux screenshot](./../../../../../images/20151209/3.png)  
나도 여기에서 에러를 경험했는데 구글링하니까 [솔루션][3]이 나왔다.  
(역시 신의 사이트 스택오버플로우 T^T)

3. npm install oracledb 명령어로 프로젝트 디렉터리 내 드라이버 모듈을 설치한다.

4. js소스 내에서 `require('oracledb')` 명령어로 설치한 모듈을 가져다 쓴다.

...위와 같이 진행하면 끝인데 순조롭게 잘 되었으면 좋겠지만 나의 경우 온갖 에러들이 oracledb 모듈 설치 도중에 짜장 나타났다.  
실제 겪은 몇가지 트러블을 정리해보겠다.

* `Cannot find $OCI_INC_DIR/oci.h`  
![linux screenshot](./../../../../../images/20151209/4.png)
이건 그냥 ORACLE_HOME만 잡고 다른 Path를 안잡아서 그렇다.

* `make: g++` 명령을 찾지 못했음  
C 관련 문제인 것 같은데 `yum install -y gcc-c++` 실행해주었더니 해결되었다.

* `gyp ERR! stack Error: Python executable "python2" is v2.4.3, which is not supported by gyp.`  
Live서버에서 발생한 오류인데 서버에 설치되어 있는 Python이 2.4.3인데 이 버전을 지원하지 않는다는 것이다.
솔루션은 역시 [스택오버플로우][4]...  
우선 npm의 설정을 Python 2.7 버전으로 설정해준다. `npm config set python python2.7`  
그 다음 Python 2.7 버전을 다운로드 받아 서버에 설치하면 잘 된다. 설치하는 방법은 이 [웹사이트][5]를 참고하였다.  
(JDK 설치하는 것보다 다소 까다롭다)

* `gyp ERR! node -v v4.2.2`  
도저히 답이 안나와서 Node.js 버전을 0.12로 다운시켜서 진행했다(...)

![linux screenshot](./../../../../../images/20151209/5.png)
oracledb 모듈 설치까지 마치고 위와 같이 간단히 오늘 날짜를 구하는 쿼리를 짜서 app.js 파일에 넣어보았다.
![linux screenshot](./../../../../../images/20151209/6.png)
잘 된다...

[1]: https://github.com/oracle/node-oracledb/blob/master/INSTALL.md
[2]: http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html
[3]: http://stackoverflow.com/questions/29330841/libclntsh-so-12-1-cannot-open-shared-object-file-error-when-running-sample-of 
[4]: http://stackoverflow.com/questions/20454199/how-to-use-a-different-version-of-python-during-npm-install 
[5]: https://wikidocs.net/8