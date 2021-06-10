---
layout: post
title: "Hacker School FTZ - Level 20"
description: "Do you know the ?"
comments: true
categories: FTZ
---

<img data-action="zoom" src='{{ "assets/ftz/level20/1.jpg" | relative_url }}' alt='relative'>  

## 1) level20/we are just regular guys 입력해 로그인  

``` c 
#include <stdio.h>
main(int argc,char **argv)
{ char bleh[80];
  setreuid(3101,3101);
  fgets(bleh,79,stdin);
  printf(bleh);
}
```
 
<img data-action="zoom" src='{{ "assets/ftz/level20/2.png" | relative_url }}' alt='relative'>  