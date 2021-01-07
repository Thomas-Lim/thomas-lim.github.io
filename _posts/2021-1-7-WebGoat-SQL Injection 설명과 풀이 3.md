---
layout: post
title: WebGoat SQL Injection 설명과 풀이 3
comments: true
---



*WebGoat의 (A1) Injection메뉴 중 (mitigation)에 대한 설명과 풀이를 다룬다.*



# 1. 설명 



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







### 첫 번째 문제, Java Code로 safe code를 만들어 보기

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

2. 첫 번째 문제의 해답과 연결
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
