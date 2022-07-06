---
layout: post  
title:  "Resilience4J를 사용한 Circuit Breaker 구현"  
date:   2021-03-22 11:31:00  
categories: JAVA SpringBoot SpringCloud 
---

새로 API 프로젝트를 구축하는데 Circuit Breaker라는 것이 무엇인지 본격적으로 알려준 Netflix hystrix의 시대가 막을 내렸다는 것을 알게 되었고 Resilience4J를 사용하여 구현한 내용을 정리해보았다.

##### Spring Cloud 2020.0 Release
* 2020-12-22 2020.0.0 버전 정식 Release
* Spring Boot 2.4.x 버전을 기반으로 하며 하위 버전을 지원하지 않음
* Spring Cloud Security 프로젝트 제거
* Spring Cloud Gateway는 Spring Cloud LoadBalancer를 기반으로 하고 Ribbon 미지원
* Netflix 계열 라이브러리 종속성 완전 제거
![project](./../../../../../../../images/20210322/1.png)

##### Netflix OSS Out
* Spring Cloud Dalston 릴리즈 이전까지는 Hystrix가 Class Path에만 있다면 Feign Client는 기본적으로 Hystrix의 메서드를 Wrapping하고 있었다.
![project](./../../../../../../../images/20210322/2.png)
* Feign에서 사용될 Client Load Balancer도 기본 Ribbon을 권장하고 있었다.
![project](./../../../../../../../images/20210322/3.png)
* Spring Cloud 2020.0 문서에는 Ribbon에 대한 언급이 없다.
![project](./../../../../../../../images/20210322/4.png)
* Ribbon은 유지보수 모드이기 때문에 Spring Cloud LoadBalancer를 권장한다.(Hystrix도 마찬가지)
![project](./../../../../../../../images/20210322/5.png)
* Spring Cloud OpenFeign은 Spring Cloud Circuit Breaker의 추상 적응을 지원한다.
Hystrix는 더 이상 Default가 아니다.
![project](./../../../../../../../images/20210322/6.png)
_(Cannot resolve configuration property)_
* Netflix에서도 다른 것 쓰겠다고 함.
![project](./../../../../../../../images/20210322/7.png)

##### Circuit Breaker (회로차단기)
MSA 아키텍처 환경에서 시스템 구성의 일부에서 결함이 발생해도 정상적 혹은 부분적으로 기능을 수행할 수 있는 내결함 시스템 (Fault Tolerance System, 장애 허용 시스템)을 구현하기 위하여 사용된다.  

특정 API 호출 등과 같은 작업에서 임계구간을 설정하고 임계점 이상 실패가 발생하면 기존에 설정한 Fallback method(대체 작업)에 맞게 동작하도록 하게 하고(Circuit Open) 일정 시간 이후 다시 시도하여 진행하게 한다.  

여러 마이크로 서비스들이 얽혀있는 MSA 환경에서 문제가 있는 서비스로의 트래픽을 차단하여 속도 저하나 장애 전파를 방지하는데 목적이 있다.
![project](./../../../../../../../images/20210322/8.png)

