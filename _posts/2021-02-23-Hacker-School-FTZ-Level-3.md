---
layout: post
title: "Hacker School FTZ - Level 3"
description: "Can you run multiple commands from the shell at once, on a single line?"
comments: true
categories: ftz
---
<img data-action="zoom" src='{{ "assets/ftz/level3/1.png" | relative_url }}' alt='relative'>  

## ﻿1) level3/can you fly? 입력해 로그인  
<img data-action="zoom" src='{{ "assets/ftz/level3/2.png" | relative_url }}' alt='relative'>  
앞선 문제와 동일하게 level3에서도 hint 파일이 있습니다.
cat 명령어를 사용해 hint 파일의 내용을 확인해보니 소스코드가 적혀있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level3/3.png" | relative_url }}' alt='relative'>

이번 문제에선 추가 힌트도 줍니다.  
<span style="color:blue">1) 동시에 여러 명령어를 사용하려면?</span>  
<span style="color:blue">2) 문자열 형태로 명령어를 전달하려면?</span>  

level4의 비밀번호를 얻기 위해선 추가 힌트로 준 질문의 답을 찾아야 하는 것 같습니다.  

## 2) autodig 파일 확인하기  
이번 문제에서도 다음 레벨로 넘어가기 위한 비밀번호를 찾아야 합니다.  
소유자 권한이 level4이고 SetUID가 설정된 파일로 비밀번호를 구할 수 있을 것으로 예상됩니다.  
그리고 그 파일 이름이 autodig 일 확률이 높습니다.

아래 명령어를 입력해 소유주가 'level4'이면서  이름이 autodig인 파일을 검색하니
/bin 디렉터리 아래 autodig 프로그램이 존재합니다.  

``` bash
find / -user level4 -name autodig 2>/dev/null
```

<img data-action="zoom" src='{{ "assets/ftz/level3/4.png" | relative_url }}' alt='relative'>   

<hr>

autodig 프로그램을 실행해보니 아래 그림과 같은 내용이 출력됐습니다.

<img data-action="zoom" src='{{ "assets/ftz/level3/5.png" | relative_url }}' alt='relative'>   

autodig 소스코드를 확인해보니 프로그램 실행 인자의 개수가 2개가 아니면 프로그램이 종료하는 것을 알 수 있습니다.  

``` c
// autodig 프로그램 소스코드 일부
int main(int argc, char **argv){

    char cmd[100];

    if( argc!=2 ){ // 실행 인자의 개수가 2개인지 확인
        printf( "Auto Digger Version 0.9\n" );
        printf( "Usage : %s host\n", argv[0] );
        exit(0);
    }
...
```

실행시키고자 하는 프로그램을 동작시킬 때 프로그램 이름을 적습니다.  
만약 프로그램이 시작할 때 옵션을 추가하고 싶다면 프로그램 이름 뒤에 옵션을 추가로 작성하면 됩니다.  
여러 개 옵션을 주고 싶다면 띄어쓰기(' ')로 구분해 주면 됩니다.  

프로그램 실행 시 지정해 준 명령행 옵션의 개수를 argc, 명령행 옵션의 문자열을 argv라고 합니다.  
즉, argc는 main 함수에 전달되는 매개 변수의 개수를 의미하고, argv는 매개 변수 문자열을 의미합니다.  
명령행 옵션 없이 프로그램을 실행하면 argc는 1이고, argv[0]에는 파일의 경로가 저장됩니다.  

아래 get_arg 소스코드는 명령행 옵션의 개수와 명령행 옵션의 문자열을 출력하는 프로그램입니다.

``` c
// get_arg.c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv) {
    int i;
    printf("argc: %d\n", argc);
    for (i = 0; i < argc; i++) {
        printf("argv[%d] is %s\n", i, argv[i]);
    }
    return 0;
}
```

프로그램 이름만 입력한 경우 argc는 1, argv[0]은 프로그램 경로를 출력합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level3/6.png" | relative_url }}' alt='relative'>   

프로그램 이름 뒤에 옵션을 하나 추가해 프로그램을 실행했습니다.
이 경우에선 argc가 2, argv[0]은 프로그램 경로, argv[1]은 옵션을 출력합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level3/7.png" | relative_url }}' alt='relative'>   

만약 여러 옵션을 추가하고 싶다면 옵션을 <span style="background-color: #fff8b2">띄어쓰기</span>로 구분하면 됩니다.  

<img data-action="zoom" src='{{ "assets/ftz/level3/8.png" | relative_url }}' alt='relative'>   

