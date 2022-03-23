---
title:  "[Spring Boot] 스프링 입문 - 코드로 배우는 스프링 부트, 웹MVC, DB 접근 기술 (인프런) #2 (View, Controller)"
date:   2022-03-22
categories: [post]
tags:
- study
- lecture
- spring
---
### View 설정

기본적으로 정적 페이지는 `resources/static` 폴더 밑에 저장한다. 해당 위치에 저장된 파일은 Spring이 최초 init 될때 자동으로 읽어온다. 기본적으로 `resources/static/index.html`을 최초 페이지로 인식한다. 하지만 Controller에서 루트(/) 에 대한 정의를 넣는다면 `Controller에서 정의한 로직이 정적페이지보다 우선`하여 인식된다.

Controller에서 리턴 값으로 String을 반환할 경우 viewResolver에 의해서 자동으로 페이지를 연결해준다. 아래 예시 코드의 경우 home.html파일을 찾아서 연결해준다.`(resources/templates/{viewName}.html)` 

![Untitled](https://user-images.githubusercontent.com/6336815/159613934-285dd594-135e-4e54-a011-6aaff7006b2b.png)

```java
@Controller
public class HomeController {

    @GetMapping("/")
    /**
     * Contorller의 정의가 정적 파일보다 우선순위가 높음
     */
    public String home() {
        return "home";  // resources/templates/home.html 로 이동
    }
}
```

### Controller

uri 대한 처리는 @RequestMapping 으로 대표되는 Annotation으로 설정할 수 있는데, 해당 Annotation에서 직접 uri과 method을 지정하거나 @GetMapping, @PostMapping... 등을 이용해서 method는 Annotation에서 미리 지정할 수도 있다. @RequestMapping에서 Method를 지정하지 않고, uri만 정의할 경우 모든 method에 매핑 된다.

```java
@GetMapping(value = "/members/new")
public String createForm() {
    return "members/createMemberForm";
}

@PostMapping(value = "/members/new")
public String create(MemberForm form) {
    Member member = new Member();
    member.setName(form.getName());

    memberService.join(member);

    return "redirect:/";
}
```

Request에서 Parameter가 있을 경우 `@RequestParam` 을 이용해서 처리할 수 있다. 위에 설명한대로 return의 경우 String을 return 하여 정적페이지로 연결할 수도 있다. 하지만 `@ResonseBody` annotation을 이용하여 객체를 직접 return 하게 되면 해당 객체를 JSON으로 변환하여 response 를 날리게 된다. 물론 Model 클래스가 아닌 자료형을 직접 반환할 수도 있다. return 시 사용 하는 객체는 getter, setter 등의 Java 전통 Model 의 형태를 갖춰야 한다. (like JAVA bean) 이를 `API 방식`으로 볼 수 있다.

![Untitled 1](https://user-images.githubusercontent.com/6336815/159613932-dfd30bc4-0704-4e52-ad6a-9e2741051dd7.png)


```java
@GetMapping("hello-string")
@ResponseBody
public String helloString(@RequestParam("name") String name) {
    return "hello " + name; // 단순히 String 문자열이 반환된다.
}

@GetMapping("hello-api")
@ResponseBody
public Hello helloApi(@RequestParam("name") String name) {
    Hello hello = new Hello();
    hello.setName(name);
    return hello; // 아래 정의되어있는 Hello 객체를 JSON 형태로 변환하여 반환한다.
}

static class Hello {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```