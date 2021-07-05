---
layout: post
title: "The Lord of the BOF - Level 13"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level13/1.jpg" | relative_url }}' alt='relative'>  

## 1) darkknight/new attacker 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level13/2.png" | relative_url }}' alt='relative'>  

이번에는 '\xbf'가 argv[1]에 포함되어 있으면 안 되는 문제입니다.  

힌트로는 RTL을 줬습니다. RTL이 무엇일까요?  

## 2) RTL이란?  

RTL은 Return to Library BOF로 return address를 함수의 시작 주소로 변경하는 것을 의미합니다.  
BOF 보호기법인 ASLR로 libc 라이브러리가 메모리에 위치한 주소는 프로그램이 실행할 때마다 매번 바뀌지만 이번 문제에서는 ASLR이 적용되어 있지 않습니다.  
libc 라이브러리 내 존재하는 함수의 위치는 메모리 상의 동일한 주소에 있습니다.  
libc 라이브러리 주소는 base address에 offset을 더해주면 되므로, 함수의 offset을 알면 해당 함수가 존재하는 메모리 주소를 알 수 있습니다.  

우선 프로그램이 실행하는 데 사용하는 공유 라이브러리를 확인하는 명령어 ldd를 사용하여 bugbear 프로그램이 사용하는 공유 라이브러리 목록을 확인하겠습니다.  

<img data-action="zoom" src='{{ "assets/lob/level13/3.png" | relative_url }}' alt='relative'>  

libc.so.6 라이브러리는 0x40018000에 위치하고 있습니다.  

그렇다면 libc.so.6 라이브러리 내에 system 함수를 사용하여 system("/bin/sh") 명령을 실행하면 문제를 풀 수 있을 것으로 예상됩니다.  

## 3) libc.so.6 라이브러리 내 system 함수의 위치(offset) 확인하기  

gdb에서 p는 print의 약자로 변수 값을 출력하거나 함수의 주소를 볼 때 사용합니다.  
r(run) 명령어를 사용해서 프로그램이 메모리 상에 올라갔기 때문에 동적 공유메모리인 libc.so.6에 정의되어 있는 system 함수를 참조할 수 있습니다.  

<img data-action="zoom" src='{{ "assets/lob/level13/4.png" | relative_url }}' alt='relative'>  

권한이 없어 bugbear 프로그램을 gdb로 실행시키지 못해 bugbear.c 소스파일을 디버깅하여 test 프로그램을 하나 만들었습니다.  
test 프로그램을 gdb로 실행시켜 system 함수의 위치를 확인해보니 0x40058ae0 주소에 위치하는 것을 확인하였습니다.  

<p><a id="binsh"></a></p>
## 4) system 함수에 인자로 넣어줄 "/bin/sh" 문자열 찾기  

"/bin/sh" 문자열을 찾는 방법은 3가지가 있습니다.  

### (1) /lib/libc.so.6 라이브러리 디버깅  

<img data-action="zoom" src='{{ "assets/lob/level13/5.png" | relative_url }}' alt='relative'>  

``` bash
shell strings -tx /lib/libc.so.6 | grep "/bin/sh"
```

libc.so.6 라이브러리를 gdb로 디버깅하여 system 함수의 offset을 확인합니다.  
system 함수의 offset은 0x40ae0 입니다.  

그다음, libc.so.6 라이브러리 내에 "/bin/sh" 문자열의 위치를 hex 값으로 출력합니다.  
"/bin/sh" 문자열은 libc.so.6 라이브러리 내 여러 곳에 위치하고 있습니다. 그 중 임의로 하나를 골라 libc.so.6 라이브러리에서 "/bin/sh" 문자열의 offset을 구합니다.  
"/bin/sh" 문자열의 주소에서 system 함수의 offset 값을 빼서 "/bin/sh" 문자열의 offset을 구합니다.  
"/bin/sh" 문자열의 offset은 0xa3519 입니다.  

프로그램을 gdb로 실행하여 메모리 상에서 system 함수가 위치하는 주소를 확인합니다.  
0x40058ae0에 system 함수가 위치하고 있습니다. 여기에 아까 구해놨던 "/bin/sh" 문자열의 offset 값을 더해주면 메모리 상에 "/bin/sh" 문자열이 위치한 주소를 구할 수 있습니다.  

"/bin/sh" 문자열은 0x400fbff9에 위치하고 있습니다.  

### (2) system 함수에서 "/bin/sh" 문자열 찾는 프로그램 작성하기  

<img data-action="zoom" src='{{ "assets/lob/level13/6.png" | relative_url }}' alt='relative'>  

``` c
// find_sh.c
#include <stdio.h>
#include <stdlib.h>
int main() {
        long system = 0x40058ae0;
        while (memcmp((void*)system, "/bin/sh", 8)) {
                system++;
        }
        printf("%p\n", system);
        return 0;
}

```

### (3) python 활용하기  

<img data-action="zoom" src='{{ "assets/lob/level13/7.png" | relative_url }}' alt='relative'>  

## 5) 스택에 system 함수를 호출하기 위한 값 채우기  

함수를 호출하기 위해 system 함수에 넣을 인자("/bin/sh"), 함수를 호출한 뒤 다시 돌아올 메모리 주소를 스택에 입력해야 합니다.  
그리고 마지막으로 함수 주소로 가기 위해 함수를 호출하는 인터럽트 과정이 필요합니다. main 함수가 종료하기 전 leave, ret 명령을 수행하기 때문에 ret에 system 함수 주소를 넣어주면 됩니다.  

그러면 buffer 배열의 길이는 40 byte이므로 44 byte는 NOP으로 채워주고 RET 위치에 system 함수의 주소를 입력하면 됩니다.  
함수가 호출되기 전 스택에 복귀 주소와 함수 인자가 스택에 쌓여있어야 하기 때문에 RET 뒤로 복귀 주소와 함수 인자를 입력하면 됩니다.  

system 함수의 위치:		0x40058ae0 (\xe0\x8a\x05\x40)  
"/bin/sh" 문자열의 위치:	0x400fbff9 (\xf9\xbf\x0f\x40)  

그러면 다음과 같은 공격 페이로드가 완성됩니다.  

``` bash
./bugbear `python -c 'print "\x90"*44+"\xe0\x8a\x05\x40"+"AAAA"+"\xf9\xbf\x0f\x40"'`
```

## 6) 공격 페이로드 입력하여 BOF 일으키기  

<img data-action="zoom" src='{{ "assets/lob/level13/8.png" | relative_url }}' alt='relative'>  
