---
layout: ftz
title: "Hacker School FTZ - Level 2"
description: "Do you know the file permission and SetUID in the Linux?"
comments: true
---
<img data-action="zoom" src='{{ "assets/ftz/level2/1.jpg" | relative_url }}' alt='relative'>  

## ﻿1) level2/hacker or cracker 입력해 로그인 
<img data-action="zoom" src='{{ "assets/ftz/level2/2.png" | relative_url }}' alt='relative'>  
level1에서와 동일하게 hint 파일이 있습니다. 
cat 명령어를 사용해 hint 파일의 내용을 확인해보니 다음과 같은 내용이 적혀있습니다. 

<img data-action="zoom" src='{{ "assets/ftz/level2/3.png" | relative_url }}' alt='relative'>  
<span style="color:blue">*﻿텍스트 파일 편집 중 쉘의 명령을 실행시킬 수 있다는데...*</span>

﻿
level2 문제의 목표는 level3 비밀번호를 찾는 것입니다. 
힌트 내용으로 유추할 수 있는 정보는 다음과 같습니다. 
1) level3 소유자 권한의 SetUID가 설정된 파일이 존재할 것이다. 
2) 그 파일이 텍스트 파일 편집기일 확률이 높다. 
3) 파일 편집기를 실행하는 도중에 쉘의 명령을 실행할 수 있다. 

1)부터 3)까지 유추한 정보가 맞는지 차례대로 확인해보겠습니다.

