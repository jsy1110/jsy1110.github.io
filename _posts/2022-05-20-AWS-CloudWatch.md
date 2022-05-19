---
title:  "[AWS] 프리티어 과금 방지를 위한 초보자 세팅"
date:   2022-05-20 01:00:00 +0900
categories: [post]
tags:
- all
- AWS
- blog
---
### CloudWatch를 이용한 Billing Alarm 만들기

해당 가이드는 한국어 설정을 기본으로 작성하였다. AWS 가이드 페이지에 한글 문서가 없는 것들이 있어 영문으로 보는게 더 편할때가 많다. Billing Alarm을 켜는 방법 또한 영문으로는 친절하게 나와있으니 영문이 편하신 분은 영문으로 된 공식 페이지를 참고 하시길([링크](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html)).  AWS 한글 페이지는 이상하게 잘 안 읽힌다…

- 먼저 우측 계정을 클릭 했을 때 볼 수 있는 **결제 대시보드 > 결제 기본 설정** 에서 `결제 알림 받기` 체크를 통해 결제 알람 기능을 활성화 시킨다.
- CloudWatch의 Billing Alarm은 US East (N. Virginia) 리전에서 확인할 수 있다. 서울 Region에서 보이지 않던 `결제 경보`가 리전 변경 후 생긴걸 알 수 있다. 

![Untitled](https://user-images.githubusercontent.com/6336815/169331087-85da8158-4c54-499b-b122-0627ae37b184.png)


![Untitled 1](https://user-images.githubusercontent.com/6336815/169331035-875fa10d-a060-4ad4-afc5-93d04ab9aa51.png)
![Untitled 2](https://user-images.githubusercontent.com/6336815/169331044-0ddd683e-b522-4aa3-a3d8-741bff790d44.png)

- 그 다음은 경보를 생성한다. 지표에서 **결제 > 예상 요금 합계** 를 선택한다.
- 결제 경보 발생 조건은 사용자가 설정할 수 있는데 예를 들어 $50 이상 사용시 경보가 발생하게 하려면 아래와 같이 설정하면 된다.

![Untitled 3](https://user-images.githubusercontent.com/6336815/169331050-455a8ffa-800c-456b-8a7e-df7cc97d7dae.png)
![Untitled 4](https://user-images.githubusercontent.com/6336815/169331056-9fc3b8ab-4f80-4167-9d09-ee671587de76.png)
- 작업 구성에서는 경보를 받을 SNS(Simple Notification Service) 주제를 정하게 된다. 기존 SNS가 없는 경우 새 주제 생성을 선택하고, 결제 관련 알림을 수신할 이메일을 입력한다. 이후 이름 및 설명, 미리 보기 완료 후 **경보 생성**을 한다.

![Untitled 5](https://user-images.githubusercontent.com/6336815/169331061-3244dc3d-29a4-4e78-9984-47ade65e61e1.png)
![Untitled 6](https://user-images.githubusercontent.com/6336815/169331064-dc87af48-87ba-4e65-a0bf-823117245603.png)
![Untitled 7](https://user-images.githubusercontent.com/6336815/169331067-da6ffd29-b6ce-4e9c-8365-7b957f5605b5.png)

이전 발생했던 과금사고 관련해서 아래와 같이 경보 발생 이력이 있는 것을 확인할 수 있다.

![Untitled 8](https://user-images.githubusercontent.com/6336815/169331071-d0f8df62-5fd3-4487-9f35-3fdc948fbd99.png)

### CloudTrail 확인

기본적으로 AWS에서 발생한 이벤트 기록([[AWS] 프리티어 과금사고 발생 (Elasti Cache - Redis)](https://jsy1110.github.io/2022/AWS-ElastiCache-incident/))은 CloudTrail을 통해 확인할 수 있다. 추가로 추적 생성을 통해 추적할 세부 Log를 설정할 수 있는데 해당 Log는 S3 버킷을 이용하게 된다. S3 버킷 또한 프리티어에서는 월 2000개의 Request 무료 제한이 있기 때문에 사용에 주의가 필요하다. 프리티어에서 제공하는 CloudTrail 기능 또한 한정적이므로 사용제 주의를 요한다. 필자의 경우는 생성을 잘못해서 5일만에 S3 Request가 1700개를 뚫고, 아래와 같이 Alert 메일을 받게 되었다 ㅠ

![Untitled 9](https://user-images.githubusercontent.com/6336815/169331076-3ba6253c-5a62-4fff-aa23-9ec6abd1adfe.png)
![Untitled 10](https://user-images.githubusercontent.com/6336815/169331080-04b85c39-42c0-4517-9c6a-a4b397cbc835.png)