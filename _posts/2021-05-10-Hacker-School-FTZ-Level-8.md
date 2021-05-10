---
layout: post
title: "Hacker School FTZ - Level 8"
description: "Do you know the ?"
comments: true
categories: ftz
---
<img data-action="zoom" src='{{ "assets/ftz/level8/1.jpg" | relative_url }}' alt='relative'>  

## 1) level8/break the world 입력해 로그인  

<img data-action="zoom" src='{{ "assets/ftz/level8/2.png" | relative_url }}' alt='relative'>  

level9의 shadow 파일이 서버 어딘가에 숨겨져 있다.  
그 파일에 대해 알려진 것은 용량이 "2700"이라는 것 뿐이다.  
라는 힌트를 제공해줍니다.  

> shadow 파일이란?  

사용자별 패스워드를 암호화하여 보호하기 위해 만들어진 파일입니다.  
/etc/passwd 파일에 저장되어 있던 비밀번호를 /etc/shadow 파일에 암호화하여 비밀번호만 별도로 저장합니다.  
또한, shadow 파일에는 패스워드 만료기간, 갱신기간, 최소기간 등 비밀번호 관리를 위한 정책 또한 저장되어 있습니다.  
/etc/passwd 파일과 /etc/shadow 파일을 read 하기 위해선 root 권한이 필요합니다.  


## 2) 용량이 2700인 파일 찾기  

``` bash
find / -size 2700c 2>/dev/null
```

<img data-action="zoom" src='{{ "assets/ftz/level8/3.png" | relative_url }}' alt='relative'>  