<img data-action="zoom" src='{{ "assets/ftz/level3/9.png" | relative_url }}' alt='relative'>   

※ autodig 프로그램의 소스코드 앞부분을 분석한 결과  
autodig 프로그램이 정상적으로 동작하기 위해선 (중간에 종료되지 않기 위해서)  

<span style="background-color: ##ffcdc0">autodig 프로그램을 실행시킬 때 한 개의 옵션을 추가 입력해 줘야 합니다.</span>  

<hr>

﻿strcpy 함수는 string copy의 약어로 <span style="background-color: #fff8b2">문자열을 복사</span>하는데 사용됩니다.  
``` bash
char* strcpy(char* dest, const char* origin);
```

﻿strcat 함수는 string concatenation의 약어로 2개의 <span style="background-color: #fff8b2">문자열을 결합</span>할 때 사용됩니다.﻿  
``` bash
char* strcat(char* dest, const char* origin);
```

아래 소스코드를 보면  
cmd 문자열 배열에 "dig @" 문자열을 복사하고,  
cmd 문자열 배열에 저장된 문자열과 argv[1] 문자열을 결합해 cmd 문자열 배열에 저장하고,  
cmd 문자열 배열에 저장된 문자열과 " version.bind chaos txt" 문자열을 결합해 cmd 문자열 배열에 저장합니다.  

``` c
// autodig 프로그램 소스코드 일부
int main(int argc, char **argv){

    char cmd[100];

...

    strcpy( cmd, "dig @" );
    strcat( cmd, argv[1] );
    strcat( cmd, " version.bind chaos txt");

...
```

﻿그러면 cmd 문자열 배열엔 다음과 같은 문자열이 저장되어 있습니다.

``` bash
dig @[argv[1]에 저장된 문자열] version.bind chaos txt
```

<hr>

그다음, system 함수는 <span style="background-color: #fff8b2">명령어를 수행하는 데</span> 사용됩니다.  
즉, 매개변수로 입력받은 문자열을 명령어로 실행합니다.  

``` bash
int system(const char* command);
```

﻿autodig의 경우 cmd 문자열 배열에 저장한 문자열을 명령어로 실행합니다.  

``` c
// autodig 프로그램 소스코드 일부
int main(int argc, char **argv){

    char cmd[100];

...

    system( cmd );

```

﻿autodig 프로그램을 실행할 때 입력한 실행 인자가 argv 변수에 저장되기 때문에,  
﻿사용자 입력에 따라 autodig 프로그램의 실행 결과가 달라집니다.  

<img data-action="zoom" src='{{ "assets/ftz/level3/10.png" | relative_url }}' alt='relative'>   

﻿autodig의 실행 인자로 '127.0.0.1' 문자열을 입력하면  
﻿cmd 문자열 배열에 'dig @127.0.0.1 version.bind chaos txt' 문자열이 저장됩니다.﻿  
﻿cmd 문자열 배열에 저장된 문자열이 system 함수를 통해 명령어로 실행되기 때문에 위의 그림과 같은 결과를 얻을 수 있습니다.  

> dig 명령어

﻿그렇다면 저 문자열이 의미하는 내용은 무엇일까요?  
﻿dig은 도메인 네임을 찾는데 사용되는 명령어입니다.  

<img data-action="zoom" src='{{ "assets/ftz/level3/11.png" | relative_url }}' alt='relative'>   

﻿구글 퍼블릭 DNS 주소를 입력하니 다음과 같은 결과를 출력합니다.  

﻿
cmd 문자열에 dig이라는 명령어가 있기 때문에 dig 뒤로 붙는 문자열은 dig 명령어의 옵션으로 인식됩니다.  

﻿하지만 한 명령어 라인에서 여러 명령을 실행할 수 있고, 앞에 명령어의 성공 또는 실패 여부와 상관없이 뒤 명령어를 실행할 수 있다면,  
﻿dig 명령어 뒤에 shell 실행 명령어를 붙여서 level4 권한으로 shell을 실행시킬 수 있습니다.  

## ﻿3) 동시에 여러 명령어를 사용하기  
﻿Linux에서 다중 명령어를 사용하는 방법은 3가지가 있습니다.  

### 1) ﻿세미콜론 (;)  
﻿한 명령어 라인의 명령어들의 성공, 실패 여부와 관계없이 전부 실행됩니다.  

``` bash
$ 명령어1; 명령어2; ...
```

<img data-action="zoom" src='{{ "assets/ftz/level3/12.png" | relative_url }}' alt='relative'>   

