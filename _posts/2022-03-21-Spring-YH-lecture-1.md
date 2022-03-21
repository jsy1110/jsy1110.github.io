---
title:  "[Spring Boot] 스프링 입문 - 코드로 배우는 스프링 부트, 웹MVC, DB 접근 기술 (인프런) #1 (강의 소개)"
date:   2022-03-21
categories: [post]
tags:
- study
- lecture
- spring
---

![Untitled](https://user-images.githubusercontent.com/6336815/159196894-b73119a8-b8e3-41e8-9737-45ed3eb83395.png)

### 강사

현 우아한형제들 기술이사 `김영한`님의 인프런 강의 ([https://www.inflearn.com/users/@yh](https://www.inflearn.com/users/@yh))

김영한 님은 특히 [자바 ORM 표준 JPA 프로그래밍](http://www.yes24.com/Product/Goods/19040233) 의 저자로 유명하신데 해당 책을 통해 국내에 JPA라는 개념이 자리 잡기 시작했다고 한다. 사실 국내에서 백엔드를 개발하는 개발자라면 `토비의 스프링`과 더불어 필독 도서라 해도 모자람이 없을 정도로 유명하다. SI회사로 시작해서 다음, SK플래닛, 우아한형제들까지 밑바닥부터 시작해서 IT 업계의 트랜드 회사들을 모두 거쳐오신 분이다. 

강의를 들어보니 개발 내공만큼 본인의 지식을 남에게 전수하는 강의 스킬까지 완벽했다. [개발바닥](https://youtu.be/Pb69UQ6f8n0) 유튜브 인터뷰를 봤는데 학원 강의 경력이 있으시다고 한다.

### 강의평

더할나위 없이 완벽하다. Spring을 처음 접하는 입문자가 쉽게 이해할 수 있는 정도로 쉽고 핵심을 잘 다룬 강의이다. 물론 블로그 주인장과 같이 Spring과 Spring boot를 사용한 경험이 있지만 “내가 사용하고 있는 Spring이 본래 의도에 맞는 Spring인가?” 의심스러운 Spring 경험자에게도 기초 개념을 다시 잡을 수 있는 강의였다. 특히나 해당 강의는 `무료`로 오픈되어 있으니 본인이 개발을 하는 사람이라면 무조건 한번은 들어보는 것을 추천한다. ([강의링크](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8#))

### Spring vs. Spring boot

Spring의 경우 개발자에게 편의를 주는 JAVA 진영의 오픈소스들을 쉽게 사용할 수 있도록 편의성을 준다는 장점과 객체지향이라는 측면에서 개발을 일관성있게 할 수 있도록 도와준다는 점에서 매우 유용한 프레임워크지만 여기엔 단점이 있었다. `초기 프로젝트 세팅이 너무나 복잡하다.`이 단점을 해결해 준 것이 Spring boot이다. 프로젝트 초기 세팅에 대한 복잡함을 줄이고, spring-boot-starter 패키지를 통해 필요한 dependency와 사용되는 라이브러리에 대한 버전관리까지 한번에 해결해준다. 개인적으로 느꼈던 가장 큰 장점은 Tomcat 설정까지 내장된 상태로 jar 파일을 한번에 배포하고, 실행 또한 `java -jar 프로젝트명.jar` 으로 간편화 했다는 점이다. 지금은 이러한 방식이 보편화 되었지만 그 이전에 개발을 해본 사람이라면 개발환경 세팅에서 was 서버 설정이 얼마나 괴로운 일인지 공감할 것이다.

### Spring boot 1.0 → 2.0

2016~2017년 회사 프로젝트로 Spring Boot를 처음 접하고 1년 반동안 Spring boot를 이용해서 프로젝트를 진행했었다. 당시 Spring 관련 도서들을 구입해서 공부했었는데 Spring boot 가 출시한지 얼마 되지 않은 시기였기 때문에 Spring 관련 내용들은 공부할 자료가 많았지만 Spring boot에 대한 자료는 많지 않았다. 하지만 Spring의 경우 공식 홈페이지에서 상세하고 친절한 Guide를 제공하고, 예제 코드 또한 잘 되어있어 Spring boot 사용이 어렵지는 않았다.

해당 강의는 Spring boot 2.0 이상의 환경에서 동작한다. 강의 영상에 나오는 버전은 2.3.1 버전이고, 2022년 3월 현재 Spring boot의 latest 버전은 2.6.4이다. 버전이 2.0 이상으로 바뀌었다고 해서 Spring boot의 핵심 로직이 변경된것은 없었다. 기존과 마찬가지로 MVC 패턴을 위한 @Controller, @Service, @Repository 등의 annotion을 통해 스프링 빈을 등록하고, @Autowired annotation을 통해 객체를 주입(DI) 받는다.

강의를 통해 느낀 초창기 Spring boot로 개발할때에 비해 발전되거나 변경되었다고 느껴지는 점들은 아래와 같다.

1. 이클립스에서 IntelliJ로 대세 IDE 변경
2. maven과 gradle 양쪽 모두 비슷하게 채용되던 과거와 달리 최근에는 gradle 쪽으로 정리되는 추세 (이전 실무 프로젝트에서는 maven으로 라이브러리 관리를 했었다)
3. Lombok 을 통한 개발 코드의 생산성 향상
4. TDD 방식의 개발은 선택이 아닌 필수