##### Resilience4J
Netflix Hystrix에 영향을 받아 만들어진 경량 내결함성 라이브러리
* 외부 의존성이 없는 [Vavr 라이브러리](https://sun-22.tistory.com/96)만 사용해서 경량 ↔ Hystrix는 Guava, Apache Commons Configuration 등의 외부 의존성이 큼
* JDK8과 함수형 프로그래밍에 맞춰 설계

###### Resilience4J에서 제공하는 Core Module
* __Circuit Breaker__ : 회로차단기 패턴을 구현
* Bulkhead : 동시 실행(Concurrent execution) 수를 제한
* RateLimiter : 일정 시간 동안 요청 수 제한
* Retry : 요청이 실패했을 때 재시도 정책에 대한 설정
* __TimeLimiter__ : 원격 서버 호출에 걸리는 시간 제한 설정
* Cache

###### Circuit Breaker Pattern
![project](./../../../../../../../images/20210322/9.png)
* 설정한 임계점 이상 실패나 지연 응답이 발생하면 Circuit은 OPEN되며 대체 작업을 수행한다.
* 대기 시간이 경과한 후 Circuit은 HALF_OPEN 상태로 변경되며 작업 수행이 가능한지 다시 확인한다.
* 실패율이 임계값보다 작으면 Circuit은 CLOSE된다.
* 실패율이 임계값보다 크면 Circuit은 다시 OPEN된다.
* Resilience4J의 Circuit Breaker는 DISABLE, FORCED_OPEN 등 2개의 특수 상태를 지원한다.

###### Resilience4J에서의 Circuit Breaker 설정값
![project](./../../../../../../../images/20210322/table01.png)
![project](./../../../../../../../images/20210322/table02.png)

###### Netflix Hystrix와의 비교
* Hystrix에서는 외부 요청은 HystrixCommand로 래핑되어야 한다. 반면 Resilience4J는 함수형 인터페이스, 람다식, 메서드 참조를 Circuit Breaker, Rate Limiter, Retry, Bulkhead로 확장하기 위한 고차 함수(decorator)를 제공한다. Circuit Breaker decorator에 Bulkhead, Rate Limiter, Retry 등의 decorator를 추가하는 것이 가능하다. 이것에 의해 필요한 decorator를 선택하여 사용할 수 있다. 모든 decorator된 함수는 동기 또는 비동기(RxJava를 사용)로 실행할 수 있다.
* Hystrix는 HALF_OPEN 상태일때 Circuit을 CLOSE 할 지 결정할 딱 한번의 요청을 수행한다. 하지만 Resilience4J는 요청의 수행 횟수와 임계점을 설정할 수 있다.
* Resilience4J는 Reactor 또는 RxJava의 Custom Operator를 지원한다.
* 두 라이브러리 모두 요청 결과와 지연 시간을 모니터링 할 수 있는 이벤트 스트림을 내보낸다.

##### Spring Boot 프로젝트에 Resilience4J 적용
1. resilience4j-spring-boot2 라이브러리 사용
build.gradle에 아래의 라이브러리 의존성을 추가한다.
* org.springframework.boot:spring-boot-starter-actuator
* org.springframework.boot:spring-boot-starter-aop
* io.github.resilience4j:resilience4j-spring-boot2
* (webflux 환경일 경우 추가) io.github.resilience4j:resilience4j-reactor

```groovy
ext {
    set('springCloudVersion', "2020.0.1")
    set('resilience4jVersion', "1.6.0")
}
 
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.boot:spring-boot-starter-aop'
    implementation "io.github.resilience4j:resilience4j-spring-boot2:${resilience4jVersion}"
}
```

application.yml에 property값 등록으로 기본적인 설정을 한다.(하지 않으면 Default 설정으로 동작)

```yaml
resilience4j:
  circuitbreaker:
    configs:
      default:
        registerHealthIndicator: true
        minimumNumberOfCalls: 4  # 요청 실패나 지연된 응답 수를 계산하기 위해 요구되는 최소 요청의 수
        failureRateThreshold: 50 # 실패율
        slidingWindowType: COUNT_BASED # COUNT_BASED는 마지막 n개의 호출 횟수
```

API 요청 Client를 작성한다.(여기에서는 Feign Client를 사용)

```java
@LoadBalancerClient(name = "sample", configuration = SampleLoadbalancerConfig.class)
@FeignClient(name = "sample")
public interface SampleClient {
 
    @GetMapping("/users/{id}")
    Sample getUser(@PathVariable("id") String id);
}
```

Client 객체 의존성을 주입한 Wrapper 클래스를 생성하고, @CircuitBreaker 어노테이션으로 Circuit의 이름과 대체 작업(Fallback Method)을 명시한다.

```java
@Slf4j
@Component
public class SampleClientWrapper {
 
    private final SampleClient sampleClient;
 
    public SampleClientWrapper(SampleClient sampleClient) {
        this.sampleClient = sampleClient;
    }
 
    @CircuitBreaker(name = "sample", fallbackMethod = "getUserFallback")
    public Sample getUser(String id) {
        log.info("Call Sample API");
        return sampleClient.getUser(id);
    }
 
    public Sample getUserFallback(Exception e) {
        log.error("user's data is null!! : {}", e.toString());
        return new Sample();
    }
}
```

확인

* 실패율 임계점 50%, 집계에 필요한 최소 요청수 4회, COUNT_BASED로 설정하고 외부 API 요청 테스트
* API 요청을 2회 수행한 다음 API 서비스 Down
* 2회 실패 후 Circuit Breaker Open 로그 확인
![project](./../../../../../../../images/20210322/10.png)

구현 후 느낀 점
* Wrapper 클래스를 만들어서 사용해야 한다. → Feign Configuration을 Override해서 따로 설정하면 Wrapper 클래스 만들지 않아도 될 듯?
* Retrofit, Rest Template를 사용할 때 쓰면 좋을 것 같음.
* 어노테이션을 사용해 Resilience4J의 모든 기능을 사용할 수 있다.
* application.yml 를 통한 설정

2. Spring Cloud Circuit Breaker 를 사용하는 방법
* Spring Cloud 프로젝트에 편입된 Resilience4J
* Hystrix, Sentinel, Spring Retry를 지원하는 버전도 존재하나 Resilience4J를 권장함
기존 Spring Boot 프로젝트에 아래의 라이브러리 의존성을 추가한다.
* (non-reactive일 경우) org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j
* (reactive일 경우) org.springframework.cloud:spring-cloud-starter-circuitbreaker-reactor-resilience4j

````groovy
ext {
    set('springCloudVersion', "2020.0.1")
}
 
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.cloud:spring-cloud-starter-config'
    implementation 'org.springframework.cloud:spring-cloud-starter-circuitbreaker-resilience4j'
    implementation 'org.springframework.cloud:spring-cloud-starter-loadbalancer'
    implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
}
````

