---
title:  "[Spring Boot] 실전! 스프링 부트와 JPA 활용1 (feat. 빌더 패턴의 적용) #extra"
date:   2022-03-23
categories: [post]
tags:
- study
- lecture
- spring
---

해당 글은 `이펙티브 자바 아이템 2 - 생성자에 매개변수가 많다면 빌더를 고려하라`에 나오는 빌더 패턴을 Lombok을 이용하여 최종 적용한 코드를 소개한다. 빌더 패턴에 대한 자세한 설명은 이전 [포스트](https://jsy1110.github.io/2022/effective-java-builder-pattern/)를 참고하면 된다.

먼저 부모 클래스인 Item와 자식클래스인 Book에 `@SuperBuilder` annotation을 추가한다. 참고로 Item 클래스에서 categories 객체는 또다른 클래스인 Category 클래스와 N:N 관계를 갖는 객체로 해당 클래스에서 `new ArrayList<>()`로 직접 초기화를 하고 있다. 빌더 패턴의 경우 객체의 변수는 모두 빌더를 통해서 세팅되기 때문에 이를 피하기 위해서 해당 변수에는 `@Builder.Default` anntation을 추가해서 기본값 설정을 허용했다. 이렇게 annotation 추가만으로 아래 컨트롤러에서 사용한 것과 같이 빌더 패턴을 사용할 수 있다.

> @SuperBuilder 는 Lombok v1.18.2. 이상 버전에서 동작한다
> 

* *참고로 한 단계 뛰어넘었지만 상속관계가 없는 class는 @Builder annotation만 해당 객체에 추가하는 것으로 충분하다.*

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
@Getter
@SuperBuilder
public abstract class Item {

    @Id @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

    @ManyToMany(mappedBy = "items")
    @Builder.Default
    private List<Category> categories= new ArrayList<>();
...
}
```

```java
@Entity
@DiscriminatorValue("B")
@SuperBuilder
public class Book extends Item {
    private String author;
    private String isbn;
}
```

```java
@PostMapping(value = "/items/new")
public String create(BookForm form) {
    Book book = Book.builder()
            .author(form.getAuthor())
            .isbn(form.getIsbn())
            .name(form.getName())
            .price(form.getPrice())
            .stockQuantity(form.getStockQuantity())
            .build();

/**
 * Builder 패턴을 이용하여 위와 같이 변경할 수 있다. (이펙티브 자바 [아이템2])
        book.setName(form.getName());
        book.setStockQuantity(form.getStockQuantity());
        book.setPrice(form.getPrice());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());
*/
    itemService.saveItem(book);
    return "redirect:/";
}
```