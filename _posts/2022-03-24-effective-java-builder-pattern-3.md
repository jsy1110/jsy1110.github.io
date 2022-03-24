---
title:  "[이펙티브자바] 아이템2 - 빌더 패턴 (Builder pattern) feat.스프링부트 JPA 활용편 (by 김영한) #3"
date:   2022-03-24 11:00:00 +0900
categories: [post]
tags:
- study
- spring
- java
---
이전 글에 설명한 것과 같이 아래의 코드는 결국 오류를 발생시키게 된다. 당장 눈에보이는 create 메서드에서는 builder를 통해 객체를 초기화 하기 때문에 오류가 없을지 몰라도 이후 Book 객체를 조회하는 로직이 추가되면 아래와 같은 오류를 발생시킨다.  참고로 해당 코드는 앞선 포스트에 이어 설명하기 위해 `@Builder` 대신 `@SuperBuilder` 를 사용한 예제를 사용한다.

```java
@GetMapping(value = "/items")
public String list(Model model) {
    List<Item> items = itemService.findItems(); // 오류 발생!!!
    model.addAttribute("items", items);
    return "items/itemList";
}
```

![Untitled](https://user-images.githubusercontent.com/6336815/159831506-d30c5b3e-9e3e-4306-852a-e25014df4f96.png)
`No default constructor for entity.` 즉, 기본 생성자가 없다는 뜻이다. 왜 이런 문제가 발생할까? 잘 살펴보면 Item 클래스와 Book 클래스 모두 생성자를 정의하지 않았다. JAVA는 개발자가 **생성자를 따로 정의하지 않으면** 자동으로 기본 생성자를 만들어준다. 그렇기 때문에 우리는 아래와 같이 생성자 없는 클래스를 정의하고 객체를 생성해서 쓸 수 있다.

```java
// 생성자가 없는 클래스
public class Dog {
		private String name;
		private int age;
		private int weight;
}

public class main {

		public static void main(String[] args) {
				Dog dog = new Dog(); // 기본 생성자
		}
}
```

즉, @Builder annotation이 기본생성자를 만들지 않도록 동작했다는 의미가 된다. 무슨일이 일어난 것인가? @Builder 에 대해 자세히 알 기 위해 [Lombok 공식 홈페이지의 설명](https://projectlombok.org/features/Builder) 보면 알 수 있다.

*Finally, applying `@Builder` to a class is as if you added `@AllArgsConstructor(access = AccessLevel.PACKAGE)` to the class and applied the `@Builder` annotation to this all-args-constructor. This only works if you haven't written any explicit constructors yourself. If you do have an explicit constructor, put the `@Builder` annotation on the constructor instead of on the class.*

그렇다. @Builder annotation 이 `@AllArgsConstructor`의 동작을 하는 constructor를 생성해 준다는 의미이다. 위에서 설명한바와 같이 기본 생성자(default constructor)는 개발자가 어떠한 생성자도 정의하지 않을 때 자바 컴파일러에 의해 생성된다. 그런데 @Builder annotation으로 인해 전체 필드를 매개변수로 하는 생성자가 정의되었으니 기본 생성자는 만들어지지 않은 것이다. 이를 해결하기 위해서 명시적으로 기본 생성자를 만들어주면 된다. `@NoArgsConstructor` 라는 annotation을 통해 기본 생성자를 만들어 주는 것이다. 이렇게 기본생성자를 추가로 정의해주면 아래 코드가 정상적으로 동작함을 알 수 있다.

일단 여기까지 했을 때 본인의 코드에서는 문제가 발생하지 않았다. 그런데 해당 문제 해결을 위해 구글링을 해보았는데 `@Builder` 와 `@NoArgsConstructor` 를 함께 썼을 때 컴파일 오류가 발생하는 경우가 있다고 한다. 이에 대한 해결방법으로 `@AllArgsConstructor` 를 함께 쓰라고 되어있다. 사실 이 부분은 잘 이해가 되지 않는 부분이다. @Builder를 통해 이미 `@AllArgsConstructor` 가 정의되었는데 또 다시 명시를 해줘야 한다니... 이 부분은 Lombok이 버전업 되면서 해결된 문제인지, `@SuperBuilder` 와 `@Builder`의 동작이 다른 것인지 차후 확인해 볼 예정이다.

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
@Getter
@SuperBuilder
@NoArgsConstructor
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
@NoArgsConstructor
public class Book extends Item {
    private String author;
    private String isbn;
}
```

```java
@GetMapping(value = "/items")
public String list(Model model) {
    List<Item> items = itemService.findItems(); // 정의된 기본생성자 사용
    model.addAttribute("items", items);
    return "items/itemList";
}
```

[[이펙티브자바] 아이템2 - 빌더 패턴 (Builder pattern) feat.스프링부트 JPA 활용편 (by 김영한) #1](https://jsy1110.github.io/2022/effective-java-builder-pattern-1/)

[[이펙티브자바] 아이템2 - 빌더 패턴 (Builder pattern) feat.스프링부트 JPA 활용편 (by 김영한) #2](https://jsy1110.github.io/2022/effective-java-builder-pattern-2/)

[[이펙티브자바] 아이템2 - 빌더 패턴 (Builder pattern) feat.스프링부트 JPA 활용편 (by 김영한) #3](https://jsy1110.github.io/2022/effective-java-builder-pattern-3/)