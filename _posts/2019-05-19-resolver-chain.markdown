---
layout: post  
title:  "SpringBoot에 Thymeleaf, JSP 같이 쓰기"  
date:   2019-05-18 22:29:00  
categories: JAVA SpringBoot JSP Thymeleaf 
---

JAVA로 구축된 대부분의 레거시 웹 소스 코드에서는 아직도 JSP가 많이 사용되고 있을 것이다.  
SpringBoot에서도 JSP를 쓰는 것을 본 적이 있다.  
이를 Thymeleaf 등 다른 템플릿 엔진으로 전환하고자 할 때 순차적으로 진행하기 위해 과도기가 필요할 수 있겠는데,  
단일 프로젝트에서 Thymeleaf와 JSP를 같이 쓰는 방법을 정리해본다.

##### JSP
* Java Server Pages. Java를 이용한 서버 사이드 스크립트 언어.
* Servlet Container가 <% … %> 스크립트 영역을 Java 코드로 컴파일 하여 실행됨
* 서버 사이드 코드와 클라이언트 사이드 코드가 섞여있음
* 디버깅이 어렵고 코드량 증가에 비해 유지보수가 대폭 어려워짐
* Spring 기본 지원에서 제외

##### Thymeleaf
* (현재) 스프링 생태계에서 권장하는 텍스트 템플릿 엔진
* Natural Template 
  * 템플릿 코드가 HTML이기 때문에 WAS기동없이 웹 브라우저에서 확인 가능
  * 프런트 엔드(HTML, Javascript, CSS), 백 엔드(JAVA) 분업에 용이
* 문법이 엄격한 편

##### 개발자들은...
![project](./../../../../../../../images/20190519/07.png)  
좋다는 사람도 있고 별로라는 사람도 있는데 솔직히 나도 별로인 거 같다. 그냥 못쓸 정도는 아니라는거...

##### SpringBoot에서 JSP를 쓰기 위한 설정

build.gradle에 2개 라이브러리 의존성 추가
```groovy
compile 'org.apache.tomcat.embed:tomcat-embed-jasper'
compile 'javax.servlet:jstl:1.2'
```

application.yml에 View Resolver 설정
```yaml
spring:
  mvc:
    throw-exception-if-no-handler-found: false
    view:
      prefix: /WEB-INF/jsp/
      suffix: .jsp
```

Controller 클래스 하나 간단하게 만든다.
```java
package com.walter.project.controller;

import com.walter.project.service.LanguageService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import reactor.core.publisher.Mono;

@Controller
@RequiredArgsConstructor
public class LanguageController {

	final private LanguageService languageService;

	@GetMapping(value = "/language")
	public Mono<String> index(Model model) {
		model.addAttribute("list", languageService.getList());
		return Mono.just("language/index");
	}
}
```

application.yml에 설정한 경로대로 src/main 디렉터리 밑에 webapp/WEB-INF/jsp 경로를 만들고 Controller 내용에 맞춰 JSP파일을 만든다.  

*index.jsp*
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
	<title>Title</title>
</head>
<body>
<div>
	Language List
	<ul>
		<c:forEach var="lang" items="${list}">
			<li>
				<a href="/language/${lang.id}">${lang.name}</a>
			</li>
		</c:forEach>
	</ul>
</div>
</body>
</html>
```

그럼 아래와 같은 구조가 된다.
![project](./../../../../../../../images/20190519/01.png)

애플리케이션 구동시킨 뒤 http://localhost:9002/language로 접속
![project](./../../../../../../../images/20190519/02.png)

이렇게 해서 일단 화면은 JSP로 뿌려진다.  
원하는 것은 여기에 Thymeleaf도 사용할 수 있는 설정을 만들고, Controller로 전달되어 온 요청을 처리한 다음 결과를 표시할 뷰를 처음에는 Thymeleaf 경로에서 찾아서 있으면 Thymeleaf 템플릿으로 보여주고, Thymeleaf 템플릿이 없으면 그 다음 JSP설정으로 넘어가서 JSP로 보여주게 하는 것이다.

##### Thymeleaf ViewResolver 설정 추가

build.gradle에 SpringBoot용 thymeleaf 라이브러리 추가
```groovy
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
```

아니면 아래처럼 일반 Spring프로젝트에 넣을 때처럼 넣어도 된다.
```groovy
compile group: 'org.thymeleaf', name: 'thymeleaf', version: '3.0.11.RELEASE'
compile group: 'org.thymeleaf', name: 'thymeleaf-spring4', version: '3.0.11.RELEASE' // SpringBoot 1.5일 때
compile group: 'org.thymeleaf', name: 'thymeleaf-spring5', version: '3.0.11.RELEASE' // SpringBoot 2일 때
```

application.yml에 JSP용으로 View Resolver 설정한 것 날려버리고 Java Config로 수동 설정한다.
```yaml
spring:
  mvc:
    throw-exception-if-no-handler-found: false
