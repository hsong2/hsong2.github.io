---
layout: post
title: "Hacker School FTZ - Level 9"
description: "Do you know the buffer over flow?"
comments: true
categories: ftz
---
<!--
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

-->