---
layout: post
title: SQL Injection 설명과 풀이 ①
comments: true
---



*WebGoat의 (A1) Injection메뉴 중 (Intro)에 대한 설명과 풀이를 다룬다.*





# 1. 설명





이번 메뉴에선 SQL이 무엇인지, 그리고 DML, DDL과 DCL등의 데이터 조작어, 질의어, 제어어를 포함한 SQL을 이용한 다양한 삽입공격에 대한 실습을 해본다.

이 중 가장 많이 사용되는 종류는 DML (Data Manipulation Language) 이다. 이는 SELECT, INSERT, CREATE, UPDATE, DELETE 등을 포함한다.

***단, 시중의 DBMS가 모두 동일한 명령어를 사용하는 것은 아니다.***

이번 장에서도 역시 CIA(기밀성, 무결성 그리고 가용성)는 중요한 가치이다.



보통 SQL Injection 공격의 목적은 아래와 같다.

- Root 권한 획득 - 권한 탈취
- 인증 우회
- 데이터 변조 혹은 탈취

이에 대한 보통의 대응 방법은 'Input validation(입력값 검증)' 이 쓰인다.





# 2. (A1) Injection - 문제들

## - SQL Injection (intro)

해당 페이지에는 직접 풀이해 볼 수 있는 9가지의 문제를 다룬다.



### 첫 번째 문제,

단순히 Bob Franco가 속한 부서의 정보를 SQL Query로 조회하는 문제이다.

아주 기본이지만, 이 기본이 되어야 SQL 구문사용이 가능하다.

