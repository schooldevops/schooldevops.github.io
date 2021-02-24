---
layout: post
title:  "Mac 에 Open JDK 설치하기. "
date:   2021-02-24 09:45:49 +0900
categories: Java
tags: [Java, OpenJDK]
toc: true
---

# 들어가기 

오랜만에 mac 을 다시 설치하고, jdk 를 설치하려고 하니 기억이 안난다. 

Oracle 에서 다운받아서 설치 하지 않고, OpenJDK 를 설치하기 위해서 brew 를 이용하려고 한다. 

 
## brew 설치하기

mac 패키지 설치 툴은 brew 가 가장 유명한것 같다. https://brew.sh/ 에서 brew 를 설치해주자. 

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

한참 설치하고 나서 brew 를 업데이트 하자. 
 
## brew update 하기

```
brew update
Already up-to-date.
``` 

## Open JDK 설치하기. 

공식적으로 아직 brew 가 OpenJDK 를 직접 지원하지 않아서, 다음 사이트에서 지원하는 AdoptOpenJDK 를 이용한다. 

```
https://github.com/AdoptOpenJDK/homebrew-openjdk
```
 

### Cask 위치 등록하기

```
brew tap AdoptOpenJDK/openJdk

==> Tapping adoptopenjdk/openjdk
Cloning into '/usr/local/Homebrew/Library/Taps/adoptopenjdk/homebrew-openjdk'...
remote: Enumerating objects: 73, done.
remote: Counting objects: 100% (73/73), done.
remote: Compressing objects: 100% (66/66), done.
remote: Total 1570 (delta 45), reused 9 (delta 6), pack-reused 1497
Receiving objects: 100% (1570/1570), 281.08 KiB | 279.00 KiB/s, done.
Resolving deltas: 100% (1103/1103), done.
Tapped 37 casks (72 files, 413.6KB).
``` 

### 지원하는 jdk 살펴보기 

```
brew search jdk

==> Formulae
openjdk    
                                                                                openjdk@11
==> Casks
adoptopenjdk-jre                    adoptopenjdk11-openj9-jre           adoptopenjdk13                      adoptopenjdk14-openj9-jre           adoptopenjdk9
adoptopenjdk-openj9                 adoptopenjdk11-openj9-jre-large     adoptopenjdk13-jre                  adoptopenjdk14-openj9-jre-large     homebrew/cask/jdk-mission-control
adoptopenjdk-openj9-jre             adoptopenjdk11-openj9-large         adoptopenjdk13-openj9               adoptopenjdk14-openj9-large         homebrew/cask/oracle-jdk
adoptopenjdk-openj9-jre-large       adoptopenjdk12                      adoptopenjdk13-openj9-jre           adoptopenjdk8                       homebrew/cask/oracle-jdk-javadoc
adoptopenjdk-openj9-large           adoptopenjdk12-jre                  adoptopenjdk13-openj9-jre-large     adoptopenjdk8-jre                   homebrew/cask/sapmachine-jdk
adoptopenjdk10                      adoptopenjdk12-openj9               adoptopenjdk13-openj9-large         adoptopenjdk8-openj9
adoptopenjdk11                      adoptopenjdk12-openj9-jre           adoptopenjdk14                      adoptopenjdk8-openj9-jre
adoptopenjdk11-jre                  adoptopenjdk12-openj9-jre-large     adoptopenjdk14-jre                  adoptopenjdk8-openj9-jre-large
adoptopenjdk11-openj9               adoptopenjdk12-openj9-large         adoptopenjdk14-openj9               adoptopenjdk8-openj9-large
``` 

### 원하는 패키지 설치하기. 

```
brew cask install adoptopenjdk8

==> Downloading https://github.com/AdoptOpenJDK/openjdk8-binaries/releases/download/jdk8u265-b01/OpenJDK8U-jdk_x64_mac_hotspot_8u265b01.pkg
Already downloaded: /Users/baegido/Library/Caches/Homebrew/downloads/33a932bba3f6b57c6a06341ab08226a51233bca1b5832aed7461b78b013cb43d--OpenJDK8U-jdk_x64_mac_hotspot_8u265b01.pkg
==> Verifying SHA-256 checksum for Cask 'adoptopenjdk8'.
==> Installing Cask adoptopenjdk8
==> Running installer for adoptopenjdk8; your password may be necessary.
==> Package installers may write to any location; options such as --appdir are ignored.
Password:
installer: Package name is AdoptOpenJDK
installer: Installing at base path /
installer: The install was successful.
package-id: net.adoptopenjdk.8.jdk
version: 1.8.0_265-b01
volume: /
location: Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk
install-time: 1596155469
🍺  adoptopenjdk8 was successfully installed!
``` 

### 설치확인하기

```
java -version

openjdk version "1.8.0_265"
OpenJDK Runtime Environment (AdoptOpenJDK)(build 1.8.0_265-b01)
OpenJDK 64-Bit Server VM (AdoptOpenJDK)(build 25.265-b01, mixed mode)
``` 

### JDK 설치 경로 확인하기. 

```
/usr/libexec/java_home -V

Matching Java Virtual Machines (1):
    1.8.0_265, x86_64:	"AdoptOpenJDK 8"	/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home

/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home
``` 

Adopopenjdk 를 이용하면 원하는 버젼을 쉽게 설치할 수 있다. 