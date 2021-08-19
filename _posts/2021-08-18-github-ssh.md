---
layout: post
title:  "GitHub SSH 접속 설정하기"
date:   2021-07-29 11:45:49 +0900
categories: GitHub
tags: [GitHub, SSH, Push]
toc: true
---

# GitHub SSH 접속 설정하기. 

- GitHub에서는 다음 3가지 방법으로 Git접근을 수행할 수 있도록 하고 있다. 
  - GitHub는 https 로 접속
  - ssh 로 인증서를 발급하여 접근하는 방법
  - git cli 를 이용하여 접근하는 방법

- 이 중 ssh 로 접근하는 방법에 대해서 알아보자. 
- ssh 접근을 설정해두면, username과 password를 입력하지 않고 git과 인터렉션 할 수 있다. 

## SSH key 생성 여부 확인하기. 

```go
ls -al ~/.ssh
```

- 위 명령으로 이미 생성된 ssh 키가 존재하는지 확인할 수 있다. 
- 디렉토리에 아무것도 존재하지 않거나 혹은 새로운 ssh 키를 생성하고자 한다면 다음 과정을 따라가자. 

## SSH key 생성하기.

```go
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- 위 명령과 같이 github에 등록된 자신의 메일 계정을 추가한 ssh key 를 생성한다. 
- ssh key 는 ssh-keygen 을 이용한다. 
- -t 옵션을 통해서 rsa, dsa, ECDSA, EdDSA 등 옵션이 올수 있다. 여기서는 ed25519 키 암호화 방식을 이용한다. 

- ed25519 는 최신의 암호화 방식이므로 동작하지 않는다면 rsa를 다음과 같이 이용할 수 있다 

```go
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### 실행해보기

```go
ssh-keygen -t ed25519 -C "schooldevops@gmail.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/kido/.ssh/id_ed25519):  <-- 그냥엔터
Enter passphrase (empty for no passphrase):   <-- 그냥엔터
Enter same passphrase again:   <-- 그냥엔터
Your identification has been saved in /Users/kido/.ssh/id_ed25519.
Your public key has been saved in /Users/kido/.ssh/id_ed25519.pub.
The key fingerprint is:
SHA256:MMzpFz1s1Xdv/6ICpuSm05ueYpJ34 schooldevops@gmail.com
The key's randomart image is:
+--[ED25519 256]--+
|         ..o     |
|     o .o+. o    |
|      *o. =.     |
|.  o o+o.o .     |
...
|ooooo..   .o.    |
+----[SHA256]-----+
```

- 위 결과처럼 ~/.ssh/id_ed25519 와 ~/.ssh/id_ed25519.pub 파일이 생성 되었다. 

## ssh-agent 에 키 등록하기. 

- ssh-agent에 등록하기 위해 키를 생성했었다. 
- ssh-add 명령으로 키를 에이전트에 등록할 수 있다. 

### ssh-agent 활성화 하기. 

```go
eval "$(ssh-agent -s)"

Agent pid 62393
```

- 위와 같이 ssa-agent 가 pid 62393으로 시작되었음을 알 수 있다. 

### config 파일 설정하기

- ~/.ssh/config 파일에 다음 내용을 추가한다. 

```go
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

### ssh-agent 에 ssh 개인키 추가 

```go
ssh-add -K ~/.ssh/id_ed25519

Identity added: /Users/kido/.ssh/id_ed25519 (schooldevops@gmail.com)
```

- 위와 같이 정상적으로 등록 되었음을 확인 할 수 있다. 

## Repository 에 ssh key 추가하기. 

### ssh_key 복사 

```go
pbcopy < ~/.ssh/id_ed25519.pub
```

- id_ed25519.pub 내용을 복사한다. 

### github 에 ssh 키 등록하기. 

#### 프로젝트 생성하기. 

- test_project 로 아래와 같이 리포지토리를 생성하자. 
- 
![github01](https://user-images.githubusercontent.com/66154381/129998506-4ec3d06f-f4f4-4b5c-a22e-23e2d19b2906.png)

#### 개인정보 > Setting 들어가기 

![github02](https://user-images.githubusercontent.com/66154381/129998502-62f3fdc5-a5e1-4ff9-b470-c061c9c378c6.png)

#### SSH and GCP 키 메뉴 클릭 

![github03](https://user-images.githubusercontent.com/66154381/129998499-d5776925-600c-4938-a0cc-913960d0eccc.png)

- New SSH Key 버튼을 클릭하여 다음과 같이 복사한 키를 등록하자. 
  
#### 키 등록하기 

![github04](https://user-images.githubusercontent.com/66154381/129998496-93d00982-e95f-4fea-bed3-bd69ede3f453.png)

#### 키 등록 결과 

![github05](https://user-images.githubusercontent.com/66154381/129998492-2c8ab07d-7522-4fa0-9b3b-2cecf39a412b.png)

- 키를 등록하면 암호 입력창이 나오는데 그때 자신의 git 암호를 입력하자. 
  
## 키 등록 테스트하기. 

```go
ssh -T git@github.com

> The authenticity of host 'github.com (IP ADDRESS)' can't be established.
> RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
> Are you sure you want to continue connecting (yes/no)? yes <-- 입력하고 엔터 

Hi schooldevops! You've successfully authenticated, but GitHub does not provide shell access.
```

- 보는바와 같이 정상적으로 github에 ssh 키로 접속 테스트가 되었음을 알 수 있다. 

## ssh 로 푸시하기. 

![github06](https://user-images.githubusercontent.com/66154381/129999047-033fbeb0-aa7a-44c7-b396-96e2ec1fe154.png)

- HTTPS | SSH 중 "SSH" 버튼을 클릭하면 사용방법이 나온다. 
- 위와 같은 커맨드로 따라하면 ssh 를 통해서 이용이 가능하다. 

## WRAP UP

- SSH 를 이용하면, username/password를 이용하지 않아도 되기 때문에 민감한 비번을 노출하는 행위를 줄일 수 있다. 
- 또한 CI/CD 서버에서 미리 등록해두면, 편리하게 GIT에 접근할 수 있어 파이프라인 구성시 한번의 설정으로 쉽게 작업이 가능하다. 