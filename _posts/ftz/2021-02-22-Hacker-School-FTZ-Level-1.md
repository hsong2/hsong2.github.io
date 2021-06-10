---
layout: post
title: "Hacker School FTZ - Level 1"
description: "Do you know the file permission and SetUID in the Linux?"
comments: true
categories: FTZ

---
<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1.jpg" | relative_url }}' alt='relative'>  

## 1) level1/level1 입력하여 로그인하기
<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_1.png" | relative_url }}' alt='relative'>  
어떤 파일이 있는지 확인해보니 'hint' 파일이 있습니다.  
cat 명령어를 사용해 hint 파일의 내용을 확인해보니 다음과 같은 내용이 적혀있습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_2.png" | relative_url }}' alt='relative'>  
<span style="color:blue">*level2 권한에 SetUID가 걸린 파일을 찾는다.*</span> 라는 내용이 hint 파일에 적혀있습니다.

## ﻿2) setuid가 걸린 파일 찾는 방법
우선 SetUID가 걸린 파일을 찾기 전에 SetUID가 무엇인지 알아보겠습니다.  

> SetUID란?  

Linux는 파일에 대한 접근 권한 및 파일 종류를 나타내기 위해 16bit를 사용합니다.
<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_3.png" | relative_url }}' alt='relative'>   

SetUID는 리눅스의 특수 권한으로, SetUID가 설정된 파일을 실행하면 파일이 동작하는 모든 작업은 파일 소유자의 권한으로 실행됩니다.

***
SetUID가 적용되지 않은 my-pass 파일로 예시를 들어보겠습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_4.png" | relative_url }}' alt='relative'>  
﻿
/bin/my-pass 파일의 권한은 -rwxr-x--x입니다.  
맨 앞에 '-'는 파일 종류를 의미하기 때문에 rwxr-x--x 이 부분만 자세히 살펴보겠습니다.  
rwx/r-x/--x이렇게 3개씩 나눠서 소유자/그룹 소유자/기타 사용자 접근 권한을 나타냅니다.  

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_5.png" | relative_url }}' alt='relative'>  
my-pass 파일의 권한을 10진수로 표현하면 751입니다.  
7 → 소유자의 경우 read(4) + write(2) + execute(1) 권한을  
5 → 그룹 소유자의 경우 read(4) + execute(1) 권한을  
1 → 기타 사용자의 경우 execute(1) 권한을 갖습니다.

만약, my-pass 파일에 SetUID 권한을 적용한다면 현재 설정되어 있는 751에 4000을 더하면 됩니다.  
정확히는 더한다는 개념보다는 751 앞에 4를 붙인다는 게 맞는 표현입니다.  
그래서 my-pass파일에 SetUID 권한이 적용된다면 rwsr-x--x(4751)로 표현됩니다.

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_6.png" | relative_url }}' alt='relative'>  
현재 로그인된 user의 uid와 gid를 id 명령어로 확인해보니 둘 다 level1에 속해 있습니다.  
앞서 my-pass 파일에서 소유자는 root이고, 그룹 소유자도 root이기 때문에 level1은 기타 사용자 접근 권한을 적용받습니다.

my-pass 파일은 기타 사용자도 실행(execute) 권한을 갖기 때문에 level1도 my-pass 파일을 실행할 수 있고, my-pass 파일이 실행하는 모든 작업은 level1의 권한으로 실행됩니다.

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_7.png" | relative_url }}' alt='relative'>  
﻿그래서 my-pass 파일을 level1 user가 실행하면 level1의 비밀번호만 얻을 수 있습니다.

***
반면, SetUID가 적용된 파일은 어떨까요?

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_8.png" | relative_url }}' alt='relative'>  
﻿﻿
/bin/ExecuteMe 파일은 SetUID가 적용된 파일입니다.  
SetUID가 적용된 파일은 소유자 접근 권한 중 실행 권한이 's' 또는 'S'로 표시됩니다.  
(소문자 's'의 경우 소유자가 실행 권한을 가진 경우고, 대문자 'S'의 경우 소유자가 실행 권한을 갖지 못한 경우입니다.)  

