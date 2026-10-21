# 📘 Assembly Language for x86 Processors (7th Edition)

> **Chapter 6, Section 6.11: Programming Exercises (Questions 1 – 10)**  
> Complete MASM / Irvine32 Solutions with Detailed Explanations and Code Walkthroughs.

![Assembly Language](https://img.shields.io/badge/Language-x86%20Assembly%20(MASM)-blue.svg)
![Library](https://img.shields.io/badge/Library-Irvine32-green.svg)
![Chapter](https://img.shields.io/badge/Chapter-6.11%20Programming%20Exercises-orange.svg)

---

## 📌 Table of Contents

* [Exercise 1: Filling an Array](#-exercise-1-filling-an-array)
* [Exercise 2: Summing Array Elements in a Range](#-exercise-2-summing-array-elements-in-a-range)
* [Exercise 3: Test Score Evaluation](#-exercise-3-test-score-evaluation)
* [Exercise 4: College Registration](#-exercise-4-college-registration)
* [Exercise 5: Boolean Calculator (1)](#-exercise-5-boolean-calculator-1)
* [Exercise 6: Boolean Calculator (2)](#-exercise-6-boolean-calculator-2)
* [Exercise 7: Probabilities and Colors](#-exercise-7-probabilities-and-colors)
* [Exercise 8: Message Encryption](#-exercise-8-message-encryption)
* [Exercise 9: Validating a PIN](#-exercise-9-validating-a-pin)
* [Exercise 10: Parity Checking](#-exercise-10-parity-checking)

---

# 🔹 Programming Exercises (6.11)

---

### 📍 Exercise 1: Filling an Array

#### 1. English Problem
> Create a procedure that fills an array of doublewords with $N$ random integers, making sure the values fall within the range $j \dots k$, inclusive. When calling the procedure, pass a pointer to the array that will hold the data, pass $N$, and pass the values of $j$ and $k$. Preserve all register values between calls to the procedure. Write a test program that calls the procedure twice, using different values for $j$ and $k$. Verify your results using a debugger.

#### 2. 한국어 해석
> 범위 $[j, k]$ ($j \dots k$, 양 끝값 포함) 사이의 $N$개의 난수(Random Integers)로 32비트 정수(DWORD) 배열을 채우는 프로시저를 작성하시오. 프로시저를 호출할 때는 데이터를 저장할 배열의 포인터, 요소 개수 $N$, 그리고 $j$와 $k$의 값을 전달해야 합니다. 프로시저 호출 시 모든 레지스터의 값이 보존되도록 하시오. 서로 다른 $j$와 $k$ 값을 사용하여 이 프로시저를 두 번 호출하는 테스트 프로그램을 작성하고, 디버거를 이용하여 결과를 확인하시오.

#### 3. 핵심 개념 및 풀이 요약
* **범위 내 난수 생성 원리:**
  * Irvine32의 `RandomRange`는 `EAX`에 입력된 값 $M$에 대해 $[0, M-1]$ 범위의 난수를 반환합니다.
  * 범위 $[j, k]$의 임의의 수를 구하기 위해 필요한 총 개수는 $(k - j + 1)$개입니다.
  * `EAX = (k - j + 1)`로 설정한 후 `RandomRange`를 호출하면 $[0, k - j]$ 사이의 값이 생성됩니다.
  * 생성된 결과에 $j$를 더하면 final range $[j, k]$를 얻습니다:  
    $$\text{RandomVal} = \text{RandomRange}(k - j + 1) + j$$
* **레지스터 보존:**
  * 프로시저 시작 시 `pushad` (또는 개별 `PUSH`), 종료 시 `popad` (또는 개별 `POP`)를 사용하여 호출 전후 레지스터 상태를 완전하게 보존합니다.

#### 4. 어셈블리 답안 코드
```assembly
INCLUDE Irvine32.inc

.data
ARRAY_SIZE = 20
array1 DWORD ARRAY_SIZE DUP(?)
array2 DWORD ARRAY_SIZE DUP(?)

.code
main PROC
    call Randomize          ; 난수 생성기 초기화

    ; --- 첫 번째 테스트: 범위 10 ~ 50 ---
    mov esi, OFFSET array1  ; ESI = 배열 포인터
    mov ecx, ARRAY_SIZE     ; ECX = N (배열 크기)
    mov ebx, 10             ; EBX = j (최소값)
    mov edx, 50             ; EDX = k (최대값)
    call FillArray

    ; --- 두 번째 테스트: 범위 -20 ~ 20 (부호 고려 시 unsigned 범위를 잘 지정) ---
    mov esi, OFFSET array2
    mov ecx, ARRAY_SIZE
    mov ebx, 100
    mov edx, 200
    call FillArray

    exit
main ENDP

;-----------------------------------------------------------------
; FillArray
; DWORD 배열을 [j, k] 범위의 난수로 채움.
; Receives: ESI = 배열 포인터, ECX = Count(N), EBX = j, EDX = k
; Returns: Nothing
;-----------------------------------------------------------------
FillArray PROC
    pushad                  ; 모든 레지스터 보존

    mov eax, edx
    sub eax, ebx
    inc eax                 ; EAX = (k - j + 1) = 난수 범위 크기

L1:
    push eax                ; 범위 크기 보존
    call RandomRange        ; EAX = 0 ~ (k - j)
    add eax, ebx            ; EAX = j ~ k
    mov [esi], eax          ; 배열에 저장
    add esi, TYPE DWORD     ; 다음 요소 이동
    pop eax                 ; 범위 크기 복원
    loop L1

    popad                   ; 모든 레지스터 복원
    ret
FillArray ENDP

END main
```

---

### 📍 Exercise 2: Summing Array Elements in a Range

#### 1. English Problem
> Create a procedure that returns the sum of all array elements falling within the range $j \dots k$ (inclusive). Write a test program that calls the procedure twice, passing a pointer to a signed doubleword array, the size of the array, and the values of $j$ and $k$. Return the sum in the `EAX` register, and preserve all other register values between calls to the procedure.

#### 2. 한국어 해석
> 배열 요소 중 지정된 범위 $[j, k]$ ($j \dots k$, 양 끝값 포함) 내에 있는 요소들의 합을 반환하는 프로시저를 작성하시오. 부호 있는 32비트 정수(SDWORD) 배열의 포인터, 배열 크기, 그리고 $j$와 $k$ 값을 전달하여 이 프로시저를 두 번 호출하는 테스트 프로그램을 작성하시오. 합계는 `EAX` 레지스터로 반환하고, `EAX`를 제외한 다른 모든 레지스터의 값은 프로시저 호출 전후로 보존되어야 합니다.

#### 3. 핵심 개념 및 풀이 요약
* **부호 있는 비교 및 수열 필터링:**
  * 배열의 각 요소 $X$에 대해 $j \le X \le k$ 조건을 충족하는지 검사합니다.
  * 부호 있는 정수(Signed Integers) 비교이므로 conditional jump 시 `JL` (Jump if Less) 및 `JG` (Jump if Greater)를 사용합니다.
* **리턴값 레지스터 및 레지스터 보존:**
  * 결과값을 반환할 `EAX`를 제외한 `EBX`, `ECX`, `EDX`, `ESI` 등은 스택에 `PUSH`/`POP`하여 보존합니다.

#### 4. 어셈블리 답안 코드
```assembly
INCLUDE Irvine32.inc

.data
myArray SDWORD -10, 20, 35, -5, 50, 15, 80, 5, 0, 40
ARRAY_SIZE = LENGTHOF myArray

.code
main PROC
    ; --- 첫 번째 테스트: 범위 0 ~ 40 내의 원소 합 ---
    mov esi, OFFSET myArray
    mov ecx, ARRAY_SIZE
    mov ebx, 0              ; j = 0
    mov edx, 40             ; k = 40
    call SumRange
    call WriteInt           ; EAX 출력
    call Crlf

    ; --- 두 번째 테스트: 범위 -10 ~ 10 내의 원소 합 ---
    mov esi, OFFSET myArray
    mov ecx, ARRAY_SIZE
    mov ebx, -10            ; j = -10
    mov edx, 10             ; k = 10
    call SumRange
    call WriteInt
    call Crlf

    exit
main ENDP

;-----------------------------------------------------------------
; SumRange
; [j, k] 범위 내에 속하는 SDWORD 배열 원소들의 합을 구함.
; Receives: ESI = 배열 포인터, ECX = 배열 크기, EBX = j, EDX = k
; Returns: EAX = 합계
;-----------------------------------------------------------------
SumRange PROC
    push ebx
    push ecx
    push edx
    push esi
    push edi

    mov edi, 0              ; EDI를 누적 합계용으로 사용 (EAX 임시 보존)

L1:
    mov eax, [esi]          ; 현재 배열 원소 읽기
    cmp eax, ebx
    jl  Skip                ; eax < j 이면 제외
    cmp eax, edx
    jg  Skip                ; eax > k 이면 제외

    add edi, eax            ; 범위 내 원소이면 합산

Skip:
    add esi, TYPE SDWORD
    loop L1

    mov eax, edi            ; 최종 합을 EAX에 복사

    pop edi
    pop esi
    pop edx
    pop ecx
    pop ebx
    ret
SumRange ENDP

END main
```

---

### 📍 Exercise 3: Test Score Evaluation

#### 1. English Problem
> Create a procedure named `CalcGrade` that receives an integer value between 0 and 100, and returns a single capital letter in the `AL` register. Preserve all other register values between calls to the procedure. The letter returned by the procedure should be according to the following ranges:
>
> | Score Range | Letter Grade |
> | :---: | :---: |
> | 90 to 100 | A |
> | 80 to 89 | B |
> | 70 to 79 | C |
> | 60 to 69 | D |
> | 0 to 59 | F |
>
> Write a test program that generates 10 random integers between 50 and 100, inclusive. Each time an integer is generated, pass it to the `CalcGrade` procedure. You can test your program using a debugger, or if you prefer to use the book’s library, you can display each integer and its corresponding letter grade.

#### 2. 한국어 해석
> 0에서 100 사이의 정수 점수를 입력받아 `AL` 레지스터에 해당하는 대문자 학점(Letter Grade)을 반환하는 `CalcGrade` 프로시저를 작성하시오. 프로시저 호출 시 `AL`을 제외한 다른 모든 레지스터의 값은 보존되어야 합니다. 학점 기준표는 위 표와 같습니다.  
> 50에서 100 사이(양 끝값 포함)의 난수 10개를 생성하는 테스트 프로그램을 작성하시오. 난수가 생성될 때마다 `CalcGrade` 프로시저에 전달하고, 각 점수와 그에 대응하는 학점을 화면에 출력하시오.

#### 3. 핵심 개념 및 풀이 요약
* **50~100 범위 난수 생성:**
  * 범위 크기: $100 - 50 + 1 = 51$
  * `EAX = 51`로 지정 후 `RandomRange` 호출 $\rightarrow$ $0 \sim 50$ 생성.
  * 결과에 $50$을 더하면 $50 \sim 100$ 범위가 됩니다.
* **단계별 범위 비교 (Cascading CMP):**
  * 높은 점수(90)부터 순서대로 비교하여 조건을 만족하면 라벨로 이동합니다.

#### 4. 어셈블리 답안 코드
```assembly
INCLUDE Irvine32.inc

.data
    msgScore BYTE "Score: ",0
    msgGrade BYTE "  Grade: ",0

.code
main PROC
    call Randomize
    mov ecx, 10             ; 10회 반복

L1:
    mov eax, 51             ; 0 ~ 50 범위
    call RandomRange
    add eax, 50             ; 50 ~ 100 범위로 변환

    ; 점수 출력
    mov edx, OFFSET msgScore
    call WriteString
    call WriteDec

    ; CalcGrade 호출 (EAX에 점수 전달)
    call CalcGrade          ; AL에 학점문자 반환

    ; 학점 출력
    mov edx, OFFSET msgGrade
    call WriteString
    mov al, al              ; AL의 학점 출력
    call WriteChar
    call Crlf

    loop L1

    exit
main ENDP

;-----------------------------------------------------------------
; CalcGrade
; 점수를 입력받아 AL 레지스터로 문자 학점 반환
; Receives: EAX = 점수 (0 ~ 100)
; Returns: AL = 'A', 'B', 'C', 'D', 'F'
;-----------------------------------------------------------------
CalcGrade PROC
    push ebx                ; 사용되는 레지스터 보호

    cmp eax, 90
    jae GradeA
    cmp eax, 80
    jae GradeB
    cmp eax, 70
    jae GradeC
    cmp eax, 60
    jae GradeD
    
    mov al, 'F'
    jmp ExitGrade

GradeA:
    mov al, 'A'
    jmp ExitGrade
GradeB:
    mov al, 'B'
    jmp ExitGrade
GradeC:
    mov al, 'C'
    jmp ExitGrade
GradeD:
    mov al, 'D'

ExitGrade:
    pop ebx
    ret
CalcGrade ENDP

END main
```

---

### 📍 Exercise 4: College Registration

#### 1. English Problem
> Using the College Registration example from Section 6.7.3 as a starting point, do the following:
> - Recode the logic using `CMP` and conditional jump instructions (instead of the `.IF` and `.ELSEIF` directives).
> - Perform range checking on the credits value; it cannot be less than 1 or greater than 30. If an invalid entry is discovered, display an appropriate error message.
> - Prompt the user for the grade average and credits values.
> - Display a message that shows the outcome of the evaluation, such as "The student can register" or "The student cannot register".

#### 2. 한국어 해석
> Section 6.7.3의 대학 등록(College Registration) 예제를 바탕으로 다음을 수행하시오:
> - `.IF` 및 `.ELSEIF` 디렉티브 대신 `CMP`와 조건 점프 명령어를 사용하여 로직을 다시 구현하시오.
> - 학점(credits) 값에 대한 범위 검사를 수행하시오. 학점은 1 미만이거나 30을 초과할 수 없습니다. 유효하지 않은 입력이 발견되면 적절한 오류 메시지를 출력하시오.
> - 사용자로부터 성적 평균(grade average)과 이수 학점(credits)을 입력받도록 작성하시오.
> - "The student can register" 또는 "The student cannot register"와 같이 평가 결과를 알리는 메시지를 출력하시오.

#### 3. 핵심 개념 및 풀이 요약
* **조건 분석 (기존 6.7.3 규칙):**
  * 학점(Credits) 입력 검증: $1 \le \text{credits} \le 30$
  * 등록 가능 조건:
    * `gradeAverage > 350` OR
    * `credits > 12` AND `gradeAverage >= 250` OR
    * `credits <= 12` AND `gradeAverage >= 350`
* **명시적 `CMP` 및 점프 구현:**
  * 고수준 제어 구문 대신 `CMP`와 `JA`, `JBE`, `JMP` 등을 활용한 단락 평가 구조를 설계합니다.

#### 4. 어셈블리 답안 코드
```assembly
INCLUDE Irvine32.inc

.data
promptGrade   BYTE "Enter grade average (0-400): ",0
promptCredits BYTE "Enter credits (1-30): ",0
msgOk         BYTE "The student can register.",0
msgNo         BYTE "The student cannot register.",0
msgErr        BYTE "Error: Credits must be between 1 and 30.",0

gradeAvg DWORD ?
credits  DWORD ?

.code
main PROC
    ; 1. 성적 평균 입력
    mov edx, OFFSET promptGrade
    call WriteString
    call ReadInt
    mov gradeAvg, eax

    ; 2. 학점 입력
    mov edx, OFFSET promptCredits
    call WriteString
    call ReadInt
    mov credits, eax

    ; 3. 학점 범위 검사 (1 <= credits <= 30)
    cmp eax, 1
    jl  InvalidCredits
    cmp eax, 30
    jg  InvalidCredits

    ; 4. 등록 자격 평가
    ; 조건 A: gradeAvg > 350 이면 즉시 가능
    mov eax, gradeAvg
    cmp eax, 350
    jg  CanRegister

    ; 조건 B: credits > 12 이고 gradeAvg >= 250 이면 가능
    mov ebx, credits
    cmp ebx, 12
    jbe CheckElseCondition
    cmp eax, 250
    jge CanRegister
    jmp CannotRegister

CheckElseCondition:
    ; 조건 C: credits <= 12 이고 gradeAvg >= 350 이면 가능
    cmp eax, 350
    jge CanRegister
    jmp CannotRegister

CanRegister:
    mov edx, OFFSET msgOk
    call WriteString
    call Crlf
    jmp Quit

CannotRegister:
    mov edx, OFFSET msgNo
    call WriteString
    call Crlf
    jmp Quit

InvalidCredits:
    mov edx, OFFSET msgErr
    call WriteString
    call Crlf

Quit:
    exit
main ENDP

END main
```

---

### 📍 Exercise 5: Boolean Calculator (1)

#### 1. English Problem
> Create a program that functions as a simple boolean calculator for 32-bit integers. It should display a menu that asks the user to make a selection from the following list:
> 1. x AND y
> 2. x OR y
> 3. NOT x
> 4. x XOR y
> 5. Exit program
>
> When the user makes a choice, call a procedure that displays the name of the operation about to be performed. You must implement this procedure using the **Table-Driven Selection** technique, shown in Section 6.5.4. (You will implement the operations in Exercise 6.)

#### 2. 한국어 해석
> 32비트 정수를 위한 간단한 부울 계산기(Boolean Calculator) 프로그램을 작성하시오. 프로그램은 다음 메뉴 목록을 표시하고 사용자로부터 선택을 받아야 합니다:
> 1. x AND y
> 2. x OR y
> 3. NOT x
> 4. x XOR y
> 5. Exit program
>
> 사용자가 메뉴를 선택하면, 실행될 연산의 이름을 출력하는 프로시저를 호출하시오. 이 프로시저는 Section 6.5.4에 나와 있는 **테이블 기반 선택(Table-Driven Selection)** 기법을 사용하여 구현해야 합니다.

#### 3. 핵심 개념 및 풀이 요약
* **테이블 기반 선택 (Table-Driven Selection):**
  * 선택 키(메뉴 번호)와 실행할 프로시저의 오프셋(포인터)을 구조체 형태의 로드 테이블로 정의합니다.
  * 테이블 항목 예: `BYTE '1', DWORD OFFSET Proc1`
  * 반복문을 통해 입력된 키와 테이블의 키를 비교하여 일치하는 프로시저를 `CALL` 합니다.

#### 4. 어셈블리 답안 코드
```assembly
INCLUDE Irvine32.inc

.data
CaseTable BYTE '1'
          DWORD ProcessAND
          BYTE '2'
          DWORD ProcessOR
          BYTE '3'
          DWORD ProcessNOT
          BYTE '4'
          DWORD ProcessXOR
          BYTE '5'
          DWORD ProcessExit
NumberOfEntries = 5

menu BYTE "--- Boolean Calculator ---",0dh,0ah
     BYTE "1. x AND y",0dh,0ah
     BYTE "2. x OR y",0dh,0ah
     BYTE "3. NOT x",0dh,0ah
     BYTE "4. x XOR y",0dh,0ah
     BYTE "5. Exit program",0dh,0ah
     BYTE "Enter choice: ",0

msgAND  BYTE "Operation Selected: AND",0dh,0ah,0
msgOR   BYTE "Operation Selected: OR",0dh,0ah,0
msgNOT  BYTE "Operation Selected: NOT",0dh,0ah,0
msgXOR  BYTE "Operation Selected: XOR",0dh,0ah,0
msgExit BYTE "Exiting Program...",0dh,0ah,0

.code
main PROC
MenuLoop:
    mov edx, OFFSET menu
    call WriteString
    call ReadChar           ; AL에 입력 문자 수신
    call WriteChar          ; 에코 출력
    call Crlf

    ; 테이블 매칭 기법 적용
    mov ebx, OFFSET CaseTable
    mov ecx, NumberOfEntries

L1:
    cmp al, [ebx]           ; 메뉴 선택 번호 일치 검사
    jne  Next
    call NEAR PTR [ebx + 1] ; 대응되는 프로시저 간접 호출
    jmp  MenuLoop           ; 연산 후 메뉴로 복귀 (Exit 선택 시 내부에서 종료)

Next:
    add ebx, 5              ; 1 바이트(키) + 4 바이트(포인터) = 5바이트 이동
    loop L1

    jmp MenuLoop            ; 잘못된 입력 시 루프 재개
main ENDP

ProcessAND PROC
    mov edx, OFFSET msgAND
    call WriteString
    ret
ProcessAND ENDP

ProcessOR PROC
    mov edx, OFFSET msgOR
    call WriteString
    ret
ProcessOR ENDP

ProcessNOT PROC
    mov edx, OFFSET msgNOT
    call WriteString
    ret
ProcessNOT ENDP

ProcessXOR PROC
    mov edx, OFFSET msgXOR
    call WriteString
    ret
ProcessXOR ENDP

ProcessExit PROC
    mov edx, OFFSET msgExit
    call WriteString
    exit
ProcessExit ENDP

END main
```

---

### 📍 Exercise 6: Boolean Calculator (2)

#### 1. English Problem
> Continue the solution program from Exercise 5 by implementing the following procedures:
> - `AND_op`: Prompt the user for two hexadecimal integers. AND them together and display the result in hexadecimal.
> - `OR_op`: Prompt the user for two hexadecimal integers. OR them together and display the result in hexadecimal.
> - `NOT_op`: Prompt the user for a hexadecimal integer. NOT the integer and display the result in hexadecimal.
> - `XOR_op`: Prompt the user for two hexadecimal integers. Exclusive-OR them together and display the result in hexadecimal.

#### 2. 한국어 해석
> Exercise 5의 솔루션 프로그램을 확장하여 다음 프로시저들을 구현하시오:
> - `AND_op`: 사용자로부터 두 개의 16진수 정수를 입력받아 `AND` 연산을 수행하고 결과를 16진수로 출력합니다.
> - `OR_op`: 사용자로부터 두 개의 16진수 정수를 입력받아 `OR` 연산을 수행하고 결과를 16진수로 출력합니다.
> - `NOT_op`: 사용자로부터 한 개의 16진수 정수를 입력받아 `NOT` 연산을 수행하고 결과를 16진수로 출력합니다.
> - `XOR_op`: 사용자로부터 두 개의 16진수 정수를 입력받아 `XOR` 연산을 수행하고 결과를 16진수로 출력합니다.

#### 3. 핵심 개념 및 풀이 요약
* **Irvine32 16진수 I/O 함수:**
  * `ReadHex`: 사용자로부터 16진수를 입력받아 `EAX`에 저장.
  * `WriteHex`: `EAX` 레지스터의 값을 16진수 형태로 화면에 출력.

#### 4. 어셈블리 답안 코드
```assembly
INCLUDE Irvine32.inc

.data
CaseTable BYTE '1', DWORD OFFSET AND_op
          BYTE '2', DWORD OFFSET OR_op
          BYTE '3', DWORD OFFSET NOT_op
          BYTE '4', DWORD OFFSET XOR_op
          BYTE '5', DWORD OFFSET Exit_op
NumberOfEntries = 5

menu BYTE 0dh, 0ah, "--- Boolean Calculator v2 ---",0dh,0ah
     BYTE "1. x AND y",0dh,0ah
     BYTE "2. x OR y",0dh,0ah
     BYTE "3. NOT x",0dh,0ah
     BYTE "4. x XOR y",0dh,0ah
     BYTE "5. Exit program",0dh,0ah
     BYTE "Choice: ",0

msgPrompt1 BYTE "Enter first hex integer: ",0
msgPrompt2 BYTE "Enter second hex integer: ",0
msgResult  BYTE "Result: ",0

.code
main PROC
MenuLoop:
    mov edx, OFFSET menu
    call WriteString
    call ReadChar
    call WriteChar
    call Crlf

    mov ebx, OFFSET CaseTable
    mov ecx, NumberOfEntries

L1:
    cmp al, [ebx]
    jne  Next
    call NEAR PTR [ebx + 1]
    jmp  MenuLoop

Next:
    add ebx, 5
    loop L1
    jmp MenuLoop
main ENDP

AND_op PROC
    mov edx, OFFSET msgPrompt1
    call WriteString
    call ReadHex
    mov ebx, eax

    mov edx, OFFSET msgPrompt2
    call WriteString
    call ReadHex

    and eax, ebx
    mov edx, OFFSET msgResult
    call WriteString
    call WriteHex
    call Crlf
    ret
AND_op ENDP

OR_op PROC
    mov edx, OFFSET msgPrompt1
    call WriteString
    call ReadHex
    mov ebx, eax

    mov edx, OFFSET msgPrompt2
    call WriteString
    call ReadHex

    or eax, ebx
    mov edx, OFFSET msgResult
    call WriteString
    call WriteHex
    call Crlf
    ret
OR_op ENDP

NOT_op PROC
    mov edx, OFFSET msgPrompt1
    call WriteString
    call ReadHex

    not eax
    mov edx, OFFSET msgResult
    call WriteString
    call WriteHex
    call Crlf
    ret
NOT_op ENDP

XOR_op PROC
    mov edx, OFFSET msgPrompt1
    call WriteString
    call ReadHex
    mov ebx, eax

    mov edx, OFFSET msgPrompt2
    call WriteString
    call ReadHex

    xor eax, ebx
    mov edx, OFFSET msgResult
    call WriteString
    call WriteHex
    call Crlf
    ret
XOR_op ENDP

Exit_op PROC
    exit
Exit_op ENDP

END main
```

---

### 📍 Exercise 7: Probabilities and Colors

#### 1. English Problem
> Write a program that randomly chooses among three different colors for displaying text on the screen. Use a loop to display 20 lines of text, each with a randomly chosen color. The probabilities for each color are to be as follows:
> - White: 30%
> - Blue: 10%
> - Green: 60%
>
> *Suggestion: Generate a random integer between 0 and 9. If the resulting integer falls in the range 0 to 2 (inclusive), choose white. If the integer equals 3, choose blue. If the integer falls in the range 4 to 9 (inclusive), choose green. Test your program by running it ten times, each time observing whether the distribution of line colors appears to match the required probabilities.*

#### 2. 한국어 해석
> 화면에 텍스트를 출력할 때 세 가지 색상 중 하나를 확률에 따라 무작위로 선택하는 프로그램을 작성하시오. 루프를 사용하여 20줄의 텍스트를 출력하되, 각 줄의 색상은 지정된 확률에 맞춰 선택되어야 합니다:
> - 하얀색 (White): 30%
> - 파란색 (Blue): 10%
> - 초록색 (Green): 60%
>
> *제안: 0에서 9 사이의 난수를 생성합니다. 난수가 0~2이면 하얀색, 3이면 파란색, 4~9이면 초록색을 선택하시오.*

#### 3. 핵심 개념 및 상세 풀이
* **확률 구간 매핑 (0 ~ 9 난수):**
  * `RandomRange(10)` 호출 $\rightarrow$ 결과: $0, 1, 2, 3, 4, 5, 6, 7, 8, 9$ (총 10가지)
  * $0 \le \text{Val} \le 2$ (3개 / 10개 = 30%) $\rightarrow$ `white`
  * $\text{Val} == 3$ (1개 / 10개 = 10%) $\rightarrow$ `blue`
  * $4 \le \text{Val} \le 9$ (6개 / 10개 = 60%) $\rightarrow$ `green`
* **콘솔 색상 변경:**
  * `SetTextColor` 프로시저 사용 (`EAX`에 (전경색 + 배경색) 지정).

#### 4. 어셈블리 답안 코드
```assembly
INCLUDE Irvine32.inc

.data
msgText BYTE "This is a line of text with probabilistic color.",0

.code
main PROC
    call Randomize
    mov ecx, 20             ; 20줄 출력

L1:
    mov eax, 10
    call RandomRange        ; EAX = 0 ~ 9

    cmp eax, 2
    jbe ChooseWhite         ; 0 ~ 2 (30%)
    cmp eax, 3
    je  ChooseBlue          ; 3 (10%)

    ; 4 ~ 9 (60%)
    mov eax, green + (black * 16)
    jmp ApplyColor

ChooseWhite:
    mov eax, white + (black * 16)
    jmp ApplyColor

ChooseBlue:
    mov eax, blue + (black * 16)

ApplyColor:
    call SetTextColor
    mov edx, OFFSET msgText
    call WriteString
    call Crlf

    loop L1

    ; 색상 원상복구 (White on Black)
    mov eax, white + (black * 16)
    call SetTextColor

    exit
main ENDP

END main
```

---

### 📍 Exercise 8: Message Encryption

#### 1. English Problem
> Revise the encryption program in Section 6.3.4 in the following manner: Create an encryption key consisting of multiple characters. Use this key to encrypt and decrypt the plaintext by `XOR`ing each character of the key against a corresponding byte in the message. Repeat the key as many times as necessary until all plain text bytes are translated. Suppose, for example, the key were equal to "ABXmv#7". This is how the key would align with the plain text bytes:
>
> | Plain text | T | h | i | s |   | i | s |   | a |   | P | l | a | i | n |   | t | e | x | t |
> | :--- | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
> | **Key** | **A** | **B** | **X** | **m** | **v** | **#** | **7** | **A** | **B** | **X** | **m** | **v** | **#** | **7** | **A** | **B** | **X** | **m** | **v** | **#** |

#### 2. 한국어 해석
> Section 6.3.4의 암호화 프로그램을 수정하여, 다중 문자로 구성된 암호화 키(Key)를 사용하도록 변경하시오. 이 키의 각 문자를 평문의 해당 바이트와 `XOR` 연산하여 암호화 및 복호화를 수행합니다. 평문의 모든 바이트가 변환될 때까지 키를 계속 반복 적용합니다.

#### 3. 핵심 개념 및 풀이 요약
* **다중 바이트 키 순환 (Cyclic Key Indexing):**
  * 평문의 길이를 $N$, 키의 길이를 $K$라 할 때, 평문의 $i$번째 바이트는 키의 $(i \bmod K)$번째 바이트와 `XOR` 수행합니다.
  * 인덱스 변수 `EDI`를 $0 \sim K-1$ 범위로 유지하면서 매 바이트마다 `INC EDI` 후 `CMP EDI, KeySize`로 범위를 벗어나면 $0$으로 초기화합니다.

#### 4. 어셈블리 답안 코드
```assembly
INCLUDE Irvine32.inc

.data
key BYTE "ABXmv#7"
keySize = $ - key

plainText BYTE "This is a secret message that needs to be encrypted!",0
textSize = $ - plainText - 1

msg1 BYTE "Original : ",0
msg2 BYTE "Encrypted: ",0
msg3 BYTE "Decrypted: ",0

.code
main PROC
    ; 1. 원문 출력
    mov edx, OFFSET msg1
    call WriteString
    mov edx, OFFSET plainText
    call WriteString
    call Crlf

    ; 2. 암호화 수행
    mov esi, OFFSET plainText
    mov ecx, textSize
    call TranslateBuffer

    ; 암호화 결과 출력
    mov edx, OFFSET msg2
    call WriteString
    mov edx, OFFSET plainText
    call WriteString
    call Crlf

    ; 3. 복호화 수행 (동일 함수 재호출)
    mov esi, OFFSET plainText
    mov ecx, textSize
    call TranslateBuffer

    ; 복호화 결과 출력
    mov edx, OFFSET msg3
    call WriteString
    mov edx, OFFSET plainText
    call WriteString
    call Crlf

    exit
main ENDP

;-----------------------------------------------------------------
; TranslateBuffer
; 반복 키를 사용하여 ESI가 가리키는 버퍼를 XOR 변환함.
; Receives: ESI = 문자열 포인터, ECX = 문자열 길이
;-----------------------------------------------------------------
TranslateBuffer PROC
    pushad
    mov edi, 0              ; 키 인덱스 (0 ~ keySize-1)

L1:
    mov al, key[edi]        ; 키 바이트 읽기
    xor [esi], al           ; 평문 바이트 XOR 변환
    inc esi                 ; 다음 평문 바이트
    
    inc edi                 ; 다음 키 바이트
    cmp edi, keySize
    jne KeepGoing
    mov edi, 0              ; 키 인덱스 리셋

KeepGoing:
    loop L1

    popad
    ret
TranslateBuffer ENDP

END main
```

---

### 📍 Exercise 9: Validating a PIN

#### 1. English Problem
> Banks use a Personal Identification Number (PIN) to uniquely identify each customer. Let us assume that our bank has a specified range of acceptable values for each digit in its customers’ 5-digit PINs:
>
> | Digit Number | Range |
> | :---: | :---: |
> | 1 | 5 to 9 |
> | 2 | 2 to 5 |
> | 3 | 4 to 8 |
> | 4 | 1 to 4 |
> | 5 | 3 to 6 |
>
> Create a procedure named `Validate_PIN` that receives a pointer to an array of byte containing a 5-digit PIN. Declare two arrays to hold the minimum and maximum range values, and use these arrays to validate each digit of the PIN passed to the procedure. If any digit is found to be outside its valid range, immediately return the digit’s position (between 1 and 5) in the `EAX` register. If the entire PIN is valid, return 0 in `EAX`. Preserve all other register values. Write a test program that calls `Validate_PIN` at least four times using both valid and invalid PINs.

#### 2. 한국어 해석
> 고객의 5자리 PIN 번호 각 자릿수에 대한 유효 범위가 위 표와 같을 때, `Validate_PIN` 프로시저를 작성하시오.
> 최소값 배열과 최대값 배열을 선언하고, 이를 활용하여 PIN의 각 자릿수를 검증합니다. 어떤 자릿수라도 유효 범위를 벗어나면 `EAX` 레지스터에 해당 자릿수 위치(1~5)를 즉시 반환하고 종료합니다. 모든 자릿수가 유효하다면 `EAX`에 0을 반환합니다. `EAX`를 제외한 모든 레지스터 값을 보존하고, 최소 4개 이상의 유효/무효 PIN을 테스트하는 프로그램을 작성하시오.

#### 3. 핵심 개념 및 풀이 요약
* **자릿수별 범주 배열 설정:**
  * `minRange BYTE 5, 2, 4, 1, 3`
  * `maxRange BYTE 9, 5, 8, 4, 6`
* **검증 조건:**
  * 각 자릿수 $i$ ($0 \le i \le 4$)에 대해:  
    $$\text{pin}[i] < \text{minRange}[i] \quad \text{OR} \quad \text{pin}[i] > \text{maxRange}[i]$$
  * 위 조건 충족 시 $EAX = i + 1$ 설정 후 즉시 리턴.

#### 4. 어셈블리 답안 코드
```assembly
INCLUDE Irvine32.inc

.data
minRange BYTE 5, 2, 4, 1, 3
maxRange BYTE 9, 5, 8, 4, 6

pin1 BYTE 5, 2, 4, 1, 3    ; Valid (EAX should be 0)
pin2 BYTE 4, 3, 5, 3, 4    ; Invalid at Digit 1 (EAX should be 1)
pin3 BYTE 6, 4, 5, 3, 5    ; Invalid at Digit 5 (EAX should be 5)
pin4 BYTE 5, 2, 9, 1, 3    ; Invalid at Digit 3 (EAX should be 3)

msgValid   BYTE "PIN is Valid! (EAX = 0)",0dh,0ah,0
msgInvalid BYTE "PIN is Invalid at Digit Position: ",0

.code
main PROC
    ; Test 1
    mov esi, OFFSET pin1
    call Validate_PIN
    call DisplayResult

    ; Test 2
    mov esi, OFFSET pin2
    call Validate_PIN
    call DisplayResult

    ; Test 3
    mov esi, OFFSET pin3
    call Validate_PIN
    call DisplayResult

    ; Test 4
    mov esi, OFFSET pin4
    call Validate_PIN
    call DisplayResult

    exit
main ENDP

;-----------------------------------------------------------------
; DisplayResult
; Validate_PIN 결과(EAX)를 확인하여 메시지 출력
;-----------------------------------------------------------------
DisplayResult PROC
    cmp eax, 0
    je  Valid
    mov edx, OFFSET msgInvalid
    call WriteString
    call WriteDec
    call Crlf
    ret
Valid:
    mov edx, OFFSET msgValid
    call WriteString
    ret
DisplayResult ENDP

;-----------------------------------------------------------------
; Validate_PIN
; Receives: ESI = 5바이트 PIN 배열 포인터
; Returns: EAX = 0 (성공) 또는 오류 자릿수 번호 1~5
;-----------------------------------------------------------------
Validate_PIN PROC
    push ebx
    push ecx
    push edx
    push esi
    push edi

    mov ecx, 5              ; 5자릿수 검사
    mov edi, 0              ; 배열 인덱스 (0~4)

CheckLoop:
    mov al, [esi + edi]     ; PIN 자릿수 가져오기
    
    ; 최소 범위 검사
    cmp al, minRange[edi]
    jb  InvalidDigit

    ; 최대 범위 검사
    cmp al, maxRange[edi]
    ja  InvalidDigit

    inc edi
    loop CheckLoop

    mov eax, 0              ; 모두 통과
    jmp ExitPin

InvalidDigit:
    mov eax, edi            ; 인덱스 0~4
    inc eax                 ; 자릿수 번호 1~5로 변환

ExitPin:
    pop edi
    pop esi
    pop edx
    pop ecx
    pop ebx
    ret
Validate_PIN ENDP

END main
```

---

### 📍 Exercise 10: Parity Checking

#### 1. English Problem
> Data transmission systems and file subsystems often use a form of error detection that relies on calculating the parity (even or odd) of blocks of data. Your task is to create a procedure that returns `True` (1) in the `EAX` register if the bytes in an array contain **even parity**, or `False` (0) if the parity is **odd**. In other words, if you count all the bits set in the entire array, their total count will be even or odd. Preserve all other register values between calls to the procedure. Write a test program that calls your procedure twice, each time passing it a pointer to an array and the length of the array.
>
> *Tip: Earlier in this chapter, we showed how you can repeatedly apply the `XOR` instruction to a sequence of byte values to determine their parity. So, this suggests the use of a loop. But be careful, since some machine instructions affect the Parity flag, and others do not. The code in your loop that checks the parity will have to carefully save and restore the state of the Parity flag to avoid having it unintentionally modified by your code.*

#### 2. 한국어 해석
> 바이트 배열 전체에 포함된 비트 중 `1`의 총 개수가 짝수이면 `EAX` 레지스터에 `True` (1), 홀수이면 `False` (0)를 반환하는 프로시저를 작성하시오. 프로시저 호출 전후로 `EAX` 외의 다른 모든 레지스터 값은 보존되어야 합니다. 최소 10바이트 이상의 짝수 패리티 배열과 홀수 패리티 배열을 각각 생성하여 프로시저를 테스트하는 프로그램을 작성하시오.
>
> *팁: 바이트 값들에 `XOR` 연산을 누적 적용하면 전체 패리티를 판별할 수 있습니다. 단, 연산 과정에서 `POPFD`/`PUSHFD` 등을 통해 Parity Flag(PF)가 명령어에 의해 의도치 않게 오염되지 않도록 세심하게 플래그 상태를 보존/복원해야 합니다.*

#### 3. 핵심 개념 및 상세 풀이
* **누적 XOR 연산을 이용한 전체 패리티 판별:**
  * 바이트 $A$와 $B$를 `XOR` 연산한 결과의 패리티 플래그(PF)는 두 바이트 $1$의 비트 합에 대한 패리티와 일치합니다.
  * 한 레지스터(예: `AL`)에 전체 배열의 바이트를 순차적으로 `XOR` 연산합니다.
* **플래그 보존의 주의점:**
  * `LOOP` 나 `ADD ESI, 1` 등의 명령어는 플래그를 수정하지 않거나 다른 플래그를 변경할 수 있으므로, 누적 `XOR` 연산 직후 `PUSHFD`로 EFLAGS 상태를 보존하고 다음 루프에서 `POPFD`로 복원한 뒤 다음 `XOR`를 수행하거나, 모든 바이트를 `XOR` 한 최종 1바이트 결과의 패리티 플래그(PF) 상태를 확인합니다.
  * 최종 `AL` 값의 하위 8비트에 대한 패리티 상태(`JPO` / `JPE`)를 검사하면 $1$ 또는 $0$을 반환할 수 있습니다.

#### 4. 어셈블리 답안 코드
```assembly
INCLUDE Irvine32.inc

.data
; Even parity array: 1의 총 개수가 짝수인 배열
evenArray BYTE 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
EVEN_SIZE = $ - evenArray

; Odd parity array: 1의 총 개수가 홀수인 배열
oddArray  BYTE 1, 2, 3, 4, 5, 6, 7, 8, 9, 11
ODD_SIZE = $ - oddArray

msgEven BYTE "Even Array Parity Test (Expected 1): ",0
msgOdd  BYTE "Odd Array Parity Test  (Expected 0): ",0

.code
main PROC
    ; Test Even Parity
    mov esi, OFFSET evenArray
    mov ecx, EVEN_SIZE
    call CheckArrayParity
    mov edx, OFFSET msgEven
    call WriteString
    call WriteDec
    call Crlf

    ; Test Odd Parity
    mov esi, OFFSET oddArray
    mov ecx, ODD_SIZE
    call CheckArrayParity
    mov edx, OFFSET msgOdd
    call WriteString
    call WriteDec
    call Crlf

    exit
main ENDP

;-----------------------------------------------------------------
; CheckArrayParity
; 전체 배열 내의 1 비트 총 개수의 짝수/홀수 여부를 판단.
; Receives: ESI = 배열 포인터, ECX = 배열 크기
; Returns: EAX = 1 (Even Parity / True), 0 (Odd Parity / False)
;-----------------------------------------------------------------
CheckArrayParity PROC
    push ebx
    push ecx
    push edx
    push esi

    mov al, 0               ; AL 초기화 (0은 1의 개수가 0개로 짝수 패리티)

ParityLoop:
    xor al, [esi]           ; 바이트 단위 누적 XOR 연산
    inc esi
    loop ParityLoop

    ; 최종 AL 값의 패리티 플래그(PF) 확인
    jp   IsEven             ; Parity Flag = 1 (짝수 패리티)이면 점프

    mov eax, 0              ; 홀수 패리티 (False)
    jmp ExitParity

IsEven:
    mov eax, 1              ; 짝수 패리티 (True)

ExitParity:
    pop esi
    pop edx
    pop ecx
    pop ebx
    ret
CheckArrayParity ENDP

END main
```