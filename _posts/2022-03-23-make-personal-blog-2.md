---
title:  "개인 블로그 만들기 #2"
date:   2022-03-23
categories: [post]
tags:
- post
- notion
- dev
---
# Github Page 시작하기 (Jekyll-Uno)

### 환경 설정

해당 내용은 window 기준으로 설명되며, 편의성을 위해 [Git bash](https://git-scm.com/downloads) 와 혼용해서 사용했다. 해당 링크에서 다운로드 가능하다.  해당 설명은 Jekyll을 활용하여 테마를 설치하였다.  Github page 공식 사이트에서도 Jekyll 사용을 권장하고 있다. 기본적으로 Jekyll은 Ruby를 쓰기 때문에 설치가 필요하다. 아래 사이트로 이동하여 아래와 같이 Ruby를 설치하고, GCC 설치를 확인한다. ([https://jekyllrb.com/docs/installation/](https://jekyllrb.com/docs/installation/))

![Untitled](https://user-images.githubusercontent.com/6336815/159629172-caf50827-20b9-4866-a39a-cba4b4154a8f.png)

1. Ruby는 최신 버전을 받으면 된다. 현재 기준 3.1.1이 최신 버전이다. ([다운로드](https://cache.ruby-lang.org/pub/ruby/3.1/ruby-3.1.1.tar.gz))
2. GCC, Make 를 설치한다.
3. 정상 설치 여부를 확인한다.

### 설치

해당 블로그는 Jekyll 을 이용한 블로그중 [jekyll-uno](https://github.com/joshgerdes/jekyll-uno) 라는 테마를 사용하였다. 설치 방법은 jekyll-uno 기준이다. (해당 테마는 어느정도 깔끔한 편인데 글을 쌓다보니 불편한 점이 있어 조만간 테마를 바꿀 예정이다) 각종 테마는 아래 사이트에서 찾아서 쓰면 된다. ([https://jekyllrb.com/resources/](https://jekyllrb.com/resources/))

![Untitled 1](https://user-images.githubusercontent.com/6336815/159629152-3cba71fd-8422-4997-971b-3a33198e30e7.png)

해당 테마의 설치 가이드를 보면 아래와 같이 되어있다. 기본적으로 순서대로 명령을 실행하면 되는데 몇가지 참고할 점이 있다.

- `git clone` 을 활용하면 해당 테마의 github에서 직접 fork 해 오게 된다. 개인 블로그의 경우 글을 자주 변경하게 되고, 이미 지난 post에 대한 변경도 심심치 않게 일어나게 된다. 그렇기 때문에 repository를 private로 설정하려고 했다. 참고로 `clone으로 가져온 repository는 private 설정이 불가`하기 때문에 직접 private repository를 생성하고, code를 다운로드 받아서 올렸다.

![Untitled 2](https://user-images.githubusercontent.com/6336815/159629159-685bc4e9-87f1-4bfb-a282-7bb7f20a6b6c.png)

- 직접 repository를 생성하여 코드를 올리게 되면 아래와 같이 visibility 설정이 활성화 된다. 하지만 무료 유저는 Github page 사용을 위해서 repository를 반드시 public으로 설정해야 한다. private으로 변경을 원할 경우 `upgrade plan` 을 요구한다. (자본주의 사회 ㅠㅠ)

![Untitled 3](https://user-images.githubusercontent.com/6336815/159629162-d7e208a5-b381-4dfa-a849-e976df17a890.png)
![Untitled 4](https://user-images.githubusercontent.com/6336815/159629165-6ebe4767-f9e4-4877-a5df-20799ed4d9cc.png)

- 기본적으로 아래와 같이 설치 순서를 맞춰서 bundler를 설치하고, ruby gem을 설치하게 되면 로컬에서 만들어진 Github page를 실행시키고, `localhost:4000` 을 통해 정상 동작을 확인할 수 있다.

![Untitled 5](https://user-images.githubusercontent.com/6336815/159629167-da7d0b9e-b066-48ee-b891-ae299a417bc3.png)

- 하지만 위의 순서대로 모든것을 설치했음에도 아래와 같이 에러가 발생할 것이다. 

`gem 'wdm', '>= 0.1.0' if Gem.win_platform?`

해당 에러는 Ruby 가 3.0+ 로 버전업이 되면서 기본 설치 패키지에 Webrick 이 설치되지 않아서 발생하는 에러인데 `gemfile`에 webrick을 설치하도록 추가해주거나, `bundle add webrick` 을 통해 설치해야 한다.

```tsx
$ bundle exec jekyll serve
Configuration file: G:/내 드라이브/지승영/개인업무/GitHub_Blog/jsy1110.github.io/_config.yml
Source: G:/내 드라이브/지승영/개인업무/GitHub_Blog/jsy1110.github.io
Destination: _site
Incremental build: disabled. Enable with --incremental
Generating...
done in 4.503 seconds.
Please add the following to your Gemfile to avoid polling for changes:
gem 'wdm', '>= 0.1.0' if Gem.win_platform?
Auto-regeneration: enabled for 'G:/내 드라이브/지승영/개인업무/GitHub_Blog/jsy1110.github.io'
C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve/servlet.rb:3:in require': cannot load such file -- webrick (LoadError)         from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve/servlet.rb:3:in <top (required)>'
from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve.rb:184:in require_relative'         from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve.rb:184:in setup'
from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve.rb:102:in process'         from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve.rb:93:in block in start'
from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve.rb:93:in each'         from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve.rb:93:in start'
from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-3.9.0/lib/jekyll/commands/serve.rb:75:in block (2 levels) in init_with_program'         from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/mercenary-0.3.6/lib/mercenary/command.rb:220:in block in execute'
from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/mercenary-0.3.6/lib/mercenary/command.rb:220:in each'         from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/mercenary-0.3.6/lib/mercenary/command.rb:220:in execute'
from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/mercenary-0.3.6/lib/mercenary/program.rb:42:in go'         from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/mercenary-0.3.6/lib/mercenary.rb:19:in program'
from C:/Ruby30-x64/lib/ruby/gems/3.0.0/gems/jekyll-3.9.0/exe/jekyll:15:in <top (required)>'         from C:/Ruby30-x64/bin/jekyll:25:in load'
from C:/Ruby30-x64/bin/jekyll:25:in `<main>'
```

```tsx
source 'https://rubygems.org'
gem 'github-pages', group: :jekyll_plugins

gem "webrick", "~> 1.7" # 해당 코드 추가
```

```tsx
$ bundle add webrick
Fetching gem metadata from [https://rubygems.org/](https://rubygems.org/).........
Resolving dependencies...
Fetching gem metadata from [https://rubygems.org/](https://rubygems.org/).........
Resolving dependencies...
Using concurrent-ruby 1.1.9
Using minitest 5.15.0
Using thread_safe 0.3.6
Using zeitwerk 2.5.4
Using bundler 2.3.7
Using public_suffix 4.0.6
Using coffee-script-source 1.11.1
Using execjs 2.8.1
Using colorator 1.1.0
.
.
.
```

- 정상적으로 설치가 성공할 경우 아래와 같이 로컬환경에서 정상동작됨을 확인할 수 있다.

```tsx
$ bundle exec jekyll serve
Configuration file: G:/내 드라이브/지승영/개인업무/GitHub_Blog/jsy1110.github.io/_config.yml
Source: G:/내 드라이브/지승영/개인업무/GitHub_Blog/jsy1110.github.io
Destination: _site
Incremental build: disabled. Enable with --incremental
Generating...
done in 4.419 seconds.
Please add the following to your Gemfile to avoid polling for changes:
gem 'wdm', '>= 0.1.0' if Gem.win_platform?
Auto-regeneration: enabled for 'G:/내 드라이브/지승영/개인업무/GitHub_Blog/jsy1110.github.io'
Server address: [http://127.0.0.1:4000/](http://127.0.0.1:4000/)
Server running... press ctrl-c to stop.
```

![Untitled 6](https://user-images.githubusercontent.com/6336815/159629168-684b16e8-2395-41c9-ab63-7a25c491443c.png)