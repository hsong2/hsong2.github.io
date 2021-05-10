---
layout: post
title: "Hacker School FTZ - Level 6"
description: "Do you know the telnet vulnerability of BBS service?"
comments: true
categories: ftz
---
<img data-action="zoom" src='{{ "assets/ftz/level6/1.jpg" | relative_url }}' alt='relative'>  

## 1) level6/what the hell 입력해 로그인  

<img data-action="zoom" src='{{ "assets/ftz/level6/2.png" | relative_url }}' alt='relative'>  

로그인하자마자 바로 힌트 내용이 출력됩니다.  

<span style="background-color: #fff8b2">인포샵 bbs의 텔넷 접속 메뉴에서 많이 사용되던 해킹 방법이다.</span>  

## 2) 인포샵? bbs?  

한국통신(현, KT)은 전화망을 활용한 부가통신 서비스의 일종인 '하이텔'을 운영하여 PC 통신을 가능하게 했습니다.  

BBS라는건 초기 PC 통신에서 제공하던 전자 게시판 시스템입니다.  
초기 PC 통신은 BBS(Bulletin board service) 형태로 글을 올리고 메시지를 전송하고 파일을 공유했습니다.    
<a href="http://bit.ly/U2c8NY">하이텔 PC통신 - BBS 형식 </a>  

또한, 하이텔망에서는 여러 서비스를 제공했는데 '인포샵'이라는 명칭으로 인터넷 서비스를 제공했습니다.  
<a href="https://m.etnews.com/199609120030">한국PC통신, 인포샵 통해 인터넷서비스</a>  
기사 내용을 보면 www, telnet, ftp, IRC, 등 인터넷의 각종 서비스를 인포샵을 통해 제공했습니다.

## 3) BBS 텔넷 접속 취약점  

현재 쉽게 접할 수 있는 서비스가 아니기 때문에 구글에서 BBS 텔넷 접속 취약점을 검색해보니,  
텔넷 접속 메뉴에서 ctrl+c(kill 시그널)를 입력하면 쉘 프롬프트로 빠져나오게 된다고 합니다.  

즉, 해당 서버에서 shell 명령어를 사용할 수 있다는 의미입니다.  

## 4) kill 시그널 주기
  
<img data-action="zoom" src='{{ "assets/ftz/level6/3.png" | relative_url }}' alt='relative'>  

ctrl+c를 입력하니 쉘 프롬프트로 빠져나왔습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level6/4.png" | relative_url }}' alt='relative'>  

password라는 파일이 존재하여 cat 명령어로 파일 내부에 적힌 내용을 출력하니 level7의 비밀번호를 획득할 수 있습니다.  