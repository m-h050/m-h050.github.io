---
title: "[CTF/Writeups] cor_2025_web/yamlquiz"
excerpt: "corCTF 2025 web/yamlquiz writeups"

categories: # 카테고리 설정
  - CTF
tags: # 포스트 태그
  - [CTF, writeups, corctf]

permalink: /CTF/cor_2025_web_yamlquiz/ # 포스트 URL

toc: true # 우측에 본문 목차 네비게이션 생성
toc_sticky: true # 본문 목차 네비게이션 고정 여부

date: 2025-11-27 # 작성 날짜
last_modified_at: 2025-11-27 # 최종 수정 날짜
---

# cor 2025 -  web/yamlquiz

# 1. 문제

![](/assets/images/ctf/cor2025/yamlquiz/web-yamlquiz-000.png)

문제에는 yaml에 대해 잘 알고있냐며, 퀴즈를 풀어보라는 내용과 함께

하나의 링크가 있고, 하나의 js 파일이 첨부되어있다.

해당 사이트로 이동해보면 아래와 같이 나온다.

![](/assets/images/ctf/cor2025/yamlquiz/web-yamlquiz-001.png)

“yamlquiz에 오신 것을 환영합니다!”

“당신은 JSON 아니 yaml에 대해 잘 알고 있나요? 이 퀴즈를 풀어보세요 !!”

“남은 기회 : 2번, 당신의 지식이 얼마나 좋은지 확인해 봅시다!”

# 2. 설명

제일 먼저 해당 링크에 접근하여, 페이지 소스를 보면 아래처럼 되어있는데

![](/assets/images/ctf/cor2025/yamlquiz/web-yamlquiz-002.png)

해당 소스코드는 간략하게 퀴즈를 풀었을때 “게임오버”, “통과”, “실패” 시 나타내는 멘트 뿐이며.

퀴즈를 잘풀던 못풀던,  플래그 찾는데에는 직접적인 연관은 없어보인다.

다음으로, 첨부파일에있는 js 파일을 살펴봄

![](/assets/images/ctf/cor2025/yamlquiz/web-yamlquiz-003.png)

js 소스코드 중 아래 post(’/submit’) 부분이 핵심인데,

POST /submit 요청을 보낼 때, result 라는 파라미터를 전달해야하고

이 result 값을 YAML 1.1 파서 / YAML 1.2 파서 각각에서 result[0].score 를 값을 추출하여

파서 결과 (score_1, score_2)가 다를 경우, score = score_1로 설정되고, 

두 파서 결과가 같으면 점수는 0으로 처리,

score === 5000 일때, {pass: ture, flag: FLAG} 반환되고, 그 외에는 {pass: false} 반환 된다는 말

핵심은 YAML 파서의 버전 차이를 이용하여, result[0].score 값이 다르게 나와야하고,

동시에 score_1 === 5000이 되어야 플래그 획득

> YAML 1.1 과 YAML 1.2 차이
> 
> - 숫자 표기 (8진수, 16진수)
>     - YAML 1.1
>         - 0123 → 8진수 = 83 (십진)
>         - 0x123 → 16진수 = 291 (십진)
>     - YAML 1.2
>         - 0123 → 그냥 123 (십진, 앞의 0은 무시)
>         - 0x123 → 문자열 “0x123” (숫자로 인식 x)

즉, 011610은 YAML 1.1 에서만 5000이고, YAML 1.2에서는 11610

011610(8진) = 4096 + 512 + 384 + 8 + 0 = 5000

# 3. 결과

## 방법1 (curl)

curl 을 사용하여 POST /submit 요청으로 yaml 문자열을 http post 형식(Content-Type: application/x-www-form-urlencoded)으로 변환해서 요청하여 응답을 받는다.

![](/assets/images/ctf/cor2025/yamlquiz/web-yamlquiz-004.png)

```bash
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "result=result%3A%0A++-+score%3A+011610" [https://yamlquiz.ctfi.ng/submit](https://yamlquiz.ctfi.ng/submit)
```

중간에 "result=result%3A%0A++-+score%3A+011610” 이게

YAML 문자열을 HTTP 형식으로 변환값

아래는 YAML 문자

```yaml
result:
 - score: 011610
```

## 방법2 (burpsuite)

버프스위트 프록시 툴을 사용하여, 퀴즈20문제 후 제출시 submit 패킷을 잡아, http body 값에 YAML 문자열을 HTTP POST 형식으로 변환한 값을 넣어 요청을 보내어 응답을 받는다.

![](/assets/images/ctf/cor2025/yamlquiz/web-yamlquiz-005.png)

flag: corctf{ihateyamlihateyamlihateyaml!!!}
