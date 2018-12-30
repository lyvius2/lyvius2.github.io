---
layout: post
title:  "OSX 엘 캐피탄에서 iPython Notebook 설치"
date:   2016-02-20 00:07:00
categories: Python
---

Python을 활용한 데이터 분석에 관련된 강의를 주말마다 수강하고 있는데, iPython Notebook이라는 모듈 설치시 에러가 나서 이를 해결하는데 조금 애를 먹었다.  
OSX 최신 버전인 엘 캐피탄의 경우 발생하는 에러인데 나중에 가면 까먹을 것 같아 여기에 간단히 기록해둔다.
```
pip install "ipython[notebook]" 
```
엘 캐피탄에서 위 명령어로 설치 시도 시 아래와 같이 오류가 발생한다.
![screenshot](./../../../../../images/20160220/1.jpg)
~~우선 아래 명령어로 Numpy, Pandas, Matplotlib 등을 사전에 설치한다. pip로도 설치가 되긴 하지만, 나의 경우는 위와 같이 빨간글자 로그가 주르륵 떴다.~~
```
sudo easy_install numpy
sudo easy_install pandas
sudo easy_install matplotlib
sudo easy_install ipython
```
**위 명령어 다 개뿔 소용없고 아래의 절차 처럼 homebrew를 이용하여 Python을 재설치한 다음 평범하게 pip로 설치하면 된다.**  
easy_install을 사용해도 ipython 설치 시에는 오류가 발생했다.
![screenshot](./../../../../../images/20160220/2.jpg)
구글링에 돌입하니 뭔가 homebrew를 통한 방법이 있는 것으로 보인다.  
구체적으로는 아래 웹사이트의 내용이 참고가 되었다.

[https://gist.github.com/stevekm/21be14a0d65cdfae5ebb][1]
[http://stackoverflow.com/questions/33004708/osx-el-capitan-sudo-pip-install-oserror-errno-1-operation-not-permitted][2]

아래 명령어로 /usr/local 디렉터리에 퍼미션을 조정한다.
```
sudo chown $(whoami):admin /usr/local && sudo chown -R $(whoami):admin /usr/local
```
그 다음 [homebrew를 설치하는 명령어를 입력한다][3].
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
그리고 나서 homebrew를 사용해 Python을 재설치한다.
```
brew install python
```
다시 한번 명령어를 날려보면,
```
sudo pip install "ipython[notebook]"
```
![screenshot](./../../../../../images/20160220/3.png)
설치가 잘 된다...

참고로 나는 이것 외에 'brew doctor', 'brew install Caskroom/cask/jupyter-notebook-ql' 등의 명령어로 날려보았는데, 해결책과는 거리가 먼 액션인 것 같다.  
왜 하필 엘 캐피탄에서만 이런 이슈가 발생하는지는 내장 Python을 사용할 경우 보안설정으로 인해 잘 작동하지 않을 수도 있다고 한다.

[1]: https://gist.github.com/stevekm/21be14a0d65cdfae5ebb
[2]: http://stackoverflow.com/questions/33004708/osx-el-capitan-sudo-pip-install-oserror-errno-1-operation-not-permitted
[3]: http://brew.sh/index_ko.html