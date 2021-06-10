---
layout: post
title: "Hacker School FTZ - Level 10"
description: "Do you know the shared memory?"
comments: true
categories: [FTZ]
tags: [FTZ]
---

<img data-action="zoom" src='{{ "assets/ftz/level10/1.jpg" | relative_url }}' alt='relative'>  

## 1) level10/interesting to hack! 입력해 로그인  

<img data-action="zoom" src='{{ "assets/ftz/level10/2.png" | relative_url }}' alt='relative'>  

공유 메모리를 이용해 비밀스런 대화를 나누고 있다고 합니다. 공유 메모리에 접근할 key를 제공해준 것으로 보입니다.  
해당 키를 사용해 공유 메모리에 접근하고 어떤 대화를 나누는지 알아보겠습니다.  
(추측하건데 level11의 비밀번호를 공유 메모리에 올려놓지 않았을까 합니다.)  

## 2) 공유 메모리 확인하기    

<img data-action="zoom" src='{{ "assets/ftz/level10/3.png" | relative_url }}' alt='relative'>  

키가 7530이고 공유 메모리 크기가 1028인 프로세스를 하나 발견했습니다.  

<img data-action="zoom" src='{{ "assets/ftz/level10/4.png" | relative_url }}' alt='relative'>  

공유 메모리에 대한 정보를 자세히 확인해보니 위의 그림과 같습니다.  

## 3) 공유 메모리 접근하는 프로그램 만들기  

공유 메모리 정보를 바탕으로 공유 메모리에 접근하는 프로그램을 만들었습니다.  

``` c

#include <sys/shm.h>
#include <sys/ipc.h>
#include <string.h>
#include <stdio.h>
#define KEY_t 7530
#define MEM_SIZE 1028
int main(void)
{
        int shmid;
        char* shmem;

        shmid = shmget((key_t)KEY_t, MEM_SIZE, 0666);
        shmem = shmat(shmid, (void *)0, 0);
        printf("%s\n", shmem);
        shmdt(shmem);

        return 0;
}

```

<img data-action="zoom" src='{{ "assets/ftz/level10/5.png" | relative_url }}' alt='relative'>  


위의 소스코드에서 쓰인 함수는 공유 메모리를 사용하기 위한 함수들입니다.  

### 0) 공유 메모리를 사용하기 위해 필요한 헤더 파일

``` c
#include <sys/shm.h>
#include <sys/ipc.h>
```

### 1) 인자로 넘겨주는 key 값과 일치하는 key를 가진 공유 메모리의 id 획득  

``` c
int shmget(key_t key, size_t size, int shmflg);
```

공유 메모리를 할당하면 고유 key값이 생깁니다. key값으로 특정 공유 메모리에 접근할 수 있습니다.   

### 2) 공유 메모리 영역을 프로세스의 메모리 영역에 attach   

``` c
void *shmat(int shmid, const void *shmaddr, int shmflg);
```

### 3) 프로세스 메모리 영역과 공유 메모리 영역 detach  

``` c
int shmdt(const void *shmaddr);
```


## 4) 공유 메모리에 접근하기  

<img data-action="zoom" src='{{ "assets/ftz/level10/6.png" | relative_url }}' alt='relative'>  

공유 메모리에 접근하는 프로그램을 만들어 공유 메모리에 올려진 내용을 읽어왔습니다.  
해당 내용을 통해 level11의 비밀번호를 획득했습니다.   