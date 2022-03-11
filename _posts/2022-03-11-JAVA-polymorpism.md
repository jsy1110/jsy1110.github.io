---
title:  "[JAVA] 다형성(Polymorpism)"
date:   2022-03-11
categories: [post]
tags:
- post
- dev
- java
---
### 다형성(Polymorpism)?
객체지향 개념에서 볼 때 `여러 가지 형태를 가질 수 있는 능력` 으로 자바에서는 한 타입의 참조변수로 여러 타입의 객체를 참조할 수 있도록 함으로써 프로그램적으로 구현

> Parent class type 참조변수로 Instance of child class를 참조할 수 있음
> 

```java
Tv t = new CaptionTv();
```

> 매개변수 또한 다형성의 성질을 잘 이용하면 구현에 편의성을 줌
> 

```java
class product {
	int price; // 제품의 가격
	int bonusPoint; // 제품구매 시 제공하는 보너스 점수
}
class Tv extends Product() {}
class Computer extends Product() {}
class Audio extends Product() {}

class Buyer { // 고객, 물건을 사는 사람
	int money = 1000; // 소유금액
	int bonusPoint = 0; // 보너스점수
}
```

위와 같은 구성에서 물건을 구매하는 Method를 Buyer Class에서 구현하게 되면 각각 다른 매개 변수로 오버라이딩 하여 구현하게 된다.

![Untitled](https://user-images.githubusercontent.com/6336815/157783326-aafe0a77-393a-40ee-b42b-ca235f6866d6.png)

```java
void buy(Tv t) {
	money = money - t.price;
	bonusPoint = bonusPoint + t.bonusPoint;
}

void buy(Computer c) {
	money = money - c.price;
	bonusPoint = bonusPoint + c.bonusPoint;
}

void buy(Audio a) {
	money = money - a.price;
	bonusPoint = bonusPoint + a.bonusPoint;
}
```

위와 같이 동일한 기능을 하는 Method도 Child class를 매개변수로 사용하게 되면 각각 구현을 해야 한다. 하지만 동일한 기능이라면 다형성(Polymorpism)을 이용하여 Parent class를 매개변수로 사용하여 method의 개수를 줄일 수 있다. 물론, 이때 Method를 사용할 때 전달하는 매개변수는 실제는 child type의 매개변수를 참조변수로 할당 받았을 것이다.

```java
void buy(Product p) {
	money = money - p.price;
	bonusPoint = bonusPoint + p.bonusPoint;
}
```

```java
Buyer b = new Buyer();
Tv t = new Tv();
Computer c = new Computer();
b.buy(t); // 실제 Method를 사용할 때는 child type의 class 를 전달
b.buy(c);
```

> 여러 종류의 객체를 다형성으로 묶는다면 객체 배열로 다룰 수 있다.
> 

```java
Product p1 = new Tv();
Product p2 = new Computer();
Product p3 = new Audio();
```

```java
Product p[] = new Product[3];
p[0] = new Tv();
p[1] = new Computer();
p[2] = new Audio();
```

> 출처 : JAVA의정석 3rd Edition
>