---
title:  "[이펙티브자바] 아이템2 - 빌더 패턴 (Builder pattern) feat.스프링부트 JPA 활용편 (by 김영한)"
date:   2022-03-24
categories: [post]
tags:
- study
- lecture
- spring
- java
---
이번 포스트에서는 이펙티브 자바에 나온 빌더 패턴을 설명하고, 이를 이용해서 김영한 님의 인프런 강의 `실전 스프링부트 JPA 활용 1편` 에 나온 코드를 빌더 패턴을 이용해서 재작성 해보았다.

사족으로 이펙티브 자바는 객체지향이란 무엇인가를 최대한 함축된 코드로 설명하고 있다. 이는 어릴적 배웠던 바둑의 사활풀이를 떠올르게 한다. 사활풀이란 죽을 위기에 빠진 돌을 살려내는 방법을 찾기 위한 수를 찾는 것인데 각각의 상황을 문제은행 식으로 제공한다. 이런 문제를 풀다보면 전혀 생각지도 못한 흔히 말하는 신의한수와 같은 방법들이 나오는데 실제 대국에서 이러한 사활 풀이들을 잘 외워두면 요긴하게 쓰이곤 했다. 이펙티브 자바에 나오는 기법들도 당장 이해하기 어렵지만 이를 이해하고 응용해보는 과정을 통해 책을 읽는 개발자를 성장 시키고, 코드의 유지보수와 생산성을 높여준다. 

정적 팩토리와 생성자에는 제약이 있다. 선택적 매개변수가 많을 경우 대응이 어렵다는 점. 이럴 경우 빌더를 이용해서 객체를 생성하는 것을 고려할 수 있다.

책에는 아래와 같이 영양소를 관리하는 클래스를 가지고 예시를 들고 있다. 아래와 같이 클래스 내 변수가 주어졌을 때 어떻게 데이터를 입력할 것인가에 대한 고민이다. 먼저 필수 변수를 포함한 `생성자`를 이용하는 방법이다. 아래와 같이 생성자의 매개변수를 늘려나가는 방식을 `점층적 생성자 패턴(telecoping constructor pattern)` 이라고 한다.

하지만 이는 값을 지정하기를 원하지 않는 변수에도 값을 지정해야 하는 경우가 생긴다.

`NutritionFacts pepsi = new NutritionFacts(180, 5, 0, 5, 0, 27);`

위와 같이 calories와 sodium의 값을 지정하지 않으려고 해도 어쩔 수 없이 0 으로 지정하게 된다.

```java
public class NutritionFacts {
    private final int servingSize;      // (ml, 1회 제공량) 필수
    private final int servings;         // (회, 총 n회 제공량) 필수
    private final int calories;         // (1회 제공량당) 선택
    private final int fat;              // (g/1회 제공량) 선택
    private final int sodium;           // (mg/1회 제공량) 선택
    private final int carbohydrate;     // (g/1회 제공량) 선택

		public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }
		...
}
```

그렇다면 전통적인 `JAVA Bean` 방식으로 Getter와 Setter를 이용하는 방법도 있다. 김영한님의 강의 예제들도 Lombok을 이용하여 Getter, Setter로 데이터를 생성한다. 최근에는 IDE에서 클래스 내 변수에 대해 getter, setter 생성을 자동으로 해주기도 하고, Lombok 을 이용하여 @Getter @Setter annotation만 넣어주면 되기 때문에 이 방법은 사용하기 간편한 편에 속한다. 하지만 이 방법 또한 하나의 객체를 만들기 위해 `메서드를 여러개 호출`해야 하는 단점이 있고, 객체가 완전히 생성되기 전까지 `일관성(consistency)`가 무너지게 된다. 이는 멀티 스레드 환경에서는 치명적인 오류를 일으킬 수 있기 때문에 추가적인 방어 코드를 넣어줘야 한다.

```java
NutritionFacts pepsi = new NutritionFacts();
pepsi.setServingSize(240);
pepsi.setServings(8);
pepsi.setCalories(100);
pepsi.setCarbohydrate(27);
```

이런 경우에 활용하기 좋은 패턴이 빌더 패턴(Builder pattern)이다. 빌더 패턴은 아래와 같이 쓸 수 있다. 여기서부터는 일관성 있는 설명을 위해 `실전 스프링부트 JPA 활용 1편` 강의에 나오는 코드 약간 변형하여 소개하겠다. 해당 예제에서는 id 값을 사용자에게 받아 저장하고, id 값을 필수값으로 지정해야 하는 상황을 가정했다.

```java
public class Book {

    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

    private String author;
    private String isbn;

    public static class Builder {
        private final Long id; // id 값은 필수 값으로 한다.

        private String name = "";
        private int price = 0;
        private int stockQuantity = 0;

        private String author = "";
        private String isbn = "";

        public Builder(Long id) {
            this.id = id;
        }

        public Builder name(String val) {
            name = val;
            return this;
        }
        public Builder price(int val) {
            price = val;
            return this;
        }
        public Builder stockQuantity(int val) {
            stockQuantity = val;
            return this;
        }
        public Builder author(String val) {
            author = val;
            return this;
        }
        public Builder isbn(String val) {
            isbn = val;
            return this;
        }
				public Book build() {
            return new Book(this);
        }
    }

    private Book(Builder builder) {
        id = builder.id;
        name = builder.name;
        price = builder.price;
        stockQuantity = builder.stockQuantity;
        author = builder.author;
        isbn = builder.isbn;
    }
}
```

