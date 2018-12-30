---
layout: post
title:  "로컬환경에서 WAS리부트 없이 Class 파일 Reloading하도록 설정 (HotSwap)"
date:   2018-03-11 20:27:00
categories: JAVA
---

[https://github.com/spring-projects/spring-loaded][1]
위 사이트에서 Spring-Loaded 라이브러리 파일(jar)를 다운받아 로컬환경 적당한 곳에 위치시킨다.

그리고 아래 같이 WAS 구동시 VM 옵션에 **-javaagent:/path/to/springloaded-1.2.8.RELEASE.jar** 를 추가한다.
![IntelliJ](./../../../../../images/20180311/995897455AA5114724.jpg)
IntelliJ Run 환경 설정에서 On 'Update' action, On frame deactivation 항목에 'Update classes and resources' 로 설정

템플릿 엔진으로 Thymeleaf를 쓴다면 application.properties 파일의 내용도 추가(수정)해준다.
```properties
spring.thymeleaf.cache=false
```
다른 방법도 있는 것 같은데 잘 될 때도 있고 안될 때도 있고... 환경을 좀 많이 타는 것 같다.

[1]: https://github.com/spring-projects/spring-loaded