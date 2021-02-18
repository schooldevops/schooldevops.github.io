---
layout: post
title:  "AWS Secret Manager With SpringBoot"
date:   2021-02-18 09:45:49 +0900
categories: AWS
tags: [AWS, Security, SpringBoot, Java, SecretManager]
toc: true
---

# 들어가기

프로그램을 개발하다보면, 단일 프로그램만이 수행되는 경우는 없다. 

- 데이터베이스에 접속한다. 
- 연관된 API 서버에 접속하여 데이터를 조회하거나 정보를 전송한다. 
- 메시지 큐를 이용한다. 

이러한 작업들은 모두 해당 서버에 접근하기 위해서 시크릿 정보가 필요하다. 

데이터베이스는 username/password, 연관 API 는 Token 등이 필요하고, 메시지 큐 역시 id/password 가 필요하다. 

일반적으로 가장 많이 사용하는 방법이 plain text 를 설정파일 application.yaml 이나 자바 코드를 이용하여 저장하여 사용한다. 
그러나 이는 매우 보안에 취약하며, 중요한 서버들의 접근 정보를 타인에게 공유하는 것이나 다름이 없다. 

회사의 보안 망 안에서만 소스코드를 관리한다고 하더라도, 이는 소스에 관련이 없는 사람이 서버의 정보를 봐야할 이유는 없기 때문에 좋은 선택이 아니다. 

## AWS Secret Manager

AWS Secret Manager 은 다양한 방법으로 secret 정보들을 저장할 수 있는 수단을 제공한다. 

우선 어떻게 만드는지 한번 알아보자. 

### AWS Secret Manager 열기 

