---
layout: post
title:  "Spring MVC에서 페이스북 소셜로그인 구현"
date:   2017-06-27 00:00:00
categories: JAVA Spring
---

개인 프로젝트에서 페이스북 소셜로그인을 구현하려는데 해보니까 진짜 쉽지만 초반에는 하루 이틀정도 헤멨던 것 같다. Spring.io에서 예제들을 열심히 찾았지만 Spring boot 사용을 전제로 해서 내놓은 것이라서 예시에서는 자동적으로 되던 것이 당연히 내 프로젝트에서는 안됐고 적지않은 구글링을 했어야 했다. 개발감 다 떨어졌다는 한탄을 하며...

내가 구현해놓은 방식이 완전히 잘했다고는 생각지는 않지만 까먹지 않기 위해 기록해둔다.

페이스북 로그인은 Spring 공식 프로젝트로 되어 있다.
![screen shot](./../../../../../images/20170627/237289445950FDD602.jpeg)
[스프링 소셜 페이스북 프로젝트 사이트][1]로 들어가서 위와 같이 메이븐 혹은 gradle의 파일에 페이스북 라이브러리의 의존성을 넣어준다. 내 프로젝트는 gradle을 사용하고 있기 때문에 build.gradle 파일의 dependencies 항목에 위 내용을 복사해서 집어넣어줬다.

