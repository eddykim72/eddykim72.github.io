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



> ## [규칙]  
>
> ## 우측에서 좌측으로 영어로 읽어 내려 가자 !



```c
int x;		// <1>
// <---		; x variable is integer type.
int * x; 	// <2>
// <---		; x variable points to integer type.
```



<1> 

x 변수는 정수형이다. = 변수 x는 정수형이다.

<2>

x 는 정수형을 가리키는 변수이다. = x는 정수형을 가리키는 포인터 변수이다.

x variable points to  => x is pointer (variable)





1. ## const

   ```c
   const int x;    // <1> : x variable is interget type which is constant.
   int const x;    // <2> : x varialbe is constant integer type.
   ```

   <1>

   변수 x는 정수형인데 상수이다. == 변수 x는 상수형 정수형이다.

   <2>

   변수 x는 상수 정수형이다.

   --> <1>과 <2> 구문은 동일한 구문이다.

   

2. ## volatile

   ```c
   volatile int x;	// <1> 
   int volatile x;	// <2>
   ```

   

3. ## const & volatile

4. with pointer