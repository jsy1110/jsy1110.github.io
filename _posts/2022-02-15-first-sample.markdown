---
title:  "The first sample from notion"
date:   2016-01-08 15:04:23
categories: [post]
tags:
- post
- notion
- test
---
# 코딩 테스트 1번

2개의 Row, N개의 Column 제시.

1번째 Row의 합은 U, 2번째 Row의 sum 은 S

배열의 값은 0 또는 1 가능.

i번째 Column의 합은 C[i] 의 형식으로 int[] 형태로 제공

위 조건을 만족하는 배열을 String 으로 return. 

예를 들어, U=2, S=3 이고, S[5] = {2,1,0,1,1} 이면 아래와 같이 나올 수 있음

이때 return 값은 “11000,10011”. (답이 여러개가 가능하면 그중 하나만 출력)

| 1 | 1 | 0 | 0 | 0 |
| --- | --- | --- | --- | --- |
| 1 | 0 | 0 | 1 | 1 |

U=3, S=5이고, S[7] = {2,2,1,0,0,2,2} 이면 해당되는 배열이 없으므로 “IMPOSSIBLE” return

```java
String Solution(int U, int S, int[] C) {

}
```
