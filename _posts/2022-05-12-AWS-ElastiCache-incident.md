---
title:  "[AWS] 프리티어 과금사고 발생 (Elasti Cache - Redis)"
date:   2022-05-12 09:00:00 +0900
categories: [post]
tags:
- all
- AWS
- blog
---

### 사건의 시작

최근 토이프로젝트로 간단한 주문 시스템을 만들면서 Redis를 최대한 활용해보고 있다. 그러다가 AWS ElastiCache를 활용하게 되었고, 여기서 사고가 발생하게 되었다. 먼저 Redis의 활용은 3가지 기능으로 접근했다.

1. Ehcache로 구현된 로컬캐시를 글로벌캐시로 확장
2. Redis 메모리 DB 사용으로 재고 관리의 신뢰성 확보 (Race condition issue 해결)
3. Spring security를 활용한 로그인에서 Session 저장소로 활용

기능 테스트 후에 어느정도 전체적인 통합 테스트를 할 수 있는 단계에 이르러서 현재 프리티어로 사용하고 있는 AWS 환경에 올려보기로 했는데 EC2 에서 자체적으로 Redis 를 설치해서 사용할 수도 있었지만 프리티어로 기능을 제공해주는 [Elasti Cache](https://aws.amazon.com/elasticache/) 로 연동해보고 싶은 욕심이 생겼다. 아래는 ElastiCache의 소개 문구이다. 프리티어 사용자는 아래 문구를 잘!! 봐야 한다. 프리티어 무료 해당 노드는 `cache.t2.micro 또는 cache.t3.micro 노드` 이다.  

***
### *무료로 Amazon ElastiCache 시작*
*AWS 프리 티어의 일부로 Amazon ElastiCache를 무료로 시작할 수 있습니다. 신규 AWS 고객은 가입 시 750시간의 ElastiCache `cache.t2.micro 또는 cache.t3.micro 노드` 사용량을 최대 12개월간 무료로 받습니다.*  
* * *

### 사건 재현

ElastiCache에서 Redis 클러스터 생성을 눌렀을 때 기본선택이 된 옵션을 살펴보면 default 선택 노드가 `cache.r6g.large` 이다. 해당 노드는 **시간당 0.206 USD** 이다. 거기다 굳이 하지 않아도 될 클러스트 모드까지 켜놨는데 기본 세팅 샤드수가 3개이다. 이렇게 만들어진 노드에 위에 언급한 Reids 기능들에 대한 테스트를 진행했고, 몇일간 CI/CD 파이프라인 구축을 공부하면서 잊고 지냈다. 그렇게 5일이 흘렀고 단 5일만에 **$267.89**가 청구되는 기적을 볼 수 있었다. 

![Untitled](https://user-images.githubusercontent.com/6336815/168192870-25994812-58dd-44a3-b636-8de6ca9c6f0c.png)
![Untitled 1](https://user-images.githubusercontent.com/6336815/168192877-356c7fac-5848-4870-905b-ec8e03122b87.png)


### 사건 수습

결국 AWS support를 이용하여 도움을 요청했다. 담당자 배정은 요청 후 바로 이루어졌다. 처음에는 실시간 Chat을 통해 자초지종을 설명하고, 사고 원인 파악을 요청했다. 상담원은 계속해서 걱정하지 말라고 나를 안심시키며 확인 후 연락을 주겠다고 했다. 상담원은 ElastiCache에서 설정한 cache.r6g.large 노드는 유료 노드임을 알려줬고, 무료 사용을 원한다면 `cache.t2.micro 또는 cache.t3.micro 노드` 를 사용하라고 안내해줬다. 그리고 과금이 되고 있는 현재 상황을 해결할 것을 요구했는데 이미 Billing 표를 보고 노드를 삭제했기 때문에 추가적인 작업을 할 필요는 없었다. 마지막으로는 CloudWatch를 통해 과금 알람을 설정하라는 안내를 해줬고, 과금 알람에 대한 설정이 확인되면 Billing에 대한 해결을 진행하겠다고 했다. 내가 취할수 있는 모든 조치를 취한 후 아래와 같이 Credit을 넣어줘서 사건은 수습할 수 있었다. 

![Untitled 2](https://user-images.githubusercontent.com/6336815/168192878-818d8294-7810-494a-8ce8-085dde78f729.png)
![Untitled 3](https://user-images.githubusercontent.com/6336815/168192879-ad29503a-5078-4259-a46a-28dd4cdacc87.png)

사건을 해결해주는 상담원은 매우 친절했다. 하지만 AWS 설정 화면 자체는 친절하지 않다. 돈을 벌어야 하는 AWS 입장에서는 당연한 일이겠지만 애초에 프리티어 무료 제공에 해당되지 않는 요소가 무엇인지 설정창에서 명확하게 보여주면 좋을 것을...정말 친절한데 친절하지 않은 AWS.

마지막으로 이 글을 읽는 모든분들께 다시 한번 강조하고 싶다. AWS 프리티어 750시간 무료 제공은 `cache.t2.micro 또는 cache.t3.micro 노드` 에 한해서 제공된다. 다음 포스트는 CloudWatch를 통한 결제 알람 설정방법을 올려보겠다.