#    view:
#      prefix: /WEB-INF/jsp/
#      suffix: .jsp
```

Java Config 클래스 파일 하나 작성한다.
```java
package com.walter.project.config;

import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;
import org.thymeleaf.spring5.SpringTemplateEngine;
import org.thymeleaf.spring5.view.ThymeleafViewResolver;

@Configuration
@EnableWebMvc
@RequiredArgsConstructor
public class WebMvcConfig {

	final private SpringTemplateEngine springTemplateEngine;

	@Bean
	public ThymeleafViewResolver thymeleafViewResolver() {
		ThymeleafViewResolver thymeleafViewResolver = new ThymeleafViewResolver();
		thymeleafViewResolver.setTemplateEngine(springTemplateEngine);
		thymeleafViewResolver.setCharacterEncoding("UTF-8");
		thymeleafViewResolver.setAlwaysProcessRedirectAndForward(true);
		thymeleafViewResolver.setOrder(1);
		return thymeleafViewResolver;
	}

	@Bean
	public InternalResourceViewResolver jspViewResolver() {
		InternalResourceViewResolver jspViewResolver = new InternalResourceViewResolver();
		jspViewResolver.setPrefix("/WEB-INF/jsp/");
		jspViewResolver.setSuffix(".jsp");
		jspViewResolver.setViewClass(JstlView.class);
		jspViewResolver.setOrder(2);
		return jspViewResolver;
	}
}
```

ViewResolver에는 setOrder()메서드로 우선 순위를 정할 수 있다.   
thymeleaf가 1순위, JSP를 2순위로 설정하였다. 우선 순위로 설정한 ViewResolver에서 view파일을 못찾으면 다음 순위로 넘어가게 되어있다.  
JSP용으로 사용하는 InternalResourceViewResolver는 view를 찾지 못하면 다음 Resolver로 넘어가는 것이 아니라 Exception을 발생시켜 버리므로 가장 마지막 순위로 설정한다.

Thymeleaf 템플릿 파일을 작성한다. Thymeleaf 기본 설정대로 프로젝트의 src/main/resources 경로 아래 templates 디렉터리를 만들고 그 안에 Controller에서 정의한 view이름으로 html파일을 생성한다.
![project](./../../../../../../../images/20190519/03.png)

*index.html*
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
	<meta charset="UTF-8"/>
	<title>Title</title>
</head>
<body>
	Language List (Thymeleaf)
	<ul>
		<th:block th:each="lang : ${list}">
			<li>
				<a th:href="@{/language/{id}(id=${lang.id})}" th:text="${lang.name}">lang.name</a>
			</li>
		</th:block>
	</ul>
</body>
</html>
```

위와 같이 템플릿 파일까지 작성한 다음 서버 재기동하고 재접속하면 아래와 같이 잘 나온다.
![project](./../../../../../../../images/20190519/04.png)

##### ViewResolver Chaining

여기까지 하면 그럼 ViewResolver의 Chaining설정까지 해놓았으니 Thymeleaf의 템플릿 파일이 없으면 당연히 다음 순위인 JSP파일을 찾아서 보여주겠지? 라는 생각이 든다.  
그래서 테스트 삼아 *index.html 파일의 이름을 index2.html로 바꿔보고* 다시 돌려본다.
![project](./../../../../../../../images/20190519/05.png)
![project](./../../../../../../../images/20190519/06.png)
위처럼 예상 외로 안돌아간다. ViewResolver에서 Exception을 뱉어내면서 500에러가 떨어진다. 
Thymeleaf ViewResolver에서 템플릿을 구성하지 못하면 ViewResolver Chaining에 의해서 다음 순위 ViewResolver로 넘어간다. 그렇지만 템플릿을 구성하려 View파일을 로딩할 때 Thymeleaf ViewResolver에서는 View파일이 실제로 존재하는지를 미리 알 수가 없다. 파일이 없으면 Chaining이 작동하기 전에 Exception이 발생해버린다.  
이를 해결하려면 두가지 방법이 있는데 가장 쉬운 방법은 ViewResolver에 ViewNames 속성을 정의하여 View파일 경로에 특정 패턴이 적용된 것만 Thymeleaf 템플릿으로 연결하고 그 이외는 Chaining이 적용되어 다음 순번 ViewResolver으로 넘어가게 하는 방법이다.

*Thymeleaf ViewResolver Bean설정에 ViewNames 셋팅*
 ```java
    thymeleafViewResolver.setViewNames(new String[] {"thymeleaf/*"});
```

