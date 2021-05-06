---
layout: post
title: "Hacker School FTZ - Level 5"
description: "Do you know the race condition attack?"
comments: true
categories: ftz
---
<img data-action="zoom" src='{{ "assets/ftz/level5/1.png" | relative_url }}' alt='relative'>  

## 1) level5/what is your name? 입력해 로그인  
<img data-action="zoom" src='{{ "assets/ftz/level5/2.png" | relative_url }}' alt='relative'>  

level5로 로그인한 뒤 hint 파일을 살펴보니 다음과 같은 내용이 적혀있습니다.

<img data-action="zoom" src='{{ "assets/ftz/level5/3.png" | relative_url }}' alt='relative'>  

<span style="background-color: #fff8b2">/usr/bin/level5 프로그램은 /tmp 디렉토리에</span>  
<span style="background-color: #fff8b2">level5.tmp라는 이름의 임시파일을 생성한다.</span>  

<span style="background-color: #fff8b2">이를 이용하여 level6의 권한을 얻어라.</span>  

## 2) /usr/bin/level5 파일 확인하기  
<img data-action="zoom" src='{{ "assets/ftz/level5/4.png" | relative_url }}' alt='relative'>  

/usr/bin/level5은 user의 경우 read, write, execute 권한이 있고,  
group의 경우 execute 권한이 있습니다.  

SetUID가 설정되어 있기 때문에 해당 파일 실행시 level6의 권한으로 프로그램이 실행됩니다.

우선 /usr/bin/level5 프로그램을 실행하면 level6 권한으로 실행되기 때문에  
/usr/bin/level5 프로그램이 동작할 때 my-pass 프로그램을 실행할 수 있으면  
level6의 비밀번호를 얻을 수 있을 것으로 예상됩니다.  


## 3) /tmp 디렉토리 확인하기
그다음 /usr/bin/level5 프로그램이 실행될 때 level5.tmp 임시 파일이 생성되는 /tmp 디렉토리를 확인해보겠습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level5/5.png" | relative_url }}' alt='relative'>  

/tmp 디렉토리의 권한을 보니 user의 경우 read, write, execute 권한이 있고,  
group 또한 read, write, execute 권한이 있고,  
other 또한 read, write, execute 권한이 있습니다.  

other 권한에서 rw't'는 Sticky bit가 설정되었다는 의미입니다.  
디렉토리에 Sticky bit가 설정되면 디렉토리 소유자나 파일 소유자 또는 슈퍼유저가 아닌 사용자들은 파일을 삭제하거나 이름을 변경하지 못합니다.  
다만, 파일 또는 디렉토리 생성은 누구나 할 수 있습니다.  
그래서 Sticky bit는 공용 디렉토리에서 주로 사용됩니다.  


/tmp 디렉토리를 살펴보니 level5.tmp 파일이 없습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level5/6.png" | relative_url }}' alt='relative'>  

/usr/bin/level5 프로그램을 실행시키고 난 후에도 임시 파일이 보이지 않습니다.  
/usr/bin/level5 프로그램이 실행될 때 level5.tmp 임시 파일이 생성됐다가 프로그램이 종료되면서 삭제되는 것으로 예상됩니다.  

## 4) 리눅스애서 임시파일이 갖는 취약점은?  

구글에 <span style="background-color: #fff8b2">'linux 임시파일 취약점'</span>이라는 키워드로 검색을 해보니  
레이스 컨디션 공격에 대한 정보가 나옵니다.  

레이스 컨디션 공격에 대해 알아보니 다음과 같습니다.  

> 1. 레이스 컨디션(Race condition)  

레이스 컨디션이란 한정된 자원을 동시에 이용하려는 여러 프로세스가 자원의 이용을 위해 경쟁을 벌이는 현상입니다.  

이러한 레이스 컨디션 현상을 악용하여 공격자가 원하는 작업을 수행하는 것을 레이스 컨디션 공격이라고 합니다.  

취약한 프로그램의 프로세스와 공격자가 만든 프로그램의 프로세스가 반복해서 실행되며 임시 파일을 이용하기 위해 경쟁을 하게 됩니다.  
그러나 임시 파일을 생성하는 프로그램이라고 모두 다 공격 대상 파일이 되지 않습니다.  
우선 레이스 컨디션 공격을 위해  
### 1) 공격자는 시스템의 관리자 권한을 얻기 위해 파일의 소유자가 root이고 SetUID가 설정되어 있고 임시 파일을 생성하는 프로그램을 찾은 뒤,  

### 2) 1번 단계에서 찾은 취약한 프로그램이 생성하는 임시 파일의 이름을 파악해야 합니다.

결국 레이스 컨디션 공격이 성공하면 공격자가 원하는 작업을 관리자 권한으로 실행할 수 있게 됩니다.  

