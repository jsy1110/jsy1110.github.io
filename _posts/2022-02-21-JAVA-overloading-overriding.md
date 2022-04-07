---
title:  "[JAVA] 상속 개념 정리 (Overriding, Overloading, Interface)"
date:   2022-02-24
categories: [post]
tags:
- all
- study
- java
---
### 오버로딩(Overloading)

- 다형성을 고려
- 동일한 이름의 Method, 다른 유형의 Parameter, return type
- Compile time에 결정됨

### 오버라이딩(Overriding)

- 상속관계일 때 사용됨 (상속은 extends 사용)
- 동일한 이름의 Method, 동일한 유형의 Parameter가 Parent, child 클래스에서 모두 구현됨
- child 에서 정의된 method 는 parent의 method를 대체하여 사용함
- Run time 에 결정 by JVM

![Untitled](https://user-images.githubusercontent.com/6336815/155447019-a3ad1ecc-c6cc-4b40-bee4-ac1154b842af.png)

![Untitled 1](https://user-images.githubusercontent.com/6336815/155447051-6e56ea00-fb43-447e-ac4a-226838caf2b9.png)

![Untitled 2](https://user-images.githubusercontent.com/6336815/155447054-8fc889b4-c57a-4d27-abb2-15fe2fe67b61.png)

### 인터페이스(Interface)

- 객체를 생성하지 않음 (생성자가 없음)
- `Parent`는 Abstract method 또는 default method 사용. 직접 method `구현 불가`
- abstract 를 붙이지 않아도 자동으로 abstract method로 정의 된다.
- 단독 개체는 사용 불가. 인터페이스를 상속받아서 구현한 클래스를 객체로 사용
- `Child`에서는 인터페이스에 정의된 `Abstract method를 반드시 구현`해야함.

```java
public interface superClass {
		void go();
}

public class childClass implements superClass {
	public void go() {
		//여기에 구현
	}
}
```

![Untitled 3](https://user-images.githubusercontent.com/6336815/155447056-00efd9cd-3833-4d97-b492-dc1191649e6b.png)
![Untitled 4](https://user-images.githubusercontent.com/6336815/155447058-87c84dcd-dc92-48b2-91c0-e9b30a40776e.png)

### 추상 클래스(Abstract Class)와 인터페이스(Interface)

- Abstract Class는 다중 상속이 불가능하다!! But, Interface 는 다중 상속(implements)가 가능하다.
- Abstract Class는 객체 생성을 할 수 없다.
- But, Abstract Class 내 Abstract method는 Child에서 상속 받을 경우 반드시 구현해야 한다. (Interface는 abstract 를 적지 않아도 반드시 child에서 구현되야함)

```java
// extends 는 1개의 class만 할 수 있음
// 부모 class에서 동일한 이름의 method가 있을 경우 어떤 것을 orverriding 해야 하는지 모름
// 아래는 에러납니다!!!
public class childClass extends AbstractParent, AbstractParent2 implements superClass {
	
	@Override
	public void go() {
		// TODO interface에 있는 함수는 반드시 구현되야 함
		
	}
}
```