ExecuteMe 파일의 경우 소문자 's'로 표시되어 있기 때문에 소유자가 실행 권한을 갖습니다.  
따라서 해당 파일의 권한을 숫자로 표현하면 4750이며, 계정별 권한은 다음과 같습니다.  
7 → 소유자의 경우 read, write, execute(SetUID, 4000) 권한을  
5 → 그룹 소유자의 경우 read, execute 권한을 갖고  
0 → 기타 사용자의 경우 아무 권한이 없습니다.  


ExecuteMe 파일은 소유자와 그룹 소유자만 파일을 실행할 수 있도록 설정되어 있습니다.  
ExecuteMe 파일의 소유자는 level2 고, 그룹 소유자는 level1로 설정되어 있기 때문에 level1 user가 ExecuteMe 프로그램을 실행할 수 있다는 것을 알 수 있습니다.  

ExecuteMe 파일에는 SetUID가 설정되어 있기 때문에 파일 소유자(level2)의 권한으로 프로그램의 작업이 실행됩니다.  
다시 말해, ExecuteMe 파일의 소유자 권한은 level2로 설정되어 있기 때문에 ExecuteMe 프로그램이 실행하는 모든 작업은 level2 권한을 갖습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_9.png" | relative_url }}' alt='relative'>  

그래서 ExecuteMe 프로그램을 실행시키고 id 명령어를 입력한 결과 uid가 level2인 것을 확인할 수 있습니다.  

그다음, Linux에서 특정 조건에 맞는 파일을 검색하는 방법에 대해 알아보겠습니다.

> find 명령어 사용법  

Linux에서 파일을 검색할 때 find 명령어를 사용합니다.  
찾고자 하는 파일의 <span style="color:blue">**이름**</span>을 알면 **-name** expression을,  
찾고자 하는 파일의 <span style="color:blue">**종류**</span>를 나타내기 위해 **-type** expression을,  
특정 <span style="color:blue">**사이즈**</span>를 가진 파일을 찾기 위해 **-size** expression을,  
특정 <span style="color:blue">**권한**</span>을 가진 파일을 찾기 위해 **-perm** expression을 설정합니다.  
이처럼 원하는 파일을 찾기 위해 다양한 표현으로 조건을 설정합니다.  

find 명령어의 사용 방법은 다음과 같습니다.  
``` bash
find [option] [path] [expression]
```

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_10.png" | relative_url }}' alt='relative'>  

/(root) 디렉터리 하위에 있는 모든 파일 중 my-pass 이름을 가진 파일을 찾는 명령입니다.  

위의 명령어로 파일을 검색하니 Permission denied라는 원치 않은 결과들도 나옵니다.

﻿﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_11.png" | relative_url }}' alt='relative'>  

이처럼 Permission denied와 같은 stderr를 무시하기 위해 '2>/dev/null' 명령어를 뒤에 붙여줍니다.  

2>/dev/null의 의미는 stderr(2)를 /dev/null에 리다이렉트('>')하겠다는 뜻입니다.  
그래서 위의 그림과 같이 명령어의 실행 결과 중 에러나지 않은 결과만을 확인할 수 있습니다.

> level2 권한에 SetUID가 걸린 파일 찾기

다음과 같은 명령어를 입력해 user가 <span style="color:blue">**level2**</span>이면서 <span style="color:blue">**SetUID(4000)**</span>가 설정된 파일을 찾습니다.
﻿
``` bash
find / -user level2 -perm -4000 2>/dev/null
```

위의 명령어를 보면 4000 숫자 앞에 '-' 기호가 붙은걸 볼 수 있습니다.

﻿파일 용량(-size)으로 파일을 검색할 경우 숫자 앞에 **'-'**가 붙으면 숫자보다 **이하**의, **'+'**가 붙으면 숫자보다 **초과**의, 아무 기호도 안 붙으면 숫자와 **똑같이 일치**하는 용량을 갖는 파일을 검색합니다. ﻿

그렇다면 **권한 표현식(-perm)**의 경우 숫자 앞에 붙는 기호는 어떤 의미일까요?  
권한 표현식에서 숫자 앞에 붙는 기호별 의미는 **AND** 또는 **OR**의 의미를 갖습니다.  

조금 더 자세히 살펴보겠습니다.  

* __'-'가 숫자 앞에 붙는 경우 AND 연산﻿__