### ﻿2) 엠퍼센트 (&&)  
﻿명령어 라인 앞 부분에 적힌 명령어부터 순차적으로 실행합니다.    
﻿명령 실행에 실패할 경우 뒤에 오는 명령어는 실행하지 않습니다.  

``` bash
$ 명령어1 && 명령어2 && ...
```

<img data-action="zoom" src='{{ "assets/ftz/level3/13.png" | relative_url }}' alt='relative'>   

### ﻿3) 더블 버티컬바 (||)  
﻿﻿엠퍼센트와 동일하게 명령어 라인 앞 부분에 적힌 명령어부터 순차적으로 실행합니다.  
﻿명령 실행에 성공한 경우 뒤에 오는 명령어는 실행하지 않습니다.  

``` bash
$ 명령어1 || 명령어2 || ...
```

<img data-action="zoom" src='{{ "assets/ftz/level3/14.png" | relative_url }}' alt='relative'>   

<hr>

위의 autodig 프로그램으로 shell을 실행하기 위해  
dig 명령어 뒤에 shell 실행 명령어가 올 수 있도록 autodig 프로그램 실행 인자로 shell 실행 명령어를 입력합니다.  
이때, dig 명령어가 실행에 성공하든 실패하든 뒤따라오는 shell 실행 명령어가 실행하도록 세미콜론(';')을 사용합니다.  

즉, cmd 문자열 배열에 아래와 같이 저장되면 됩니다.  

``` bash
dig @;sh; version.bind chaos txt
```

``` bash
dig @;/bin/sh; version.bind chaos txt
```

``` bash
dig @;bash; version.bind chaos txt
```

﻿세미콜론 사이에는 shell을 실행할 수 있는 어떤 명령어든 들어올 수 있습니다.  
﻿level4 권한으로 shell이 실행되는 것이 중요하기 때문에 앞, 뒤 명령어를 신경쓸 필요 없습니다.  

## 4) 문자열 형태로 명령어를 전달하기  
autodig 프로그램을 실행할 때 '; bash;' 문자열을 실행 인자로 입력해야 합니다.  
명령행에서 세미콜론(';')을 입력하면 다중 명령어를 사용한다는 명령어로 인식합니다.  
실행 인자로 넘길때는 문자열로 넘겨줘 세미콜론(';')을 문자로 인식하도록 해야 합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level3/15.png" | relative_url }}' alt='relative'>  

﻿Linux에서 문자열 형태로 명령어를 전달하기 위해선 따옴표를 사용하면 됩니다.  
﻿문자열을 나타낼 때는 큰따옴표든 작은따옴표든 상관없습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level3/16.png" | relative_url }}' alt='relative'>   

﻿아래 그림과 같이 ';bash;' 문자열을 실행 인자로 넘겨줬더니 level4 권한의 shell이 실행된 것을 확인할 수 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level3/17.png" | relative_url }}' alt='relative'>    

``` bash
autodig ';bash;'
```

﻿cmd 문자열 배열에 저장된 문자열은 "dig @;bash; version.bind chaos txt" 입니다.﻿  
﻿system 함수로 cmd 문자열 배열에 저장된 문자열을 명령어로 실행합니다.  

﻿이 외에도 아래와 같은 방법으로도 level4 권한의 shell을 실행시킬 수 있습니다.  

``` bash
autodig ';bash||'
```

<img data-action="zoom" src='{{ "assets/ftz/level3/18.png" | relative_url }}' alt='relative'>    

``` bash
autodig ';bash&&'
```

<img data-action="zoom" src='{{ "assets/ftz/level3/19.png" | relative_url }}' alt='relative'>    

## 5) level4의 비밀번호 획득  
﻿1) 파일의 소유자가 level4이면서 SetUID가 설정된 파일을 찾았고,  
﻿2) 소스코드 분석으로 system 함수의 취약성을 확인했으며,  
﻿3) 다중 명령어를 사용하는 방법과 문자열 형태로 명령어를 전달하는 방법을 확인했습니다.  

﻿<span style="background-color: #ffcdc0">SetUID가 설정된 파일에 system 함수의 취약성으로 level4 권한으로 shell을 실행할 수 있었고</span>  
﻿<span style="background-color: #ffcdc0">level4 권한으로 쉘을 획득했기 때문에 my-pass 명령어로 level4의 비밀번호를 얻을 수 있었습니다.</span>  

<img data-action="zoom" src='{{ "assets/ftz/level3/20.png" | relative_url }}' alt='relative'>    