![secret_manager01](https://user-images.githubusercontent.com/66154381/108319488-ce35cc80-7204-11eb-907b-d069b02efb9b.png)

새 보안 암호 저장 을 선택한다. 

### AWS Secret Manager 

![secret_manager02](https://user-images.githubusercontent.com/66154381/108319550-e86faa80-7204-11eb-93bd-d46731f56e87.png)

- RDS 데이터베이스에 대한 자격 증명: RDS 의 계정/암호를 저장할 수 있으며, 대상 계정에 계정과 암호를 세팅할 수 있다. 
- DocumentDB 데이터베이스에 대한 자격 증명: NoSQL 인 DocumentDB 에 계정/암호를 저장한다. 
- Redshift 클러스터에 대한 자격 증명: 빅데이터 저장소에 대해 계정/암호를 저장한다. 
- 기타 데이터베이스에 대한 자격 증명: 기타 데이터베이스에 대해 계정/암호를 저장한다. 
- 다른 유형의 보안 암호: 키-값 형태로 시크릿을 저장할 수 있게 한다. 

우리는 여기서 `다른 유형의 보안 암호` 를 선택할 것이다. 

그리고 하단에 `보안 암호 키/값` 탭에서 이미지와 같이 입력하자. 

사실 아무 값이나 입력해도 된다. 

### 새 보안 암호 이름 설정 

![secret_manager03](https://user-images.githubusercontent.com/66154381/108319577-f4f40300-7204-11eb-8060-ef2f3627df45.png)

보안 암호 이름: 보안 암호 이름은 디렉토리 구조로 지정한다. 보통 하나의 계정에서 여러 보안 암호를 이용하기 때문에 이런 디렉토리 구조로 팀/프로젝트/대상서버 등의 형태로 작성해 주면 좋다. 

설명, 태그도 같이 입력해 주자.

### 암호 교체 방식 설정 

![secret_manager04](https://user-images.githubusercontent.com/66154381/108319673-16ed8580-7205-11eb-8eb7-52a0953c3b99.png)

자동 교체 구성을 설정한다. 

- 자동 교체 비활성화: 자동 교체 비활성화는 고정된 암호를 사용하는 것이다. 이것은 이미 설정한 암호를 지정하는 경우 사용하며, 프로그램에 시크릿이 고정되어야 할 경우 주로 사용한다. 보통의 케이스는 자동 교체 비활성화를 선택하면 된다. 
- 자동 교체 활성화: 자동교체 활성화는 자동으로 특정 시스템의 암호가 변경되기를 원하는 경우 사용하면 된다. 더욱 강화된 보안을 제공하지만 해당 시스템에 접근할때마다 보안 암호를 가져와서 접속을 해야하는 경우 적합하다. 

### 샘플 코드 보기 

![secret_manager05](https://user-images.githubusercontent.com/66154381/108319712-2371de00-7205-11eb-90e1-db5080ee2afd.png)

보는바와 같이 여러 방법으로 시크릿에 접근할 수 있도록 샘플 코드를 제공하고 있다. 
선호하는 프로그래밍 언어를 선택해서 이를 이용하면 될 것이다. 

### 생성 결과 보기 

![secret_manager06](https://user-images.githubusercontent.com/66154381/108319741-2e2c7300-7205-11eb-8d89-f1a946fe82bf.png)

우리가 이전에 입력한 보안 암호 이름으로 보안 정보를 볼 수 있다. 

해당 보안 내용을 클릭하면 상세 정보를 볼 수 있다. 

![secret_manager07](https://user-images.githubusercontent.com/66154381/108320960-d4c54380-7206-11eb-82ca-d216218767e1.png)

지금까지 보안 암호 설정을 알아 보았다. 

이제 프로그래밍을 해보자. 

## SpringBoot 프로젝트에서 서버가 올라갈때 암호 가져오기. 

https://start.spring.io/ 에서 프로젝트를 하나 생성하자. 

### AWS Dependency 등록하기. 

pom.xml 파일에 다음과 같이 의존성을 추가해준다. 

```
... 중략 ...
<dependency>
  <groupId>com.amazonaws</groupId>
  <artifactId>aws-java-sdk-secretsmanager</artifactId>
  <version>1.11.549</version>
</dependency>
... 중략 ...
```

### 서버가 올라올때 aws 자격 정보를 가져 올 수 있도록 Listener 등록하기. 

PropertyListener.java 파일을 아래와 같이 만들어 준다. 

```java
@Component
public class PropertyListener implements ApplicationListener<ApplicationPreparedEvent> {

    ObjectMapper objectMapper = new ObjectMapper();
    private static final Logger LOGGER = LoggerFactory.getLogger(PropertyListener.class);
    private static final String AWS_SECRET_NAME = "myproject/schooldevops/db";

    @Override
    public void onApplicationEvent(ApplicationPreparedEvent event) {
        String secretJson = getSecretV1();

        ConfigurableEnvironment env = event.getApplicationContext().getEnvironment();
        Properties properties = new Properties();

        properties.put("myproject.schooldevops.db.username", getValue(secretJson, "username"));
        properties.put("myproject.schooldevops.db.password", getValue(secretJson, "password"));
        properties.put("myproject.schooldevops.db.token", getValue(secretJson, "usertoken"));

        env.getPropertySources().addFirst(new PropertiesPropertySource("myproject.schooldevops.db", properties));
    }

    private String getSecretV1() {
        // Create a Secrets Manager client
        AWSSecretsManager client  = AWSSecretsManagerClientBuilder.standard()
                .withRegion(Regions.AP_NORTHEAST_2)
                .build();

        String secret, decodedBinarySecret;
        GetSecretValueRequest getSecretValueRequest = new GetSecretValueRequest()
                .withSecretId(AWS_SECRET_NAME);
        GetSecretValueResult getSecretValueResult = null;

        try {
            getSecretValueResult = client.getSecretValue(getSecretValueRequest);
        } catch (DecryptionFailureException e) {
            // Secrets Manager can't decrypt the protected secret text using the provided KMS key.
            // Deal with the exception here, and/or rethrow at your discretion.
            throw e;
        } catch (InternalServiceErrorException e) {
            // An error occurred on the server side.
            // Deal with the exception here, and/or rethrow at your discretion.
            throw e;
        } catch (InvalidParameterException e) {
            // You provided an invalid value for a parameter.
            // Deal with the exception here, and/or rethrow at your discretion.
            throw e;
        } catch (InvalidRequestException e) {
            // You provided a parameter value that is not valid for the current state of the resource.
            // Deal with the exception here, and/or rethrow at your discretion.
            throw e;
        } catch (ResourceNotFoundException e) {
            // We can't find the resource that you asked for.
            // Deal with the exception here, and/or rethrow at your discretion.
            throw e;
        }

        // Decrypts secret using the associated KMS CMK.
        // Depending on whether the secret is a string or binary, one of these fields will be populated.
        if (getSecretValueResult.getSecretString() != null) {
            secret = getSecretValueResult.getSecretString();
            return secret;
        }
        else {
            decodedBinarySecret = new String(Base64.getDecoder().decode(getSecretValueResult.getSecretBinary()).array());
            return decodedBinarySecret;
        }
    }

    private String getValue(String json, String key) {
        try {
            JsonNode jsonNode = objectMapper.readTree(json);
            return jsonNode.path(key).asText();
        } catch (JsonProcessingException e) {
            LOGGER.error(e.getMessage(), e);
            return null;
        }
    }
}
```

위 코드 내용을 보면 ApplicationListener 가가 처음 서버가 부팅될때, 리스너가 동작하도록 해준다. 

ApplicationListener 구현체는 @Override public void onApplicationEvent(ApplicationPreparedEvent event) 를 구현해야한다. 

여기서는 aws 에서 시크릿을 가져오도록 getSecretV1() 을 호출하여, 시크릿 설정 정보를 조회한다. 

이때 secretName 은 우리가 이전에 만들었던 `myproject/schooldevops/db` 시크릿을 가져오는 역할을 한다. 

결과값을 jsonString 으로 반환하며, getValue 를 이용하여 json 에서 키에 해당하는 값을 추출한다. 

```java
  ConfigurableEnvironment env = event.getApplicationContext().getEnvironment();
  Properties properties = new Properties();

  properties.put("myproject.schooldevops.db.username", getValue(secretJson, "username"));
  properties.put("myproject.schooldevops.db.password", getValue(secretJson, "password"));
  properties.put("myproject.schooldevops.db.token", getValue(secretJson, "usertoken"));
```

프로퍼티 속성을 설정해주는 역할을 한다. 

조금 있다가 이 값이 어떻게 사용하는지 보게 될 것이다. 

### 프로퍼티 사용하는 예제 컨트롤러 

SourceController.java 

```java
@RestController
@RequestMapping("/api")
public class SecretController {

    @Value("${myproject.schooldevops.db.username}")
    private String username;

    @Value("${myproject.schooldevops.db.password}")
    private String password;

    @Value("${myproject.schooldevops.db.token}")
    private String token;

    @GetMapping("/secret/{value}")
    public String getSecretValue(@PathVariable String value) {
        if ("username".equals(value)) {
            return username;
        } else if ("password".equals(value)) {
            return password;
        } else if ("token".equals(value)) {
            return token;
        } else {
            return "NOE";
        }
    }

}
```

우리는 시크릿 정보를 @Value 를 이용하여 가져왔다. 

이렇게 가져온 값을 이용하여 DB DataSource 등에도 적용할 수 있을 것이다. 

### Listener 등록하기. 

이제는 어플리케이션이 올라올때 Listener 이 동작하도록 등록해보자. 

TestsecretApplication.java

```java
@SpringBootApplication
public class TestsecretApplication {

	public static void main(String[] args) {
		SpringApplication app = new SpringApplication(TestsecretApplication.class);
		app.addListeners(new PropertyListener());
		app.run(args);
	}

}
```

app.addListener(new PropertyListener()); 을 이용하여 등록한 것을 알수 있을 것이다. 

### 로컬 환경에서 aws configure 설정하기. 

시크릿 정보를 가져 오기위해서는 로컬에 aws 세션 정보가 들어 있어야한다. 

이를 위해서 다음과 같이 aws 시크릿 정보를 조회할 수 있도록 access_key, secret_access_key 를 설정하자. 

```
aws configure
AWS Access Key ID [****************]: 
AWS Secret Access Key [****************]: 
Default region name [ap-northtast-2]: ap-northtast-2
Default output format [json]: json
```

### 결과 확인하기. 

서버를 실행하고, 다음 curl 을 실행하면 우리가 원하는 값이 넘어오는 것을 알 수 있다. 

```
Ξ Downloads/testsecret git:(main) ▶ curl localhost:8080/api/secret/username
schooldevops

Ξ Downloads/testsecret git:(main) ▶ curl localhost:8080/api/secret/password  
pass!@#qwe

Ξ Downloads/testsecret git:(main) ▶ curl localhost:8080/api/secret/token   
EzAFK6lV5AzEy4VFv2ND44g3nhqo3bTgnt3SMZFARLA=
```

## 고민해보기. 

이제 고민해보자. 

위 코드로 우리는 secret manager 에 접근하여 시크릿 정보를 가져 왔다. 

그러나 이 코드가 정말 좋은 코드인지는 생각해 볼 문제이다. 무슨 문제가 있는지 알아보자. 

- 소스 코드에 시크릿 정보를 하드코딩 하고 있다. 
  - 시크릿 정보가 변경이 일어난다면 코드를 수정해야한다. 이것은 좋은 신호가 아니다. 
- aws secret manager 와 코드가 커플링 되어 있다. 
  - 우리가 작성하는 코드에서 secret manager 가 수행하는 역할은 무엇인가? 단순히 시크릿 정보를 프로퍼티로 가져오는 역할을 하고, 더이상은 aws 의 sdk 를 사용해야할 이유는 없다. 
  - 비즈니스에 집중한 코드 측면에서 보면 이 방법은 좋지 않다. 
- 서버를 배포하는 곳에 일일이 aws configure 값을 세팅해야한다. 
  - 이것은 보안상 매우 위험한 코드이다. aws secret 만 접근하는 롤을 가진 유저의 시크릿을 이용하면 되지만 위험은 여전하다. 

위 내용만 보더라도 좋은 방법은 아니다. 

그럼 어떻게 하는것이 좋을까? 이에 대한 하나의 해결책은 다음 아티클에서 알아 볼 것이다. 

## 결론

지금까지 aws secret manager 를 이용한 Spring Boot 코드를 알아 보았다. 

직접 시크릿을 생성하고, 어떻게 시크릿을 조회하여 프로퍼티로 등록하는지도 알아 보았다. 

그리고 이런 방식으로 코드가 작성되는 것이 좋은지에 대해서도 고찰해 보았다. 

어쨌든 시크릿을 하드코딩 하는 것 보다 훨씬 안전한 코드가 되었다. 이제 이 코드가 깃헙에 올라가더라도 서버의 정보를 노출하는 일은 없을 것이다. 

다음 아티클에서는 이보다 더 좋은 방법을 알아보고 함께 고찰해보자. 

위 전체 소스는 [Git](https://github.com/schooldevops/aws-secret-manager-java) 에서 살펴볼 수 있다. 