``` bash
find / -user level2 -perm -4520 2>/dev/null
```

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_13.png" | relative_url }}' alt='relative'>  

﻿-4520 조건으로 파일을 검색한다면, ① SetUID가 설정되어 있고 ② 소유자에게 read 권한이 있고 ③ 소유자에게 execute 권한이 있고 ④ 그룹 소유자에게 write 권한이 있는 파일이 출력됩니다.﻿

즉, ① AND ② AND ③ AND ④ == True를 만족하는 파일을 찾습니다.  
find 명령어 수행 결과 위 조건을 만족하는 파일이 없는지 아무 결과도 출력되지 않았습니다.


﻿﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_12.png" | relative_url }}' alt='relative'>  
﻿ExecuteMe 실행파일의 경우 user가 level2로 설정되어 있습니다.﻿

﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_14.png" | relative_url }}' alt='relative'>  

다만, ExecuteMe 파일 권한과 권한에 대한 검색 조건을 비교해본 결과 ①, ②, ③은 bit가 서로 일치하지만, ④의 경우 일치하지 않습니다.  
그래서 '-' 검색 조건(-4520)으로는 ① AND ② AND ③ AND ④ == False이기 때문에 ExecuteMe 파일이 검색되지 않습니다.  

* __'+'가 숫자 앞에 붙는 경우 OR 연산__

``` bash
find / -user level2 -perm +4520 2>/dev/null
```

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_15.png" | relative_url }}' alt='relative'>  
﻿
+4520 조건으로 파일을 검색한다면, ① SetUID가 설정되어 있거나 ② 소유자에게 read 권한이 있거나 ③ 소유자에게 execute 권한이 있거나 ④ 그룹 소유자에게 write 권한이 있는 파일이 출력됩니다.﻿

즉, ① OR ② OR ③ OR ④ == True를 만족하는 파일을 찾습니다.  
find 명령어 수행 결과 위 조건을 만족하는 ExecuteMe 파일의 경로가 출력되었습니다.
﻿

﻿﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_16.png" | relative_url }}' alt='relative'>  
﻿
ExecuteMe 파일과 검색 조건 권한을 비교해본 결과 ①, ②, ③은 bit가 서로 일치하지만, ④의 경우 일치하지 않습니다.  
그러나 '+' 검색 조건(+4520)은 ① OR ② OR ③ OR ④ == True이기 때문에 ExecuteMe 파일이 검색됐습니다.  
﻿
* __ 아무런 기호도 붙지 않은 경우 __

``` bash
find / -user level2 -perm 4520 2>/dev/null
```

﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_17.png" | relative_url }}' alt='relative'>  

아무 기호도 붙지 않은 경우 검색 조건과 일치하는 권한을 갖는 파일만 출력됩니다.  
파일의 권한을 표현하기 위해 총 12bit를 사용하는데 이 12bit가 검색 조건과 모두 일치하는 파일을 찾습니다.  

﻿﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_18.png" | relative_url }}' alt='relative'>  
﻿
ExecuteMe 파일과 검색 조건 권한을 비교해본 결과 소유자 write 권한과 그룹 소유자의 모든 권한이 일치하지 않습니다.  
권한을 나타내는 12bit 중 검색 조건(4520)과 일치하지 않은 bit가 있기 때문에 ExecuteMe 파일이 검색되지 않습니다.  


﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_19.png" | relative_url }}' alt='relative'>  

그러나 ExecuteMe 파일 권한과 똑같이 4750 권한으로 검색하니 ExecuteMe 파일이 검색된 것을 확인할 수 있습니다.  
권한을 나타내는 12bit 모두가 검색 조건(4750)과 일치하기 때문에 ExecuteMe 파일이 검색됐습니다.  

﻿﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_20.png" | relative_url }}' alt='relative'>  

## ﻿3) ExecuteMe 실행파일 확인  
﻿ExecuteMe 실행파일을 실행해보니 아래 사진과 같은 내용이 나옵니다.  

﻿﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_21.png" | relative_url }}' alt='relative'>  

레벨 2의 권한으로 원하는 명령어를 한 가지 실행시킬 수 있다고 합니다.  

﻿﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_22.png" | relative_url }}' alt='relative'>  


