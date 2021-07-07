---
layout: post
title: "The Lord of the BOF - Level 17"
comments: true
categories: [LOB]
tags: [LOB]
---
<img data-action="zoom" src='{{ "assets/lob/level17/1.jpg" | relative_url }}' alt='relative'>  

## 1) zombie_assassin/no place to hide 입력하여 로그인하기  

<img data-action="zoom" src='{{ "assets/lob/level17/2.png" | relative_url }}' alt='relative'>  

``` c
/*
        The Lord of the BOF : The Fellowship of the BOF
        - succubus
        - calling functions continuously
*/

#include <stdio.h>
#include <stdlib.h>
#include <dumpcode.h>

// the inspector
int check = 0;

void MO(char *cmd)
{
        if(check != 4)
                exit(0);

        printf("welcome to the MO!\n");

        // olleh!
        system(cmd);
}

void YUT(void)
{
        if(check != 3)
                exit(0);

        printf("welcome to the YUT!\n");
        check = 4;
}

void GUL(void)
{
        if(check != 2)
                exit(0);

        printf("welcome to the GUL!\n");
        check = 3;
}

void GYE(void)
{
        if(check != 1)
                exit(0);

        printf("welcome to the GYE!\n");
        check = 2;
}

void DO(void)
{
        printf("welcome to the DO!\n");
        check = 1;
}

main(int argc, char *argv[])
{
        char buffer[40];
        char *addr;

        if(argc < 2){
                printf("argv error\n");
                exit(0);
        }

        // you cannot use library
        if(strchr(argv[1], '\x40')){
                printf("You cannot use library\n");
                exit(0);
        }

        // check address
        addr = (char *)&DO;
        if(memcmp(argv[1]+44, &addr, 4) != 0){
                printf("You must fall in love with DO\n");
                exit(0);
        }

        // overflow!
        strcpy(buffer, argv[1]);
        printf("%s\n", buffer);

        // stack destroyer
        // 100 : extra space for copied argv[1]
        memset(buffer, 0, 44);
        memset(buffer+48+100, 0, 0xbfffffff - (int)(buffer+48+100));

        // LD_* eraser
        // 40 : extra space for memset function
        memset(buffer-3000, 0, 3000-40);
}
```

