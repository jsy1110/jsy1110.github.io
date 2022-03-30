---
title:  "[Spring Boot] 스프링 입문 - 코드로 배우는 스프링 부트, 웹MVC, DB 접근 기술 (인프런) #3 (MVC 패턴)"
date:   2022-03-30 11:00:00 +0900
categories: [post]
tags:
- study
- lecture
- spring
---

### 일반적인 MVC 패턴 구조
MVC 패턴을 사용하는 일반적인 웹 애플리케이션은 아래와 같은 구조로 되어있다. 전통적인 Model, View, Controller의 역할을 나누고 각 계층마다 역할을 명확하게 나눈다. 유지보수 측면에서 보더라도 이러한 패턴 구조를 지키는 것이 서로간의 로직 이해에 도움이 된다. 물론 기본적인 구조는 `컨트롤러 → 서비스 → 리포지토리 → DB` 순으로 접근하도록 되어있지만 단순 조회와 같은 특정 로직에서는 컨트롤러에서 리포지토리를 직접 접근하는 것도 나쁘지 않다. 해당 수업에서는 Repository를 인터페이스로 구현했다. 처음 구현은 더미역할을 하는 Memory DB를 만들어서 로직을 완성하고, 이후에 H2라는 실제 DB와 연동했다. 향후에 자세한 설명이 담긴 포스트를 올리겠지만 DB가 변경되는 커다란(?) 구조 변경에도 불구하고 코드 변경은 최소화 되었다. 객체지향의 5대 원칙 중 하나인 `개방-폐쇄 원칙(OCP, Open-Closed Principle)`과 스프링의 `DI(Dependencies Injection)` 특성을 살린 코드인데 자바와 스프링의 맛을 잘 살린 너무나도 훌륭한 예제코드였다.

![Untitled 0](https://user-images.githubusercontent.com/6336815/160737441-c695bde9-c53c-429d-ab37-e9493eb1ac20.png)
![Untitled 1](https://user-images.githubusercontent.com/6336815/160737432-34a16d8a-9857-4c9c-b574-ecd301b676eb.png)

- 컨트롤러: 웹 MVC의 컨트롤러 역할
- 서비스: 핵심 비즈니스 로직 구현
- 리포지토리: 데이터베이스에 접근, 도메인 객체를 DB에 저장하고 관리
- 도메인: 비즈니스 도메인 객체, 예) 회원, 주문, 쿠폰 등등 주로 데이터베이스에 저장하고 관리됨

### MyBatis를 활용한 예제 코드

현업에서 운영중인 서비스도 JPA 대신 MyBatis를 사용하고 있지만 구조는 동일하다. 문제가 되지 않는 범위에서 코드를 살짝 보면 아래와 같다.

`Controller`에서는 Request를 받기 위한 매핑이 되어있고, 실질적인 비즈니스 로직은 `Service`에 넘어가 있다. 여기서 비즈니스 도메인의 역할은 `VO(Value Object)`객체에서 하고있고 DB와 연동되는 있는 `Repository`는 `DAO(Data Access Object)`로 구성되어 있다. 해당 DAO를 통해 아래 쿼리가 실행되고 DB와 Transaction을 갖게 된다.

```java
//Controller
@RequestMapping("saveRedmsEvlList")
public NexacroResult saveRedmsEvlList(@ParamDataSet(name = "DS_USRSF008") List<Usrsf008VO> gridData) {
    chanSvc.saveRedmsEvlList(gridData);

    NexacroResult result = new NexacroResult();
    return result;
}
```

```java
//Service
public void saveRedmsEvlList(List<Usrsf008VO> dataList) {
    for(int i=0; i < dataList.size(); i++){
        Usrsf008VO vo = dataList.get(i);
        switch(vo.getRowType()){
            case RowType.UPDATE :
                dao.update("ChanQry.updateUsrsc007Q01", vo);
                break;
        }
    }
}
```

```sql
#Query
<update id="updateUsrsc007Q01">
        UPDATE  테이블
        SET  데이터
<where>
        AND  조건
</where>
</update>
```