setuid가 설정된 파일은 프로그램이 실행되는 동안 파일의 uid 권한으로 모든 작업들이 실행됩니다.  
ExecuteMe 실행파일이 level2의 uid를 갖기 때문에 ExecuteMe 실행파일이 실행되는 동안 작업한 내용은 level2 권한으로 실행됩니다.  
그래서 whoami 명령어를 입력하니 level2를 출력합니다.  
그러나 입력을 받고 결과를 출력함과 동시에 ExecuteMe 프로그램은 종료합니다.  

ExecuteMe 실행파일을 실행시켜 동작을 확인해본 결과, 명령어를 딱 한 번만 입력할 수 있고, 입력한 명령어는 level2 권한으로 실행됨과 동시에 ExecuteMe 프로그램이 종료합니다.

## ﻿4) my-pass 명령어 확인  

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_23.png" | relative_url }}' alt='relative'>  
쓰인 대로 my-pass 명령어는 사용할 수 없습니다.

ExecuteMe 프로그램을 실행해본 결과 레벨 2의 권한으로 my-pass와 chmod 명령어를 제외한 딱 한개의 명령어를 실행할 수 있다는 것을 확인했습니다.

그렇다면 level1 권한으로 my-pass 명령어가 어떤 동작을 수행하는지 확인해보겠습니다.
<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_24.png" | relative_url }}' alt='relative'>  

my-pass 명령어를 실행해본 결과, 현재 로그인한 user의 비밀번호를 출력해 주는 실행파일입니다.  
level2 권한으로 my-pass 명령어를 실행하면 level2로 접속할 수 있는 비밀번호를 얻을 수 있습니다.  
하지만, ExecuteMe 실행파일에서 my-pass 명령어를 바로 사용할 수 없습니다.

그렇다면 level2 user의 비밀번호를 어떻게 얻을 수 있을까요?
﻿
## ﻿5) level2의 비밀번호 획득하는 방법  
ExecuteMe 실행파일이 level2의 uid를 갖기 때문에 ExecuteMe 실행파일이 실행되는 동안 작업한 내용은 level2 권한으로 실행됩니다.  
그래서 my-pass 명령어를 입력할 수 있다면 level2의 비밀번호를 획득할 수 있습니다.

하지만 ExecuteMe 프로그램은 my-pass 명령어 사용을 막아놨습니다.

vi 명령어나 touch 명령어를 사용해 my-pass 명령어를 실행시키는 프로그램을 만들고자 해도 파일을 쓸 권한이 없습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_25.png" | relative_url }}' alt='relative'>  

**위와 같은 과정으로 도출된 결과는 /bin 디렉터리에 있는 명령어만 실행할 수 있다는 것입니다.**

사용할 수 있는 명령어를 확인해보니 다음과 같습니다.  
ExecuteMe 프로그램을 실행시키고 입력할 수 있는 명령어는 아래 나열된 명령어만 가능합니다.

﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_26.png" | relative_url }}' alt='relative'>  
ExecuteMe 프로그램이 종료되지 않고 명령어를 계속 입력할 수 있는 방법은 level2 권한으로 shell을 실행하는 것입니다.

<span style="color:blue">**Shell을 실행하는 방법**</span>

1. sh 명령어 입력  
﻿﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_27.png" | relative_url }}' alt='relative'>  

2. bash 명령어 입력  
<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_28.png" | relative_url }}' alt='relative'>  

3. bash2 명령어 입력  
<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_29.png" | relative_url }}' alt='relative'>  


ExecuteMe 프로그램에서 shell을 실행하니 사용자가 level2로 변경되었습니다.  
shell은 exit 명령어를 입력하기 전까지는 종료되지 않습니다.  
그래서 level2 권한으로 원하는 명령어를 계속해서 실행할 수 있습니다.
﻿
## ﻿6) level2의 비밀번호 획득  
해당 shell에서 실행되는 명령은 로그인한 사용자 즉, level2 권한으로 실행됩니다.  

my-pass 명령어는 현재 로그인한 사용자의 비밀번호를 출력하는 명령어입니다.  
level2의 비밀번호를 얻기 위해 my-pass 명령어를 입력합니다.  

﻿﻿<img data-action="zoom" src='{{ "assets/ftz/level1/ftz_level1_30.png" | relative_url }}' alt='relative'>  