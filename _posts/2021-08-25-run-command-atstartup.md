---
layout: post
title:  "AWS EC2 시스템 시작시 Command 실행하기"
date:   2021-08-25 11:45:49 +0900
categories: AWS
tags: [AWS, EC2, Bootup, Crontab, rc.local, bash]
toc: true
---

# Amazon Linux2 시스템 부트업시 커맨드 실행하기

- EC2 인스턴스를 사용하다 보면, 인스턴스 재기동을 수행해야하는 경우가 있다. 
- 항상 시스템과 함께 실행되어야하는 프로그램의 경우에는 시스템이 부트업될때 자동으로 수행되도록 설정이 필요하다. 
- 이러한 설정을 수행하는 방법에 대해서 알아볼 것이다. 

## 커맨스 실행방법 

- CronTab을 이용하는 방법
- rc.local 파일에 스크립트를 등록하는 방법
- Linux 쉘을 로그인/호그아웃시 실행하는 방법
- UserData 를 이용하는 방법 

### CronTab을 이용하는 방법

- Cron 은 주기적으로 프로그램이 수행되도록 해주는 리눅스 스케줄링 프로그램중에 하나이다. 
- 다음과 같이 실행하고자 하는 작업을 등록하자. 

```go
crontab -e

@reboot sdk use java 8.302.08.1-amzn
@reboot /home/ec2-user/nexus-3.33.1-01/bin/nexus start
```

- 위와 같이 /myscript-location 디렉토리에 script.sh 파일을 부트시 실행해 준다. 

- 오류가 있다면 다음과 같이 확인할 수 있다. 

```go
sudo cat /var/log/cron

,,, 로그 
```