그리고 페이스북 개발자 사이트로 접속한다. 주소는 [https://developers.facebook.com][2] 이다.

페이스북 개발자 사이트에서 페이스북 계정으로 로그인 한 다음 새 앱을 만들고 앱의 대시보드에서 제품 추가를 하여 Facebook 로그인을 추가한다. 클라이언트 OAuth 설정에서 클라이언트와 웹 OAuth 로그인을 활성화하고 리디렉션 URI도 설정해준다. 대충 아래의 화면 처럼 될 것이다.
![screen shot](./../../../../../images/20170627/2714B3385951008D34.jpeg)
아직 로컬에서만 돌아가고 있어서 유요한 리디렉션 URI에는 그냥 로컬 주소로 써넣었는데 잘 돌아간다.

그리고 설정 > 기본 설정에서 앱 ID와 앱 시크릿코드를 메모해준다. 이 두 개의 코드는 페이스북 로그인 외 페이스북 관련된 API를 연동할 때에도 모두 사용되는 모양이다.

스프링 페이스북 라이브러리에는 요청을 받아서 페이스북 API에 전달해주는 컨트롤러 단과 실제로 통신하고 페이스북 데이터를 가져오는 부분까지 통째로 다 있다. 컨트롤러까지 다 들어있으므로 여기까지만 했으면 세가지 작업만 더 해주면 페이스북 로그인은 바로 구현된다.

1. XML이나 JAVA Config에 앱 ID와 시크릿코드를 Argument로 넣어서 Bean 설정해 준다.

2. component-scan의 base-package에 소셜 로그인 관련된 라이브러리 패키지 경로(org.springframework.social)를 넣어준다.

3. 페이스북의 경우 페이스북 데이터에 접근하기 위한 URL은 /connect/__facebook__ (POST)이며 페이스북에서 인증처리한 후 똑같은 경로로 GET호출을 통해 코드값 등을 넘겨주므로 로그인 요청할 facebookConnect.jsp와 데이터를 받아서 표시할 facebookConnected.jsp 두 개 파일을 생성한다. 경로는 view 단의 /connect 디렉터리 밑이다. 왜 페이스북의 경우라고 하냐면 이 것이 페이스북 뿐만 아니라 다른 서비스 소셜 로그인 구현 시에도 공통으로 적용되는 사항이기 때문이다. Google Plus 소셜 로그인 구현 시에는 요청 URL이 /connect/**google**이 되게 되어 있다. 아래 소스를 보시면 이해가 가실 것이다.  
![screen shot](./../../../../../images/20170627/237613395951069A2C.jpeg)  

위 세 가지 작업만 다 하면 페이스북 로그인은 바로 사용할 수 있지만 대부분은 그대로 사용하지 않을 것이다. 왜냐면 우리는 페이스북 로그인 정보를 받아오는 것 뿐만이 아니라 그 정보들을 받아서 회원정보 Model 객체에 매핑하거나 아님 그대로 UsernamePasswordAuthenticationToken을 생성할 적에 매개변수로 그대로 집어넣거나 해서 페이스북이 아닌 내 웹사이트에 사용자 인증처리를 시켜야 하고 그리고 라이브러리에서 정해놓은 대로 /connect/facebookConnected.jsp 파일을 사용할 생각이 없기 때문이다.

실제로 대부분 구현 사례를 구글링 해본 결과 대부분 페이스북에 연결하고 데이터를 Facebook.class 객체로 받아오는 부분만 라이브러리를 사용하고, 사용자 요청받고 응답하고 Token 생성하는 부분은 직접 구현한 사례가 많았다.

그래서 나도 우선은 최대한 라이브러리를 활용하되 인증 처리하는 부분은 직접 구현을 했다. 사용자 요청받고 응답하는 컨트롤러 부분은 직접 구현하려니 너무 복잡하고 굳이 이 내용들을 일일히 까볼 필요가 없을 것 같아 가장 손쉬운 방법으로 처리했다. 라이브러리 안의 org.springframework.social.connect.web.ConnectController 클래스를 상속받는 클래스를 내가 만든 패키지 안에 생성한 다음에 POST로 페이스북 API에 연결하는 메서드(connect)와 페이스북 인증 후 GET으로 코드값 받아오는 메서드(oauth2Callback) 두 개만을 오버라이드하여 내 입맛대로 바꿨다.

**1.** src/main/resources 경로 밑에 application.properties 파일을 생성한다. 아래처럼 페이스북 앱 ID와 앱 시크릿코드를 붙여 넣는다.
```properties
spring.social.facebook.appId=233668646673605
spring.social.facebook.appSecret=33b17e044ee6a4fa383f46ec6e28ea1d
```
참고로 위의 앱 ID와 앱 시크릿코드는 내 것이 아니라 Spring.io에 공개되어 있는 테스트용 코드다. -_-

**2.** JAVA Config 혹은 context.xml에 bean 설정과 application.properties 파일을 읽어서 사용할 수 있도록 설정한다. 나는 아직 XML 기반의 설정을 사용해서 아래와 같이 context.xml 파일에 설정해놓았다. 나중에 JAVA Config로 바꿀 생각이지만.  
{% highlight xml %}
<!-- application.properties 설정 -->
<context:property-placeholder location="classpath:/application.properties" />

<beans:bean id="connectionFactoryLocator" class="org.springframework.social.connect.support.ConnectionFactoryRegistry">
  <beans:property name="connectionFactories">
    <beans:bean class="org.springframework.social.facebook.connect.FacebookConnectionFactory">
      <beans:constructor-arg value="${spring.social.facebook.appId}" />
      <beans:constructor-arg value="${spring.social.facebook.appSecret}" />
    </beans:bean>
  </beans:property>
</beans:bean>

<beans:bean id="inMemoryConnectionRepository" class="org.springframework.social.connect.mem.InMemoryConnectionRepository">
  <beans:constructor-arg ref="connectionFactoryLocator" />
</beans:bean>
{% endhighlight %}
XML의 내용을 보면 connectionFactoryLocator와 inMemoryConnectionRepository 라는 이름 Bean 두 개를 생성한다. connectionFactoryLocator라는 Bean은 페이스북 앱 ID와 앱 시크릿코드를 Argument로 받아 생성하는 클래스 Bean을 connectionFactories라는 이름의 Property로 가지고 있다. 대충 이 놈이 페이스북 API에 연결해서 인증과 관련된 역할을 할 것이라는 느낌이 온다.(Property 이름이 복수로 되어 있는 걸 보면 알 수 있겠지만 실제로 List 객체 타입으로 되어있다. 나중에 추가 소셜로그인 구현을 하다보면 페이스북 이외에 다른 서비스들의 CoonnectionFactory Bean들을 추가로 add해서 쓴다.) 그리고 inMemoryConnectionRepository 라는 녀석은 Repository라는 단어에서 아마도 페이스북 인증 후 정보를 담는 역할을 할 것이라는 Feel이 전해져 올 것이다. 이 두 개의 Bean은 소셜로그인을 위해 org.springframework.social.connect.web.ConnectController 클래스를 상속해서 만든 컨트롤러 Class 생성자의 매개변수로 주입된다.  

**3.** org.springframework.social.connect.web.ConnectController 클래스를 상속받은 커스터마이징(?) 컨트롤러 클래스 생성.
{% highlight java %}
package com.walter.controller;

import com.walter.config.authentication.SignInUserDetailsService;
import com.walter.model.MemberVO;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.social.connect.Connection;
import org.springframework.social.connect.ConnectionFactoryLocator;
import org.springframework.social.connect.ConnectionRepository;
import org.springframework.social.connect.web.ConnectController;
import org.springframework.social.facebook.api.Facebook;
import org.springframework.social.facebook.api.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.servlet.view.RedirectView;

import javax.annotation.Resource;
import javax.inject.Inject;
import javax.servlet.http.HttpServletRequest;

@Controller
@RequestMapping("/connect")
public class FacebookTestController extends ConnectController {
	final Logger logger = LoggerFactory.getLogger(this.getClass());
	private String TARGET_URL = new String();

	@Resource(name="signInUserDetailsService")
	private SignInUserDetailsService signInUserDetailsService;

	@Resource(name="inMemoryConnectionRepository")
	private ConnectionRepository connectionRepository;

	@Inject
	public FacebookTestController(ConnectionFactoryLocator connectionFactoryLocator, ConnectionRepository connectionRepository) {
		super(connectionFactoryLocator, connectionRepository);
	}

	@RequestMapping(value="/{providerId}", method=RequestMethod.POST)
	public RedirectView connect(@PathVariable String providerId, NativeWebRequest request) {
		HttpServletRequest httpServletRequest = (HttpServletRequest)request.getNativeRequest();
		TARGET_URL = httpServletRequest.getHeader("REFERER");
		return super.connect(providerId, request);
	}

	@RequestMapping(value="/{providerId}", method= RequestMethod.GET, params="code")
	public RedirectView oauth2Callback(@PathVariable String providerId, NativeWebRequest request) {
		RedirectView redirectView = super.oauth2Callback(providerId, request);

		// 사용자 정보 가져오기
		Connection<Facebook> connection = connectionRepository.findPrimaryConnection(Facebook.class);
		Facebook facebook = connection.getApi();
		String [] fields = { "id", "age_range", "email", "first_name", "gender",
				"last_name", "link", "locale", "name", "third_party_id", "verified" };
		User userProfile = facebook.fetchObject("me", User.class, fields);

		// 로그인 처리
		signInUserDetailsService.onAuthenticationBinding(new MemberVO(), userProfile);
		redirectView.setUrl(TARGET_URL);
		return redirectView;
	}
}
{% endhighlight %}
프런트엔드에서 소셜로그인을 위해 최초로 요청을 받게 되는 메서드는 **connect**이다. 여기에는 요청을 보낸 페이지의 URL을 받아 TARGET_URL이라는 이름의 String 변수에 대입하는 부분만을 추가했는데 페이스북 페이지에 가서 인증을 하고 나서는 바로 처음으로 소셜로그인 요청을 했던 원래 페이지로 돌아오게 하기 위해서다. 예를 들면 *http://a.com/abc* 페이지에서 페이스북 로그인 요청을 해서 페이스북 사이트가 갔다 오면 다시 *http://a.com/abc* 로 돌아오게 하려고. 이를 처리하기 위해 페이스북 개발자 센터에서도 별도로 Callback 페이지 설정하는 부분도 있고 분명 더 좋은 방법이 있을 것인데 혹시 알고 계시는 분이 계시면 알려주기 바람. 라이브러리의 컨트롤러를 그대로 사용하자면 분명 Client Side에서 Callback 처리해줘야 할 거 같은데 나같은 경우는 백엔드 - 프런트엔드를 여러번 왔다갔다 하는 것을 꺼려해서 그냥 이렇게 처리해버렸다.

그리고 페이스북 페이지에서 로그인 한 뒤에는 페이스북 API에서 code 값을 파라메터로 담아 Callback 요청을 하게 되는데 이 요청을 받는 메서드가 마지막 **oauth2Callback** 메서드이다. 여기에서는 수퍼클래스의 메서드 내용을 수행한 뒤 connectionRepository에서 사용자 정보를 User 객체에 fetch하고 그 User 객체를 이용하여 로그인 처리를 한 다음 TARGET_URL로 리디렉션하도록 해놨다.

로그인 처리는 별도의 signInUserDetailsService 클래스의 onAuthenticationBinding 메서드에서 수행하는데 내용은 아래와 같다.
{% highlight java %}
public void onAuthenticationBinding(MemberVO memberVO, User facebookUser) throws NullPointerException {
    memberVO.setUsername(facebookUser.getId());
    memberVO.setEmail(facebookUser.getEmail());
    memberVO.setFirst_name(facebookUser.getFirstName());
    memberVO.setKr_name(facebookUser.getName());
    memberVO.setLast_name(facebookUser.getLastName());
    memberVO.setAuthorities(ROLE.DEFAULT.getRoleList());
    memberVO.setAccountNonExpired(true);
    memberVO.setAccountNonLocked(true);
    memberVO.setCredentialsNonExpired(true);
    memberVO.setEnabled(true);

    // Token 생성하고 로그인 세션 생성
    UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(
            memberVO, null, ROLE.DEFAULT.getRoleList()
    );
    SecurityContextHolder.getContext().setAuthentication(authenticationToken);
}
{% endhighlight %}
메서드 내용 상단의 memberVO.set~으로 시작되는 부분은 기존의 회원정보 Model 객체로 사용되던 MemberVO 클래스에 User 객체의 내용들을 바인딩하는 내용이고 하단은 사용자 정보를 담은 Model 객체와 자격증명(대개 비밀번호 암호화한 값을 넣는데 여기선 없으므로 null로 넣었다), 그리고 권한을 매개변수로 던져서 authenticationToken을 생성하게 되고 마지막으로 Spring Security에 태워보내는 내용이다.

**4.** 마지막으로 프런트엔드에서 페이스북 로그인 요청하는 부분을 만들어야 하는데, form에 /connect/facebook 이라는 주소로 POST 요청을 보내면 된다.
{% highlight jsp %}
  <form action="/connect/facebook" method="post" id="facebook-form">
    <input type="hidden" name="scope" value="public_profile, email"/>
    <button type="submit">Sign In with Facebook</button>
  </form>
{% endhighlight %}
파라메터로 **scope** 라는 항목이 있는데 이 값에 따라 페이스북 API로부터 가져오는 사용자 정보 항목이 달라진다. 자세한 내용은 [https://developers.facebook.com/docs/facebook-login/permissions/][3] 여기를 참고하도록...
![screen shot](./../../../../../images/20170627/22173C35595120540B.jpeg)
'SIGN IN WITH FACEBOOK' 클릭  
(scope 파라메터에 'public_profile, email' 라는 값을 담아서 /connect/facebook에 POST전송)
![screen shot](./../../../../../images/20170627/25140035595120540B.jpeg)
페이스북 로그인 화면으로 전환되고 로그인을 한다.  
(앱에 대한 사용자 정보 사용 동의에 대한 화면이 나옴)
![screen shot](./../../../../../images/20170627/231C31355951205503.jpeg)
로그인 처리 후 다시 원래 화면으로 돌아온다('SIGN IN'이 'SIGN OUT'으로 바뀜)  

잘 된다...

[1]: http://projects.spring.io/spring-social-facebook/
[2]: https://developers.facebook.com
[3]: https://developers.facebook.com/docs/facebook-login/permissions/