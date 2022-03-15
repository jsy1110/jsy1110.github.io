---
title:  "[JAVA] 예외처리 (Exception handling)"
date:   2022-03-15
categories: [post]
tags:
- post
- dev
- java
---

### JAVA의 예외처리(Exception handling)

프로그램 에러는 발생 시점에 따라 **컴파일 에러**와 **런타임 에러**로 나눌 수 있다. 그리고 JAVA에서는 런타임에 발생할 수 있는 프로그램 오류를 **Error**와 **Exception**으로 구분한다.

- Error : 프로그램 코드에 의해 수습될 수 없는 심각한 오류(Out of memory, Stack overflow 등)
- Exception : 프로그램 코드에 의해 수습될 수 있는 다소 미약한 오류

자바는 실행 시 발생할 수 있는 Exception과 Error를 Class로 정의

- Exception은 RuntimeException class와 그 자손, 그리고 Exception class와 그 자손으로 나뉨
    - RuntimeException classes : 프로그래머의 실수로 발생하는 예외 `(Runtime, unchecked exception, 예외처리 필수X)`
    - Exception classes : 사용자의 실수와 같은 외적인 요인에 의해 발생하는 예외 `(Compile time, checked exception, 예외처리 필수)`

<aside>
💡 **단순 JAVA에서 Exception 종류에 따라 transaction rollback이 결정되는 것은 아니다.** 일부 블로그에 unchecked exception은 rollback이 필수라고 되어있는데 이는 Spring에서 제공하는 exception 관련 옵션에서 unchecked exception 처리에 대한 default가 rollback이기 때문에 나오는 오해일 것이다.

</aside>

![Untitled](https://user-images.githubusercontent.com/6336815/158289345-e62375ef-2bc3-4845-b125-137b7180f137.png)

### Exception Handling 방법

- try - catch 문 사용 (finally 구문은 catch, return 에 상관없이 항상 실행된다)
- throw를 이용해서 고의로 예외 발생
- method에 throws Exception 을 이용하여 예외 선언

```java
void method() throws Exception1, Exception2, ... ExceptionN {}
```