---
title:  "MySQL Regular Expression"
excerpt: "MySQL 정규표현식을 대하여 설명합니다."

categories:
  - MySQL
tags:
  - MySQL
  - Regular Expression
  - DATABASE
last_modified_at: 2020-01-22T09:48:00
---

MySQL은 데이터의 특정 패턴을 검색하기 위하여 다음과 같은 패턴 매칭 연산자를 제공한다.
1. Like
2. REGEXP

또한, 임의의 문자나 문자열을 대체하기 위하여 와일드카드(Wildcard) 문자를 사용할 수 있다.

**Like**
~~~
* like 연산자는 특정 패턴을 포함하는 데이터를 찾기 위해 사용한다.

SELECT * FROM table WHERE column LIKE '%blahblah%';
- '%'는 0개 이상의 문자라는 의미의 Wildcard 문자다. 
- 반대의 경우 NOT LIKE 사용
~~~

**Wildcard**
~~~
1. % : 0개 이상의 문자를 대체
2. - : 1개의 문자를 대체
~~~

**REGEXP**
~~~
LIKE 연산자보다 더욱 복잡한 패턴을 검색하고 싶을 때 사용

SELECT * FROM table WHERE column REGEXP 'blah|blahblah$blah';
~~~

1. `.`  줄바꿈 문자를 제외한 임의의 한 문자 
2. `*` 해당 문자 패턴이 0번 이상 반복됨 
3. `+` 해당 문자 패턴이 1번 이상 반복됨 
4. `^` 문자열의 처음을 의미 
5. `$` 문자열의 끝을 의미 
6. `|` or 
7. `[...]` 괄호 안의 어떠한 문자를 의미 
8. `[^...]` 괄호 안에 있지 않은 어떠한 문자를 의미 
9. `{n}` n회 반복 
10. `{m, n}` 반복되는 횟수의 최소, 최대 
