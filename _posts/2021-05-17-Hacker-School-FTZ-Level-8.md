---
layout: post
title: "Hacker School FTZ - Level 8"
description: "Do you know the John the Ripper?"
comments: true
categories: ftz
---
<!--
<img data-action="zoom" src='{{ "assets/ftz/level8/1.jpg" | relative_url }}' alt='relative'>  

## 1) level8/break the world 입력해 로그인  

<img data-action="zoom" src='{{ "assets/ftz/level8/2.png" | relative_url }}' alt='relative'>  

level9의 shadow 파일이 서버 어딘가에 숨겨져 있다.  
그 파일에 대해 알려진 것은 용량이 "2700"이라는 것 뿐이다.  
라는 힌트를 제공해줍니다.  

> shadow 파일이란?  

사용자별 패스워드를 암호화하여 보호하기 위해 만들어진 파일입니다.  
/etc/passwd 파일에 저장되어 있던 비밀번호를 /etc/shadow 파일에 암호화하여 비밀번호만 별도로 저장합니다.  
또한, shadow 파일에는 패스워드 만료기간, 갱신기간, 최소기간 등 비밀번호 관리를 위한 정책 또한 저장되어 있습니다.  
/etc/passwd 파일과 /etc/shadow 파일을 read 하기 위해선 root 권한이 필요합니다.  


## 2) 용량이 2700인 파일 찾기  


### 1) 파일 용량 기준으로 검색  

검색 기준 이상의 용량을 가진 파일을 검색할 땐 '+'를 숫자 앞에 붙이고, 검색 기준 이하의 용량을 가진 파일을 검색할 땐 '-'를 숫자 앞에 붙입니다.  
검색 기준과 일치하는 용량을 가진 파일을 검색할 때는 숫자 앞에 아무런 기호도 붙이지 않습니다.  
 
volume 크기의 용량을 가진 파일 검색 시,  

``` bash
find [검색할 디렉토리] -size [volume] 
```

volume 크기 이상의 용량을 가진 파일 검색 시,  

``` bash
find [검색할 디렉토리] -size +[volume] 
```

volume 크기 이하의 용량을 가진 파일 검색 시,  

``` bash
find [검색할 디렉토리] -size -[volume] 
```

### 2) 파일 용량 단위  

그 다음 용량의 크기 단위를 지정해줘야 합니다.  
위의 힌트에서처럼 파일의 용량이 2700이라고 알려주면 2700바이트인지, 2700키로 바이트인지, 2700메가 바이트인지 컴퓨터는 알 수 없기 때문입니다.  

'b': blocks (512-byte)  
'c': bytes  
'w': two-byte words  
'k': Kibibytes (KiB, units of 1024 bytes)  
'M': Mebibytes (MiB, units of 1024 X 1024 = 1048576 bytes)  
'G': Gibibytes (GiB, units of 1024 X 1024 X 1024 = 1073741824 bytes)  


위의 정보를 토대로 힌트가 제시한 파일을 검색해보았습니다.  

``` bash
find / -size 2700c 2>/dev/null
```

용량이 2700인 파일을 검색해보니 4개의 파일이 검색됩니다.   
아무래도 파일 경로와 이름을 보아 /etc/rc.d/found.txt가 level9의 shadow 파일로 추측됩니다.  

<img data-action="zoom" src='{{ "assets/ftz/level8/3.png" | relative_url }}' alt='relative'>  

/etc/rc.d/found.txt 파일에 적혀진 내용을 출력해보니 shadow 파일입니다.  
level9의 비밀번호가 암호화되어 저장되어 있는 것을 확인할 수 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level8/4.png" | relative_url }}' alt='relative'>   


## 3) /etc/rc.d/found.txt 파일 확인하기  

<img data-action="zoom" src='{{ "assets/ftz/level8/5.png" | relative_url }}' alt='relative'>   

shadow 파일의 각 필드는 ':'로 구분됩니다.  
총 9개의 필드가 있으며 각 필드가 나타내는 정보는 다음과 같습니다.  

1) 사용자명
2) 암호화된 패스워드
3) 패스워드 파일 최종 수정일
4) 패스워드 최소 변경일 (최소 사용일)
5) 패스워드 최대 변경일 (최대 사용일)
6) 패스워드 경고일 (패스워드 만료까지 남은 기간)
7) 비활성기간 (패스워드 만료 후 계정이 비활성되는 기간으로 비활성기간이 지나면 계정이 잠김)
8) 계정 만료 기간
9) 예약 필드

위의 그림을 참고하면 level9 사용자의 비밀번호는 '$1$vkY6sSlG$6RyUXtNMEVGsfY7Xf0wps.' 입니다.
2번 필드에서는 '$'로 3가지 정보를 구분합니다.  

앞에서부터 순서대로 $<span style="background-color: #fff8b2">id</span>$<span style="background-color: #fff8b2">salt</span>$<span style="background-color: #fff8b2">암호화된 패스워드</span> 입니다.  

id는 해시 알고리즘의 종류를 나타내고, salt는 평문에 임의의 값을 추가하여 동일한 비밀번호를 사용하더라도 서로 다른 해시값이 나오도록 하는 유도합니다.  

level9 비밀번호가 저장된 shadow 파일이 나타내는 정보를 바탕으로 확인할 수 있는 것은  
id가 1인 경우 MD5 해시 알고리즘이 사용됐음을 의미하고, salt는 'vkY6sSlG'이 사용된 것을 알 수 있습니다.  

## 4) MD5 해시값 복호화  

<a href="https://www.krcert.or.kr/data/trendView.do?bulletin_writing_sequence=2304">MD5를 사용하여 해시 값으로 저장된 16자리 비밀번호를 한 시간 안에 해독할 수 있다는 것이 실험을 통해 밝혀졌습니다.</a>  
level9의 비밀번호가 md5 해시 알고리즘으로 암호화됐기 때문에 복호화해서 level9의 비밀번호를 얻도록 하겠습니다.  

구글에 검색해보니 해당 md5 해시 값은 John the Ripper라는 도구로 복호화가 가능하다고 합니다.  
<span style="background-color: #fff8b2">John the Ripper</span> 도구는 패스워크 크래커로 유명한 도구입니다.  

해당 툴을 다운로드 받아 패스워드 크래킹을 해보겠습니다.  
<a href="https://www.openwall.com/john/">John the Ripper password cracker</a> 사이트에서 운영체제에 맞게 설치해줍니다.  

README 파일을 살펴보니 John the Ripper 도구 사용방법은 다음과 같습니다.  

> How to use.  
To run John, you need to supply it with some password files and  
optionally specify a cracking mode, like this, using the default order  
of modes and assuming that "passwd" is a copy of your password file:  

``` bash
	john passwd
```

passwd 파일에 level9의 shadow 파일 정보를 저장했습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level8/6.png" | relative_url }}' alt='relative'>   

<img data-action="zoom" src='{{ "assets/ftz/level8/6.png" | relative_url }}' alt='relative'>   

John the Ripper 패스워드 크래킹 툴을 사용하여 level9의 비밀번호를 획득할 수 있습니다.  

-->