![IMAGE 002](https://user-images.githubusercontent.com/52769104/103758399-7c225880-5055-11eb-9e74-d3701b1c1e8a.png)



 **풀이 방법은 다음과 같다.**

```
1. Select, From, Where 문 사용
2. Where에는 first_name='Bob' 만 지정해도 됨.
```

![IMAGE 003](https://user-images.githubusercontent.com/52769104/103758401-7d538580-5055-11eb-84f4-e428982f7059.png)



### 두 번째 문제, 

### DML (Data Manipulation Language)에 대한 문제이다.

위에 설명했듯 가장 많이 사용되는 SQL 언어 종류이고, 문제 1번의 내용을 포함하고 있다.

**Tobi Barnett이 속한 부서명을 바꾸면 된다.**

![IMAGE 001](https://user-images.githubusercontent.com/52769104/103758433-89d7de00-5055-11eb-9214-b8dd480a5e50.png)



 **풀이 방법은 다음과 같다.**

```html
1. Select, From, Where, Update (+set) 문 사용
2. Where에는 first_name='Tobi' 만 지정해도 됨.
3. Update에서 다시 테이블명을 지정해준 후, 부서명을 바꿔줌.
```



![IMAGE 004](https://user-images.githubusercontent.com/52769104/103758426-88a6b100-5055-11eb-8b21-e5eff14add51.png)







### 세 번째 문제, DDL (Data Definition Language) 에 대한 설명이다.

데이터 정의어라고도 불리는 이 부분은 테이블의 생성, 수정, 삭제 등을 다룬다.

**employees 테이블에 phone (varchar(20)) Column을 추가하면 된다.**



![IMAGE 005](https://user-images.githubusercontent.com/52769104/103758459-95c3a000-5055-11eb-8753-023fa3e66d42.png)





 **풀이 방법은 다음과 같다.**

```html
1. Alter (+add) 문 사용
2. table은 employees 지정
```



![IMAGE 006](https://user-images.githubusercontent.com/52769104/103758463-96f4cd00-5055-11eb-99d5-5f8b9f338b31.png)



### 네 번째 문제, DCL (Data Control Language) 에 대한 설명이다.

데이터 설정어는 주로 사용자의 권한 부여, 수정, 철회와 관련이 있다.,

 **usergroup "UnauthorizedUser"에 권한을 부여하는 문제이다.**



![IMAGE 007](https://user-images.githubusercontent.com/52769104/103758466-96f4cd00-5055-11eb-81ac-085fd89f8e2c.png)



 **풀이 방법은 다음과 같다.**

```html
1. Grant, Alter (+to) 문 사용
2. table 전체 지정, to 이후 권한 부여할 계정 지정
```



![IMAGE 008](https://user-images.githubusercontent.com/52769104/103758468-978d6380-5055-11eb-8567-3269a9deb718.png)



### 다섯 번째 문제, 본격적으로 SQL Injection 공격 실습이다.

기본적으로 SQL Injection 공격은 String 인젝션과 Numeric 인젝션으로 구분 할 수 있다.

**이번 문제는 String SQL Injection을 이용**하여 
**사용자 테이블에서 모든 사용자를 검색하는 문제이다.** 
문제 해결을 위해 특정사용자의 이름을 알 필요는 없다.



![IMAGE 009](https://user-images.githubusercontent.com/52769104/103758469-978d6380-5055-11eb-81ae-118c0ab9156d.png)

 **풀이 방법은 다음과 같다.**

```html
1. '1'='1' 은 항상 True 이다.
2. SELECT * FROM user_data WHERE first_name = 'John' and last_name = '' or TRUE 형식이다.
```

결국, or 앞에 조건이 어떻게 되었던 간에 행을 한줄 한줄 비교하며 전체 True로 만둘어버리기 때문에 모든 행의 출력이 가능해진다.

**이와 비슷한 해결 방법으로는 1=1 -- 등의 주석처리를 행하는 방법이 있다.**

![IMAGE 010](https://user-images.githubusercontent.com/52769104/103758470-9825fa00-5055-11eb-9d5d-17747c175508.png)





### 여섯 번째 문제, Numeric SQL Injection 실습이다.

ID 값, Count 값 등을 담고있는 칼럼 명에 숫자를 대입하는 방법으로 주로 사용된다.

![IMAGE 011](https://user-images.githubusercontent.com/52769104/103758472-9825fa00-5055-11eb-9e5d-36b0af0a69a8.png)



 **풀이 방법은 다음과 같다.**

```html
1. '1'='1' 은 항상 True 이다.
2. 임의의 수 or Ture 가 포인트 이다.
```



![IMAGE 012](https://user-images.githubusercontent.com/52769104/103758474-98be9080-5055-11eb-9f8d-f5f034b681a5.png)



### 일곱 번째 문제, 기밀성 손상 !

### SQL Injection을 통한 인증 TAN 우회 공격이다.

공격의 목표는 다음과 같다.

1. 직원들의 급여를 보고 싶다
2. 이를 위해 employees 테이블에서 모든 직원 데이터를 검색한다.

![IMAGE 013](https://user-images.githubusercontent.com/52769104/103758476-98be9080-5055-11eb-9979-26d2197c6953.png)



**풀이 방법은 다음과 같다.**

```html
1. 이미 '" + auth_tan "'은 싱글쿼터로 감싸져 있다.
2. '글자' or '1' = '1'; 형태가 와야 한다.
3. 앞에 어떤 것이 오던 무조건 True만 되면 된다.
```



**아래의 해결 방법이 정석적인 방법이 아님을 염두해두시길 바란다 !**



![IMAGE 014](https://user-images.githubusercontent.com/52769104/103758478-99572700-5055-11eb-8cd2-7e776d701f27.png)





### 여덟 번째 문제, 무결성 손상!

### Query Chaining 을 이용해 Query 추가로 존재하는 DB의 내용 바꾸기

우리의 John 은 자신의 급여를 테이블 상의 직원 중 가장 높게 바꾸고 싶어한다.

이를 위해 DML 을 사용해 바꿔준다.



![IMAGE 015](https://user-images.githubusercontent.com/52769104/103758480-99572700-5055-11eb-8e8f-71a147396f91.png)



**풀이 방법은 다음과 같다.**

```html
1. 위 쿼리에 이어서 SQL문을 작성한다고 생각.
2. Employee Name 에 Update, (+Set), Where 을 사용한다.
3. TAN은 주석처리 해준다.
4. 이후 TAN에 아무 값이나 넣어도 OK!
```

**Smith'; update employees set salary = 100000 where last_name = 'Smith';--** 이 방식으로 해결했다.

![IMAGE 016](https://user-images.githubusercontent.com/52769104/103758482-99efbd80-5055-11eb-9554-3fda36cb4ca4.png)



### 아홉 번째 문제, 가용성 손상! 

### 계정손상 혹은 로그삭제로 SQL Injection을 실행한다.

급여테이블 조작을 성공한 John을 위해 access_log를 삭제시키는 실습이다.

이 역시 Query Chaining을 이용해 삭제하면 된다.



![IMAGE 017](https://user-images.githubusercontent.com/52769104/103758485-9a885400-5055-11eb-9097-6c9d3c4004da.png)



**풀이 방법은 다음과 같다.**

```html
1. 위 쿼리에 이어서 SQL문을 작성한다고 생각.
2. 테이블 명은 access_log 이다. 
3. Drop, 주석을 사용한다.
```



 **1' drop table access_log --** 이 방식으로 해결했다.



![IMAGE 018](https://user-images.githubusercontent.com/52769104/103758486-9a885400-5055-11eb-81a4-b2e61afb888b.png)



## 여기까지 잘 진행했으면 SQL Injection (intro) 메뉴가 초록색 체크박스로 채워져 있을 것이다.

![IMAGE 019](https://user-images.githubusercontent.com/52769104/103758487-9b20ea80-5055-11eb-80ac-8d1f853a66da.png)
