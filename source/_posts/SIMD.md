---
title: MFC SIMD Vector Class 사용법 정리
date: 2013-06-25 15:28:00
categories:
- CPP
- MFC
tags:
- CPP
- SIMD
- MFC
---

- Header File : `<dvec.h>`


## SIMD Vector Class 명명법

```
 <Type><Signed><Bits>vec<Nums>
{ F | I } { s | u } { 64 | 32 | 16 | 8 } vec { 8 | 4 | 2 | 1 }
```
- F : 실수 , I : 정수

- s : signed, u : unsigned ( I에만 사용됨)

- 64 : double, __int64 , 32 : float, int , 16: short , 8 : char

- 8,4,2,1 : pack 개수 (Bits x Nums 는 128을 넘어 갈 수 없다. )

```
Ex) unsigned int 4개pack class : Iu32vec4 , float 4개 packclass : F32vec4
```
 
## 초기화

- 괄호 안에 각 변수들을 지정
```
Ex) Is32vec4 pint4(10,20,30,40); 
```
### 1. 연산자 ( vec = vec op vec)

```
    = : 대입
+, += : 덧셈
- , -= : 뺄셈
* , *=  : 곱셈
/ , /+ : 나눗셈 (실수 연산만 가능)
& : 논리연산 AND
| : 논리연산 OR
^ : 논리연산 XOR  
```

### 2. 연산자 (vec = vec op n ) 

```
<<, <<== : Left Shift ( x2 )
>> , >>== : Right Shift ( /2 ) 
```

### 3. 함수 (vec = func(vec, vec) 

```
simd_min() : 최소값
simd_max() : 최대값
andnot() : 논리연산 And Not
```

### 4. 비교함수 ( 참이면 0xff…, 거짓이면 0 )

```
cmpeq : ==
cmpneq : !=
cmpgt : >
cmpge : >=
cmplt : <
cmple : <=
cmpnlt : !(A < B)
cmpnle : !(A <= B)
cmpngt : !(A > B)
cmpnge : !(A >= B)
```

### 5. Select : R = Select(A,B,C,D)=> if (A 비교 B) R = C else R = D

```
select_eq : ==
select_neq : !=
select_gt : >
select_ge : >=
select_lt : <
select_le : <=
select_nlt : !(A < B)
select_nle : !(A <= B)
select_ngt : !(A > B)
select_nge : !(A >= B)
```

### 6. Unpacked / Pack (예제는 Is32vec4)

```
R = unpack_low( A, B) => R = ( B1 , A1 , B0 , A0 )
R = unpack_high(A, B) => R = ( B3 , A3 , B2 , A2 )
R = pack_sat(A, B) => Is16vec8 R = (B3, B2, B1, B0, A3, A2, A1, A0) : 정수형만지원
```

### 7.정수 Vector <-> Array :Vector Class에서 지원하지 앟음 intrinsic 함수 이용

```
Is32vec4 A = _mm_load_si128((__m128i*) p) : 배열 Pointer를 __m128i* 로 casting 후 저장  
                                                            (정렬된 메모리)

s32vec4 A = _mm_loadu_si128((__m128i*) p) : 정렬되지 않은 메모리에서 데이터 읽어오기
_mm_store_si128((_m128i*) p , A) : A에 저장된 값들을 배열 p에순서대로 저장
                                            (정렬된 메모리)
_mm_storeu_si128((_m128i*) p , A) : 정렬되지 않은 메모리에 데이터 쓰기
```


### 8. 실수 Vector <-> Array

```
loadu (F32vec4 R, float* a) : a[0] ~ a[3] 4개의 float 값을읽어서 R에 저장
storeu(float* a, F32vec4 R) : R의 값을 a[0] ~ a[3]에저장
store_nta(float* a, F32vec$ R) : 버퍼 없이 메모리 a[0] ~ a[3]에저장
```


### 9.수학 함수 (실수 연산만 가능)

```
R = sqrt(A) : 제곱근 Root
R = rcp(A) : 역분수 : R = 1 / A
R = rsqrt(A) : 제곱근 역분수 : R = 1 / Root A
R = rcp_nr(A) : Newton-Raphson법의 역분수 : R = rcp(A) * 2 –A * rcp(A) * rcp(A)
R = rsqrt_nr(A) : Newton-Raphson법의 제곤근 역분수 : R = rsqrt(A)/ 2 * ( 3 – A * rsqrt(A) * rsqrt(A) )
F = add_horizontal(A) : float F = A0 + A1 + A2 + A3
```