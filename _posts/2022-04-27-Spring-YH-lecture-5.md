---
title:  "[Spring Boot] 스프링 입문 - 코드로 배우는 스프링 부트, 웹MVC, DB 접근 기술 (인프런) #5 - END (AOP)"
date:   2022-04-27 10:31:00 +0900
categories: [post]
tags:
- all
- study
- lecture
- spring
---
## AOP (Aspect Oriented Programming)
관점지향 프로그래밍. 객체지향 프로그램에서 하나의 모듈로 독립하기 어렵고, 프로그램 전반에 걸쳐있는 공통 기능(관심사)을 분리해주는 Spring을 쓰는 주요 이유중 하나이다. 주요 비즈니스 로직은 아니지만 전체적으로 처리해야할 부가 기능들을 비즈니스 로직과 분리 시켜준다. 프록시를 활용하는 데코레이션 패턴에서 발전된 기법으로 로깅, 보안, 트랜잭션 등에서 쓰인다.

 
아래 그림과 같이 시간 측정 로직(부가 기능)이 전체 프로그램에 공통적으로 필요할 경우 시간 측정 로직(부가 기능)을 주요 비즈니스 로직과 분리할 수 있다. 스프링에서는 직접적으로 `AspectJ`에서 제공하는 `@Aspect, @Around` 등의 annotation을 제공하여 개발자가 직접 AOP 요소를 사용할 수 있도록 만들어 준다.

![Untitled](https://user-images.githubusercontent.com/6336815/165420072-4b771647-1944-4998-aa99-b05deaab566f.png)
![Untitled 1](https://user-images.githubusercontent.com/6336815/165420076-a84a99fb-43f2-43a0-b41c-420f0caa89f4.png)

```java
package hello.hellospring.aop;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class TimeTraceAop {
    @Around("execution(* hello.hellospring..*(..))")
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        System.out.println("START: " + joinPoint.toString());
        try {
            return joinPoint.proceed();
        } finally {
            long finish = System.currentTimeMillis();
            long timeMs = finish - start;
            System.out.println("END: " + joinPoint.toString()+ " " + timeMs + "ms");
        }
    }
}
```

AspectJ를 이용하여 제공하는 기능 외에도 스프링에서 직접적으로 제공하는 기능 중 AOP의 특징을 가진 것들이 있다. 그 중 한 가지를 소개하면 `@RestControllerAdvice`가 있는데 RESTful API를 처리하면서 나올 수 있는 각종 http exception을 분리하여 처리할 수 있게 도와준다. 해당 기능을 통해 exception 발생 시 처리하는 로직을 분리할 수 있고, exception에 대한 공통적인 처리 로직을 정의할 수 있다. 아래 코드와 같이예외가 발생할 때 클라이언트에 바로 exception을 던지는 것이 아니라 서버 단에서 한번 처리 후 클라이언트에 에러 원인을 정확히 파악할 수 있도록 서로 규약된 에러 코드와 메시지를 던져주는 것이다.

```java

import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import javax.servlet.http.HttpServletRequest;

@Slf4j
@RestControllerAdvice
public class ExceptionAdvice {

    @ExceptionHandler(CustomException.class)
    public ResponseEntity<ExceptionEntity> exceptionHandler(HttpServletRequest request, final CustomException e) {
        return ResponseEntity
                .status(e.getError().getStatus())
                .body(ExceptionEntity.builder()
                        .errorCode(e.getError().getCode())
                        .errorMessage(e.getError().getMessage() + " " + e.getExtraMessage())
                        .build());
    }
}
```

```java
@Getter
@ToString
public class ExceptionEntity {

    private String errorCode;
    private String errorMessage;

    @Builder
    public ExceptionEntity(HttpStatus status, String errorCode, String errorMessage) {
        this.errorCode = errorCode;
        this.errorMessage = errorMessage;
    }
}
```

AOP로 인해 전체적인 코드 간결성이 좋아지고, 객체지향의 SOLID 5대 원칙 중 하나인 **SRP (단일책임의 원칙: Single Responsibility Principle)** 을 지키며 프로그래밍 할 수 있게 된다. 비즈니스 로직에서는 비즈니스 로직만을 책임지고, 그 외 부가 기능은 AOP로 인해 분리된 객체가 책임진다.