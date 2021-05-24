---
layout: post
title: "Hacker School FTZ - Level 11"
description: "Do you know the BOF?"
comments: true
categories: ftz
---

<img data-action="zoom" src='{{ "assets/ftz/level11/1.jpg" | relative_url }}' alt='relative'>  

## 1) level11/what!@#$? 입력해 로그인  

<img data-action="zoom" src='{{ "assets/ftz/level11/2.png" | relative_url }}' alt='relative'>  

힌트를 살펴보니 어떤 프로그램의 소스코드 같습니다.    

``` c
// what.c
#include <stdio.h>
#include <stdlib.h>

int main( int argc, char *argv[] )
{
        char str[256];

        setreuid( 3092, 3092 );
        strcpy( str, argv[1] );
        printf( str );
}
```

1) uid가 3092로 설정됩니다.  

<img data-action="zoom" src='{{ "assets/ftz/level11/3.png" | relative_url }}' alt='relative'>  

level11의 uid가 3091이니 3092는 level12의 uid로 예상됩니다.  

2) 프로그램 실행 시 첫번째 인자로 입력된 문자열이 str 배열에 복사됩니다.  

3) str 배열에 저장된 문자열이 출력됩니다.  

소스코드를 분석해보면 1번부터 3번 순서대로 프로그램이 진행되는 것을 확인할 수 있습니다.  

한 번 소스코드를 복사해 해당 프로그램을 만들고 실행시켜보겠습니다.  

## 2) hint에 있는 소스코드를 프로그램으로 만들기  

<img data-action="zoom" src='{{ "assets/ftz/level11/4.png" | relative_url }}' alt='relative'>  

프로그램을 인자 없이 실행해보니 segmentation fault가 발생합니다.  
그래서 인자를 하나 입력하니 입력한 문자열 그대로 출력하고 프로그램은 종료합니다.  

shell을 실행시키는 쉘코드를 str 배열에 입력하고 RET에 str 배열의 주소를 입력하면 level12 권한으로 shell을 실행시킬 수 있을 것 같습니다.  

## 3) 쉘코드 만들기  

쉘코드를 만드는 방법에 대해 궁금하다면 <a href="https://hsong2.github.io/ftz/2021/05/24/Hacker-School-FTZ-Shellcode.html">쉘코드 만드는 방법</a>을 확인하세요.  

<span style="background-color: #fff8b2">25바이트</span> 쉘코드를 만들었습니다.  

``` bash
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80
```

## 4) str 배열의 위치 찾기  

<img data-action="zoom" src='{{ "assets/ftz/level11/5.png" | relative_url }}' alt='relative'>  

어셈블리 코드를 보면 ebp-264 위치에 str 배열의 공간이 할당되어 있는 것으로 보입니다.  
main 함수에 breakpoint를 설정하고 실행(r)합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level11/6.png" | relative_url }}' alt='relative'>  

strcpy 함수를 실행시키기 위해 인자를 2개 입력 받습니다.  

``` c
strcpy( str, argv[1] );
```

스택에는 함수 인자의 뒷 부분부터 push 합니다. 그래서 argv[1]부터 push 하는 어셈블리 코드를 확인할 수 있습니다.  

<main+43>부터 <main+49>를 보면 argv[1]의 주소를 eax 레지스터에 입력하고 eax가 담고있는 주소를 스택에 push 합니다.  

그 다음 str 배열의 주소인 ebp-264 값을 eax 레지스터에 입력하고 eax가 담고있는 주소를 스택에 push 합니다.  


<img data-action="zoom" src='{{ "assets/ftz/level11/7.png" | relative_url }}' alt='relative'>  

<main+58>인 0x80483ce까지 실행한 뒤 스택에 쌓인 모습을 보면 위의 그림과 같습니다.  
0xbfffdc40은 str 배열의 주소이고 0xbffffb27은 argv[1]의 주소입니다.  

실제로 argv[1](0xbffffb27)에 저장된 값을 보면 처음 실행시 인자로 입력한 내용이 저장되어 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level11/8.png" | relative_url }}' alt='relative'>  

위의 그림에서 보여지는 스택을 도식화하면 아래 그림과 같습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level11/9.jpg" | relative_url }}' alt='relative'>  


## 5) str 배열에 쉘코드 입력한 뒤 리턴 주소 변경하기  

스택에 str 배열이 할당받은 주소를 알았다면 bof 취약점을 이용해 RET 주소를 str 배열의 주소로 바꿔보겠습니다.  

str 배열에서 SFP까지 총 264바이트만큼 떨어져 있습니다. 264바이트를 NULL값(\x90)과 쉘코드(25바이트)로 채우겠습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level11/10.png" | relative_url }}' alt='relative'>  

그림과 같이 200바이트 NULL값 + 25바이트 쉘코드 + 43바이트 NULL값 + RET주소(str 배열 주소)를 프로그램 실행시 인자로 입력합니다.  

4번에서 예상했던 str 배열 주소 값이 이번 실행에서는 다른 위치에 할당됐는지 Segmentation fault가 발생합니다.  

RET 주소를 바꿔서 여러 번 시도해보니 shell을 획득할 수 있습니다.  

## 6) attackme에 bof 시도  

위와 같은 방법으로 attackme 프로그램에서도 bof를 시도해보겠습니다.  

str 배열의 주소를 특정하는 것이 어려워 argv[1]의 주소를 RET에 적은 뒤 bof를 시도했습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level11/11.png" | relative_url }}' alt='relative'>  


<img data-action="zoom" src='{{ "assets/ftz/level11/12.png" | relative_url }}' alt='relative'>  

위 그림과 같이 argv[1] 주소와 인접한 주소를 RET에 대입하니 shell 코드가 실행되며 level12의 비밀번호를 획득할 수 있습니다.  