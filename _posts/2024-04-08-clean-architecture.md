---
title:  "[독서후기] 클린 아키텍쳐"
date:   2024-04-08 22:00:00 +0900
categories: [post]
tags:
- all
- book
- blog
---

### 같은 책을 두번 읽는 이유

로버트 C. 마틴 저/송준이 역 | 인사이트(insight)


![image](/assets/2024-04-08-clean-architecture/image1.png)


최근 1~2년 간 모놀리식의 시스템에서 신규 마이크로 서비스들을 분리해내는 프로젝트를 연속으로 진행하였다. MSA 기반의 시스템에서 개발을 하다보니 그동안 가지고 있었던 개발 철학이 많은 부분 바뀌게 되었다. 이전에 이 책을 읽었을 때는 MSA에 대한 경험이 부족한 상태에서 이론적인 지식을 얻었다면, 이번에 이 책을 다시 읽으면서는 그 동안 MSA 개발을 하면서 어렴풋이 `잘못 됐다` `이상하다` 라고 느꼈던 부분들에 대해 `왜` 라는 이유를 조금은 더 구체화하는 기회가 되었다.

### 콘웨이의 법칙

> 시스템을 설계하는 조직이라면 어디든지 그 조직의 의사소통 구조와 동일한 구조의 설계를 만들어 낼 것이다

현재 근무중인 회사의 시스템은 계속해서 발전하고 있고, 이에 따라 수많은 마이크로 서비스들이 만들어지고 있다. 어떤 경우에는 너무 쓸데없는 마이크로 서비스가 생기는것이 아닌가 할때도 있고, 어떤 경우에서는 특정 마이크로 서비스의 역할이 너무 작아서 조금 더 확장을 시켜야 하는게 아닌가 싶을 때도 있다. 개인적으로 이는 회사의 조직 구조와 팀 간의 정치력(?) 싸움의 결과라고 이 책에 명확한 용어로 정의되어 있었다. `콘웨이의 법칙` 결국 시스템 설계는 조직의 구조를 따라 간다. 즉, 소프트웨어 회사의 잦은 조직 변경은 시스템 설계에 따라 변경되는 것이 자연스럽다. 

*조직을 먼저 만들어놓고, 그 조직에 맞게 시스템 설계를 하게 되는 우를 범하면 안되겠다*

### 모놀리식은 오답이고, MSA가 정답인가?

아키텍쳐의 관점에서 소프트웨어는 결합이 최대한 분리되고 변경에 부드러워야 한다. 이 책에서는 결합의 분리모드를 **소스 수준, 배포 수준, 서비스 수준** 세 단계의 수준으로 표현했다. 가장 높은 수준의 분리는 서비스 수준의 분리이고, 이 것이 우리가 알고 있는 MSA이다. 하지만 책이 아니더라도 모두들 직감적으로, 경험적으로 알고 있다. MSA는 개발 시간과 시스템 자원 모두 많이 드는 아키텍쳐이다. 그렇기 때문에 규모가 작은 회사(개발팀)에서 도입하기엔 현실적인 어려움이 있다.

그렇다면 내가 현재의 규모가 아닌 모놀리식으로 운영되는 서비스를 개발할 때는 어떻게 해야 할 것인가? 나는 MSA 시스템을 직접 개발하고 운영하면서 MSA가 가진 강점을 직접 경험하고 있다. 내가 모놀리식 서비스를 운영하는 회사에서 일하게 된다고 하더라도 앵무새처럼 우리의 지향점은 MSA에 있음을 주장할 것이다. 이런 태도는 기존 서비스를 운영하고 있는 기존 멤버가 봤을 때는 "너희는 틀렸고, 내가 맞아" 라는 모습으로 비춰질 수 있다.

이 책은 이런 고민을 해결할 수 있는 힌트를 주고 있다. 실제로 인프라까지 MSA로 구분되는 서비스로 개발하지 않더라도 컴포넌트를 잘 분리해서 설계해 놓고, 시스템 발전에 따라 소스 수준에서 배포 수준으로, 다시 서비스 수준으로 분리 수준을 높여갈 수 있다는 것이다. 사실 아키텍쳐 관점에서 컴포넌트 분리의 경계를 정하는 것은 매우 어려운데 여기서 나의 강점이 잘 드러날 수 있다. 모놀리식 서비스가 MSA 서비스로 진화되고 있는 과정을 지켜보고 있기 때문에 어떤 컴포넌트들이 먼저 분리되어 나오는지 경험을 하고 있기 때문이다.

### 패션은 사라지지만 스타일은 그대로 남는다


![image](/assets/2024-04-08-clean-architecture/image2.png)


사실 위의 내용들은 이전에 이 책을 읽었을 때는 크게 신경을 쓰지 않았고, 기억에 남아 있지도 않았다. 책을 읽으면서 이런 부분이 있었나? 하고 새로운 책을 읽는 기분이었다. 그런데 나의 경험이 쌓이고 나서 읽었을 때 그동안 보이지 않던 부분이 비로소 보이기 시작했다. 여기 적은 내용 외에도 그 동안 경험적으로, 관성적으로 다른 코드들을 모방하면서 개발했던 부분들이 실제로 좋은 아키텍쳐가 될 수 있는 이유를 명확하게 알 수 있었다. 이러한 이유로 당분간 개발 기본서 읽기는 지속될 것 같다.
