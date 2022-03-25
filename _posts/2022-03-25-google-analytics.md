---
title:  "[Github Page] Google Analytics로 블로그 방문자 관리하기 #3"
date:   2022-03-25 16:00:00 +0900
categories: [post]
tags:
- post
- blog
- notion
---
개인적인 스터디 용도로 만든 블로그지만 실제 블로그의 글을 보고 있는 사람이 있는지 궁금해졌다. 현재 사용중인 Jekyll-Uno 테마는 GA 설정을 쉽게 할 수 있도록 config를 제공하기 때문에 GA를 생성해서 테스트 적용시켜 보았다.

### 계정 생성

[Google Analytics](https://analytics.google.com/) 계정을 생성한다. 계정을 생성할 때 가장 중요한 것은 `고급 옵션 보기` 를 열어 `유니버설 애널리틱스 속성 만들기` 를 활성화 하는 것이다. 현재 구글 애널리틱스는 GA4로 업그레이드 된 상태인데 github page의 Jekyll은 최신 버전인 GA4를 지원하지 않는다. 그렇기 때문에 계정을 만들 때 이전 버전인 `유니버설 애널리틱스` 속성을 활성화한 후 만들어야 한다.

![Untitled 0](https://user-images.githubusercontent.com/6336815/160074040-8ec592ab-d990-4cc7-8266-f0d8c0fd2aef.png)

![Untitled 1](https://user-images.githubusercontent.com/6336815/160074044-8507ae6d-a94d-4eae-aa9a-433fa40bbc4e.png)

생성된 계정에서 관리 탭을 확인해보면 `추적ID`를 확인할 수 있다. `UA-` 로 시작되는 ID를 얻었다면 

유니버설 애널리틱스로 생성이 된 것이다. 그렇지 않다면 GA4 이다. 아마 처음 생성했을 경우 유니버설 애널리틱스와 GA4 두 개가 생성됐을 것이다.

![Untitled 2](https://user-images.githubusercontent.com/6336815/160074045-aa15393a-cf84-4dee-8e5e-e33d31387301.png)

### Blog 설정에 추적ID 입력

Jekyll-uno 의 경우 `_config.yml` 파일 google_analytics 속성에 복사한 추적ID를 넣어주기만 하면 세팅이 완료된다. 추적ID를 입력한 후 커밋, 푸쉬까지 완료하면 적용 끝.

```tsx
# Site settings
title: Zzi`s Laboratory
description: 'Development & Study'
url: ''
baseurl: ''
google_analytics: 'UA-22XXXXXXX'
```

만약 본인이 사용중인 테마의 `_config.yml` 파일에 GA 설정이 없다면 직접 GA 관련 파일을 추가해줘야 한다. Jekyll-uno 의 `_includes/footer.html` 파일을 보면 아래와 같이 GA를 자동으로 생성해주는 코드가 들어있다.  이렇게 적용 된 GA는 크롬 등의 브라우저에서 개발자 도구로 확인할 수 있다. 아래 개발자 도구의 콘솔 화면은 크롬에서 [Google Analytics 확장 프로그램](https://chrome.google.com/webstore/detail/google-analytics-debugger/jnkmfdileelhofjcijamephohjechhna?hl=ko)을 설치하여 볼 수 있다.

![image](https://user-images.githubusercontent.com/6336815/160146736-f7f7f5f7-ff03-4fbd-8e34-f6a7fe3e84aa.png)

![Untitled 3](https://user-images.githubusercontent.com/6336815/160074047-7197f1e2-a31d-4ee2-ad7d-d2707d6f3a63.png)

### 통계 보기

이제 Google Analytics 가 정상 동작하는지 확인을 하면 되는데 실시간 통계 페이지를 열어놓고 본인의 github 블로그에 접속하면 실시간으로 통계가 수집되는 것을 확인할 수 있다.

![Untitled 4](https://user-images.githubusercontent.com/6336815/160074050-68596823-9d14-4f82-98b2-aaa48611594a.png)
