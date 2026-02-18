---
layout: post  
title:  "DEVELOPERS: SAMESITE=NONE; SECURE 쿠키 설정을 위한 준비"  
date:   2020-01-30 11:08:00  
categories: Environment
---

5월에 Chrome은 새로운 쿠키 분류 시스템을 통해 쿠키에 대한 기본 보안 모델을 발표했습니다. 이 이니셔티브는 웹에서 개인 정보 및 보안을 향상시키기위한 지속적인 노력의 일환입니다.  
Chrome 변경 사항이 몇 개월 남았지만 쿠키를 관리하는 개발자는 준비해야 합니다. 개발자 지침은 web.dev에 설명 된 SameSite 쿠키를 참조하십시오.

##### 교차 사이트 및 동일 사이트 쿠키 컨텍스트 이해
##### Understanding Cross-Site and Same-Site Cookie Context

웹 사이트는 일반적으로 광고, 콘텐츠 추천, 타사 위젯, 소셜 임베드 및 기타 기능을 위한 외부 서비스를 통합합니다. 웹을 탐색 할 때 이러한 외부 서비스는 브라우저에 쿠키를 저장한 다음 해당 쿠키에 액세스하여 개인화 된 경험을 제공하거나 사용자 참여를 측정할 수 있습니다. 모든 쿠키에는 도메인이 연결되어 있습니다. 쿠키와 연결된 도메인이 사용자 주소 표시 줄의 웹 사이트가 아닌 외부 서비스와 일치하는 경우 이는 교차 사이트 (또는 "타사 third party") 컨텍스트로 간주됩니다.  
동일한 엔터티가 쿠키와 웹 사이트를 소유하지만 쿠키의 도메인이 쿠키에 액세스하는 사이트와 일치하지 않는 경우에도 여전히 크로스 사이트 또는 "타사" 컨텍스트로 인식됩니다.

![project](./../../../../../../../images/20200130/1.png)
_웹 페이지의 외부 리소스가 사이트 도메인과 일치하지 않는 쿠키에 액세스하는 경우 이는 사이트 간 또는 "타사" 컨텍스트입니다._

반대로 쿠키 도메인이 사용자 주소 표시 줄의 웹 사이트 도메인과 일치하면 동일 사이트 "first-party" 컨텍스트에서 쿠키 액세스가 발생합니다. 동일 사이트 쿠키는 일반적으로 사람들이 개별 웹 사이트에 로그인하고 선호 사항을 기억하며 사이트 분석을 지원하는 데 사용됩니다.

![project](./../../../../../../../images/20200130/2.png)
_웹 페이지의 리소스가 사용자가 방문하는 사이트와 일치하는 쿠키에 액세스하면 동일한 사이트 또는 "first-party" 컨텍스트입니다._

##### A New Model for Cookie Security and Transparency
##### 쿠키 보안 및 투명성을 위한 새로운 모델

현재 쿠키를 first-party 컨텍스트에서만 액세스하려는 경우 개발자는 외부 액세스를 방지하기 위해 두 가지 설정 (SameSite = Lax 또는 SameSite = Strict) 중 하나를 적용할 수 있습니다. 그러나 이 권장 사항을 따르는 개발자는 극소수이므로 [사이트 간 요청 위조 공격](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)과 같은 위협에 많은 same-site 쿠키가 불필요하게 노출됩니다.  
더 많은 웹 사이트와 사용자를 보호하기 위해 새로운 기본 보안 모델은 달리 명시되지 않는 한 모든 쿠키가 외부 액세스로부터 보호되어야 한다고 가정합니다. 개발자는 새로운 쿠키 설정인 SameSite=None을 사용하여 사이트 간 접속을 위한 쿠키를 지정해야 합니다.  
SameSite=None 속성이 있는 경우 HTTPS 연결을 통해서만 사이트 간 쿠키에 액세스할 수 있도록 추가로 Secure 속성을 사용해야 합니다. 이것은 사이트 간 접속(cross-site access)과 관련된 모든 위험을 완화시키지는 않지만 네트워크 공격에 대한 보호를 제공할 것입니다.  
즉각적인 보안상의 이점 이외에도, 사이트 간 쿠키를 명시적으로 선언하면 투명성과 사용자 선택이 향상됩니다. 예를 들어, 브라우저는 여러 사이트에서 액세스하는 쿠키와는 별도로 단일 사이트에서만 액세스하는 쿠키를 관리하기 위한 세분화 된 제어 기능을 사용자에게 제공할 수 있습니다.

**Chrome Enforcement Starting in February 2020**

