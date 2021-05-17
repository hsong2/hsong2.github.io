---
layout: post
title: "Hacker School FTZ - Level 12"
description: "Do you know the ?"
comments: true
categories: ftz
---

<img data-action="zoom" src='{{ "assets/ftz/level12/1.jpg" | relative_url }}' alt='relative'>  

## 1) level12/it is like this 입력해 로그인  

<img data-action="zoom" src='{{ "assets/ftz/level10/2.png" | relative_url }}' alt='relative'>  

``` c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main( void )
{
        char str[256];

        setreuid( 3093, 3093 );
        printf( "문장을 입력하세요.\n" );
        gets( str );
        printf( "%s\n", str );
}
```