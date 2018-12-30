---
layout: post
title:  "Node.js 웹 애플리케이션을 Heroku에 Deploy 하기"
date:   2016-11-07 14:56:00
categories: Node.js
---

간단한 API 서버를 하나 구축해야 해서 일단 Node.js로 만들고 heroku를 이용해서 서비스를 띄워보려고 하는데...  
해보니 정말 간단해서 깜짝 놀랐다. 그냥 사이트에 나와 있는데로 따라해보니 2~3분도 안걸린다.  
우선 사전에 Node.js와 git이 로컬환경에 설치되어 있어야 하며 Path도 다 잡혀 있어야 한다.

1. Node.js로 구동되는 Express 앱을 하나 만든다.  
eclipse에서 nodeclipse 플러그인을 이용하거나 IntelliJ 또는 WebStorm으로 새 프로젝트 만들기를 하여 Node.js(Express) 프로젝트를 선택하면 금방 간단한 껍데기 앱이 만들어진다. Express 앱 생성 후 해당 프로젝트 디렉터리에 가서 `npm install` 명령어를 실행하여 필요한 모듈들을 설치하고 그 후 `npm start` 명령어 실행...
![screenshot](./../../../../../images/20161107/1.png)

2. **http://localhost:3000** 접속해서 위와 같이 뜨면 일단 문제없이 되었다.

3. Express 앱의 루트 디렉터리에 Procfile 이라는 이름의 파일을 생성하고 내용으로 `web: node ./bin/www` 을 입력해놓고 저장.  
`web:` 다음 내용은 heroku 서버에서 앱을 Deploy 할 때 수행되는 명령어이다. forever나 2pm을 사용해서 돌린다면 얘네들을 이용해서 돌리는 명령어를 집어넣으면 될 듯...

4. heroku 상에서 Deploy 할 때, package.json 파일의 내용을 가지고 필요한 라이브러리들을 설치하므로, 앱 상에서 필요한 라이브러리는 package.json 상에 의존성이 정확히 적시되어 있어야 한다.  
로컬 개발환경에서 전역설치해놓으면 package.json에 적시 안되어 있어도 앱은 잘 돌아가는 경우가 있어, 이 부분을 주의.  
앱 소스 디렉터리에서 npm 라이브러리를 설치할 때 항상 npm install 라이브러리_명 -save 명령어로 설치해놓는다.
![screenshot](./../../../../../images/20161107/2.png)

5. Express 앱은 일단 그대로 냅두고 **heroku.com** 에 접속하여 Sign up을 클릭하여 계정을 만든다.
![screenshot](./../../../../../images/20161107/3.png)

6. 계정 만들고 로그인하면 위와 같이 대쉬보드로 접속되고 화면 오른쪽 상단의 New 버튼을 클릭하여 앱을 생성한다. 나는 앱을 두 개 만들어 놨다.  
앱은 heroku 전용 CLI로 만드는 방법이 일반적인 것 같은데 나는 UI쓰는게 편해서 생성은 그냥 웹사이트 상에서 했다.  
앱 생성 시에 입력해야 할 항목은 단 두 개인데 앱 이름과 서버 위치이다. 앱 이름은 소문자만 되고 서버는 유럽과 미국 두 군데 중 한 곳을 고르면 된다.
![screenshot](./../../../../../images/20161107/4.png)

7. 앱을 생성하고 나서 대쉬보드에서 해당 앱 항목을 클릭해 Deploy 탭을 선택하면 어떻게 Deploy 하는지 친절하게 설명되어 있다.  
그냥 이 페이지에 나와 있는대로 따라서 하면 올린 소스에 문제가 있지 않는 한 바로 된다.  
대략 원리는 heroku에서 제공하는 Git Repository를 remote로 잡고 로컬에 있는 소스를 push하면 pipeline을 통해 소스 전송받고 프레임워크 식별한 다음 자동으로 Deploy하여 띄우는 것 같다.

8. [https://devcenter.heroku.com/articles/heroku-command-line][1] 에서 heroku 전용 CLI툴을 사용 운영체제에 맞는 것을 골라 설치한다.(여기서는 윈도우즈10)
![screenshot](./../../../../../images/20161107/5.png)

9. 별도의 CLI 콘솔 프로그램을 구동시켜야 하는 줄 알았더니 그냥 윈도우즈의 프롬프트 창에서 heroku 배치파일 명령어를 실행하는 것으로 구현한다.  
`heroku login` 명령어를 입력해서 **heroku.com**의 계정정보를 입력하여 로그인 세션 생성.

10. `heroku apps` 명령어로 생성해 둔 앱 목록을 확인한다.

11. 예전 1번에서 만들어 놓은 Express 앱 소스 디렉터리로 들어가서 `git init` 명령어로 로컬 Repository를 생성하고 나서 `heroku git:remote -a heroku에_생성해놓은_앱_이름` 명령어로 heroku.com의 Git Repository를 원격 Repository로 설정한다.

12. `git add .`, `git commit -am "init commit"` 명령어로 로컬 Repository에 소스를 커밋.
![screenshot](./../../../../../images/20161107/6.png)

13. 커밋하고 나서 `git push heroku master` 명령어로 heroku의 원격 Repository에 push하면 소스가 전송되면서 자동으로 프레임워크를 탐지하고 Deploy되는 과정이 로그로 쫘악~ 펼쳐진다.
![screenshot](./../../../../../images/20161107/7.png)

14. **https://앱_이름.herokuapp.com** 으로 접속해보면 잘 뜬다.  
3000포트를 URL뒤에 붙여줘야 하나? 생각했는데, 이것도 알아서 웹 서버 상에서 80포트로 서비스 되도록 설정을 해주는 듯...

이렇게 해놓고 보니 참 쉽고 생산성 부분에 대해서는 다시금 감탄하게 되는데 아무래도 정식 서비스로 사용하기에는 부족한 듯하다.
일단 서버가 해외에 있는 탓인지 내가 올려놓은 Node 소스에는 국내 서버와 I/F하는 부분이 있는데 좀 느린 것 같다.  
웹 서버 말고도 PostgreSQL DB도 호스팅하고 있길래 DB계정도 하나 만들어봤는데 사용 못할 정도로 느리다.
그리고 접속을 잘 안하고 있으면 웹 서버가 Sleep 모드로 되서, 간만에 재접속하려고 하면 깨우느라(?) 응답에 조금 시간이 걸린다.

결국 쓸만하게 만드려면 유료로 전환해야 할 듯... 그래도 프로토타입이나 테스트 용도로 쓰기에는, 매우 괜찮아 보인다.

[1]: https://devcenter.heroku.com/articles/heroku-command-line