## ﻿2) level3 소유 권한을 갖고 SetUID가 걸린 파일 찾기 
﻿[level1](https://hsong2.github.io/ftz/2021/02/22/Hacker-School-FTZ-Level-1.html)에서 파일을 검색하는 명령어와 SetUID에 대해 알아봤었습니다. 

``` bash
find / -user level3 -perm -4000 2>/dev/null
```

<img data-action="zoom" src='{{ "assets/ftz/level2/4.png" | relative_url }}' alt='relative'>   

소유자가 level3이면서 SetUID가 설정된 파일을 검색하니 /usr/bin 디렉토리 아래 editor 이름을 가진 파일이 하나 존재합니다.  
파일 이름부터가 텍스트 파일 편집기 같습니다.

<img data-action="zoom" src='{{ "assets/ftz/level2/5.png" | relative_url }}' alt='relative'>  

파일의 정보를 자세히 살펴보니,  
→ 소유자의 경우 read, write, execute(SetUID) 권한을  
→ 그룹 소유자의 경우 read, execute 권한을 갖고  
→ 기타 사용자의 경우 아무 권한이 없습니다.

editor 파일의 소유자는 level3이고, 그룹 소유자는 level2로 설정되어 있어 level2 user가 editor 프로그램을 실행할 수 있습니다.

## ﻿﻿3) editor 실행파일 확인  
﻿editor 파일을 실행해보니 아래 사진과 같은 내용이 나옵니다.

<img data-action="zoom" src='{{ "assets/ftz/level2/6.png" | relative_url }}' alt='relative'> 

﻿editor 프로그램은 Linux 문서 편집기인 vi, vim이었습니다.  
﻿Linux에서는 vi 또는 vim 명령어를 입력하여 텍스트 파일을 편집할 때 사용합니다.  

## ﻿4) vi로 쉘 명령 실행하기  
힌트에서 '텍스트 파일 편집 중 쉘의 명령을 실행할 수 있다'라고 했습니다.

vi 편집기는 3가지 모드를 갖습니다.  
### 1) 명령 모드 (Command mode)  
→ vi 편집기를 처음 실행했을 때 명령 모드로 설정됩니다. vi 명령어를 사용하기 위해선 명령 모드에 설정된 상태여야 합니다.  
### 2) 입력 모드 (Insert mode)  
→ 명령 모드에서 입력 모드로 바꾸기 위해 'i' 또는 'a'를 입력합니다. 입력 모드에선 일반적인 메모장 프로그램처럼 사용자가 자유롭게 내용을 작성할 수 있습니다. 명령 모드로 다시 돌아가기 위해서는 'esc' 키를 누르면 됩니다.  
### 3) 마지막 행 모드 (Last line mode)  
→ 명령 모드에서 ':'(콜론)을 입력하면 vi 편집기 화면 맨 하단에 명령어를 입력할 수 있습니다. 수정된 내용을 저장(w)하거나 파일 편집을 종료(q)할 때 사용됩니다. 또한, 파일의 라인 번호를 출력(set nu)하거나 문자열을 탐색(?검색 문자열)할 때 사용됩니다.

<span style="background-color: #ffcdc0;">이 외에도 vi 편집기에서 마지막 행 모드는 쉘의 명령어도 실행할 수 있습니다.</span>

#### 1) 쉘(shell)로 나가기  
마지막 행 모드에서 sh를 입력하면 쉘로 나갈 수 있습니다.

﻿``` bash
:sh
```

<img data-action="zoom" src='{{ "assets/ftz/level2/7.png" | relative_url }}' alt='relative'> 

editor 파일이 level3 소유 권한으로 SetUID가 설정되어 있기 때문에 editor 파일이 실행되는 동안엔 level3의 권한으로 모든 작업이 수행됩니다.  
그래서 :sh를 입력해 쉘로 나가면 level3 권한으로 shell이 실행됩니다.


<img data-action="zoom" src='{{ "assets/ftz/level2/8.png" | relative_url }}' alt='relative'> 

﻿sh를 입력해 쉘로 나오니 user가 level3으로 바뀐걸 확인할 수 있습니다.

#### ﻿2) shell 명령어 실행하기  
﻿마지막 행 모드에서 '!' 뒤에 쉘 명령어를 입력하면 명령어가 실행됩니다.

``` bash
:!명령어
```

``` bash
:!/bin/sh
```

``` bash
:!/bin/bash
```

``` bash
:!/bin/bash2
```

위의 그림처럼 shell을 실행하는 명령어를 입력하면 shell이 실행됩니다.

<img data-action="zoom" src='{{ "assets/ftz/level2/9.png" | relative_url }}' alt='relative'>  

﻿그래서 위 그림과 같이 level3 권한으로 shell이 실행된 것을 확인할 수 있습니다.

``` bash
:!my-pass
```

﻿level2의 목표는 level3의 비밀번호를 알아내는 것입니다.  
﻿그래서 level3 권한으로 shell을 실행할 필요 없이 바로 비밀번호를 확인하는 명령어 'my-pass'를 입력해도 됩니다.  

<img data-action="zoom" src='{{ "assets/ftz/level2/10.png" | relative_url }}' alt='relative'>  

﻿위 그림과 같이 level3의 비밀번호를 확인할 수 있습니다.  

#### ﻿3) 명령어 실행 결과(console text) 가져오기  
﻿마지막 행 모드에서 'r!' 뒤에 쉘 명령어를 입력하면 명령어가 실행되고 그 결과를 vi 편집기에 출력합니다.

``` bash
:r!명령어
```

``` bash
:r!my-pass
```

<img data-action="zoom" src='{{ "assets/ftz/level2/11.png" | relative_url }}' alt='relative'>  

명령어 앞에 'r!'를 붙여 실행하니 위 그림과 같은 화면이 나옵니다.  
위 화면과 같은 상태에서 enter 키를 누르고 다시 편집기 화면으로 돌아오니,  
아래 그림과 같이 명령어가 실행되고 난 뒤 결과를 편집기에 출력합니다.  

<img data-action="zoom" src='{{ "assets/ftz/level2/12.png" | relative_url }}' alt='relative'>  

## ﻿5) level3의 비밀번호 획득  
1) 파일의 소유자가 level3이면서 SetUID가 설정된 파일인 editor 프로그램을 찾았고,  
2) editor 프로그램이 텍스트 파일 편집기인 vi라는 것을 확인했으며,  
3) vi 명령어에서 쉘 명령어를 사용하는 방법을 확인했습니다.  

<span style="background-color: #ffcdc0;">level2 문제도 level1과 동일하게 SetUID가 설정된 파일에서 shell을 실행할 수 있는 취약점으로 level3의 비밀번호를 얻을 수 있었습니다.</span>  

<img data-action="zoom" src='{{ "assets/ftz/level2/13.png" | relative_url }}' alt='relative'>  
