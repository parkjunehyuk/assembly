# Chapter 8: Advanced Procedures (고급 프로시저) - 핵심 요약 및 8.10.1 Short Answer 해설

이 문서는 Kip Irvine의 *Assembly Language for x86 Processors* 교재 **8장 "Advanced Procedures"**의 핵심 개념 및 **8.10.1 Short Answer** 연습문제의 정답과 초보자의 눈높이에 맞춘 상세 해설을 담고 있습니다.

---

## 1. 8장 핵심 개념 한눈에 보기

### 1) 스택 프레임 (Stack Frame)
* **개념**: 함수(프로시저)가 호출될 때 매개변수 전달, 반환 주소 저장, 지역 변수 할당, 레지스터 보존을 위해 스택에 생성되는 **'임시 작업 공간(상자)'**입니다.
* **주요 레지스터 역할**:
  * `EBP` (Base Pointer): 현재 함수 내부의 **고정 기준점(출발점 깃발)**. 매개변수(`[EBP + 8]`)나 지역 변수(`[EBP - 4]`)를 찾을 때 기준이 됩니다.
  * `ESP` (Stack Pointer): 스택의 맨 위(Top)를 가리키는 **움직이는 포인터**. `PUSH`/`POP`에 따라 실시간으로 위치가 바뀝니다.
* **프롤로그 (Prologue)**: 함수 진입 시 `push ebp`, `mov ebp, esp` 등을 수행하여 스택 프레임을 형성하는 준비 단계입니다.
* **에필로그 (Epilogue)**: 함수 종료 직전 `mov esp, ebp`, `pop ebp`, `ret` 등을 통해 쓰던 공간을 청소하고 원래 함수로 되돌아가는 뒷정리 단계입니다.

### 2) 호출 규약 (Calling Conventions)
* **cdecl (C 호출 규약)**: 매개변수를 오른쪽에서 왼쪽 순으로 푸시하며, **호출한 쪽(Caller)**이 `add esp, n`으로 스택을 직접 정리합니다. (가변 인자 지원)
* **STDCALL (표준 호출 규약)**: 매개변수를 오른쪽에서 왼쪽 순으로 푸시하며, **호출당한 함수(Callee)**가 종료될 때 `RET n` 명령어로 스택을 청소하고 나옵니다.

---

## 2. 8.10.1 Short Answer 문제 풀이 및 상세 해설

### Q1. Which statements belong in a procedure’s epilogue when the procedure has stack parameters and local variables?
**스택 매개변수와 지역 변수를 사용하는 프로시저의 에필로그(Epilogue)에는 어떤 문장(명령어)들이 들어가는가?**

* **정답:**
  ```assembly
  mov esp, ebp    ; 1. 지역 변수 공간 해제 (ESP를 기준점 EBP 위치로 복원)
  pop ebp         ; 2. 원래 불렀던 이전 함수의 EBP 값 복원
  ret n           ; 3. 원래 코드 위치로 돌아가며 매개변수 스택 청소 (STDCALL 방식 기준)
  ```
  *(또는 `leave` 명령어로 `mov esp, ebp`와 `pop ebp`를 한 번에 실행한 뒤 `ret n` 실행)*
* **쉬운 해설:**
  에필로그는 **"함수의 퇴근 및 뒷정리 과정"**입니다. 남의 공간을 빌려 쓴 뒤 자기가 챙겨온 짐(지역 변수)을 지우고(`mov esp, ebp`), 원래 주인(이전 함수)의 책상 위치(`EBP`)를 원상복구 해놓은 뒤(`pop ebp`), 돌아갈 문을 열고 나가는(`ret`) 과정입니다.

---

### Q2. When a C function returns a 32-bit integer, where is the return value stored?
**C 언어 함수가 32비트 정수를 반환할 때, 반환 값은 어디에 저장되는가?**

* **정답:** `EAX` 레지스터
* **쉬운 해설:**
  컴퓨터 세상에서는 함수가 일을 끝내고 결과물(반환값)을 돌려줄 때 사용하는 **공식 선물 상자가 바로 `EAX` 레지스터**입니다. 32비트 숫자나 주소 값은 항상 `EAX`에 담아 보냅니다.

---

### Q3. How does a program using the STDCALL calling convention clean up the stack after a procedure call?
**STDCALL 호출 규약을 사용하는 프로그램은 프로시저 호출 후 스택을 어떻게 정리하는가?**

* **정답:**
  **호출당한 프로시저(Callee)** 내부에서 `RET n` 명령어(예: `RET 8`)를 실행하여 나가는 동시에 전달받았던 매개변수 크기($n$ 바이트)만큼 스택 포인터(`ESP`)를 올리며 스스로 청소합니다.
