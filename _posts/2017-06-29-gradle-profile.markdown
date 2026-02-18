---
layout: post
title:  "Gradle 배포 환경 별 설정파일 분리"
date:   2017-06-29 23:49:00
categories: JAVA
---

대부분 웹 애플리케이션을 개발할 시 DB를 개발용과 운영용으로 구분하여 개발하는데 WAS에 DB커넥션풀을 생성한 뒤 JNDI로 설정해두고 Spring의 context.xml에서 데이터 소스로 가져와서 쓰게 하는 방법을 쓰지 않는 한.... 소스 안에 직접 DB접근정보들을 프로퍼티로 해서 넣게 된다.

이렇게 하면 운영환경에 배포할 때마다 일일히 이 프로퍼티 파일의 개발DB 정보를 운영DB 정보로 바꿔서 올려야 하는 수고를 하게 된다.

회사 플젝의 경우 DB커넥션풀은 대부분 JNDI로 설정해두고 쓸거고(...아닌가?) 개인적으로도 커넥션풀은 WAS에서 생성해두고 쓰는게 낫다고 생각하고 있는데... 개인 플젝은 헤로쿠(heroku.com)에 올릴 것인데 이게 JNDI 설정하기가 좀 까다로운 거 같아서(사실 아직 방법을 못찾았다) 어쩔 수 없이 DB접근정보를 웹 애플리케이션 내부 프로퍼티 파일에 넣어놓고 deploy 하게 되었다.

배포환경 별로 설정파일을 분리하는 방법은 여러가지가 있는데 그냥 properties 파일 안에 개발인지 운영인지 구분하는 프로퍼티를 설정하고 deploy 할 때마다 그 프로퍼티만 dev에서 live로 바꿔서 올리는 방법이 있고 WAS 기동시 argument를 넘겨서 Spring의 context.xml에서 인식하게 하여 구분하게 하는 방법, 그리고 maven이나 gradle 로 빌드할 때 프로퍼티를 줘서 build를 배포환경 별로 달리 수행되게 하는 방법이 있는데 내가 보기엔 마지막 방법이 가장 나아보인다. 왜냐면 첫번째나 두번째 방법은 운영에 배포할 때 개발환경의 프로퍼티 파일이 war에 포함되서 결국 더미 파일을 가지고 있을 수 밖에 없는데 maven이나 gradle를 활용하는 방법은 아예 배포환경에 필요한 파일들만 포함시켜서 deploy 할 수 있기 때문이다.

gradle을 사용하여 설정파일 분리하는 방법이다.

1. 우선 main 경로 밑에 기존의 resources 디렉터리 외 임의의 다른 디렉터리를 만들고, 그 밑에 기존의 프로퍼티 파일을 개발용 운영용으로 분리하여 집어넣는다.
![IntelliJ](./../../../../../images/20170629/252FA5365955101F01.png)
resources-env라는 디렉터리를 만들고 그 밑에 개발용과 운영용을 분리하여 'dev' 와 'live' 디렉터리를 만들고 프로퍼티 파일들을 옮겨 놓았다.

2. build.gradle에서 빌드대상 소스파일들의 경로를 설정한 아래 부분을
```
sourceSets {
    main.java.srcDirs=['src/main/java']
    main.resources.srcDirs=['src/main/resources']
}
```
아래와 같이 바꿔준다. 상단의 조건문은 'profile'이라는 프로퍼티가 없거나 profile의 값이 없을 때는 profile의 값을 기본 'dev'로 정하는 내용이고 전달되어오는 profile 프로퍼티의 값에 따라서... resources-env/dev 혹은 resources-env/live 둘 중 한 곳만 war에 포함되어 빌드 될 것이다.
```
sourceSets {
    if (!project.hasProperty('profile') || !profile) {
        ext.profile = 'dev'
    }
    main {
        java {
            srcDirs "src/main/java"
        }
        resources {
            srcDirs "src/main/resources", "src/main/resources-env/${profile}"
        }
    }
}
```
3. gradle 명령어로 빌드할 때... 아래와 같이 프로퍼티 붙여서 수행하면 잘 된다...  
개발 : ***gradle build -Pprofile=dev***  
운영 : ***gradle build -Pprofile=live***