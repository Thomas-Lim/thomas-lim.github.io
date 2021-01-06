---
layout: post
title: WebGoat SQL Injection 설명과 풀이 ②
comments: true
---



*WebGoat의 (A1) Injection메뉴 중 (advanced)와 (mitigation)에 대한 설명과 풀이를 다룬다.*





# 1. 설명





이번 메뉴에선 좀 더 심화된 SQL Injection 기법을 다룬다.

일반적으로 더 이상 '1' = '1' 등은 통하지 않는다.

많이 사용되는 것이 Query Chaining을 통한  Inner/Outer Join 속성 결합 또는 Union 속성과의 결합이다.

문자열이 필터링 되는 경우 char(97)등의 asciicode를 이어붙이는 경우도 있다. 
**그렇다면 ''+(char97)+'pple'은 'apple'이 되는 것이다.**

이와 함께 Blind SQL Injection을 다룬다.





# 2. (A1) Injection - 문제들 ②

## - SQL Injection (advanced)

해당 페이지에는 직접 풀이해 볼 수 있는 3가지의 문제를 다룬다.



### 첫 번째 문제,

두 가지 Table을 결합해 Dave의 password를 알아내는 것이 문제이다.

Union 속성을 사용해서 풀이할 수 있다. 

**Union은 이 구문을 기준으로 좌측의 table 열 갯수와 우측의 table 열 갯수가 동일해야지만 올바르게 작동한다.**

