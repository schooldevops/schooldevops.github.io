---
layout: post
title:  "AWS STS(AssumeRole) 실습  "
date:   2021-03-30 10:01:49 +0900
categories: AWS
tags: [AWS, aws, STS, assumeRole]
toc: true
---

# AWS STS(AssumeRole) 실습

AWS 를 이용하는 경우 안전한 서비스를 위해서 STS(Security Token Service)를 활용할 수 있다. 

사용자 계정을 생성하고, 임시의 롤을 사용자에게 부여하는 방식으로 이용할 수 있다. 

## 진행과정

1. 사용자 계정을 생성한다. 
2. 롤을 생성한다. 
   1. 롤에 대해서 신뢰하는 사용자를 등록한다. (AssumeRole을 잉요할 수 있게 된다.)
3. AssumeRole 을 부여 받고, CLI 를 이용하여 임시 계정과 토큰을 (Credential) 획득한다. 
4. 획득한 크레덴셜을 활용하여 지정된 리소스에 접근한다. 
5. 임시 크레덴셜을 삭제한다. 

## 사용자 계정 생성하기. 

AWS Console 에서 IAM 을 선택한다. 

![sts01](https://user-images.githubusercontent.com/66154381/112921714-44074f00-9146-11eb-8dc9-d99b2a88455d.png)

위와 같이 신규 사용자를 추가한다. 

- 사용자 이름은 assume-test-user 로 지정했다. 
- Access 유형: 프로그래밍 방식 엑세스를 지정한다. 

이후 다음 으로 넘어간다. 

아래 권한 설정은 별다른 설정없이 진행하자. 

![sts02](https://user-images.githubusercontent.com/66154381/112921730-48336c80-9146-11eb-927a-653dbb21a2aa.png)

나머지 부분도 모두 지정하지 않고 사용자 생성을 완료한다. 

사용자를 생성하고 나서. ACCESS_ID, SECRET_ACCESS_KEY 를 반드시 다운로드 받거나, 별도로 저장해 두자. 

## 역할 생성하기. 

이제는 IAM > 역할 에서 역할을 신규로 생성한다. 

"역할 만들기" 버튼을 클릭한다. 

![sts03](https://user-images.githubusercontent.com/66154381/112921733-49649980-9146-11eb-971e-f7dcb45ea861.png)

아래와 같이 역할 만들기 화면이 나타나면 "일반 사용 사례 > EC2" 를 선택하자. 

![sts04](https://user-images.githubusercontent.com/66154381/112921736-4a95c680-9146-11eb-8966-40bd27567041.png)

열할 만들기 항목에서는 우리가 테스트해볼 S3 를 입력한다. 

그리고 AmazonS3FullAccess 를 체크하고 다음으로 넘어간다. 

![sts05](https://user-images.githubusercontent.com/66154381/112921738-4b2e5d00-9146-11eb-954e-acf968fa1f63.png)

아래와 같이 역할 태그를 설정하자. 

- Name: AssumeRoleTestRole 로 지정했다. 
  
![sts06](https://user-images.githubusercontent.com/66154381/112921739-4bc6f380-9146-11eb-96d2-fa7e8db081ad.png)

검토 부분에서는 역할 이름을 지정한다. 

test-assume-role 로 역할 이름을 지정하고 생성했다. 

![sts07](https://user-images.githubusercontent.com/66154381/112921741-4c5f8a00-9146-11eb-86dd-56e0345b4d0f.png)

생성된 역할 목록을 선택한다. 

![sts08](https://user-images.githubusercontent.com/66154381/112921744-4cf82080-9146-11eb-9bc2-93030c3dc7cc.png)

요약 > 신뢰 관계 탭을 클릭한다. 

- 신뢰 관계 편집 을 클릭하여 사용자를 추가해 줄 것이다. 

![sts09](https://user-images.githubusercontent.com/66154381/112921745-4d90b700-9146-11eb-81e0-f344255d7db6.png)

사용자를 추가해 주기 위해서는 IAM > User 에서 조금전에 만든 "assume-test-user" 를 선택하고, 사용자 ARN 을 복사한다. 

![sts10](https://user-images.githubusercontent.com/66154381/112921749-4e294d80-9146-11eb-9fe0-be5f642b135a.png)

그리고 역할에서 > 신뢰관계 편집 항목 내용을 변경해주자. 

![sts11](https://user-images.githubusercontent.com/66154381/112921751-4ec1e400-9146-11eb-829e-731e09d69591.png)

우리가 변경한 부분은 Principal 부분이며

- AWS: "arn ....." 이 부분이다. 

이로서 AssumeRole 을 생성할 준비가 완료 되었다. 

## AWS CLI 설치하기. 

이제 부터는 콘솔을 이용하여 STS 를 조회할 것이기 때문에 AWS CLI 를 설치하자. 

설치는 : https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-macos.html 을 참조하자. 

## jq 설치

jq 는 Json 을 파싱하는 매우 강력한 커맨드라인 툴이다. 

보통 AWS 의 결과가 JSON 으로 받게 되며, 이를 파싱해서 원하는 값을 조회해야한다. 그러므로 jq 를 설치하자. 

```go
brew install jq
```

위 명령을 이용하여 jq 를 설치한다. 

## 생성된 사용자의 AccessKeyID와 SecretAccessKey 를 이용하여 접근해보기. 

```go
AWS_ACCESS_KEY_ID=AKIARQxxxxxxxxX2HHWFT AWS_SECRET_ACCESS_KEY=rVV1/wm3xxxxxD1hlxxxxG9PEOa/XU0fb0Ri aws s3 ls

An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied
```

보는 바와 같이 우리가 생성한 사용자로는 S3에 접근할 수 없다.

즉, 사용자만 생성했고, 권한을 직접 부여하지 않았기 때문에 이 사용자는 STS(Assume Role) 를 부여 받아서 리소스에 접근할 수 밖에 없다. 

## AWS CLI 를 이용하여 Assuem Role 얻어내기

이제는 Assume Role 를 이용하여 Assume Role 얻어내 보자. 

```go
CREDS=`AWS_ACCESS_KEY_ID=AKIARQE......HHWFT AWS_SECRET_ACCESS_KEY=rVV1/...ZlG9PEO...fb0Ri \
   aws sts assume-role --role-arn "arn:aws:iam::10XXXXXXX946:role/test-assume-role" --role-session-name "AssumeS3"`
```

이전에 받아둔 AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY 를 우선 변수에 할당하고, aws sts assume-role 커맨드를 실행한다. 

### 형식

```go
aws sts assume-role --role-arn "<Role 로 생성한 Role의 리소스 이름>" --role-session-name "원하는 이름" 
```

위 명령을 수행하고 나면 CREDS 라는 환경 변수에 임시로 받은 값이 존재하게 된다. 

```
echo $CREDS

{
    "Credentials": {
        "AccessKeyId": "ASIARQEQRWMJJ77EQH6Q",
        "SecretAccessKey": "wnEj/wGmK7mtGKWj0aJN8EafVqlXooLvaGyRmcGk",
        "SessionToken": "IQoJb3JpZ2luX2VjECEaDmFwLW5vcnRoZWFzdC0yIkcwRQIgCKEQ0dgp26SqD869UyX7m92LVMNHI9IqjDSRuj1SsugCIQCAj2/9UQiEmQDV3YGrVYIbqvhQ8HgBpD0627SyiMU8iCqVAghqEAIaDDEwMzM4MjM2NDk0NiIMEc5tOvDzvcqiskKLKvIBJeDkQQe97d5uVOnRYymNXcrbCu6G1myUBHeTPW8Td1xdEJjC8fgylXTPzrjRx66B9yKEfRbXjCc2AY7tvna+gl2aS0jknzZXTdl+H14raIOLjVUxlSZsgqdJiz8yV5LtxRSZql14YQF82qteYdjD4T/wyC40nD0tZmRUI1bXyQAeO0hEeRh4D1ZCFwruX8NBTxh962j7XjaH0uHL3mPV7Q0+Yj+GJbCy1QqYIck9oedlpedeY7P3khlUntTR6c0uPqzmzN5F+mmNpavuAM8GpTNuyUNyEBj3Duyq6yXwXLjgfL9rc7HXW972Lfbz1Hus2BYwpemJgwY6nQE99wG9OTMuuY+cOiIBxXi2SJxLPJ+Ea5IxPbGpiRbaFDx4oVrkqCkvGK+BAvZ2GEDXuDFnD5HEC/2SJmyQ/xAzaRbFaEGfaBhUKkCpmUkgys4ALFBLZg0yFfbf32csonEuRemK6prLsWpjHPH3E+UQ/2bilbBl14m/EAmEDhbSlHMo1qVsJN8iZ9/xfydrDJ2Xv7mE/P7uqnd2yauS",
        "Expiration": "2021-03-30T01:45:25Z"
    },
    "AssumedRoleUser": {
        "AssumedRoleId": "AROARXXXXXXXXDV:AssumeS3",
        "Arn": "arn:aws:sts::103XXXXXX946:assumed-role/test-assume-role/AssumeS3"
    }
}
```

- Credentials.AccessKeyId: 임시 엑세스 키 아이디이다. 
- Credentials.SecretAccessKey: 임시 시크릿 키 아이디이다. 
- Credentials.SessionToken: 임시 토큰
- Credentials.Expiration: 마료 시간 (만료 시간역시 assume-role 요청시 별도로 지정이 가능하다. )

- AssumeRoleUser: 롤을 할당 받을 정보를 알 수 있다. 

## Credential 환경 변수 등록하기. 

이제는 리소스에 접근할 수 있도록 환경 변수를 등록해보자. 

jq 를 이용하여 위에서 보여준 내용과 같이 Credentials 의 값을 추출하여 환경 변수로 각각 등록 하였다. 

```go
export AWS_ACCESS_KEY_ID=`echo $CREDS | jq -r '.Credentials.AccessKeyId'`
export AWS_SECRET_ACCESS_KEY=`echo $CREDS | jq -r '.Credentials.SecretAccessKey'`
export AWS_SESSION_TOKEN=`echo $CREDS | jq -r '.Credentials.SessionToken'`
```

### jq 명령

jq 명령중 ```-r``` 옵션은 raw data 를 획득하는 것이다. json 경우 double quota 가 있어서 이를 제거하기 위해서 이 옵션을 지정했다. 

자세한 사용법은 다음을 참조하자. 

- https://stedolan.github.io/jq/manual/#Basicfilters

### 요청 크레덴셜 정보 확인하기. 

위와 같이 환경 변수를 셋팅 했으니 실제 aws cli 를 이용할때 어떠한 계정 정보를 이용하는지 알아보자. 

```go
aws sts get-caller-identity

{
    "UserId": "AROARXXXXXXXXPHVS7CDV:AssumeS3",
    "Account": "103xxxxx946",
    "Arn": "arn:aws:sts::103xxxxx946:assumed-role/test-assume-role/AssumeS3"
}
```

위와 같이 새로 획득한 사용자 계정과, assumeRole arn 을 확인할 수 있다. 

### S3에 접근하기. 

```go
aws s3 ls

2021-03-01 21:22:40 elasticbeanstalk-ap-northeast-2-103XXXX946
2021-03-17 10:18:14 schooldevops-sdk-test
2021-02-17 16:19:05 schooldevops-vault-01
```

위와 같이 S3에 접근할 수 있다는 것을 확인했다. 

### 환경변수 릴리즈 하기. 

이제는 환경 변수 값을 릴리즈 하자. 

```go
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
```

```go
aws s3 ls

Partial credentials found in env, missing: AWS_SECRET_ACCESS_KEY
```