<hr width="20px">

> 2. 심볼릭 링크(Symbolic link)  

Windows의 바로가기와 동일하게 Linux에서는 원본 파일에 링크를 연결하여 원본 파일을 직접 사용할 수 있게 합니다.  
심볼릭 링크를 설정하면 원본 파일에 수정한 모든 내용이 공유됩니다.  

<img data-action="zoom" src='{{ "assets/ftz/level5/7.png" | relative_url }}' alt='relative'>  

그림과 같이 비어있는 /home/level5/tmp 디렉토리에 origin 파일을 생성합니다.  
origin 파일 내부에는 'hello world'라는 글이 적혀있습니다.  

```bash
ln -s [origin file] [new file]
```

심볼릭 링크를 생성하는 명령어를 입력하여 new라는 이름을 가진 파일을 생성합니다.  
new 파일은 origin 파일과 심볼릭 링크로 연결되어 있습니다.  


<img data-action="zoom" src='{{ "assets/ftz/level5/8.png" | relative_url }}' alt='relative'>  

새로 생성된 new 파일을 읽어보니 origin 파일과 동일한 내용이 출력됩니다.  
new 파일을 수정해서 'my name is r0sa'라는 글을 추가하고 저장했습니다.  

new 파일에서 수정한 내용이 origin 파일에서도 수정된 것을 확인할 수 있습니다.


<img data-action="zoom" src='{{ "assets/ftz/level5/9.png" | relative_url }}' alt='relative'>  

동일하게 origin 파일에서도 내용을 수정해봤습니다.  
origin 파일에 'hello everyone'이라는 글을 추가하고 저장했습니다.  

이번에도 origin 파일을 수정한 내용이 new 파일에서도 수정된 것을 확인할 수 있습니다.  

<hr width="20px">

/usr/bin/level5 프로그램이 실행되는 절차는 다음과 같이:  
프로그램 실행 (level5 user가 실행) -> SetUID로 인한 프로세스 권한이 level6으로 상승 -> 임시 파일(level5.tmp) 생성 -> 프로그램 동작 -> 임시 파일 삭제 -> 프로그램 종료 순서로 진행됩니다.  

앞서 살펴본 레이스 컨디션 공격과 심볼릭 링크에 대한 내용으로 유추해보면,  
level5.tmp와 동일한 이름의 심볼릭 링크 파일을 만들어 /usr/bin/level5 프로그램이 실행됐을 때  
내가 만든 심볼릭 링크 파일이 실행되도록 합니다.  
해당 심볼릭 링크 파일이 /usr/bin/level5 프로그램의 임시 파일로 사용되며 작업을 수행하면 심볼릭 링크 파일과 연결된 파일에  
/usr/bin/level5 프로그램이 작업한 내용이 저장될 것 입니다.  

## 5) /tmp 디렉토리에 level5.tmp 파일 생성하기  
/usr/bin/level5가 만들어내는 임시 파일(level5.tmp)과 동일한 이름의 **심볼릭 링크**를 만들어보겠습니다.  

힌트 내용에서 /usr/bin/level5 프로그램은 /tmp 디렉토리에 level5.tmp 임시 파일을 만든다고 했으니  
/tmp 디렉토리로 이동하여 level5.tmp 이름의 심볼릭 링크를 만들어 주겠습니다.  

심볼릭 링크를 만들기 위해선 해당 심볼릭 링크 파일과 연결될 원본 파일이 필요하기 때문에  
touch 명령어로 빈 파일 하나를 만들어줍니다.  

<img data-action="zoom" src='{{ "assets/ftz/level5/10.png" | relative_url }}' alt='relative'>  

/usr/bin/level5 프로그램이 실행하면 /tmp 디렉토리에 있는 level5.tmp 심볼릭 링크 파일을 임시 파일로 이용하고  
level5.tmp 파일과 연결된 a 파일에도 /usr/bin/level5 프로그램이 수행한 작업 내용이 저장됩니다.  
 
위에서 심볼릭 링크에 대해 알아본 내용처럼 심볼릭 링크가 수정되면 원본 파일도 수정되기 때문에  
원본 파일인 a에서 /usr/bin/level5 프로그램이 어떤 작업을 수행했는지 확인할 수 있습니다.  

## 6) /usr/bin/level5 프로그램 동작하기  

<img data-action="zoom" src='{{ "assets/ftz/level5/11.png" | relative_url }}' alt='relative'>  

/usr/bin/level5 프로그램 실행 전에는 a 파일의 사이즈가 0이었는데 프로그램 실행 후 사이즈가 31로 바뀐 것을 확인할 수 있습니다.  

a 파일에 담긴 내용을 cat 명령어로 확인해보니 level6의 비밀번호를 획득할 수 있었습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level5/12.png" | relative_url }}' alt='relative'>  
