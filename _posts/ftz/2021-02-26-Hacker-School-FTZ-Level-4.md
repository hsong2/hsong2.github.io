---
layout: post
title: "Hacker School FTZ - Level 4"
description: "Do you know the vulnerability of the finger service?"
comments: true
categories: FTZ
---
<img data-action="zoom" src='{{ "assets/ftz/level4/1.png" | relative_url }}' alt='relative'>  

## 1) level4/suck my brain 입력해 로그인  
<img data-action="zoom" src='{{ "assets/ftz/level4/2.png" | relative_url }}' alt='relative'>  

level4로 로그인한 뒤 어떤 파일이 있는지 확인해봅니다.  
어김없이 hint 파일이 있습니다. cat 명령어로 hint 파일 내용을 확인해보니 누군가 /etc/xinetd.d/에 백도어를 심어놨다고 합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/3.png" | relative_url }}' alt='relative'>  

> /etc/xinetd.d  
<a href="https://www.linux.co.kr/lecture/lec_linux_01/lec-data/13data_1.pdf">참고자료</a>  

리눅스에는 데몬이라는 백그라운드로 실행되며 클라이언트의 요청에 서비스하는 서버 프로세스가 있습니다.  
가장 대표적인 서비스 중에는 Telnet, SSH, HTTP, FTP 등이 있는데,  
클라이언트가 이런 서비스를 이용하기 위해선 서버에 해당 데몬이 실행 중이어야 합니다.  
그래서 데몬은 서버 메모리에 상주하면서 클라이언트로부터 요청이 들어오면 바로 응답할 수 있도록 대기하고 있습니다.  


이러한 여러 가지 데몬을 관리하는 게 xinetd입니다.  
여러 가지 데몬들을 제어하고 각 서비스의 연결을 담당하기 때문에 슈퍼 데몬이라고 불립니다.  
xinetd는 /etc/xinetd.d/ 디렉터리 아래 서비스별 설정 파일을 저장하고, 설정 파일에 정의된 내용으로 서비스를 제어합니다.  


클라이언트가 서비스 접속을 시도하면 xinetd에 의해 인가된 사용자인지 TCP Wrapper(접근제어)로 검사받습니다.  
인가된 사용자인 경우 xinetd의 서비스 설정 파일을 참조하고, 설정 파일에 정의된 데몬과 연결합니다.  
위의 과정을 거쳐 서비스 데몬과 연결된 사용자는 서비스를 이용할 수 있습니다.   


> 백도어란?  

백도어(Backdoor)는 문자 그대로 뒷문을 의미합니다.  
컴퓨터 시스템의 백도어는 인증 없이 전산 시스템에 접속 가능한 비밀통로라고 할 수 있습니다.  
관리자 권한의 백도어가 심어져 있다면 제3자가 보안 인증 과정 없이 관리자 권한으로 로그인한 뒤,  
개인·기밀 정보를 손쉽게 열람하거나 유출할 수 있습니다.  

 
해커들은 백도어를 들키지 않기 위해 정상적인 프로그램처럼 위장하기도 하고 숨겨놓기도 합니다.  
백도어는 소프트웨어 프로그램 형태에 국한되지 않고 하드웨어에 칩으로 숨겨져 있기도 합니다.  
심지어는 제조사에서 편리하게 제품을 관리 또는 사용자 시스템의 정보를 수집하기 위해 의도적으로 백도어를 심어놓기도 합니다.  

## 2) /etc/xinetd.d/ 디렉터리에 심어진 백도어 찾기  
/etc/xinetd.d/ 디렉터리에 어떤 설정 파일이 있는지 살펴보니 backdoor라는 이름을 가진 파일이 제일 먼저 눈에 띕니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/4.png" | relative_url }}' alt='relative'>  


backdoor 파일의 내용을 살펴보니 service finger라고 적혀있습니다.  
backdoor는 finger 서비스에 대한 설정 파일이라는 것을 확인할 수 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/5.png" | relative_url }}' alt='relative'>  

> finger  
finger 서비스는 로컬 서버 또는 원격 서버에서 시스템 사용자 정보를 확인하는 서비스입니다.  

### 1) 현재 시스템에 로그인된 모든 사용자를 확인할 수 있습니다.  

``` bash
finger
```

<img data-action="zoom" src='{{ "assets/ftz/level4/6.png" | relative_url }}' alt='relative'>  

### 2) 로컬 서버에서 특정 user 정보를 확인할 수 있습니다.  

``` bash
finger [user]
```

<img data-action="zoom" src='{{ "assets/ftz/level4/7.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level4/8.png" | relative_url }}' alt='relative'>  

### 3) finger 서비스로 원격지 서버의 사용자들로부터 요청되는 로컬 사용자 정보를 확인할 수 있습니다.  
→ 호스트 서버에 로그인된 모든 사용자를 확인할 수 있습니다.  

``` bash
finger @[host]
```

→ 호스트 서버의 특정 user 정보를 확인할 수 있습니다.  

``` bash
finger [user]@[host]
```

또는  

``` bash
finger @[host] [user]
```

현재 서버에서도 /etc/services 파일에서 finger 문자열을 검색해보니, finger 서비스가 사용되고 있습니다.  
또한, finger 서비스는 TCP 79번 포트를 사용한다고 정의되어 있습니다.  
※ /etc/services 파일: 리눅스 서버에서 사용하는 모든 포트를 정의합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/9.png" | relative_url }}' alt='relative'>  

