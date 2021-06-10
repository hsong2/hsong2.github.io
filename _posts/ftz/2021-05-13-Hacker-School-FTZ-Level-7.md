---
layout: post
title: "Hacker School FTZ - Level 7"
description: "Do you know the ASCII?"
comments: true
categories: FTZ
---

<img data-action="zoom" src='{{ "assets/ftz/level7/1.jpg" | relative_url }}' alt='relative'>  

## 1) level7/come together 입력해 로그인  

<img data-action="zoom" src='{{ "assets/ftz/level7/2.png" | relative_url }}' alt='relative'>  

/bin/level7 명령을 실행하면 패스워드 입력을 요청한다고 합니다.  

1. 패스워드는 가까운곳에..  
2. 상상력을 총동원하라.  
3. 2진수를 10진수로 바꿀 수 있는가?  
4. 계산기 설정을 공학용으로 바꾸어라.  

4개의 힌트를 줬는데 우선 /bin/level7을 실행해서 어떻게 동작하는지 확인해보겠습니다.  

## 2) /bin/level7 실행하기  

<img data-action="zoom" src='{{ "assets/ftz/level7/3.png" | relative_url }}' alt='relative'>  

/bin/level7을 실행해보니 비밀번호를 입력받고 있습니다.  
우선은 비밀번호를 모르니 바로 엔터를 눌렀습니다.  

/bin/wrong.txt 파일이 없다고 나옵니다.  

<img data-action="zoom" src='{{ "assets/ftz/level7/4.png" | relative_url }}' alt='relative'>  

이번에는 아무 값이나 입력해봤습니다.  
똑같이 /bin/wrong.txt 파일이 없다고 나옵니다.  

## 3) /bin/wrong.txt 파일 확인하기  

<img data-action="zoom" src='{{ "assets/ftz/level7/5.png" | relative_url }}' alt='relative'>  

/bin/wrong.txt 파일에 대한 정보를 확인하기 위해 ls 명령어를 사용해 검색해보니 해당 파일은 없다고 합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level7/6.png" | relative_url }}' alt='relative'>  

/bin 디렉토리 아래 파일을 생성하는 권한이 없어 해당 파일을 만들 수 없습니다.  

**구글에 검색을 해보니 FTZ 로컬서버를 구축해서 발생한 오류라고 합니다.**  
<span style="background-color: #fff8b2">root 계정으로 들어가 /bin 디렉토리 아래 wrong.txt 파일을 추가했습니다.</span>  

root 계정 정보: root/hackerschool  

<img data-action="zoom" src='{{ "assets/ftz/level7/7.png" | relative_url }}' alt='relative'>  

``` bash

올바르지 않은 패스워드 입니다.

패스워드는 가까운곳에...

--_--_- --____- ---_-__ --__-_-

```

위 내용을 /bin/wrong.txt 파일에 입력하면 됩니다.  
제 경우 한글이 입력이 안 돼 맨 아래 줄만 입력했습니다.  

## 4) 힌트 ... 상상력 총동원!  

<span style="background-color: #fff8b2">--_--_- --____- ---_-__ --__-_-</span>  

힌트에서 주는 정보는 모스부호를 의미하는 것으로 보입니다.  
(파일 메타데이터에 모스부호를 삽입하여 데이터를 은닉하거나 비밀 통신하는 경우가 더러 있습니다.)  

힌트 내용을 고려해 /bin/wrong.txt 파일에 적인 내용을 모스부호라고 가정한 뒤 '-'는 1을 '_'는 0으로 해석하면,  
'1101101 1100001 1110100 1100101'이라는 데이터를 얻을 수 있습니다.  

## 5) 2진수를 10진수로  

7비트씩 나눠서 표기한 것을 보고 이 2진수 값은 아스키 코드를 나타낸다는 것을 유추했습니다.  

1101101: 109 -> m  
1100001: 97 -> a  
1110100: 116 -> t  
1100101: 101 -> e  

7비트로 쪼개서 하나씩 계산해보니 'mate'라는 단어를 구했습니다.  

## 6) 비밀번호 입력하기  

<img data-action="zoom" src='{{ "assets/ftz/level7/8.png" | relative_url }}' alt='relative'>  

/bin/level7 프로그램을 실행시켜 'mate'를 입력하니 level8의 비밀번호를 획득할 수 있었습니다.  
