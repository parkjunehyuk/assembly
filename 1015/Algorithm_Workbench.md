# 📘 Assembly Language for x86 Processors (7th Edition)

> **Chapter 6: Conditional Processing — Algorithmic Exercises & Logic Implementation**

이 리포지거리는 Kip Irvine의 *Assembly Language for x86 Processors (7th Edition)* **6장 (Conditional Processing)**의 **Algorithm Workbench 및 Programming Logic 10문제**에 대한 상세한 풀이, 개념 설명, 그리고 어셈블리 코드를 제공합니다.

---

## 📌 Table of Contents

* [Part 1: Bit Manipulation & Conditional Jump Exercises (Q1 ~ Q6)](#-part-1-bit-manipulation--conditional-jump-exercises-q1--q6)
  * [Question 1: ASCII Digit to Binary Conversion](#-question-1)
  * [Question 2: Parity Calculation of 32-bit Memory Operand](#-question-2)
  * [Question 3: Set Difference Operation ($SetX \setminus SetY$)](#-question-3)
  * [Question 4: Unsigned Integer Comparison (`DX` vs `CX`)](#-question-4)
  * [Question 5: Signed Integer Comparison (`AX` vs `CX`)](#-question-5)
  * [Question 6: Bit Clearing & Zero Flag Branching](#-question-6)
* [Part 2: High-Level Logic & Control Structure Implementation (Q7 ~ Q10)](#-part-2-high-level-logic--control-structure-implementation-q7--q10)
  * [Question 7: Compound Logical AND with Short-Circuit](#-question-7)
  * [Question 8: Compound Logical OR with Short-Circuit](#-question-8)
  * [Question 9: Complex Logical Condition `(A AND B) OR C`](#-question-9)
  * [Question 10: While Loop with Nested Short-Circuit Conditions](#-question-10)

---

# 🔹 Part 1: Bit Manipulation & Conditional Jump Exercises (Q1 ~ Q6)

---

### 📍 Question 1

#### 1. English Question
> Write a single instruction that converts an ASCII digit in `AL` to its corresponding binary value. If `AL` already contains a binary value (`00h` to `09h`), leave it unchanged.

#### 2. 한국어 해석
> `AL` 레지스터에 저장된 ASCII 문자로 된 숫자를 그에 대응하는 이진(정수) 값으로 변환하는 단일 명령어를 작성하시오. 만약 `AL`이 이미 이진 값(`00h` ~ `09h`)을 포함하고 있다면, 변경되지 않고 유지되어야 합니다.

#### 3. 핵심 개념 및 상세 풀이
* **ASCII 숫자의 구조:**
  * ASCII 문자의 숫자 `'0'` ~ `'9'`는 16진수로 `30h` ~ `39h`, 이진수로 `0011 0000b` ~ `0011 1001b`입니다.
  * 순수한 정수 값 `0` ~ `9`는 16진수로 `00h` ~ `09h`, 이진수로 `0000 0000b` ~ `0000 1001b`입니다.
* **차이점 분석:**
  * 상위 4비트(High Nibble)가 ASCII의 경우 `0011b` (`3h`)이고, 정수의 경우 `0000b` (`0h`)입니다.
  * 하위 4비트(Low Nibble)는 두 경우 모두 정확히 숫자 값(`0000b` ~ `1001b`)을 나타냅니다.
* **해결 방법:**
  * 비트 마스크 연산인 `AND`를 사용하여 상위 4비트를 `0`으로 만들고 하위 4비트만 남기면 됩니다.
  * 비트 마스크: `0000 1111b` (`0Fh`)
  * 이미 정수인 값(`00h` ~ `09h`)에 `AND 0Fh`를 적용해도 상위 비트가 원래 `0`이므로 값이 바뀌지 않습니다.

| 입력 형태 | 원래 값 (이진수) | 마스크 연산 | 결과 (이진수) | 결과 (16진수/정수) |
| :--- | :--- | :--- | :--- | :--- |
| ASCII `'5'` | `0011 0101b` | `AND 0000 1111b` | `0000 0101b` | `05h` (정수 5) |
| 정수 `5` | `0000 0101b` | `AND 0000 1111b` | `0000 0101b` | `05h` (정수 5) |

#### 4. 어셈블리 답안 코드
```assembly
and al, 0Fh    ; AL의 상위 4비트를 0으로 지우고 하위 4비트만 유지 (00001111b)
```

---

### 📍 Question 2

#### 1. English Question
> Write instructions that calculate the parity of a 32-bit memory operand. *Hint: Use the formula presented earlier in this section: $B_0 \oplus B_1 \oplus B_2 \oplus B_3$.*

#### 2. 한국어 해석
> 32비트 메모리 피연산자의 패리티(Parity)를 계산하는 지시어들을 작성하시오. *(힌트: 본문 앞부분에서 제시된 공식 $B_0 \text{ XOR } B_1 \text{ XOR } B_2 \text{ XOR } B_3$를 사용하시오.)*

#### 3. 핵심 개념 및 상세 풀이
* **패리티 플래그 (Parity Flag, PF):**
  * x86 아키텍처에서 패리티 플래그(PF)는 **결과값의 하위 8비트(1바이트)**에 포함된 `1`의 개수가 **짝수개이면 `PF = 1`**, **홀수개이면 `PF = 0`**으로 설정됩니다.
* **32비트 전체 패리티 구하기:**
  * 32비트 정수(4바이트)를 $B_3, B_2, B_1, B_0$ 바이트로 나눕니다.
  * `XOR` 연산의 특성: 비트 단위 $1$의 개수가 홀수인지 짝수인지 여부(동등성)를 유지해 줍니다.
  * $B_0 \oplus B_1 \oplus B_2 \oplus B_3$를 계산하여 최종 1바이트 결과를 얻으면, 이 1바이트의 패리티 플래그(PF) 상태가 32비트 전체의 패리티 상태와 정확히 일치하게 됩니다.

#### 4. 어셈블리 답안 코드
```assembly
.data
    memVal DWORD 12345678h    ; 32비트 메모리 변수 예시

.code
    mov al, BYTE PTR [memVal]       ; AL = B0 (첫 번째 바이트)
    xor al, BYTE PTR [memVal + 1]   ; AL = B0 XOR B1
    xor al, BYTE PTR [memVal + 2]   ; AL = (B0 XOR B1) XOR B2
    xor al, BYTE PTR [memVal + 3]   ; AL = (B0 XOR B1 XOR B2) XOR B3
    ; 연산 완료 후 Parity Flag(PF)에 32비트 데이터 전체의 패리티가 저장됨
```

---

### 📍 Question 3

#### 1. English Question
> Given two bit-mapped sets named `SetX` and `SetY`, write a sequence of instructions that generate a bit string in `EAX` that represents members in `SetX` that are not members of `SetY`.

#### 2. 한국어 해석
> `SetX`와 `SetY`라는 이름의 두 비트맵 집합이 주어졌을 때, `SetX`의 원소이면서 `SetY`의 원소가 아닌 원소들을 나타내는 비트 스트링을 `EAX`에 생성하는 명령어 시퀀스를 작성하시오.

#### 3. 핵심 개념 및 상세 풀이
* **비트 집합(Bit-mapped Set)과 집합 연산:**
  * 각 비트 위치(0~31)가 집합의 원소 포함 여부(`1` = 포함, `0` = 미포함)를 의미합니다.
* **차집합 개념:**
  * 구하고자 하는 집합: 차집합 $SetX \setminus SetY$ (SetX - SetY)
  * 집합론 공식: $SetX \setminus SetY = SetX \cap SetY^c$
  * 논리 연산 공식: $\text{SetX} \land (\neg \text{SetY})$
* **연산 순서:**
  1. `SetY`를 반전(`NOT`)하여 `SetY`에 속하지 않는 원소들의 비트를 `1`로 만듭니다.
  2. 그 결과와 `SetX`를 논리곱(`AND`) 연산합니다.

#### 4. 어셈블리 답안 코드
```assembly
mov eax, SetY     ; EAX에 SetY 값 로드
not eax           ; EAX = NOT(SetY) [SetY의 여집합 구하기]
and eax, SetX     ; EAX = SetX AND (NOT SetY) [차집합 SetX \ SetY 완성]
```

---

### 📍 Question 4

#### 1. English Question
> Write instructions that jump to label `L1` when the unsigned integer in `DX` is less than or equal to the integer in `CX`.

#### 2. 한국어 해석
> `DX`에 있는 부호 없는(unsigned) 정수가 `CX`에 있는 정수보다 작거나 같을 때(`DX` $\le$ `CX`) 라벨 `L1`으로 점프하는 지시어들을 작성하시오.

#### 3. 핵심 개념 및 상세 풀이
* **부호 없는(Unsigned) 비교 점프 명령어:**
  * 부호 없는 비교에서는 **Below / Above** 용어를 사용하는 점프 명령어를 써야 합니다.
  * $\le$ (작거나 같다 / 이하): `JBE` (*Jump if Below or Equal*) 또는 `JNA` (*Jump if Not Above*)
* **작동 원리:**
  * `CMP DX, CX` 수행 시 내부적으로 `DX - CX` 연산 수행
  * `DX` $\le$ `CX` 일 때 Carry Flag(`CF = 1`) 또는 Zero Flag(`ZF = 1`)가 설정됩니다.

#### 4. 어셈블리 답안 코드
```assembly
cmp dx, cx        ; DX와 CX를 비교 (DX - CX 연산 수행)
jbe L1            ; DX <= CX (부호 없는 비교)이면 라벨 L1으로 점프
```

---

### 📍 Question 5

#### 1. English Question
> Write instructions that jump to label `L2` when the signed integer in `AX` is greater than the integer in `CX`.

#### 2. 한국어 해석
> `AX`에 있는 부호 있는(signed) 정수가 `CX`에 있는 정수보다 크다면(`AX` $>$ `CX`) 라벨 `L2`로 점프하는 지시어들을 작성하시오.

#### 3. 핵심 개념 및 상세 풀이
* **부호 있는(Signed) 비교 점프 명령어:**
  * 부호 있는 비교에서는 **Less / Greater** 용어를 사용하는 점프 명령어를 써야 합니다.
  * $>$ (초과 / 크다): `JG` (*Jump if Greater*) 또는 `JNLE` (*Jump if Not Less or Equal*)
* **주의사항:**
  * 부호 없는 비교용 명령어인 `JA` (*Jump if Above*)를 사용하면, 음수 값을 올바르게 비교할 수 없습니다.

#### 4. 어셈블리 답안 코드
```assembly
cmp ax, cx        ; AX와 CX를 비교 (부호 있는 정수 해석)
jg  L2            ; AX > CX (부호 있는 비교)이면 라벨 L2로 점프
```

---

### 📍 Question 6

#### 1. English Question
> Write instructions that first clear bits 0 and 1 in `AL`. Then, if the destination operand is equal to zero, the code should jump to label `L3`. Otherwise, it should jump to label `L4`.

#### 2. 한국어 해석
> 먼저 `AL` 레지스터의 비트 0과 비트 1을 0으로 지우시오(clear). 그 후, 연산 결과 피연산자가 0과 같다면 라벨 `L3`으로 점프하고, 그렇지 않다면 라벨 `L4`로 점프하는 코드를 작성하시오.

#### 3. 핵심 개념 및 상세 풀이
* **특정 비트 클리어 (Clear Bits):**
  * 특정 비트를 `0`으로 만들려면 지우고자 하는 위치에 `0`, 유지할 위치에 `1`을 넣은 마스크와 `AND` 연산을 수행합니다.
  * 비트 0과 1을 지우는 비트 마스크: `1111 1100b` (`0FCh`)
* **비교 및 분기:**
  * `AND` 명령어는 결과값이 `0`이 되면 자동으로 **Zero Flag (`ZF = 1`)**를 설정합니다.
  * 따라서 별도의 `CMP AL, 0` 명령어 없이 곧바로 `JZ` (*Jump if Zero*)를 사용할 수 있어 효율적입니다.

#### 4. 어셈블리 답안 코드
```assembly
and al, 11111100b   ; AL의 비트 0과 1을 0으로 설정 (마스크: 0FCh)
jz  L3              ; AND 결과가 0이면(ZF = 1) L3로 점프
jmp L4              ; 0이 아니면 L4로 점프
```

---

# 🔸 Part 2: High-Level Logic & Control Structure Implementation (Q7 ~ Q10)

---

### 📍 Question 7

#### 1. English Question
> Implement the following pseudocode in assembly language. Use short-circuit evaluation and assume that `val1` and `X` are 32-bit variables.
> ```c
> if( val1 > ecx ) AND ( ecx > edx ) 
>    X = 1
> else
>    X = 2;
> ```

#### 2. 한국어 해석
> 다음 의사코드를 어셈블리 언어로 구현하시오. 단락 평가(short-circuit evaluation)를 사용하고 `val1`과 `X`는 32비트 변수라고 가정합니다.

#### 3. 핵심 개념 및 상세 풀이
* **단락 평가 (Short-Circuit Evaluation - AND):**
  * `A AND B` 구조에서 **첫 번째 조건 `A`가 거짓(False)이면, 두 번째 조건 `B`는 검사할 필요도 없이 전체가 거짓**이 됩니다.
  * 따라서 첫 번째 조건이 거짓이면 즉시 `else` 블록으로 점프합니다.
* **조건 반전 (Inverted Condition Jump):**
  * 의사코드 조건: `val1 > ecx`
  * 거짓일 때 점프할 조건: `val1 <= ecx` (`JNG` 또는 `JLE` 사용)

#### 4. 어셈블리 답안 코드
```assembly
    mov eax, val1
    cmp eax, ecx
    jng L_Else           ; val1 <= ecx (거짓)이면 바로 else로 단락 점프

    cmp ecx, edx
    jng L_Else           ; ecx <= edx (거짓)이면 else로 단락 점프

    ; 두 조건 모두 참인 경우 (if 본문)
    mov X, 1
    jmp L_Exit

L_Else:
    ; 한 조건이라도 거짓인 경우 (else 본문)
    mov X, 2

L_Exit:
```

---

### 📍 Question 8

#### 1. English Question
> Implement the following pseudocode in assembly language. Use short-circuit evaluation and assume that `X` is a 32-bit variable.
> ```c
> if( ebx > ecx ) OR ( ebx > val1 ) 
>    X = 1
> else
>    X = 2
> ```

#### 2. 한국어 해석
> 다음 의사코드를 어셈블리 언어로 구현하시오. 단락 평가(short-circuit evaluation)를 사용하고 `X`는 32비트 변수라고 가정합니다.

#### 3. 핵심 개념 및 상세 풀이
* **단락 평가 (Short-Circuit Evaluation - OR):**
  * `A OR B` 구조에서 **첫 번째 조건 `A`가 참(True)이면, 두 번째 조건 `B`를 검사하지 않고 즉시 전체가 참**이 됩니다.
  * 따라서 첫 번째 조건이 참이면 바로 `if` 실행 블록(`X = 1`)으로 점프합니다.

#### 4. 어셈블리 답안 코드
```assembly
    cmp ebx, ecx
    jg  L_Then           ; ebx > ecx (참)이면 두 번째 조건 검사 없이 if 본문으로 단락 점프

    cmp ebx, val1
    jg  L_Then           ; ebx > val1 (참)이면 if 본문으로 점프

    ; 두 조건 모두 거짓인 경우 (else 본문)
    mov X, 2
    jmp L_Exit

L_Then:
    ; 한 조건이라도 참인 경우 (if 본문)
    mov X, 1

L_Exit:
```

---

### 📍 Question 9

#### 1. English Question
> Implement the following pseudocode in assembly language. Use short-circuit evaluation and assume that `X` is a 32-bit variable.
> ```c
> if( ebx > ecx AND ebx > edx ) OR ( edx > eax ) 
>    X = 1
> else
>    X = 2
> ```

#### 2. 한국어 해석
> 다음 의사코드를 어셈블리 언어로 구현하시오. 단락 평가(short-circuit evaluation)를 사용하고 `X`는 32비트 변수라고 가정합니다.

#### 3. 핵심 개념 및 상세 풀이
* **복합 논리 조건 분해: `(A AND B) OR C`**
  1. **좌측 `(A AND B)` 그룹:**
     * `A (ebx > ecx)`가 거짓이면 `(A AND B)` 그룹 전체가 거짓이므로, 오른쪽 `OR` 조건인 `C` 검사로 넘어갑니다.
     * `A`가 참이고 `B (ebx > edx)`도 참이면, 전체 `OR` 조건이 성립하므로 즉시 `X = 1`로 점프합니다.
  2. **우측 `C (edx > eax)` 조건:**
     * 좌측 그룹이 실패했을 때 검사하며, 이 조건이 참이면 `X = 1`, 거짓이면 최종 `X = 2`가 됩니다.

#### 4. 어셈블리 답안 코드
```assembly
    ; 1. 좌측 (A AND B) 그룹 검사
    cmp ebx, ecx
    jle L_TestC          ; ebx <= ecx (거짓)이면 오른쪽 OR 조건(C) 검사로 이동

    cmp ebx, edx
    jg  L_Then           ; ebx > edx (참)이면 (A AND B) 전체 참 -> if 본문으로 점프

L_TestC:
    ; 2. 우측 C 조건 검사 (edx > eax)
    cmp edx, eax
    jg  L_Then           ; edx > eax (참)이면 if 본문으로 점프

    ; 모든 조건이 거짓인 경우 (else 본문)
    mov X, 2
    jmp L_Exit

L_Then:
    mov X, 1             ; if 본문: X = 1

L_Exit:
```

---

### 📍 Question 10

#### 1. English Question
> Implement the following pseudocode in assembly language. Use short-circuit evaluation and assume that `A`, `B`, and `N` are 32-bit signed integers.
> ```c
> while N > 0
>   if N != 3 AND (N < A OR N > B)
>      N = N – 2
>   else
>      N = N – 1
> end while
> ```

#### 2. 한국어 해석
> 다음 의사코드를 어셈블리 언어로 구현하시오. 단락 평가를 사용하고 `A`, `B`, `N`은 32비트 부호 있는 정수라고 가정합니다.

#### 3. 핵심 개념 및 상세 풀이
* **구조 분석:**
  * **외부 루프:** `while (N > 0)`
  * **내부 조건:** `if (Condition1 AND (Condition2 OR Condition3))`
* **단락 평가 흐름:**
  1. `N <= 0` 이면 루프 종료 (`Loop_Exit`).
  2. `N == 3` 이면 `AND` 조건이 실패하므로 `else` 블록(`N = N - 1`)으로 이동.
  3. `N < A` 이면 `OR` 조건이 참이므로 `if` 블록(`N = N - 2`)으로 이동.
  4. `N > B` 이면 `OR` 조건이 참이므로 `if` 블록(`N = N - 2`)으로 이동.
  5. 위 `OR` 조건 둘 다 거짓이면 `else` 블록(`N = N - 1`)으로 이동.

#### 4. 어셈블리 답안 코드
```assembly
L_WhileLoop:
    ; 1. 루프 조건 평가: while (N > 0)
    cmp N, 0
    jle L_LoopExit       ; N <= 0 이면 루프 탈출

    ; 2. if 조건 평가 1: N != 3
    cmp N, 3
    je  L_Else           ; N == 3 이면 거짓이므로 else로 이동

    ; 3. if 조건 평가 2: (N < A OR N > B)
    mov eax, N
    cmp eax, A
    jl  L_IfThen         ; N < A (참)이면 if 본문으로 점프
    
    cmp eax, B
    jg  L_IfThen         ; N > B (참)이면 if 본문으로 점프

L_Else:
    ; if 조건 불충족 시: N = N - 1
    sub N, 1
    jmp L_WhileLoop      ; 루프 처음으로 이동

L_IfThen:
    ; if 조건 충족 시: N = N - 2
    sub N, 2
    jmp L_WhileLoop      ; 루프 처음으로 이동

L_LoopExit:
    ; 루프 종료 지점
```