application.yml에 feign.circuitbreaker.enabled: true로 설정한다.

````yaml
feign:
  circuitbreaker:
    enabled: true
````

Java Config로 Resilience4J의 CircuitBreakerFactory Bean을 설정한다.

````java
import io.github.resilience4j.circuitbreaker.CircuitBreakerConfig;
import org.springframework.cloud.circuitbreaker.resilience4j.Resilience4JCircuitBreakerFactory;
import org.springframework.cloud.circuitbreaker.resilience4j.Resilience4JConfigBuilder;
import org.springframework.cloud.client.circuitbreaker.CircuitBreakerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
import java.util.function.Function;
 
import static io.github.resilience4j.circuitbreaker.CircuitBreakerConfig.SlidingWindowType.COUNT_BASED;
 
@Configuration
public class Resilience4JConfig {
 
    @Bean
    public CircuitBreakerFactory circuitBreakerFactory() {
        final CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
                .minimumNumberOfCalls(4)
                .failureRateThreshold(50)
                .slidingWindowType(COUNT_BASED)
                .build();
 
        final CircuitBreakerFactory factory = new Resilience4JCircuitBreakerFactory();
        factory.configureDefault((Function<String, Resilience4JConfigBuilder.Resilience4JCircuitBreakerConfiguration>) s ->
            new Resilience4JConfigBuilder(s).circuitBreakerConfig(circuitBreakerConfig).build());
        return factory;
    }
}
````

API 요청을 수행할 Feign Client와 Fallback Method가 정의된 클래스를 생성한다.
__Feign Client__

````java
@FeignClient(name = "array", url = "http://localhost:3000", fallbackFactory= ArrayClientFallbackFactory.class)
public interface ArrayClient {
    @RequestMapping("/users")
    List<Sample> getUsers();
}
````

__Fallback Factory__

````java
@Slf4j
@Component
public class ArrayClientFallbackFactory implements FallbackFactory<ArrayClient> {
 
    @Override
    public ArrayClient create(Throwable cause) {
        return () -> {
            log.error("[Access Error] : {}", cause.toString());
            return Collections.emptyList();
        };
    }
}
````

확인
* 실패율 임계점 50%, 집계에 필요한 최소 요청수 4회, COUNT_BASED로 설정하고 외부 API 요청 테스트
* API 요청을 2회 수행한 다음 API 서비스 Down
* 2회 실패 후 Circuit Breaker Open 로그 확인
![project](./../../../../../../../images/20210322/11.png)

구현 후 느낀 점
* Feign과 유기적으로 잘 결합함 (Wrapper 클래스를 꼭 만들지 않아도 됨)
  * Wrapper 클래스를 만들어서 사용한 예제

````java
@Slf4j
@Component
public class SampleClientWrapper {
    private final SampleClient sampleClient;
    private final CircuitBreaker circuitBreaker;
 
    public SampleClientWrapper(SampleClient sampleClient, CircuitBreakerFactory circuitBreakerFactory) {
        this.sampleClient = sampleClient;
        this.circuitBreaker = circuitBreakerFactory.create("sample");
    }
 
    public Sample getUser(String id) {
        return circuitBreaker.run(() -> sampleClient.getUser(id), throwable -> getUserFallback(throwable));
    }
 
    private Sample getUserFallback(Throwable t) {
        log.error("user's data is null!! : {}", t.toString());
        return new Sample();
    }
}
````

* 기존 코드에 변경이 거의 일어나지 않는다.
* Circuit Breaker, TimeLimiter 2개의 기능만 제공
* Java Config로 설정한다.

##### 참고 자료
* Resilience4J 레퍼런스
  * https://resilience4j.readme.io
* Spring Cloud Circuit Breaker 레퍼런스
  * https://docs.spring.io/spring-cloud-circuitbreaker/docs/current/reference/html/
* Spring Cloud OpenFeign 레퍼런스
  * https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/
* Spring Cloud 2020.0 Release Notes
  * https://github.com/spring-cloud/spring-cloud-release/wiki/Spring-Cloud-2020.0-Release-Notes
* Circuit Breaker (Resilience4j)
  * https://rusyasoft.github.io/java/2020/03/30/Circuit-Breaker-Resilience4j/