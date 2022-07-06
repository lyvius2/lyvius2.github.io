---
layout: post  
title:  "Queue에 메시지 전송하는 Producer 개발"  
date:   2022-05-27 13:22:00  
categories: AWS, Spring Cloud AWS, Kafka
---

Kafka와 SQS에 메시지를 발송하는 Producer의 개발 방법이다.  
솔직히 해보면 매우 간단하고, 단일 프로젝트에 같이 구현하는 것도 가능하다.
SpringBoot로 기본적인 API 프로젝트를 만들고 Controller에서 호출할 메시지 송신 메서드를 구현해본다.

##### Kafka Producer

1.build.gradle 에 spring-kafka 라이브러리 의존성을 추가한다.

```groovy
dependencies {
    // Kafka
    implementation 'org.springframework.kafka:spring-kafka:2.8.2'
}
```

2.application.yml에 메시지를 송신할 Kafka Broker의 서버 정보를 설정한다.

```yaml
spring:
  kafka:
    producer:
      bootstrap-servers: kafka.01.server.com:9092,kafka.02.server.com:9092,kafka.03.server.com:9092
```

3.Service 클래스를 하나 만들고 KafkaTemplate 빈 의존성을 주입한다.

```java
@Service
public class KafkaSendingService {
    private final KafkaTemplate kafkaTemplate;
 
    public KafkaSendingService(KafkaTemplate kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }
}
```

KafkaTemplate 빈은 자동 생성되므로 Override 할 필요가 없다면 따로 생성하지 않아도 된다.

4.TOPIC명과 메시지 내용을 매개변수로 받아 Kafka Broker로 보내는 메서드를 아래와 같이 작성한다.

```java
public void sendMessage(String topicName, String message) {
    final ListenableFuture<SendResult<String, String>> future = kafkaTemplate.send(topicName, message);
    future.addCallback(new ListenableFutureCallback<SendResult<String, String>>() {
         @Override
         public void onSuccess(SendResult<String, String> result) {
             log.info("Kafka sent message='{}' with offset={}", message, result.getRecordMetadata().offset());
         }
 
         @Override
         public void onFailure(Throwable ex) {
             log.error("Kafka unable to send message='{}'", message, ex);
             throw new RuntimeException();
         }
     });
}
```

Controller나 다른 요소에서 위 sendMessage 메서드를 호출하면 메시지 전송이 된다.

##### SQS Producer

1.build.gradle에 Spring Cloud AWS의 라이브러리 중 SQS에 관련된 요소의 의존성을 추가한다.

```groovy
dependencies {
    // Sqs
    implementation 'io.awspring.cloud:spring-cloud-aws-autoconfigure:2.4.1'
    implementation 'io.awspring.cloud:spring-cloud-aws-messaging:2.4.1'
    implementation 'io.awspring.cloud:spring-cloud-starter-aws:2.4.1'
}
```

2.application.yml에 AWS Credential 과 Region 관련 설정을 넣는다.

```yaml
cloud:
  aws:
    credentials:
      accessKey: # access-key 사용시 적시
      secretKey: # secret-key 사용시 적시
      use-default-aws-credentials-chain: true
    region:
      static: ap-northeast-2
    stack:
      auto: false
```

cloud.aws.stack.auto는 기본값 설정이 true인데 [AWS CloudFormation](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/Welcome.html)이 셋팅되어 있지 않으면 에러를 발생시키므로 false로 설정한다.  
위 설정에서 accessKey와 secretKey는 AWS가 아닌 다른 환경에서 구동할 때 필요하다.  
AWS SDK에서는 credential 관련해서 6가지의 인증 옵션을 제공하는데, 아무런 설정을 하지 않으면 우선 순위에 의해서 인증 옵션이 순차적으로 적용된다.(Provider Chain)

Default Credential Provider Chain : https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html

별도의 설정을 하고 싶지 않으면 use-default-aws-credentials-chain의 값을 true로 지정해줘야 Provider Chain이 작동된다. (기본값이 false)  
특정한 인증수단을 지정해주고 싶으면, SQS나 S3의 Client 빈을 별도로 생성해주고 인증수단을 적시해놓는다.(아래 예시)

```java
@Bean
public AmazonSQS amazonSQS() {
    return AmazonSQSAsyncClientBuilder
            .standard()
            .withCredentials(new WebIdentityTokenCredentialsProvider())
            .withRegion(Regions.AP_NORTHEAST_2)
            .build();
}
```

3.QueueMessagingTemplate 빈을 생성한다.

```java
import com.amazonaws.services.sqs.AmazonSQS;
import com.amazonaws.services.sqs.AmazonSQSAsync;
import io.awspring.cloud.messaging.core.QueueMessagingTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
@Configuration
public class SqsConfig {
    @Bean
    public QueueMessagingTemplate queueMessagingTemplate(AmazonSQS amazonSQS) {
        return new QueueMessagingTemplate((AmazonSQSAsync) amazonSQS);
    }
}
```

4.Service 클래스를 하나 만들고 QueueMessagingTemplate 빈 의존성을 주입한다.

```java
@Service
public class SqsSendingService {
    private final QueueMessagingTemplate queueMessagingTemplate;
 
    public SqsSendingService(QueueMessagingTemplate queueMessagingTemplate) {
        this.queueMessagingTemplate = queueMessagingTemplate;
    }
}
```

5.Queue 이름과 메시지를 매개변수로 받아 SQS로 보내는 메서드를 아래와 같이 작성한다.

```java
public void sendMessage(String queueName, String message) {
    final Message<String> newMessage = MessageBuilder.withPayload(message).build();
    queueMessagingTemplate.send(queueName, newMessage);
}
```

여기서 Queue 이름만 넣어서 전송하면 메시지는 당연하게도(?) credential 설정으로 인증된 AWS Account의 SQS에서 같은 이름의 Queue를 찾아간다.  
만약 Queue가 없거나 하면 Error가 발생하고, 다른 AWS Account에서 생성된 SQS Queue로 보내고자 한다면 Queue의 Full URL을 넣어서 전송해야 한다.
서울 리전이라면 SQS의 Full URL은 아래의 형식으로 되어 있을 것이다.

> https://sqs.ap-northeast2.amazonaws.com/{aws account 번호}/{Queue 대기열 이름}

Queue의 엑세스 정책에도 특정한 타 AWS Account에서도 접근이 가능하게 설정이 되어 있어야 한다.  
Controller나 다른 요소에서 위 sendMessage 메서드를 호출하면 메시지 전송이 된다.