2월 Chrome 80에서는 Chrome에서 SameSite 값이 선언되지 않은 쿠키를 SameSite = Lax 쿠키로 취급합니다. 오직 SameSite=None; Secure 설정이 된 쿠키만이 보안 연결을 통해 액세스하는 경우에만 외부 액세스가 가능합니다. 

**How to Prepare; Known Complexities**

cross-site 쿠키를 관리하는 경우 SameSite = None; Secure를 적용해야 합니다. 대부분의 개발자에게는 구현이 간단해야하지만 다음과 같은 복잡성과 특수 사례를 식별하기 위해 지금 테스트를 시작하는 것이 좋습니다.

* 모든 언어와 라이브러리가 아직 None 값을 지원하는 것은 아니며 개발자가 쿠키 헤더를 직접 설정해야합니다. 이 [Github 리포지토리](https://github.com/GoogleChromeLabs/samesite-examples)는 SameSite = None을 구현하기 위한 지침을 제공합니다. 다양한 언어, 라이브러리 및 프레임 워크로 보호하십시오.
* 일부 Chrome, Safari 및 UC 브라우저 버전을 포함한 일부 브라우저는 의도하지 않은 방식으로 None 값을 처리 할 수 있으므로 개발자가 해당 클라이언트에 대한 예외를 코딩해야 합니다. 여기에는 이전 버전의 Chrome으로 구동되는 Android WebView가 포함됩니다. 호환되지 않는 알려진 클라이언트 목록은 [다음](https://www.chromium.org/updates/same-site/incompatible-clients/)과 같습니다.
* 앱 개발자는 HTTP (S) 헤더와 Android WebView의 CookieManager API를 통해 액세스하는 쿠키 모두에 대해 None 값과 호환되는 Chrome 버전을 기반으로 Android WebViews에 적절한 SameSite 쿠키 설정을 선언하는 것이 좋습니다. 나중에까지 Android WebView에서 시행됩니다.
* 엔터프라이즈 IT 관리자는 Single Sign On 또는 내부 애플리케이션과 같은 일부 서비스가 2월 출시에 준비가 되지 않은 경우 일시적으로 Chrome Browser를 레거시 동작으로 되돌리기 위한 [특별한 정책](https://www.chromium.org/administrators/policy-list-3/cookie-legacy-samesite-policies/)을 구현해야 합니다.
* first-party 및 third-party 컨텍스트 모두에서 액세스하는 쿠키가 있는 경우, first-party 컨텍스트에서 SameSite=Lax의 보안 혜택을 얻기 위해 별도의 쿠키를 사용하는 것을 고려할 수 있습니다.

[SameSite Cookies Explained](https://web.dev/samesite-cookies-explained/)는 위 상황에 대한 특정 지침과 문제 및 질문을 제기하기 위한 채널을 제공합니다.  
귀하가 관리하는 사이트 또는 쿠키에 대한 새로운 Chrome 동작의 영향을 테스트하려면 Chrome 76 이상에서 chrome://flags로 이동하여 "SameSite by default cookies" 및 "Cookies without SameSite must be secure(SameSite가 없는 쿠키는 안전해야 함)" 옵션을 활성화 하십시오. 또한 이러한 실험은 크롬 79 베타 사용자 중 일부에 대해 자동으로 활성화됩니다. 테스트가 가능한 일부 베타 사용자는 아직 새 모델을 지원하지 않는 서비스와 호환되지 않는 문제가 발생할 수 있습니다.  
same-site 컨텍스트 (same-site 쿠키)로만 액세스되는 쿠키를 관리하는 경우 사용자가 취해야 할 조치는 없습니다. Chrome은 SameSite 속성이 없거나 값이 설정되지 않은 경우에도 외부 엔터티가 쿠키에 액세스하지 못하게 합니다.  
그러나 모든 브라우저가 기본적으로 동일한 사이트 쿠키를 보호하는 것은 아니기 때문에 적절한 SameSite 값 (Lax 또는 Strict)을 적용하고 기본 브라우저 동작에 의존하지 않는 것이 좋습니다.  
마지막으로, 웹 사이트에 서비스를 제공하는 공급 업체 및 다른 사람의 준비 상태에 대해 우려가 되는 경우 페이지에 필요한 설정이 누락된 cross-site 쿠키가 포함 된 경우 Chrome 77 이상에서 개발자 도구 콘솔 경고를 확인할 수 있습니다.

![project](./../../../../../../../images/20200130/3.png)

Posted by Barb Palser, Chrome and Web Platform Partnerships  
원문 : https://blog.chromium.org/2019/10/developers-get-ready-for-new.html