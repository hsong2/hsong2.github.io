---
layout: post
title: "Hacker School FTZ - Level 9"
description: "Do you know the buffer over flow?"
comments: true
categories: FTZ
---

<img data-action="zoom" src='{{ "assets/ftz/level9/1.jpg" | relative_url }}' alt='relative'>  

## 1) level9/apple 입력해 로그인  

<img data-action="zoom" src='{{ "assets/ftz/level9/2.png" | relative_url }}' alt='relative'>  

힌트에서 /usr/bin/bof 프로그램의 소스코드를 보여줬습니다.  
프로그램 이름처럼 bof 문제인가 봅니다.  

``` c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

main(){

  char buf2[10];
  char buf[10];

  printf("It can be overflow : ");
  fgets(buf,40,stdin);

  if ( strncmp(buf2, "go", 2) == 0 )
   {
        printf("Good Skill!\n");
        setreuid( 3010, 3010 );
        system("/bin/bash");
   }

}
```

buf2 배열에 "go" 문자열이 들어있으면 level10 권한으로 bash가 동작되나 봅니다.   
사용자에게 입력받은 데이터는 buf 배열에 저장되는데 어떻게 하면 buf2에 문자열을 입력할 수 있을까요?  

## 2) /usr/bin/bof 프로그램 실행

<img data-action="zoom" src='{{ "assets/ftz/level9/3.png" | relative_url }}' alt='relative'>  

/usr/bin/bof 프로그램을 실행하면 소스코드의 fgets 함수에 의해 입력을 대기하고 있습니다.  


<img data-action="zoom" src='{{ "assets/ftz/level9/4.jpg" | relative_url }}' alt='relative'>  

소스코드 내용을 참고하면 buf2 배열이 선언된 다음 buf 배열이 선언되기 때문에 스택에는 위 그림과 같이 공간이 할당되어 있을 것으로 예상됩니다.  
buf2 배열에 go 문자열이 들어있으면 level10 권한으로 bash가 실행되니 buf 배열을 아무 값으로 채우고 buf2 배열에 go 문자열을 채워넣어야 합니다.  
buf 배열에 표준입력(stdin)을 문자열 길이로 40까지 받기 때문에 buf 배열을 넘어서는 입력을 받을 가능성이 생깁니다.  
이 취약점을 이용해 buf 배열 아래 buf2 배열에 go 문자열을 채워넣겠습니다.  


<img data-action="zoom" src='{{ "assets/ftz/level9/5.png" | relative_url }}' alt='relative'>  

buf 배열에 a를 10개 채우고 buf2 배열에 go를 입력해도 level10 권한으로 bash가 실행되지 않습니다.  
아무래도 buf 배열과 buf2 배열 사이에 공간이 있는 것 같습니다.  

## 3) /usr/bin/bof 프로그램 디버깅  

/usr/bin/bof 프로그램을 디버깅할 수 있는 권한이 없기 때문에 hint 파일에서 제공한 소스코드를 복사해 동일한 프로그램을 하나 만들었습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level9/6.png" | relative_url }}' alt='relative'>  


새로 만든 파일을 gdb 디버거로 디버깅을 해보니 main 함수에 대한 어셈블리 코드가 나타납니다.  

<img data-action="zoom" src='{{ "assets/ftz/level9/7.png" | relative_url }}' alt='relative'>  


어셈블리 코드를 참고하여 buf 배열과 buf2 배열이 할당받은 공간을 그려보면 아래 그림과 같습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level9/8.jpg" | relative_url }}' alt='relative'>  


변수들이 스택에 할당받은 공간을 보면 buf 배열과 buf2 배열 사이에 작은 공간이 있습니다. bof 공격을 위해 이 공간까지 덮어써서 buf2에 go 문자열을 넣어야 합니다.  
입력받을 수 있는 문자열의 길이는 40이니 buf 배열(length: 10)을 채우고 buf 배열과 buf2 배열 사이의 공간(length: 6)을 채우고 buf2에 go 문자열(length: 3)을 입력해도 충분합니다.  

## 4) bof 공격하기

<img data-action="zoom" src='{{ "assets/ftz/level9/9.png" | relative_url }}' alt='relative'>  

buf 배열과 buf 배열과 buf2 배열 사이의 공간을 아무 값이나 채우고 buf2에 go 문자열을 입력하기 위해 위 그림과 같이 입력합니다.  

level10 권한으로 bash가 실행된 것을 볼 수 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level9/10.png" | relative_url }}' alt='relative'>  

my-pass 명령어를 실행해 level10 비밀번호를 획득합니다.  