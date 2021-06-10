---
layout: post
title: "Hacker School FTZ - Level 16"
description: "Do you know how to call the function?"
comments: true
categories: ftz
---

<img data-action="zoom" src='{{ "assets/ftz/level16/1.jpg" | relative_url }}' alt='relative'>  

## 1) level16/about to cause mass 입력해 로그인  

``` c
#include <stdio.h>

void shell() {
  setreuid(3097,3097);
  system("/bin/sh");
}

void printit() {
  printf("Hello there!\n");
}

main()
{ int crap;
  void (*call)()=printit;
  char buf[20];
  fgets(buf,48,stdin);
  call();
}
```

<img data-action="zoom" src='{{ "assets/ftz/level16/2.png" | relative_url }}' alt='relative'>  

힌트에서 제공하는 코드 내용을 보면 void (\*call)()=printit; 이 부분을 공략 지점 같습니다.  
printit 함수 주소 대신 shell 함수의 주소를 call 함수에 입력하면 main 함수에서 call 함수를 호출할 때 shell 함수를 호출할 것 입니다.  

## 2) attackme 프로그램 스택 구조 살펴보기  

<img data-action="zoom" src='{{ "assets/ftz/level16/3.png" | relative_url }}' alt='relative'>  

빨간 박스가 가리키는 부분이 void(\*call)()=printit; 코드에 대한 어셈블리어 코드 같습니다.  
맞는지 0x8048500 주소가 담고 있는 내용을 확인해보겠습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level16/4.png" | relative_url }}' alt='relative'>  

0x8048500 주소에서 printf 함수가 호출되는 것을 보면 printit 함수 내 코드에 대한 어셈블리어 코드 같습니다.  
만약 0x80485c0에 "Hello there!\n" 문자열이 저장되어 있다면 printit 함수라는 것을 확신할 수 있겠습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level16/5.png" | relative_url }}' alt='relative'>  

역시나 "Hello there!\n" 문자열이 저장되어 있습니다.  

그렇다면 shell 함수가 저장된 코드는 어느 주소에 저장되어 있을까요?  

## 3) shell 함수 코드가 저장된 주소는 어디에?  

함수가 선언된 순서를 보면 shell => printit => main 함수 순서로 작성되어 있습니다.  

앞서 main 함수와 printit 함수의 주소를 주의깊게 보셨다면 눈치챘을 수도 있습니다.   

<img data-action="zoom" src='{{ "assets/ftz/level16/6.png" | relative_url }}' alt='relative'>  

이처럼 함수가 선언된 순서대로 code 세그먼트에 저장되어 있습니다.  
따라서 printit 함수의 주소를 통해 shell 함수의 주소를 유추할 수 있었습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level16/7.png" | relative_url }}' alt='relative'>  

## 4) bof 일으키기!  

RET의 위치를 찾기위해 main 함수의 스택 구조를 분석했습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level16/8.png" | relative_url }}' alt='relative'>  

0x0804853c(\<main+36\>) 위치에 어셈블리어 코드를 보면  
$ebp-16에 저장된 값을 eax 레지스터에 add한 뒤 eax 레지스터에 저장된 값으로 call(호출)하는 코드 같습니다.  

즉, $ebp-16에 shell 함수의 주소를 넣으면 shell 함수가 호출될 것 입니다.  

buf 배열의 시작 주소는 $ebp-56이기 때문에,  
buf 20 byte + dummy 20 byte + shell 함수 주소 4 byte + dummy 12 byte + SFP 4 byte + RET 4 byte  
이러한 순서로 저장되어 있습니다. 

bof 공격을 위해 입력해야 할 문자열은 다음과 같습니다.  

``` bash
(python -c 'print "\x90"*40+"\xd0\x84\x04\x08"'; cat) | ./attackme
```

<img data-action="zoom" src='{{ "assets/ftz/level16/9.png" | relative_url }}' alt='relative'>  

shell 함수가 실행되어 level17 권한으로 shell을 실행시켜 level17의 비밀번호를 획득했습니다.  