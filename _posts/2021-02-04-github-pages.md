---
layout: post
title:  "GitHub Pages with Jekyll"
date:   2021-02-04 16:45:49 +0900
categories: jekyll update
tags: [jekylly]
toc: true
---
# GitHub Pages 

GitHub 으로 포스팅 할 수 있는 블로그 사이트를 GitHub Pages 라고 명명 되고 있네요. 

마크다운으로 페이지를 생성하면, 정적 홈페이지가 만들어진다. 

이때 블로그 페이지를 만들어 주는 것이 Jekyll 이라고 하는 프로그램으로 정적 페이지를 만들어 주며, GitHub Pages 에서 확인할 수 있다. 

## 사전 필요사항 

### Ruby 설치하기. 

[Ruby install](https://www.ruby-lang.org/en/documentation/installation/) 을 참조하자. 

```
brew install ruby
```

### Jekyll 설치하기. 

[Jekyll](https://jekyllrb.com/docs/) 을 참조해서 설치하자.

#### 설치하기. 

```
gem install jekyll bundler
```

#### Jekyll 블로그 생성하기. 

```
jekyll new myblog
```

#### 번들링 대상을 테스트하기 

```
cd myblog 

bundle install 

bundle exec jekyll serve
```

## 새 블로그 포스팅하기

새 블로그 포스팅을 해보자. 

jekyll new myblog 로 만들었다면 디렉토리 하위에 _posts 라는 디렉토리가 생성이 된다. 

### 파일 이름 규칙 

파일 형식은 항상 다음과 같이 작성해야한다. 

- YYYY-MM-DD-Namd-of-post.md 

예)
```
2021-02-04-welcome-to-jekyll.markdown or 2021-02-04-welcome-to-jekyll.md
```

### 파일 헤더 달기 

파일 헤더는 다음과 같이 작성할 수 있다. 

```
layout: post
title: "POST TITLE"
date: YYYY-MM-DD hh:mm:ss -0000
categories: CATEGORY-1 CATEGORY-2
```

헤더 이후에 컨텐츠는 마크다운을 활용하여 작성해주면 된다. 

### 블로그 설정 변경하기. 

_config.yml 파일의 내용을 변경하면 블로그의 제목, 푸터등의 내용을 변경할 수 있다. 

```
title: Welcome School Dev Ops Stubs.
email: baekido@gmail.com
description: >- # this means to ignore newlines until "baseurl:"
  Welcome This is School Dev Ops's Stubs. 
baseurl: "" # the subpath of your site, e.g. /blog
url: "" # the base hostname & protocol for your site, e.g. http://example.com
github_username:  schooldevops

# Build settings
theme: minima
plugins:
  - jekyll-feed
```

내용의 일부를 바꿔보자. 

## GitHub Repository 연동

생성한 사이트를 블로그와 연동하기 위해서 [Creating a GitHub Pages site with Jekyll](https://docs.github.com/en/github/working-with-github-pages/creating-a-github-pages-site-with-jekyll) 을 참조하여 리포지토리를 만들자. 