*Controller*
```java
    @GetMapping(value = "/language")
    public Mono<String> index(Model model) {
        model.addAttribute("list", languageService.getList());
        return Mono.just("language/index");     // JSP
    }
	
    @GetMapping(value = "/test")
    public Mono<String> test(Model model) {
        model.addAttribute("test", new Date());
        return Mono.just("thymeleaf/test");     // Thymeleaf
    }
```

이게 사실 정석이라 할 수 있는데 이렇게 하면 JSP를 Thymeleaf로 마이그레이션이 다 끝나도 설정을 원복하지 않는 한 View이름에 앞으로 계속 `thymeleaf/`를 붙여야 한다. 이게 솔직히 좀 마음에 안든다. -_-  
그래서 좀 변칙적인 수단일 수도 있지만 Thymeleaf ViewResolver 클래스를 상속해서 템플릿 파일이 존재하는지 여부를 체크한 다음 없으면 false를 리턴하게 하여 Chaining이 동작하게 하는 방법이 있다.

*CustomThymeleafViewResolver.java*
```java
package com.walter.project.config;

import lombok.RequiredArgsConstructor;
import lombok.Setter;
import org.apache.commons.lang3.StringUtils;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;
import org.springframework.core.io.UrlResource;
import org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver;
import org.thymeleaf.spring5.view.ThymeleafViewResolver;

import java.net.MalformedURLException;
import java.util.Locale;

@RequiredArgsConstructor
public class CustomThymeleafViewResolver extends ThymeleafViewResolver {

	public static final String RESOURCE_PREFIX_CLASSPATH = "classpath:";
	public static final String RESOURCE_PREFIX_FILE = "file:";

	final private SpringResourceTemplateResolver springResourceTemplateResolver;

	@Setter
	private String prefix;

	@Setter
	private String suffix;

	@Override
	protected boolean canHandle(String viewName, Locale locale) {
		boolean isExistView = isExistView(viewName);
		if (isExistView) {
			return super.canHandle(viewName, locale);
		}
		return isExistView;
	}

	protected boolean isExistView(String viewName) {
		String viewPath =
				StringUtils.defaultString(this.prefix, springResourceTemplateResolver.getPrefix()) +
				viewName +
				StringUtils.defaultString(this.suffix, springResourceTemplateResolver.getSuffix());

		Resource resource = null;
		if (viewPath.startsWith(RESOURCE_PREFIX_CLASSPATH)) {
			resource = new ClassPathResource(StringUtils.removeStart(viewPath, RESOURCE_PREFIX_CLASSPATH));
		} else if (viewPath.startsWith(RESOURCE_PREFIX_FILE)) {
			resource = new FileSystemResource(StringUtils.removeStart(viewPath, RESOURCE_PREFIX_FILE));
		} else {
			try {
				resource = new UrlResource(viewPath);
			} catch (MalformedURLException e) {
				return false;
			}
		}

		if (!resource.exists()) {
			return false;
		}
		return true;
	}
}
```

isExistView() 메서드에서 View파일 존재 여부를 체크하여 canHandle() 메서드에서 파일이 있으면 수퍼 클래스의 메서드를 수행하고, 반대의 경우 false를 리턴한다.  
Java Config파일에는 ThymeleafViewResolver클래스를 상속한 CustomThymeleafViewResolver클래스를 Bean에 셋팅한다.

*WebMvcConfig.java*
```java
package com.walter.project.config;

import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;
import org.thymeleaf.spring5.SpringTemplateEngine;
import org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver;
import org.thymeleaf.spring5.view.ThymeleafViewResolver;

@Configuration
@EnableWebMvc
@RequiredArgsConstructor
public class WebMvcConfig {

	final private SpringResourceTemplateResolver springResourceTemplateResolver;
	final private SpringTemplateEngine springTemplateEngine;

	@Bean
	public ThymeleafViewResolver thymeleafViewResolver() {
		ThymeleafViewResolver thymeleafViewResolver = new CustomThymeleafViewResolver(springResourceTemplateResolver);
		thymeleafViewResolver.setTemplateEngine(springTemplateEngine);
		thymeleafViewResolver.setCharacterEncoding("UTF-8");
		thymeleafViewResolver.setAlwaysProcessRedirectAndForward(true);
		thymeleafViewResolver.setOrder(1);
		return thymeleafViewResolver;
	}

	@Bean
	public InternalResourceViewResolver jspViewResolver() {
		InternalResourceViewResolver jspViewResolver = new InternalResourceViewResolver();
		jspViewResolver.setPrefix("/WEB-INF/jsp/");
		jspViewResolver.setSuffix(".jsp");
		jspViewResolver.setViewClass(JstlView.class);
		jspViewResolver.setOrder(2);
		return jspViewResolver;
	}
}
```

이렇게 하면 오류없이 처음에는 Thymeleaf 템플릿을 찾고, 템플릿 파일이 없으면 JSP로 넘어간다.