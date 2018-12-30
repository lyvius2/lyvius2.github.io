---
layout: post
title:  "AWS Essentials"
date:   2015-12-29 22:36:00
categories: AWS
---

**2015.12.29 T아카데미 AWS강좌를 수강하면서 필기.**

[http://skpla.net/ta1229][1]
비트윈, 디스패치가 AWS로 구현되었음.  
아마존 웹서비스는 public cloud computing 을 제공하는 세계 최대의 벤더이다.

##### 온 디맨드(On-demand) : 주문형 서비스
1. 서비스를 구축하려면 서버, 스토리지, 네트워크 엔지니어가 필요하다. -> 지금은 cloud computing을 통해 on-demand하게 access할 수 있다.
2. 내가 쓴 만큼 비용을 지불한다.
3. 안정적이다. 전문적인 인력들이 나의 인프라를 관리해준다.
4. 인프라의 각 구성 요소가 API로 연결되어 있다.(서버 자원 및 사용량 셋팅 등...)  
시간과 시기에 따라 자원의 투입량을 CRON등의 스크립트를 사용해서 자동으로 조절할 수 있다.

* 전통적인 서비스 구축 : Packaged Software
* Cloud Computing :  
Infrastructure(물리적인 인프라의 장애 대응은 벤더사에 위임),  
Platform(물리적인 인프라와 OS, Middleware 환경까지 on-demand하게 제공-개발만 한다-),  
Software(소프트웨어까지 위임한다. All. MS애저, 구글플랫폼, 구글Docs 등)

##### Cloud Computing의 특징
1. Cloud라고 해서 무조건 저렴하지는 않다.
2. 사용량과 서비스 활성화에 따라 자원 투입을 플렉서블하게 운용할 수 있다. -> 이 상황에 따라 비용이 많이 들 수도, 적게 들 수도 있다.
3. 초기 비용이 들지 않는 대신 매달 유지비용이 든다.
4. 서비스와 장애 대응의 신속성
5. 병렬로 확장 (2코어 4기가 서버를 여러대 증설 -> 병렬로 요청을 받게 됨), 직렬로 확장 (2코어 4기가 서버를 8코어 32기가 서버로 업그레이드)
6. 관리의 용이성 (서비스를 관리하기 위한 스크립트 기반의 API가 잘 되어있다)

##### Cloud Computing의 고려 사항
1. 보안성(서비스 관리 계정 정보가 유출될 경우 누구나 접근이 가능한 맹점
2. 국가간, 대륙간 데이터 이동, 백업이 발생하는데 개인정보와 미디어, 컨텐츠 데이터의 경우 법률적인 문제가 될 수 있다.
3. 인프라 리소스에 대한 보안을 보장하지 데이터베이스에 대한 보안은 보장해주지 않는다.
4. 보안패치 및 장애 대응 작업의 경우 AWS에서 진행하기 때문에 서비스 중단 시점에 대한 주도권을 쥘 수 없다.(인프라를 위임했기 때문에)
5. 위의 문제를 보완하기 위해 가용성 측면에서 Cloud 상에서 이중화로 구축해놓아야 한다.
6. 커스터마이징 불가 (서버OS 라이브러리의 업데이트나 미들웨어 설정의 변경 등...)

AWS가 기술적으로 MS애저보다 2년 빠른 것 같다. - 활용도와 레퍼런스 서비스가 발전해있다.  
**JAVA - AWS, .Net - MS Azure**

##### AWS 의 데이터 센터

[https://prezi.com/cxpwi_og7lht/aws-regions-and-availability-zones/#][2]  
REGION > Availability Zone (IDC. 논리적인 묶음일수도, 물리적인 집합일 수도 있다.)  
논리적인 집합이라 하더라도 네트워크를 통해 같은 Zone내의 서버 간의 속도를 빠르게 한다. > EC2 Instance (가상머신, 서버. 물리적 장비 위에 탑재된다. )

##### 3개의 지불 모델 :
* On Demand - 주문형 (구글은 계속 사용하면 Reserved로 전환된다). 위치한 Region에 따라 사용 가격이 다르다. 미국이 제일 저렴.
* Reserved - Cloud벤더에서 제안하는 표준적인 사양. 가격대가 30% 저렴
* Spot - 약 70% 가량 저렴. 남는 자원을 입찰을 통해 배분한다. 24시간 서비스 보장을 하지 않아 일시적이고 정기적인 대용량 데이터 처리에 적합.

**EBS** : Storage Level. 확장 및 축소가 간편. 내구성이 뛰어나 거의 장애가 발생하지 않는다.(chunk 단위)

**AMI** : EC2 Instance에 포함되어 있는 부가적인 서비스. OS + Software의 가상 이미지.  
OS와 커스터마이징 된 설정, 그 위에 탑재된 미들웨어까지 포함된 설치-배포용 표준 이미지.  
AMS 마켓플레이스에서는 오라클DB, SAP 등의 상용 소프트웨어가 설치된 이미지도 제공한다.  
(물론 비용 상승은 덤. 볼륨 라이선스나 연단위로 사용정책을 체크하는 일반적인 상용 소프트웨어 설치와 달리 AWS에서는 시간 단위로 라이선스를 책정한다.)

**RDS Instance** : 기존의 DB는 관리적인 소모가 너무 많이 든다(보안패치, 백업 작업, 이중화 작업의 어려움, 고난이도의 장애 대응)  
위와 같은 관리를 AWS에서 해 줌. 대쉬보드를 통해 설치, 백업 정책 설정, 데이터 롤백과 리커버리가 간편하다.(Click 두 번으로 가능)

**S3** : 웹 베이스로 파일(Object) 단위의 I/O를 위한 스토리지. 파일 단위 당 5TB까지의 용량 제한. 스토리지 사용 용량 자체는 제한없다.(쓴만큼 돈 낸다)

**DynamoDB** : NoSQL DB. 비정형 데이터를 취급하기 때문에 용량 제한이 없다(무제한). Request Base로 쿼리를 몇 번 날렸는지에 따라 가격이 달라진다.  
속도가 중요하기 때문에 SSD로 물리적 장비가 구성되어 있다. 자동으로 이중화 구성이 된다.

##### Auto Scaling Group : AWS을 사용하는 가장 중요한 이유.
* 실시간 모니터링을 통해 서비스 사용 자원량이 임계점에 도달하기 전에 자동으로 자원 스케일링(확장)을 해준다.
* 트래픽 폭주 시 자동으로 병렬 서버를 생성하여 다중화(부하 분산)로 돌입하도록 설정할 수 있다.
* 임계값 설정(CPU 기준으로? 트래픽 기준으로? 디스크 I/O기준으로?)이 중요하다.

**Elastic Load Balancer** : Health Check 등을 활용하여 부하 분산 처리.

AWS에서는 시스템 뿐만 아니라 billing에 관한 모니터링도 가능하다. 넷플릭스는 자체 서버가 없이 모두 AWS의 자원을 사용한다.  
디스패치는 원래 워드프레스 기반으로 10대 미만의 자체 서버로 운영되었었다. → 스캔들, 이슈 발생 시를 대비해 AWS으로 마이그레이션 (Auto Scaling이 주 목적)  
최대 초당 11,500 request 발생, 60대까지 서버가 늘어남.  
AZ간 이중화를 해야 한다 → AWS의 인프라 관리 체계에 종속되기 때문(AWS에서 정기점검시 서비스도 정지시켜야 하는 상황이 발생할 수 있음)

AWS의 SNS 서비스를 이용해 Push 서버 없이 Push 기능을 구현할 수 있다.

대용량 메일 발송 서비스를 제공한다.(SMTP 서버도 제공)  
[http://www.slideshare.net/AmazonWebServices?utm_campaign=profiletracking&utm_medium=sssite&utm_source=ssslideview][3]

##### AWS Mobile Hub
- Auth, Push 적용 여부, 라이브러리, iOS/Android SDK, 앱 빌드까지 모두 제공
- 6개월 안에 시장조사부터 앱 런칭까지 끝내야 한다.
- 기존 개발방법론으로 하면 인프라 구축에만 6개월 걸리고, 이미 시장은 급변해 있다(한발 늦음)
[https://www.lucidchart.com][4]

**AWS 실습 웹사이트** : [https://qwiklab.com][5]
```
$sudo su -
#yum update
#yum install httpd
#service httpd start
```

[1]: http://skpla.net/ta1229
[2]: https://prezi.com/cxpwi_og7lht/aws-regions-and-availability-zones/#
[3]: http://www.slideshare.net/AmazonWebServices?utm_campaign=profiletracking&utm_medium=sssite&utm_source=ssslideview
[4]: https://www.lucidchart.com
[5]: https://qwiklab.com