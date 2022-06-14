---
title:  "[Spring Boot] JAVA Enum 클래스 Validation 방법"
date:   2022-05-14 16:00:00 +0900
categories: [post]
tags:
- all
- study
- lecture
- spring
---
스프링에서 @Validated (순수 JAVA의 @Valid) 어노테이션은 MVC 패턴에서 클라이언트의 Request의 값을 손쉽게 validation 할 수 있도록 도와준다. JAVA에서 제공하는 원시타입(Primitive type)의 경우 @NotBlank, @NotNull, @Email 등을 통해 일반적으로 예상되는 validation을 기본적으로 지원한다. 하지만 개발자가 직접 정의한 Enum class의 경우는 Validation 방법 또한 개발자가 직접 추가해야한다.

아래의 코드를 살펴보면 username과 age의 경우는 javax 의 validation 기능을 사용하면 된다. 하지만 Gender 의 값은 어떻게 Validation 해야할까? 아래의 경우 gender 값은 MEN or WOMEN 이외에는 들어올 수 없으므로 exception 처리를 해야한다.

```java

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;

public class UserUpdateRequest {

    @NotBlank
    private String username;
    @NotNull
    private Integer age;

    @?????
    private Gender gender;

}
```

```java
public enum Gender {
    MEN, WOMEN
}
```

위와 같은 상황에서 Gender 라는 Enum class를 처리하는 방법에는 여러가지가 있겠지만 해당 프로젝트에 사용하는 Enum class가 Gender만 있는 것은 아니다. Role, Place 등 여러가지 값들을 Enum class로 정의하여 사용하고 있을 수 있다. 그렇기 때문에 Enum class 로 정의된 전체 값들의 validation을 체크하는 방법을 찾아야 한다.

### 공통 처리를 위한 @interface 추가

아래와 같은 인터페이스 어노테이션을 추가해준다. `@Constraint(validatedBy = EnumValidator.class)` 는 구체적인 validation 방법을 정의한 class 이다. 아래 인터페이스 어노테이션을 이용한 `@Enum` 을 통해 request를 validation 한다. `Class<? extends java.lang.Enum<?>> enumClass();` 우리가 정의한 Enum class를 파라미터로 넘길것이기 때문에 해당 코드를 유심히 봐두자.

```java
@Constraint(validatedBy = EnumValidator.class)
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidEnum {
    String message() default "Invalid value. This is not permitted.";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
    Class<? extends java.lang.Enum<?>> enumClass();
}
```

### Validator 추가

위에서 지정했던 EnumValidator 를 구현한다. 해당 클래스는 ConstraintValidator 인터페이스를 상속받아 구현한다. `ConstraintValidator<ValidEnum, Enum>` 부분이 있는데 우리가 앞에 정의한 어노테이션 인터페이스 `ValidEnum`과 우리가 Validation에 사용할 Type은 Enum class type이므로 `Enum` 값을 선언한다. 

실제 Validation에 사용할 코드는 isValid 를 통해 구현하였다. 현재 Enum 값을 String으로 사용하고 있는데 Enum 의 경우 직접 == 비교가 가능한다. 대소문자 구분 등 추가로 필요한 비교 validation 로직이 있다면 추가할 수 있다.

```java
public class EnumValidator implements ConstraintValidator<ValidEnum, Enum> {
    private ValidEnum annotation;

    @Override
    public void initialize(ValidEnum constraintAnnotation) {
        this.annotation = constraintAnnotation;
    }

    @Override
    public boolean isValid(Enum value, ConstraintValidatorContext context) {
        boolean result = false;
        Object[] enumValues = this.annotation.enumClass().getEnumConstants();
        if (enumValues != null) {
            for (Object enumValue : enumValues) {
                if (value == enumValue) {
                    result = true;
                    break;
                }
            }
        }
        return result;
    }
}
```

### DTO에 적용

위에 정의한 @ValidEnum 과 EnumValidator를 통해 Enum 값에 대한 validation 정의가 끝났다. 앞으로는 사용할 DTO에서 아래와 같이 @ValidEnum과 사용할 enum class를 지정해주면 손쉽게 request validation을 할 수 있다.

```java

import com.univ.xoxo.exception.ValidEnum;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;

public class UserUpdateRequest {

    @NotBlank
    private String username;
    @NotNull
    private Integer age;

    @ValidEnum(enumClass = Gender.class)
    private Gender gender;

}
```

### Reference

- [Annotation으로 Enum 검증하기](https://velog.io/@hellozin/Annotation%EC%9C%BC%EB%A1%9C-Enum-%EA%B2%80%EC%A6%9D%ED%95%98%EA%B8%B0)