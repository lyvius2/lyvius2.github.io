---
layout: post
title:  "Tomcat 8.5에서의 특수문자 파일명/경로 처리"
date:   2019-01-05 00:37:10
categories: Tomcat WEB/WAS
---

최근 웹서버를 새로 셋팅할 일이 생겨 하는 김에 웹 애플리케이션의 WAS(Tomcat 7)를 Tomcat 8.5 버전으로 업그레이드를 시도했는데 이슈 하나를 발견하여 적용을 연기했다.
이제 해결했기 때문에 조만간 다시 반영 시도를 할 것이지만...

웹서버에는 NAS가 마운트되어 있고 웹에서 이 NAS에 위치한 (다른 시스템에서 업로드한)이미지 파일을 읽어들여 보여줘야 하는데 유독 몇개 파일에서 404에러가 떨어져서 웹 브라우저에서 엑박으로 표시되는 것이었다.  
같은 경로의 다른 파일들은 멀쩡히 잘 보이는데, 처음에는 한글파일명을 인식못하나? 싶어서 봤더니 이상없이 잘 보이는 이미지파일 중에서도 한글명으로 된 파일들이 있었고 Tomcat의 **server.xml** 설정 파일에도 Connector 속성에 `URIEncoding="UTF-8"` 옵션이 잘 들어가 있었다.

404에러가 발생하는 CASE를 보니 파일명 형식이
```
(#알파벳#)#한글#[#숫자].#확장자# 
```
예를 들면 `(abc)한글명[123].jpg` 이런 식이다.

파일명 앞의 괄호가 문제가 되나? 싶어서 파일명을 바꿔봤더니 그래도 안된다. 혹시나 해서 파일명 뒤의 대괄호를 URI Encoding해서 `(abc)한글명%5B123%5D.jpg` 로 바꾸어서 호출해봤더니  
**된다...**

UTF-8 처리해줬는데 여기에서 더 뭘 추가해줘야 하나 찾아봐야 했는데, WAS 보안상 URL경로와 파라메터에서 특수문자를 아예 막아놓은 것이었다.  
AS-IS의 Tomcat 7 설정파일에서
{% highlight xml %}
<Listener className="org.apache.catalina.core.JasperListener" />
{% endhighlight %}
이 부분만 주석처리해서 엎어친 다음(8버전에서 삭제된 클래스라 안하면 오류발생) 해봐도 안되는걸 봐서는 아마도 버전업하면서 기본 보안 설정을 더 엄하게 한 모양이다.

어떻게 처리해야 할지는 Tomcat 설정 파일 중 catalina.properties 파일 내용을 보면 가장 마지막 라인에 주석으로 명시해놓았다.
{% highlight properties %}
# This system property is deprecated. Use the relaxedPathChars relaxedQueryChars
# attributes of the Connector instead. These attributes permit a wider range of
# characters to be configured as valid.
# Allow for changes to HTTP request validation
# WARNING: Using this option may expose the server to CVE-2016-6816
#tomcat.util.http.parser.HttpParser.requestTargetAllow=|
{% endhighlight %}
원래는 catalina.properties 파일에 `tomcat.util.http.parser.HttpParser.requestTargetAllow` 프로퍼티 주석을 풀고 허용할 특수문자를 넣어줬으면 됐지만 deprecated 되었고 server.xml의 Connector 속성에 `relaxedPathChars`, `relaxedQueryChars` 두 옵션을 추가하라는 것이다.

`relaxedQueryChars`는 GET 메서드로 호출할 때 쿼리 스트링 형식의 파라메터에서 특수문자를 허용하는 옵션이고 나는 파일명 문제이므로 `relaxedPathChars` 옵션만 추가했다.
{% highlight xml %}
    <Connector port="8080" protocol="HTTP/1.1" 
               URIEncoding="UTF-8"
               relaxedPathChars="[]"
               ... />
{% endhighlight %}
이번 경우는 우선 대괄호가 문제여서 대괄호만 추가해서 되는지 안되는지 테스트해보았고 해보니 잘된다...

파일명에 다른 특수문자도 들어있을지 모르니, 대괄호 외에 다른 특수문자도 허용하고 싶으면 `relaxedQueryChars` 옵션에 허용할 특수문자를 추가로 넣으면 된다.  
사실 보안 처리가 된 것을 뚫는거라 좋은 해결 방법은 아닌 것 같다.

그렇기 때문에 왠만하면 사용자가 파일 업로드 하면 시스템에서 파일명 바꿔서 저장하도록 하고,  
어지간하면 웹에 올라갈 이미지에 한글파일명은 쓰지말자... -_-  
(개인적으로 파일명에 한글쓰는 것을 지양하는 편임)


