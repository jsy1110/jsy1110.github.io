---
title:  "[Github Page] Google 검색에 블로그 노출 시키기"
date:   2022-03-27 23:00:00 +0900
categories: [post]
tags:
- post
- blog
- notion
---

개인 자료 보관용으로 만든 블로그지만 실제로 검색을 했을 때 내 블로그 글이 검색이 되는지 궁금해졌다. 아무래도 공개된 웹 상에 올리는 글이다 보니 포스트를 작성할 때 잘못된 정보가 없는지 신경이 쓰이는 것이 사실이고, 실제로 기술적인 내용이 들어간 글은 90%이상 만들어놓고 해당 포스트의 정보가 정확한지 확인하지 못해 포스트로 올리지 못하는 경우도 있다. 그러다보니 블로그가 점점 개인 일기장화 되고 있다는 느낌이 있지만...

### Google Search Console

이전 글([[Github Page] Google Analytics로 블로그 방문자 관리하기](https://jsy1110.github.io/2022/google-analytics/))에서 Google Analytics를 통해 블로그의 방문자 추이를 확인할 수 있게 되었다. 해당 블로그는 Github page로 만들어졌기 때문에 자동으로 구글 검색에 조회될 것이라 생각했다. 그런데 테스트를 해보니 검색이 되지 않는다. 구글 검색에 추가하는것도 몇가지 과정이 필요했고, 이 과정에서 겪은 문제들은 공유하고자 한다.

### 속성 추가, 소유권 확인

Google search console에 본인의 github 주소를 추가하면, 아래와 같이 소유권 확인을 위한 창이 뜬다. 여기 나온 html을 다운받은 후 본인 github repository의 루트에 해당 파일을 커밋, 푸쉬를 하면 자동으로 소유권이 확인된다.

![Untitled 0](https://user-images.githubusercontent.com/6336815/160339426-6230ce93-d9df-450e-a274-1de6bf615a85.png)

![Untitled 1](https://user-images.githubusercontent.com/6336815/160339430-51bb4e1d-b975-4219-a284-30f07415e0be.png)

![Untitled 2](https://user-images.githubusercontent.com/6336815/160339434-6a5dc55d-acee-42af-98d0-6b51a37fb854.png)

### 색인 생성

색인 생성을 위해 URL 검사를 해보면 아마 처음에는 `URL이 Google에 등록되어 있지 않음` 이라는 메시지가 뜰 것이다. 여기서 `색인 생성 요청` 을 하게되면 자동으로 URL 테스트를 하고, 색인 생성 요청 상태가 되는데 아래와 같이 크롤링 대기열에 추가되기 때문에 바로 적용되지는 않는다. 해당 블로그의 경우 1~2일 정도 소요되었다. 색인 생성이 완료되면 아래와 같이 `URL이 Google에 등록되어 있음` 으로 메시지가 바뀐다.

![Untitled 3](https://user-images.githubusercontent.com/6336815/160339439-5a1bd1ee-acc6-4ec9-bdfa-87b54fc40710.png)
![Untitled 4](https://user-images.githubusercontent.com/6336815/160339441-ccba9b0c-12fd-47cf-b9d2-167cd7f1de6a.png)
![Untitled 5](https://user-images.githubusercontent.com/6336815/160339443-bb6aefdc-3408-409c-812f-d5e5645b005a.png)

### Sitemap 추가

Sitemap을 추가해준다. Jekyll theme 을 쓰는 경우 종류에 따라 조금씩 다른데 기본적으로 jekyll은 sitemap을 추가해주는 plugin을 제공한다. 해당 페이지의 기본 테마인 Jekyll-Uno의 데모페이지는 sitemap plugin을 사용하지 않고, sitemap 파일을 직접 제공하고 있는데 해당 포스트에서는 어떤 theme를 쓰던 적용할 수 있도록 plugin을 사용하는 방법을 소개한다. 해당 플러그인에 대한 설명은 Github에 잘 설명이 되어있다.([링크](https://github.com/jekyll/jekyll-sitemap))

1. `**Gemfile`에 `gem 'jekyll-sitemap'` 추가.** 
    - `해당 gem은 가장 뒤에 추가한다.` sitemap plugin의 github 페이지를 보면 아래와 같이 설명되어 있다.

be sure to require `jekyll-sitemap` either *after* those other gems if you *want*  the sitemap to include the generated content, or *before* those other gems if you *don't want* the sitemap to include the generated content from the gems. (Programming is *hard*.)

```tsx
source 'https://rubygems.org'
gem 'github-pages', group: :jekyll_plugins

gem "webrick", "~> 1.7"
gem 'jekyll-sitemap'
```

2. `**_config.yml` 파일 url에 본인의 블로그 주소를 등록하고 jekyll-plug인 추가** 

```tsx
url: "https://jsy1110.github.io" # the base hostname & protocol for your site
plugins:
  - jekyll-sitemap
```

![Untitled 6](https://user-images.githubusercontent.com/6336815/160339445-08002a6a-7f19-4e2b-b5c0-497d3b1fd82b.png)

3. **`robots.txt` 파일 작성**

repository 루트에 robots.txt 를 추가하여 아래와 같이 sitemap을 추가하여 작성한다.

```tsx
User-agent: *
Allow: /

Sitemap: [https://jsy1110.github.io/sitemap.xml](https://jsy1110.github.io/sitemap.xml)
```

4. **Build 후 파일 생성 확인**

`bundle exec jekyll serve` 명령어를 통해 로컬 폴더에 `sitemap.xml`이 생성된것을 확인한다. 빌드시 해당 폴더 의 `_site 폴더`에 빌드된 파일이 생성된다. 생성된 파일은 `http://localhost:4000/sitemap.xml` 에 접속해보면 확인할 수 있다.

![Untitled 7](https://user-images.githubusercontent.com/6336815/160339446-282d7eff-f127-446f-8fde-c3cc556c4149.png)
![Untitled 8](https://user-images.githubusercontent.com/6336815/160339449-057da662-91a6-439a-9b17-96f0a924319b.png)

5. **변경사항 배포(Commit&Push), XML 파일 업로드**

여기까지 완료가 되었다면 변경사항을 다시 한번 배포 한 후 Google Search Console 에서 Sitemap을 추가해준다. sitemap에 추가되는 html 페이지들의 태그에 `noindex`가 있는 경우 검색에서 제외한다는 의미이다. 오류가 발생할 경우 해당 부분도 체크해본다.

```tsx
---
layout: default
robots: noindex
---
```

*현재는 해당 사이트 또한 아래와 같이 오류가 발생한 상태이다. 구글링 결과 색인 생성 후 sitemap을 다시 등록하고 [기다리면 해결되는 경우](https://maejinkim.github.io/%EB%B8%94%EB%A1%9C%EA%B7%B8/%EA%B2%80%EC%83%89%EC%B6%94%EA%B0%80/)가 있다고 한다. (실제로 구문오류가 있는 sitemap.xml과 정상 sitemap.xml 파일을 바꿔가면서 올려봤는데 오류 내용은 바뀌지 않았다. **(해당 부분은 추후 업데이트 예정)***

![Untitled 9](https://user-images.githubusercontent.com/6336815/160339451-1575981e-6827-4a61-a508-0e3e0c74a226.png)
![Untitled 10](https://user-images.githubusercontent.com/6336815/160339455-f18ef8d4-84de-4f63-8cc0-6e38433f6c6c.png)