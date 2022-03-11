---
title:  "[JAVA] 인터페이스를 이용한 다중상속 (다형성)"
date:   2022-03-12
categories: [post]
tags:
- post
- dev
- java
---
### JAVA의 다중상속

JAVA에서 다중상속이 필요한 경우 인터페이스를 사용하는 것은 맞지만 인터페이스의 존재 목적이 다중 상속은 아니다. 오히려 다중상속 성격의 구현이 필요한 경우가 있더라도 비중이 큰 Parent를 상속하고, 나머지 부분을 클래스 내부 멤버로 구현하거나 인터페이스로 만들어서 구현한다.

아래와 같이 TV의 성격과 VCR의 성격을 모두 갖는 TVCR 클래스를 만든다고 할 때, 아래와 같이 한 쪽만 선택하여 상속받고 나머지는 클래스 내에 포함시켜서 사용할 수 있다.

![Untitled](https://user-images.githubusercontent.com/6336815/157811355-b96026aa-e1fc-48e7-b9ac-89a49727ebc0.png)

```java
public class Tv {
	protected boolean power;
	protected int channel;
	protected int volume;

	public void power() { power = !power; }
	public void channelUp() { channel++; }
	public void channelDown()  { channel--; }
	public void volumeUp() { volume++; }
	public void volumDown() { volume--; }
}

public class VCR {
	protected int counter;

	public void play() {
		// Tape를 재생한다.
	}
	public void reset() {
		counter = 0;
	}

	public int getCounter() {
		return counter;
	}

	public void setCounter(int c) {
		counter = c;
	}
}
```

위와같이 상속받을 Parent class가 두 개일 경우 TV 클래스는 상속 받고, VCR class는 인터페이스화 하여 child class에 인스턴스화 할 수 있다.

```java
public interface IVCR {
	public void play();
	public void stop();
	public void reset();
	public int getCounter();
	public void setCounter(int c);
}
```

```java
public class TVCR extends Tv implements IVCR {
	VCR vcr = new VCR();

	public void play() {
		vcr.play();
	}
	public void stop() {
		vcr.stop();
	}
	public void reset() {
		vcr.reset();
	}
	public int getCounter() {
		return vcr.getCounter();
	}
	public void setCounter(int c) {
		vcr.setCounter(c);
	}
}
```

### 인터페이스를 이용한 다형성

> 다형성 : Child class의 인스턴스를 Parent type의 참조변수로 참조하는 것이 가능
> 

```java
Fightable f = new Fighter();
```

인터페이스는 아래와 같이 Method의 매개변수 Type으로도 사용 가능

```java
void attack(Fightable f) { // 인터페이스를 매개변수로 사용

}
```

```java
class Fighter extends Unit implements Fightable {
	public void move(int x, int y) { /* 내용 생략 */ }
	public void attack(Fightable f) { /* 내용 생략 */ }
}
```

위와 같이 구현된 Class가 있다면 `attack(new Fighter())`와 같이 실제 매개 변수로는 Child의 인스턴스를 매개변수로 사용할 수 있다. 또한 아래와 같이 return type으로 인터페이스 타입을 설정하는 것도 가능하다.

```java
Fightable method() {
	Fighter f = new Fighter();
	return f;
}
```

> Return type이 인터페이스라는 것은 메서드가 해당 인터페이스를 구현한 클랜스의 인스턴스를 반환한다는 것을 의미한다.
> 

```java
interface Parseable {
	public void parse(String fileName);
}

class ParserManager {
	public static Parseable getParser(String type) {
		if(type.equals("XML")) {
			return new XMLParser();
		} else {
			return new HTMLParser();
		}
	}
}

class XMLParser implements Parseable {
	public void parse(String fileName) {
		System.out.println(fileName + " - XML parsing completed.");
	}
}

class HTMLParser implements Parseable {
	public void parse(String fileName) {
		System.out.println(fileName + " - HTML parsing completed.");
	}
}

class ParserTest {
	public static void main(String args[]) {
		Parseable parser = ParserManager.getParser("XML");
		parser.parse("document.xml");
		Parseable parser = ParserManager.getParser("HTML");
		parser.parse("document2.html");
	}
}
```

인터페이스의 다형성은 아래와 같이 활용될 수도 있다.

![Untitled 1](https://user-images.githubusercontent.com/6336815/157811365-9deb078c-3f49-4aff-847e-b93b7b499502.png)

```java
interface Liftable {
	void liftOff();
	void move(int x, int y);
	void stop();
	void land();
}

class LiftableImpl implements Liftable {
	public void liftOff() { /* 내용 생략 */ }
	public void move(int x, int y)  { /* 내용 생략 */ }
	public void stop()  { /* 내용 생략 */ }
	public void land()  { /* 내용 생략 */ }
}
```

```java
class Barrack extends Building implements Liftable {
	LiftableImpl l = new LiftableImpl();

	void liftOff() { l.liftOff(); }
	public void move(int x, int y)  { l.move(x, y); }
	public void stop()  { l.stop(); }
	public void land()  { l.land(); }

	void makeMarine() { /* 내용 생략 */ }
	// ...
}

class Barrack extends Building implements Liftable {
	LiftableImpl l = new LiftableImpl();

	void liftOff() { l.liftOff(); }
	public void move(int x, int y)  { l.move(x, y); }
	public void stop()  { l.stop(); }
	public void land()  { l.land(); }

	void makeTank() { /* 내용 생략 */ }
	// ...
}
```

### Default 메서드와 Static 메서드

기본적으로 인터페이스에는 Abstract method만 사용 가능했다. 하지만 JDK1.8부터는 Default method와 static method를 사용할 수 있게 되었다.

 애초에 static method는 인스턴스와 관계없이 독립적으로 사용되는 method로 인터페이스에서 사용하지 못할 이유는 없었지만 규칙의 통일성을 위해 제공하지 않았다. 대표적으로 java.util.Collection 인터페이스에서 사용되는 method들이 static method이다. 이 인터페이스를 Collections라는 클래스에 넣어서 사용했고, 사용 또한 Collections.sort() 와 같이 사용했었다.

 Default method는 인터페이스에 신규 method 추가가 필요한 상황이 생길경우 유연성을 위해 추가되었다. 예를 들어 아래와 같이 신규 newMethod라는 신규 method가 필요할 경우 해당 인터페이스를 구현한 모든 class에서 변경이 불가피해진다. 이를 해결하기 위해 Default method가 추가되었다.

```java
interface MyInterface {
	void method();
	void newMethod(); // 추상 메서드
}
```

```java
interface MyInterface {
	void method();
	default void newMethod();
}
```

> Default method는 Abstract method가 아니기 때문에 child class에서 구현하지 않아도 된다.

> 출처 : JAVA의정석 3rd Edition