![IMAGE 001](https://user-images.githubusercontent.com/52769104/103781133-41cab280-5079-11eb-81b5-a1baaa3309db.png)

 **풀이 방법은 다음과 같다.**

```
1. SELECT * FROM user_data WHERE last_name = '' 까지가 입력되어 있다고 가정. 
2. 이 이후에 '; 부터 select문을 이어나간다.
3. user_system_data의 칼럼 갯수를 user_data 테이블과 맞춰주기 위해 임의의 열 이름을 SQL에 추가 (데이터 형식에 유의)
```

해당 문제 풀이에 사용된 SQL문은 다음과 같다.

**'; SELECT * FROM user_system_data;-- or ' UNION SELECT *, user_name, password, cookie, 'A', 'B', 2 from user_system_data;--**



![IMAGE 002](https://user-images.githubusercontent.com/52769104/103781138-42fbdf80-5079-11eb-8594-e8ce467bf3b4.png)





### 두 번째 문제, 

### Blind SQL Injection 문제이다.

간단하게 생각해서, 일반적인 SQL Injection은 실패 시 공격대상에서 어떤 오류를 뿜는다. 이를 바탕으로 어느정도 추측 할 수 있다.

하지만 Blind SQL Injection 은 DB에 True or False로 돌아오는 답변을 통해 끊임없이 유추하여 시도하고 성공시키는 공격기법이지만, 일반적으로 공격 실패시에도 사이트 내에서 어떤 변화가 없거나 공격성공 단서를 유추할 수 없는 에러만 나오는 경우 사용한다.

풀어야 할 문제는 **Tom의 계정으로 로그인 하기** 이다.



![IMAGE 003](https://user-images.githubusercontent.com/52769104/103781159-4d1dde00-5079-11eb-82ca-173320e7fc08.png)



## ※ 이번 문제는 상당히 난이도 있다! 

### 준비물 : Burp Suite





 **풀이 방법은 다음과 같다.**

```html
1. tom 계정 존재 여부 확인
2. REGISTER 메뉴에서 tom' and '1'='1 등의 것으로 에러메시지 확인
  -> tom' and 조합으로 존재 여부 확인 (length(password)>=23--, 
     substring(password,1,1)='a'-- 등등)
3. Burp Suite 프록시 상태에서 Intercept is on 하여 PUT 메소드의 패킷 잡아내기
4. Send to Intruder 로 전송해 무차별 대입 공격 준비.
5. 전송된 Payloads 부분에 변경 될 부분 선택
6. 변경 범위에 대입 순서, Brute Force 방법 선택
7. 공격 시도 후 Intruder attack 공격화면에서 Length 값이 다른 것의 Response 확인
8. 해당 Password 조합으로 로그인.
```



**LOGIN 화면에서는 어떠한 답도 알아낼 수 없다. REGISTER 부분을 집중 공략해야 한다.**

**생성화면에서 <u>User {0} already exists please try to register with a different username.</u> 이 공격 성공의 HINT!**



**아래 사진은 'tom' 계정의 존재여부 확인**

![IMAGE 006](https://user-images.githubusercontent.com/52769104/103781180-55761900-5079-11eb-9ea6-14b36438225c.png)



![IMAGE 007](https://user-images.githubusercontent.com/52769104/103781218-645ccb80-5079-11eb-8c7a-5dba8fa16d00.png)

![IMAGE 009](https://user-images.githubusercontent.com/52769104/103781220-64f56200-5079-11eb-9523-b350013b1a77.png)

![IMAGE 012](C:\Users\ultrarisk\Pictures\IMAGE 012.png)![IMAGE 010](https://user-images.githubusercontent.com/52769104/103781237-6d4d9d00-5079-11eb-91e4-3a10c8306f1f.png)

<u>**다음과 같이 ~~ created 또한 힌트이다.**</u> 이렇게 에러 메세지를 보면서 공격 힌트를 좁혀 나갈 수 있다.

![IMAGE 008](https://user-images.githubusercontent.com/52769104/103781281-78083200-5079-11eb-9ca0-8c574e651922.png)

![IMAGE 011](https://user-images.githubusercontent.com/52769104/103781283-78a0c880-5079-11eb-96ae-89aa330b328b.png)





이후 Burp Suite 에서 캡쳐 된 PUT 메소드의 패킷이다. 이것으로 Brute Forcing 할 준비를 한다.

**두 번째 사진의 초록생 부분이 실제 변경이 이뤄지며 대입될 부분이다.** 
**우측의 ADD$로 설정 가능하다.**

![IMAGE 004](https://user-images.githubusercontent.com/52769104/103781315-82c2c700-5079-11eb-8929-d3cfef6c2e05.png)

![IMAGE 005](https://user-images.githubusercontent.com/52769104/103781316-835b5d80-5079-11eb-802e-ff89a13d46a9.png)



Intruder의 Payload 는 다음과 같이 설정한다. 준비 후 **Start Attack**을 누른다. 무료 버전은 기능제한이 있다는 메세지는 덤. 실제로 엄청 느리게 진행된다.

![IMAGE 013](https://user-images.githubusercontent.com/52769104/103781350-91a97980-5079-11eb-8ffd-29cb895a03ed.png)

![IMAGE 014](https://user-images.githubusercontent.com/52769104/103781353-92421000-5079-11eb-89f2-4f47629c3a32.png)

![IMAGE 015](https://user-images.githubusercontent.com/52769104/103781355-92daa680-5079-11eb-9e79-fbb3e8e86871.png)





실제로 이와같이 Length가 다른 패킷의 Response를 조사해 보면, 
반가운 힌트인 already exists를 메세지로 대응해준다.

![IMAGE 016](https://user-images.githubusercontent.com/52769104/103781379-9ec66880-5079-11eb-8643-6129a4a813bc.png)





해결된 화면 !!! 무료 버전 기준 1시간 넘게 걸렸다......

![IMAGE 017](https://user-images.githubusercontent.com/52769104/103781391-a4bc4980-5079-11eb-9580-145a9e93a880.png)



### 세 번째 문제, 

### 지금까지 학습한 SQL Injection에 대한 지식테스트이다. 

5문제로 이루어져 있다.



 **풀이 방법은 다음과 같다.**

```html
1. 4개의 문항 중 알맞은 것을 선택 !
```



![IMAGE 018](https://user-images.githubusercontent.com/52769104/103781406-abe35780-5079-11eb-9359-7522922d799f.png)





## - SQL Injection (mitigation)



이번 장은 SQL Injection `공격의 완화기법`에 대해 다룬다. 

Immutable Queries는 상당히 효과적인 SQL Injection 방어 기법이다. Static Query를 다은과 같이 사용한다거나

```
SELECT * FROM users WHERE user = "'" + session.getAttribute("UserID") + "'";
```

`Parameterized Query` 로 사용해 예방할 수 있다.

```
String query = "SELECT * FROM users WHERE last_name = ?";
PreparedStatement statement = connection.prepareStatement(query);
statement.setString(1, accountName);
ResultSet results = statement.executeQuery();
```

Parameterized Query 는 Java 코드 등과도 결합해 사용 가능하다.

이 외에도 DB에 연결 된 Application 별로 Least Privilege 할당이 좋은 예방방법이 된다.







### 네 번째 문제, Java Code로 safe code를 만들어 보기

![IMAGE 020](https://user-images.githubusercontent.com/52769104/103781439-b69dec80-5079-11eb-8eef-a2a804605660.png)





 **풀이 방법은 다음과 같다.**

```html
1. DriverManager 클래스에 정의된 static 메서드인 getConnection() 사용
2. Connection 객체의 prepareStatement()라는 메서드를 사용
3. Both the name and the mail are in the string format
```



![IMAGE 019](https://user-images.githubusercontent.com/52769104/103781453-bb62a080-5079-11eb-9795-524a4b3470de.png)





### 다섯 번째 문제, Java.sql 관련된 Hard Coding 문제이다.

위의 문제와 연관되며, 빈칸채우기를 손으로 직접 코딩한다.



![IMAGE 021](https://user-images.githubusercontent.com/52769104/103781476-c1f11800-5079-11eb-9d32-23af56582b3b.png)

**풀이 방법은 다음과 같다.**

```html
1. 예문 활용

try {
    Connection conn = null;
    System.out.println(conn);   //should output 'null'
} catch (Exception e) {
    System.out.println("Oops. Something went wrong!");
}

2. 네 번째 문제의 해답과 연결
```



![IMAGE 022](https://user-images.githubusercontent.com/52769104/103781492-c87f8f80-5079-11eb-9b8f-c71808629b6e.png)





### 여섯 번째 문제, Order By를 활용한 취약점 실습이다.

<u>**문제의 목표는 webgoat-prd  라는 호스트의 아이피를 알아내는 것이다.**</u>

<u>**이를 위해 최소한의 아이피를 알려주고 있다. xxx.130.219.202**</u>



Order By는 CASE문과 결합해 Blind SQL Injection에서 IF문과 비슷한 용도로 사용가능하다.

Order By는 오름차순 정렬이 기본값이며, 주입을 막기 위해 만약 어떤 열을 정렬해야 하면 정렬되는 값을 검증하기 위해 화이트리스트 기반으로 구현해야 한다.



![IMAGE 033](https://user-images.githubusercontent.com/52769104/103781503-cddcda00-5079-11eb-976b-3e9484073392.png)



**이번에도 역시 Burp Suite 가 사용되며, `Repeater` 라는 새로운 기능을 사용한다.**

> **Repeater는 Intercept is on과 비슷한 역할을 하지만, 다음 행동에 대해 반복적으로 테스트해볼 수 있다는 점에서 차이가 있다.**



 **풀이 방법은 다음과 같다.**

```html
1. Burp Suite 프록시 상태로 준비하기
2. 문제에서 각종 Sorting 기능 활용해보기
3. 캡쳐 된 패킷을 Send to Repeater 하기
4. GET 메소드에서 칼럼명의 파라미터 값들을 CASE문 사용해 변환
5. Send 버튼을 눌러 오름차순으로 정렬되는지 여부 확인 (오름차순 정렬이 공격성공)
```

**<u>CASE 문은 Order By 절에서 SQL Injection 공격 시 함께 사용한다.</u>**

CASE문은 정렬로 인한 참과 거짓에 대한 반응을 직접 확보하여 공격에 쓰이도록 한다.

문법은 다음에 나오는 사진에서 확인 가능하다.



Burp Suite 가동 상태에서 칼럼의 Order 기능 사용시 Parameter 값이 칼럼명으로 잡힌다.

![IMAGE 023](https://user-images.githubusercontent.com/52769104/103781525-d46b5180-5079-11eb-8d8b-33a76db40ce8.png)



이렇게 넘어온 패킷을 Send to Repeater로 넘긴다.

![IMAGE 024](https://user-images.githubusercontent.com/52769104/103781527-d59c7e80-5079-11eb-8133-99a440475170.png)

![IMAGE 025](https://user-images.githubusercontent.com/52769104/103781528-d6351500-5079-11eb-8889-77487ddd18f0.png)



이후 CASE 문법을 활용해 Response 값이 오름차순으로 정렬되는지 확인한다.

![IMAGE 026](https://user-images.githubusercontent.com/52769104/103781576-e3ea9a80-5079-11eb-95b3-1e49cd6e5afc.png)

![IMAGE 027](https://user-images.githubusercontent.com/52769104/103781583-e51bc780-5079-11eb-9cad-2679283cc726.png)

![IMAGE 028](https://user-images.githubusercontent.com/52769104/103781585-e5b45e00-5079-11eb-93a8-0ef76f5d181c.png)





최종적으로 알아낸 IP를 입력 후 문제를 해결

![IMAGE 030](https://user-images.githubusercontent.com/52769104/103781632-f49b1080-5079-11eb-8516-e9a9171090bd.png)



## 여기까지 잘 진행했으면 SQL Injection 메뉴의 모든 부분이 초록색 체크박스로 채워져 있을 것이다.

![IMAGE 032](https://user-images.githubusercontent.com/52769104/103781663-fc5ab500-5079-11eb-9697-5127cc6cae5c.png)