따라서 웬만하면 서버 보안을 위해 /etc/xinetd.d/finger 파일을 삭제하고 /etc/services 파일 내에 finger 행을 삭제해야 합니다.  

## 3) /etc/xinetd.d/backdoor 파일 분석하기  

<img data-action="zoom" src='{{ "assets/ftz/level4/10.png" | relative_url }}' alt='relative'>  

<img data-action="zoom" src='{{ "assets/ftz/level4/11.png" | relative_url }}' alt='relative'>  

finger 서비스의 데몬 프로그램이 /home/level4/tmp/ 디렉터리 아래 backdoor라는 이름으로 저장되어 있습니다.  
/home/level4/tmp/backdoor 프로그램이 실행될 때, level5 권한으로 실행된다고 합니다.  

## 4) /home/level4/tmp/backdoor 파일  

/home/level4/tmp/ 디렉터리 내 파일을 검색해보니 아무 파일도 존재하지 않습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/12.png" | relative_url }}' alt='relative'>  

해당 디렉터리에 finger 서비스가 실행될 때 연결될 backdoor 프로그램을 직접 작성해야 합니다.  
level5의 비밀번호를 얻어야 하기 때문에 level5의 비밀번호를 출력하는 명령어를 실행하도록 프로그램을 제작합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/13.png" | relative_url }}' alt='relative'>  

``` c
// backdoor.c
#include <stdio.h>
int main() {
    system("my-pass");
    return 0;
}
```

<img data-action="zoom" src='{{ "assets/ftz/level4/14.png" | relative_url }}' alt='relative'>  

/home/level4/tmp/backdoor 프로그램은 level5 권한으로 실행되기 때문에,  
my-pass 명령어를 실행하면 level5의 비밀번호를 출력합니다.  

## 5) finger 서비스를 통한 /home/level4/tmp/backdoor 실행 방법  
위에서 4) backdoor 프로그램을 만들었지만,  
finger 명령어를 사용하면 서버 사용자 정보만 출력되고 level5의 비밀번호를 출력하지 않습니다.  

finger 서비스를 찾아보니 /etc/xinetd.d/ 디렉터리에 finger 서비스 설정 파일과 /usr/bin/finger 파일이 존재합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/15.png" | relative_url }}' alt='relative'>  


우선 /etc/xinetd.d/finger 파일을 살펴보니 disable이 yes로 설정되어 있습니다.  
원격에서 finger 서비스에 접속하면 이 파일은 xinetd 제어를 받지 않기 때문에 in.fingerd 데몬과 연결되지 않습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/16.png" | relative_url }}' alt='relative'>  

원격으로 finger 서비스에 접속하면 /etc/xinetd.d/finger 설정 파일이 아닌  
/etc/xinetd.d/backdoor 설정 파일을 참조하고,  
설정 파일에 정의된 /home/level4/tmp/backdoor가 데몬으로 실행됩니다.

<hr width="20px">

리눅스에서는 $PATH 환경 변수에 명령 등이 속해 있는 경로를 저장합니다.  
$PATH에 저장된 경로에 있는 명령어는 명령어 이름만 쳐도 실행이 가능합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/17.png" | relative_url }}' alt='relative'>  

ls 파일은 /bin/ 디렉터리에 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/18.png" | relative_url }}' alt='relative'>  

/bin 디렉터리가 $PATH에 저장되어 있기 때문에 /bin/ls 또는 ls로 쓸 수 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/19.png" | relative_url }}' alt='relative'>  


/usr/bin/finger 파일도 $PATH에 /usr/bin 디렉터리가 저장되어 있기 때문에 finger만 쳐도 실행됩니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/20.png" | relative_url }}' alt='relative'>  

로컬에서 finger 명령어를 치면 /usr/bin/finger 파일이 실행됩니다.  

<hr width="20px">

<span style="background-color: #ffcdc0">즉, finger로 원격지 서버에 접속해야 xinetd에 의해 backdoor(finger)의 my-pass 데몬이 실행되니,</span>  
<span style="background-color: #ffcdc0">원격으로 finger 서비스에 접속해야 level5의 비밀번호를 알아낼 수 있습니다.</span>  

## 6) 원격으로 finger 서비스 접속 및 level5의 비밀번호 획득  

우선 finger 서비스가 동작 중인지 확인해봅니다.  
아래 그림과 같이 연결 가능한 TCP 네트워크 정보를 확인합니다.  

네트워크 정보를 확인해보니 finger 서비스가 사용하는 port는 TCP 79번이라고 합니다.  
79번 포트를 통해 원격지에서 서버로 연결이 가능한 것을 확인할 수 있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level4/21.png" | relative_url }}' alt='relative'>  


finger 서비스에 원격으로 접속해 /home/level4/tmp/backdoor 프로그램을 실행합니다.  
finger 서비스에 원격으로 접속하는 방법은 finger 서비스를 사용하거나 telnet 서비스를 사용하면 됩니다.  

### 1) finger 서비스 사용 방법  

``` bash
finger @localhost
```

<img data-action="zoom" src='{{ "assets/ftz/level4/22.png" | relative_url }}' alt='relative'>  


### 2) telnet 서비스 사용 방법  

``` bash
telnet localhost 79
```

<img data-action="zoom" src='{{ "assets/ftz/level4/23.png" | relative_url }}' alt='relative'>  
