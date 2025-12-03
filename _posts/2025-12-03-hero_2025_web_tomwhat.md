---
title: "[CTF/Writeups] hero_2025_web_tomwhat"
excerpt: "HeroCTF 2025 web_tomwhat writeups"

categories: # 카테고리 설정
  - ctf
  - web
tags: # 포스트 태그
  - [CTF, writeups, heroctf]

permalink: /ctf/web/hero_2025_web_tomwhat/ # 포스트 URL

toc: true # 우측에 본문 목차 네비게이션 생성
toc_sticky: true # 본문 목차 네비게이션 고정 여부

date: 2025-12-03 # 작성 날짜
last_modified_at: 2025-12-03 # 최종 수정 날짜
---

# web/Tomwhat

![image.png](/assets/images/ctf/hero2025/web/tomwhat/heroctf_web_tomwhat-000.png)

> 키워드: Tomcat / Session / Cross-Context Session Sharing
> 
> 
> **목표:** `username = darth_sidious` 세션 만들기 → `/dark/admin`에서 플래그 획득
> 

---

## ✅ 1) 문제 설명

배포된 인스턴스에 접속하면 **Apache Tomcat 10.1.49** 기본 페이지가 노출되고, 문제에서 제공한 웹앱이 두 개 존재한다.

- `/light/` : Light Side
- `/dark/` : Dark Side
- `/dark/admin` : 조건 만족 시 플래그 출력

하지만 `/light/`에서 `darth_sidious`를 입력하면 **Forbidden username**으로 막혀서, 정상 흐름으로는 `/dark/admin`의 조건을 만족시키기 어렵다.

📌 **목표 조건**

- 세션 attribute `username`이 `darth_sidious`일 때만 `/dark/admin`에서 플래그 출력

![image.png](/assets/images/ctf/hero2025/web/tomwhat/heroctf_web_tomwhat-001.png)

---

## 📦 2) 파일 분석

### 2-1) 애플리케이션 동작 요약

 ✅ `/light/`

- 사용자가 입력한 `username`을 세션에 저장함
- 단, `darth_sidious`는 **입력 차단**

 ✅ `/dark/`

- 세션의 `username`이 있으면 화면에 출력함
- 링크로 `/dark/admin` 이동 가능

 ✅ `/dark/admin`

- 세션의 `username`이 `darth_sidious`이면 플래그 출력
- 아니면 **Access denied**

---

### 2-2) 진짜 핵심: 세션 공유 설정

제공 파일의 실행 스크립트(`run.sh`)에서 중요한 설정이 들어감

- `sessionCookiePath="/"`

➡️ 의미:

**JSESSIONID 쿠키가 `/` 경로로 설정되며, 서로 다른 웹앱(/light, /dark, /examples 등)이 같은 세션을 공유할 수 있음**

📌 결론적으로

`/light`에서 막혀도 **다른 컨텍스트에서 세션에 `username`을 넣을 방법이 있으면** 우회 가능.

---

## 🧠 3) 분석 절차

> 핵심 아이디어:
> 
> 
> **/light에서 세션에 값을 못 넣으면, /light 말고 다른 곳에서 세션에 넣으면 된다.**
> 

 Step 1) Light에서 일반 방법이 막혀있음 확인

- `/light/`에서 `darth_sidious` 입력 → Forbidden
- 즉, LightServlet 경로로는 세션에 `username=darth_sidious` 저장 불가

---

 Step 2) “다른 앱에서 세션을 조작할 수 있나?”를 찾기

Tomcat은 기본적으로 `/examples`라는 예제 앱을 포함할 수 있고, 거기에 **세션 값 저장 기능(SessionExample)** 이 있다.

➡️ SessionExample은 파라미터로 세션 attribute를 저장할 수 있음:

- `dataname` = 세션 key
- `datavalue` = 세션 value

📌 즉, 여기서 `username=darth_sidious`를 넣으면 끝.

---

 Step 3) Cross-Context 세션 공유 확인

`sessionCookiePath="/"` 설정 때문에:

- `/examples`에서 만든 세션 값이
- `/dark`에서도 그대로 보이고
- `/dark/admin`에서도 그대로 사용됨

---

## 🧪 4) 재현 절차

---

 ✅ 0. 준비

브라우저로 아래 인스턴스 접속:

```
http://dyn01.heroctf.fr:10590/
```

---

 ✅ 1. (확인) /light에서 darth_sidious가 막히는지 확인

```
http://dyn01.heroctf.fr:10590/light/
```

- `darth_sidious` 입력 → **Forbidden username** 확인

---

 ✅ 2. 세션에 username을 강제로 주입 (핵심)

아래 URL을 그대로 호출:

```
http://dyn01.heroctf.fr:10590/examples/servlets/servlet/SessionExample?dataname=username&datavalue=darth_sidious
```

페이지에서 세션 데이터가 설정된 것을 확인:

- `username = darth_sidious`

![image.png](/assets/images/ctf/hero2025/web/tomwhat/heroctf_web_tomwhat-002.png)

---

 ✅ 3. /dark에서 값이 반영되는지 확인

```
http://dyn01.heroctf.fr:10590/dark/
```

![image.png](/assets/images/ctf/hero2025/web/tomwhat/heroctf_web_tomwhat-003.png)

---

 ✅ 4. /dark/admin에서 플래그 획득

```
http://dyn01.heroctf.fr:10590/dark/admin
```

![image.png](/assets/images/ctf/hero2025/web/tomwhat/heroctf_web_tomwhat-004.png)

---

## 🏁 5) 결론

> /light는 darth_sidious를 막지만, Tomcat /examples의 SessionExample로 같은 세션에 username=darth_sidious를 주입할 수 있었고, sessionCookiePath="/" 설정 때문에 
/dark/admin 인증을 그대로 우회할 수 있었다.
> 

---