* **쉬운 해설:**
  * **STDCALL 방식**: "손님이 음식을 먹고 나갈 때 자기 그릇을 치우고 나가는 셀프 카페" (함수 내부에서 `RET n`으로 청소)
  * **cdecl 방식**: "손님이 그냥 나가고, 카페 주인이 뒤따라와 그릇을 치우는 일반 식당" (외부에서 `add esp, n`으로 청소)

---

### Q4. How is the LEA instruction more powerful than the OFFSET operator?
**`LEA` 명령어는 `OFFSET` 연산자보다 어떤 점에서 더 강력한가?**

* **정답:**
  `OFFSET`은 조립 시점(컴파일 타임)에 이미 위치가 정해진 정적 변수의 주소만 알 수 있는 반면, `LEA`는 프로그램이 실제로 **실행되는 시점(런타임)에 계산되는 주소**(예: 스택에 만들어진 지역 변수 `[EBP - 4]`의 주소)까지 계산해 올 수 있습니다.
* **쉬운 해설:**
  `OFFSET`은 이사 가기 전 건축 도면에 적힌 "아파트 고정 주소"만 읽을 수 있는 반면, `LEA`는 집이 지어지고 실시간으로 변하는 "현재 짐이 있는 물리적 공간의 위치 주소"를 즉석에서 알아낼 수 있습니다.

---

### Q5. In the C++ example shown in Section 8.2.3, how much stack space is used by a variable of type int?
**Section 8.2.3의 C++ 예시에서 `int` 타입 변수 하나가 차지하는 스택 공간은 얼마인가?**

* **정답:** **4 바이트 (4 bytes)** 또는 **1 dword**
* **쉬운 해설:**
  32비트 x86 컴퓨터 환경에서 기본 정수형(`int`)은 32비트 = 4바이트 칸 크기를 차지합니다.

---

### Q6. What advantages might the C calling convention have over the STDCALL calling convention?
**C 호출 규약(cdecl)이 STDCALL 호출 규약보다 가지는 장점은 무엇인가?**

* **정답:**
  **가변 인자 함수(Varargs)**를 지원할 수 있습니다. (예: 인자의 개수가 그때그때 바뀌는 `printf`, `scanf` 등)
* **쉬운 해설:**
  STDCALL은 함수 스스로 나갈 때 몇 개를 청소할지(`RET 8` 등) 고정해야 해서 인자 개수가 변하면 청소를 못 합니다. 하지만 C 방식은 호출한 놈(Caller)이 몇 개를 넣었는지 알고 있으므로 외부에서 맞춤형으로 치워줄 수 있어서 인자 개수가 자유롭습니다.

---

### Q7. (True/False): When using the PROC directive, all parameters must be listed on the same line.
**`PROC` 지시어를 사용할 때, 모든 매개변수는 반드시 한 줄에 모두 나열해야 한다.**

* **정답:** **False (거짓)**
* **쉬운 해설:**
  매개변수가 너무 많아 줄이 길어지면 `\` (줄 바꿈 기호)를 쓰거나 여러 줄로 나누어 보기 좋게 작성할 수 있습니다.

---

### Q8. (True/False): If you pass a variable containing the offset of an array of bytes to a procedure that expects a pointer to an array of words, the assembler will flag this as an error.
**Byte 배열의 오프셋을 가진 변수를 Word 배열 포인터를 요구하는 프로시저에 전달하면 어셈블러가 이를 에러로 감지한다.**

* **정답:** **False (거짓)**
* **쉬운 해설:**
  어셈블리는 C++처럼 "이 상자는 문자용, 이 상자는 숫자용" 같은 꼼꼼한 타입 검사(Type Checking)를 해주지 않습니다. 단순한 숫자 주소값으로만 인식하므로 어셈블러가 경고나 에러를 발생시키지 않고 그냥 넘어가버립니다.

---

### Q9. (True/False): If you pass an immediate value to a procedure that expects a reference parameter, you can generate a general-protection fault.
**참조 매개변수(Reference Parameter)를 원하는 프로시저에 상숫값(Immediate Value)을 직접 넘기면 General-Protection Fault(일반 보호 오류/메모리 오류)가 발생할 수 있다.**

* **정답:** **True (참)**
* **쉬운 해설:**
  참조 전달은 "내용물이 든 집 주소"를 주길 기대하는 방식입니다. 그런데 주소가 아닌 단순 숫자인 `10`을 직접 던져주면, 함수는 `10번지`라는 메모리 공간을 찾아가서 뚜껑을 열려고 시도합니다. 이때 엉뚱하거나 보호받는 윈도우 OS 시스템 영역을 무단 침입하는 꼴이 되어 프로그램이 튕기게(General-Protection Fault) 됩니다.
