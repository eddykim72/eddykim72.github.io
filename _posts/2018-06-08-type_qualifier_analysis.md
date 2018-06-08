---
title: "복합적인 형식 한정자(type qualifier) 구문 분석 방법"
categories:
  - posts
tags:
  - C
  - type qualifier
  - const
  - volatile
---



복잡한 타입 한정자(type qualifier)로 구성된 구문을 분석 규칙 및 방법



```
[규칙] 우측에서 좌측으로 영어로 읽어 내려 가자 !
```



int x;	// variable x is integer type. ( 변수 x  integer 타입이다.)

int * x; 	

// variable x points to integer type (variable x points to ==  x pointer variable)

// => 



1. const

   ```c
   const int x;    // <1> : x variable is interget type which is constant.
   int const x;    // <2> : x varialbe is constant integer type.
   ```

   

2. volatile

3. const & volatile

4. with pointer