위와 같이 Builder 패턴으로 클래스를 설계할 경우 아래와 같이 사용하게 된다. 위 코드는 생각보다 간단하다. Book 이라는 클래스 내에 static으로 선언된 Builder 클래스를 inner 클래스로 정의하고, 해당 클래스 내에서 값을 세팅한다. 여기서 생성자와 final 키워드를 활용해서 id는 필수값으로 입력하도록 정의했다. 그리고 마지막으로 build() 함수를 선언하여 해당 함수를 통해 Book 클래스를 생성하도록 했고, 해당 클래스를 생성하면서 Book 클래스 내에 정의해 둔 private 생성자(constructor)를 호출해 값을 초기화 하도록 한 코드이다.

해당 코드는 아래와 같이 사용할 수 있다. id값은 필수로 지정하였기 때문에 3333L 은 필수 파라미터로 입력되도록 컴파일 단계에서 걸러지게 된다. 그리고 나머지 인자에 대해서는 입력하지 않기를 원한다면 입력하지 않아도 된다.

```java
Book book = new Book.Builder(3333L)
                .name("Effective JAVA")
                .price(35000)
                .stockQuantity(5)
                .author("zzi")
                .isbn("00101")
                .build();

// 아래와 같이 price와 author를 지정하지 않아도 객체 생성 가능
Book book2 = new BookForm.Builder(3333L)
                .name("Effective JAVA")
//                .price(35000)
                .stockQuantity(5)
//                .author("zzi")
                .isbn("00101")
                .build();
```

여기까지 이해가 되었다면 그 다음은 상속관계가 있는 class에서 빌더 패턴을 어떻게 사용하는지 확인해볼 차례이다. 김영한 님의 강의에서 나오는 예제 코드에서도 아래와 같이 상속 관계를 갖는 클래스가 정의되어 있다. 각 엔티티의 관계도와 코드를 살펴보면 아래와 같은데 이 글에서는 패턴 소개를 위해 Spring boot 동작을 위해 사용되는 코드는 최대한 제거했다. (참고로 아래 코드에서 id는 @GeneratedValue 로 정의되어 있어 세팅하지 않아도 된다)

![Untitled](https://user-images.githubusercontent.com/6336815/159743092-15ec3be9-edee-486c-b89e-d351d26b1770.png)

아래 코드는 Book과 Movie 두 개의 클래스만 정의했다. Movie 클래스는 director와 actor를 필수값으로 지정하도록 정의하였고, Book 클래스는 artist와 etc를 선택적으로 세팅하도록 정의하였다. 여기서 주목할 점은 `protected abstract T self()` 코드인데 부모 클래스에서 해당 클래스를 Type generic 
**”T"** 를 이용해서 반환값을 지정하였고, 추상 메서드로 정의하여 자식 클래스에서 반드시 해당 메서드를 완성하도록 하였다. 자식 클래스에서는 각각 본인의 Builder Type(Movie.Builder, Book.Builder)를 반환하도록 self() 클래스를 완성시켰다. 여기에 Item 의 생성자에서 매개변수를 `Builder<?>` 로 넘기도록 해서 자식 클래스의 모든 Builder를 변수로 사용할 수 있도록 했다.

```java
public abstract class Item {

    @Id @GeneratedValue
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

		abstract static class Builder<T extends Builder<T>> {
        private Long id;

        private String name;
        private int price;
        private int stockQuantity;

        public T Price(int val) {
            price = val;
            return self();
        }

        public T StockQuantity(int val) {
            stockQuantity = val;
            return self();
        }

        public T Name(String val) {
            name = val;
            return self();
        }
        abstract Item build();

        protected abstract T self();
    }

    Item(Builder<?> builder) {
        name = builder.name;
        price = builder.price;
        stockQuantity = builder.stockQuantity;
    }
}
```

```java
public class Movie extends Item {
    private String director;
    private String actor;

    /**
     * 빌더 패턴을 이용한 생성자 매개변수 전달 (이펙티브 자바 [아이템2])
		 * 아래의 경우 Builder 생성자와 final 키워드 의해 director와 actor는 필수값이 된다.
     */
    public static class Builder extends Item.Builder<Movie.Builder> {
        private final String director;
        private final String actor;

        public Builder(String director, String actor) {
            this.director = director;
            this.actor = actor;
        }

        @Override
        public Movie build() {
            return new Movie(this);
        }

        @Override
        protected Movie.Builder self() {
            return this;
        }
    }

    private Movie(Movie.Builder builder) {
        super(builder);
        director = builder.director;
        actor = builder.actor;
    }

}
```

```java
public class Book extends Item {
    private final String author;
    private final String isbn;

    /**
     * 빌더 패턴을 이용한 생성자 매개변수 전달 (이펙티브 자바 [아이템2])
     */
    public static class Builder extends Item.Builder<Builder> {
        private String author;
        private String isbn;

        public Builder author(String val) {
            author = val;
            return this;
        }

        public Builder isbn(String val) {
            isbn = val;
            return this;
        }

        @Override
        public Book build() {
            return new Book(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Book(Builder builder) {
        super(builder);
        author = builder.author;
        isbn = builder.isbn;
    }
}
```

```java
@PostMapping(value = "/items/new")
public String create(BookForm form) {
/**
 * Builder 패턴을 이용하여 아래와 같이 변경할 수 있다. (이펙티브 자바 [아이템2])
        book.setName(form.getName());
        book.setStockQuantity(form.getStockQuantity());
        book.setPrice(form.getPrice());
        book.setAuthor(form.getAuthor());
        book.setIsbn(form.getIsbn());
*/

    Book book = new Book.Builder()
            .name(form.getName())
            .stockQuantity(form.getStockQuantity())
            .price(form.getPrice())
            .author(form.getAuthor())
            .isbn(form.getIsbn())
            .build();

    itemService.saveItem(book);
    return "redirect:/";
}
```