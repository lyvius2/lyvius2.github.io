---
layout: post  
title:  "HTTPS접속에서 MIXED MESSAGE가 더 이상 없다."  
date:   2020-01-30 11:07:00  
categories: Environment
---

오늘은 https:// 페이지가 안전한 https 프로토콜인 하위 리소스만 로드할 수 있도록 Chrome이 점차 변화될 것이라고 발표합니다. 아래에 설명하는 일련의 단계에서 기본적으로 mixed content (https:// 페이지의 안전하지 않은 http:// 하위 리소스)를 차단하기 시작합니다. 이 변경은 웹에서 사용자 개인 정보 보호 및 보안을 향상시키고 사용자에게보다 명확한 브라우저 보안 UX를 제공합니다.

지난 몇 년 동안 웹은 HTTPS로 전환하는데 큰 진전을 보였습니다. 이제 Chrome 사용자는 주요 플랫폼에서 90% 이상을 HTTPS로 사용합니다. 이제 웹에서 HTTPS 구성이 안전하고 최신 상태가 되도록 주의를 기울이고 있습니다.

HTTPS 페이지는 일반적으로 페이지의 하위 자원이 http://를 통해 안전하지 않게 로드되는 mixed content라는 문제로 인해 어려움을 겪습니다. 브라우저는 스크립트 및 iframe과 같은 여러 유형의 mixed content를 기본적으로 차단하지만 이미지, 오디오 및 비디오는 계속 로드되어 사용자의 개인 정보 및 보안을 위협합니다. 예를 들어 공격자는 주식 차트의 혼합된 이미지를 조작하여 투자자를 오도하거나 추적 쿠키를 로드되는 혼합된 리소스 사이에 주입 할 수 있습니다. mixed content를 로드하면 페이지가 안전하지 않다고 표시되어 브라우저 UX가 혼란스러워집니다.

Chrome 79에서 시작하는 일련의 단계에서 Chrome은 기본적으로 모든 mixed content를 점차 차단합니다. 파손을 최소화하기 위해 자원을 https://로 자동 업그레이드하므로 하위 자원이 https://를 통해 이미 사용 가능한 경우 사이트가 계속 작동합니다. 사용자는 특정 웹 사이트에서 mixed content 차단을 거부하도록 설정을 활성화 할 수 있으며, 아래에서는 개발자가 mixed content를 찾고 수정하는 데 도움이 되는 리소스에 대해 설명합니다.

##### Timeline

모든 mixed content를 한 번에 모두 차단하는 대신 이러한 변경을 단계적으로 진행할 것입니다.

* 2019년 12월 출시되는 Chrome 79에서는 특정 사이트에서 mixed content를 차단 해제할 수 있는 새로운 설정을 도입합니다. 이 설정은 Chrome에서 현재 기본적으로 차단하는 mixed script, iframe 및 기타 유형의 컨텐츠에 적용됩니다. https:// 페이지에서 잠금 아이콘을 클릭하여 사이트 설정을 클릭하여 이 설정을 전환할 수 있습니다. 이전 버전의 데스크탑 Chrome에서 mixed content를 차단 해제하기 위해 검색 주소창 오른쪽에 표시되는 방패 아이콘이 대체됩니다.
![project](./../../../../../../../images/20200130/4.png)
* Chrome 80에서는 mixed 오디오 및 비디오 리소스가 https://로 자동 업그레이드되며 https://를 통해 로드하지 못하면 Chrome에서 기본적으로 이를 차단합니다. Chrome 80은 2020년 1월 초기 릴리즈 될 예정입니다. 사용자는 위에서 설명한 설정으로 영향을 받는 오디오 및 비디오 리소스를 차단 해제 할 수 있습니다.
* 또한 Chrome 80에서는 mixed 이미지를 계속 로드할 수 있지만 검색 주소창에 Chrome에 '보안되지 않음 Not Secure' 칩이 표시됩니다. 이는 사용자에게 보다 명확한 보안 UI이며 웹 사이트에서 이미지를 HTTPS로 마이그레이션하도록 동기를 부여 할 것으로 예상됩니다. 개발자는 이 보안 경고를 피하기 위해 upgrade-insecure-requests 또는 block-all-mixed-content 보안 정책을 사용할 수 있습니다. 계획은 다음과 같습니다.
![project](./../../../../../../../images/20200130/5.png)
* Chrome 81에서는 mixed 이미지가 https://로 자동 업그레이드되며 https://를 통해 로드하지 못하면 Chrome에서 기본적으로 이미지를 차단합니다. Chrome 81은 2020년 2월 초기 릴리즈 될 예정입니다.

##### Resources for Developers

개발자는 웹 경고와 파손을 피하기 위해 mixed content를 https://로 즉시 마이그레이션해야 합니다. 다음은 그 방안입니다.

* [컨텐츠 보안 정책](https://web.dev/fixing-mixed-content/) 및 [Lighthouse’s mixed content 감사](https://web.dev/what-is-mixed-content/)를 사용하여 사이트에서 mixed content를 검색하고 수정하십시오.
* 서버를 HTTPS로 마이그레이션하는데 대한 일반적인 조언은 이 가이드를 참조하십시오.
* CDN, 웹 호스트 또는 컨텐츠 관리 시스템에 mixed content 디버깅을 위한 특수 도구가 있는지 확인하십시오. 예를 들어 Cloudflare는 mixed content를 https://로 다시 작성하는 도구를 제공하며 [WordPress 플러그인](https://en-gb.wordpress.org/plugins/ssl-insecure-content-fixer/)도 사용할 수 있습니다.

Posted by Emily Stark and Carlos Joan Rafael Ibarra Lopez, Chrome security team
원문 : https://blog.chromium.org/2019/10/no-more-mixed-messages-about-https.html