# Assembly Language Chapter 10.8 Programming Exercises 해설

이 문서는 Kip Irvine의 *Assembly Language for x86 Processors* 10장(Structures and Macros)의 **10.8 Programming Exercises** 10문제에 대한 **영어 원문, 한국어 번역, 한 줄 한 줄 친절한 주석이 달린 전체 코드**, 그리고 **왕초보 맞춤 기초 가이드**를 정리한 학습 노트입니다.

---

## 📌 시작 전 필수 기초 가이드 (헷갈리는 문법 & 함수 총정리)

코드를 읽기 전에, 어셈블리어에서 처음 보거나 자꾸 까먹기 쉬운 5가지만 먼저 짚고 넘어가세요!

### 1. `mReadkey`는 원래 있는 명령어가 아닙니다!
* **1번 문제의 `mReadkey`**나 **3번의 `mMove32`** 등은 어셈블리어 내장 명령어가 아니라, **문제에서 "이런 이름의 매크로를 직접 만들어봐!"라고 지시해서 우리가 새로 창조하는 매크로 이름**입니다. (보통 매크로 이름 앞에는 `m`을 붙이는 관습이 있습니다.)

### 2. 왜 항상 `dest`가 왼쪽에 올까요?
* `dest`는 **Destination(목적지)**, `source`는 **Source(원본)**의 약자입니다.
* 어셈블리어에서는 `mov dest, source`처럼 **무조건 오른쪽(`source`)의 값을 왼쪽(`dest`)으로 집어넣는(대입하는) 방향**이 철칙입니다. (`dest = source` 순서와 똑같다고 외우시면 됩니다!)

### 3. `IF / ELSEIF` 와 `TYPE arg EQ 1` 완벽 해부
어셈블리어에는 `IF`문이 두 종류가 있습니다.
1. **점 없는 `IF` / `ELSEIF` / `ENDIF` (매크로 전용):** 프로그램이 실행될 때가 아니라, **코드를 번역(어셈블)할 때** 변수 크기 등을 검사해서 알맞은 코드를 골라내는 문법입니다. *(※ 실행 중에 값을 비교할 때는 앞에 점을 찍어 `.IF` / `.ELSEIF`를 씁니다.)*
2. **`IF TYPE arg EQ 1` 해석법:**
   * **`TYPE arg`**: `arg` 변수의 크기가 몇 바이트인지 숫자로 알려줍니다. (`BYTE`=1, `WORD`=2, `DWORD`=4)
   * **`EQ`**: **Eq**ual (`==`, 같다)의 약자입니다.
   * **합치면:** *"만약 `arg` 변수의 크기(`TYPE`)가 `1`바이트와 같다면(`EQ`)..."* 이라는 뜻이 됩니다!

### 4. 자꾸 까먹는 Irvine32 라이브러리 함수 사전 (`call ...`)
메소드(프로시저)가 많아서 헷갈릴 때마다 아래 표를 커닝페이퍼처럼 보세요!

| 함수 이름 | 하는 일 | 호출 전 준비할 것 (입력) | 호출 후 결과 (출력) |
| :--- | :--- | :--- | :--- |
| **`ReadChar`** | 키보드에서 문자 1개 입력받기 (화면에 안 보임) | 없음 | **`AL`** = ASCII 코드, **`AH`** = 스캔 코드 |
| **`WriteChar`** | 화면에 문자 1개 출력하기 | **`AL`** = 출력할 문자 (`'A'`, `','` 등) | 화면에 문자 출력 |
| **`WriteString`** | 화면에 문자열 출력하기 | **`EDX`** = 문자열의 시작 주소 (`OFFSET`) | 화면에 문자열 출력 |
| **`SetTextColor`** | 콘솔 글자색 바꾸기 | **`EAX`** = 색상 이름 (`white`, `yellow` 등) | 이후 출력되는 글자색 변경 |
| **`Randomize`** | 실행할 때마다 다른 난수가 나오게 시드 초기화 | 없음 | 난수 시드 설정 |
| **`RandomRange`** | `0` ~ `(N - 1)` 사이의 랜덤 숫자 뽑기 | **`EAX`** = 범위 숫자 `N` | **`EAX`** = 뽑힌 랜덤 숫자 |
| **`ReadInt`** | 키보드로 부호 있는 정수(`+`, `-`) 입력받기 | 없음 | **`EAX`** = 입력받은 32비트 정수 |
| **`WriteInt`** | 부호 있는 정수(`+`, `-`)를 화면에 출력하기 | **`EAX`** = 출력할 32비트 정수 | 화면에 `+10`, `-42` 형태로 출력 |
| **`WriteDec`** | 부호 없는 양수(10진수)를 화면에 출력하기 | **`EAX`** = 출력할 32비트 양수 | 화면에 `10`, `25` 형태로 출력 |
| **`WriteHex`** | 16진수 형태로 화면에 출력하기 | **`EAX`** = 출력할 32비트 숫자 | 화면에 `12345678` 형태로 출력 |
| **`Crlf`** | 줄바꿈 (엔터키 효과) | 없음 | 커서가 다음 줄로 이동 |

---

## 💻 10.8 Programming Exercises 문제 원문, 번역 및 코드

### 1. `mReadkey` Macro (난이도 ★)
> **영어 원문:** Create a macro that waits for a keystroke and returns the key that was pressed. The macro should include parameters for the ASCII code and keyboard scan code. Hint: Call `ReadChar` from the book’s link library. Write a program that tests your macro.
>
> **한국어 번역:** 키 입력을 기다렸다가 눌린 키를 반환하는 매크로를 작성하시오. 매크로는 ASCII 코드와 키보드 스캔 코드(Scan code)를 위한 매개변수를 포함해야 합니다. (힌트: 책의 링크 라이브러리에 있는 `ReadChar`를 호출하시오.) 작성한 매크로를 테스트하는 프로그램을 작성하시오.

* **정답 및 테스트 코드:**
```masm
INCLUDE Irvine32.inc

; [매크로 정의] mReadkey라는 이름의 매크로를 새로 만듭니다.
mReadkey MACRO asciiCode, scanCode
    push eax                ; 기존 EAX 레지스터 값을 스택에 임시 보관(보호)
    call ReadChar           ; 키 입력 대기 -> 눌리면 AL에 아스키코드, AH에 스캔코드가 들어옴
    mov asciiCode, al       ; 오른쪽(AL)의 아스키코드를 왼쪽(asciiCode 변수)에 저장
    mov scanCode, ah        ; 오른쪽(AH)의 스캔코드를 왼쪽(scanCode 변수)에 저장
    pop eax                 ; 임시 보관했던 EAX 값 원래대로 복구
ENDM

.data
ascii BYTE ?                ; 아스키코드를 담을 1바이트 빈 상자
scan  BYTE ?                ; 스캔코드를 담을 1바이트 빈 상자
msg1  BYTE "Press any key... ", 0
msg2  BYTE "ASCII: ", 0
msg3  BYTE "  Scan Code: ", 0

.code
main PROC
    mov edx, OFFSET msg1    ; 출력할 문자열 주소를 EDX에 넣음
    call WriteString        ; "Press any key... " 출력

    mReadkey ascii, scan    ; 우리가 만든 매크로 호출! (키 입력 대기)

    call Crlf               ; 줄바꿈 (엔터)
    mov edx, OFFSET msg2
    call WriteString        ; "ASCII: " 출력
    mov al, ascii           ; 저장된 아스키코드를 AL에 넣음
    call WriteChar          ; 문자 1개 화면 출력

    mov edx, OFFSET msg3
    call WriteString        ; "  Scan Code: " 출력
    movzx eax, scan         ; 1바이트 scan 값을 0으로 채우며(MOVZX) 4바이트 EAX로 복사
    call WriteDec           ; 스캔 코드를 10진수 숫자로 출력
    call Crlf

    exit
main ENDP
END main
```

---

### 2. `mWritestringAttr` Macro (난이도 ★)
> **영어 원문:** (Requires reading ahead to Section 11.1.11.) Create a macro that writes a null-terminated string to the console with a given text color. The macro parameters should include the string name and the color. Hint: Call `SetTextColor` from the book’s link library. Write a program that tests your macro with several strings in different colors.
>
> **한국어 번역:** (11.1.11절 예습 필요) 주어진 텍스트 색상으로 널 종료 문자열(끝에 0이 붙은 문자열)을 콘솔에 출력하는 매크로를 작성하시오. 매크로 매개변수에는 문자열 이름과 색상이 포함되어야 합니다. (힌트: 책의 링크 라이브러리에 있는 `SetTextColor`를 호출하시오.) 여러 문자열을 다양한 색상으로 출력하며 매크로를 테스트하는 프로그램을 작성하시오.

* **정답 및 테스트 코드:**
```masm
INCLUDE Irvine32.inc

; [매크로 정의] 문자열 이름(strName)과 색상(color)을 받아 색깔 글씨를 출력
mWritestringAttr MACRO strName, color
    push eax                ; EAX 보호
    push edx                ; EDX 보호
    mov eax, color          ; 색상 상수를 EAX에 넣음 (SetTextColor의 약속)
    call SetTextColor       ; 콘솔 글자색 변경!
    mov edx, OFFSET strName ; 문자열의 시작 주소를 EDX에 넣음 (WriteString의 약속)
    call WriteString        ; 변경된 색상으로 문자열 출력!
    pop edx                 ; EDX 복구
    pop eax                 ; EAX 복구
ENDM

.data
str1 BYTE "This is White text.", 0dh, 0ah, 0        ; 0dh, 0ah는 줄바꿈, 0은 문자열 끝
str2 BYTE "This is Yellow text.", 0dh, 0ah, 0
str3 BYTE "This is LightGreen text.", 0dh, 0ah, 0

.code
main PROC
    mWritestringAttr str1, white        ; 흰색으로 str1 출력
    mWritestringAttr str2, yellow       ; 노란색으로 str2 출력
    mWritestringAttr str3, lightGreen   ; 연두색으로 str3 출력

    ; 출력이 끝난 후 콘솔 색상을 원래 기본값(밝은 회색)으로 되돌림
    mov eax, lightGray
    call SetTextColor
    exit
main ENDP
END main
```

---

### 3. `mMove32` Macro (난이도 ★)
> **영어 원문:** Write a macro named `mMove32` that receives two 32-bit memory operands. The macro should move the source operand to the destination operand. Write a program that tests your macro.
>
> **한국어 번역:** 두 개의 32비트 메모리 피연산자(변수)를 받는 `mMove32`라는 매크로를 작성하시오. 이 매크로는 원본(source) 피연산자의 값을 목적지(destination) 피연산자로 이동시켜야 합니다. 매크로를 테스트하는 프로그램을 작성하시오.

* **정답 및 테스트 코드:**
```masm
INCLUDE Irvine32.inc

; [매크로 정의] 오른쪽(source) 메모리 값을 왼쪽(dest) 메모리로 복사
mMove32 MACRO dest, source
    push eax                ; 징검다리로 쓸 EAX의 기존 값 보호
    mov eax, source         ; 1단계: 메모리(source) -> 레지스터(EAX)로 이동
    mov dest, eax           ; 2단계: 레지스터(EAX) -> 메모리(dest)로 이동
    pop eax                 ; EAX 복구
ENDM

.data
val1 DWORD 12345678h        ; 원본 데이터 (16진수)
val2 DWORD 0                ; 복사받을 목적지 데이터

.code
main PROC
    mMove32 val2, val1      ; val2(목적지, 왼쪽)에 val1(원본, 오른쪽) 값을 복사!

    mov eax, val2           ; 잘 복사되었는지 확인하기 위해 EAX에 넣음
    call WriteHex           ; 화면에 12345678 출력
    call Crlf
    exit
main ENDP
END main
```

---

### 4. `mMult32` Macro (난이도 ★)
> **영어 원문:** Create a macro named `mMult32` that multiplies two 32-bit memory operands and produces a 32-bit product. Write a program that tests your macro.
>
> **한국어 번역:** 두 개의 32비트 메모리 피연산자를 곱하여 32비트 곱셈 결과를 생성하는 `mMult32`라는 매크로를 작성하시오. 매크로를 테스트하는 프로그램을 작성하시오.

* **정답 및 테스트 코드:**
```masm
INCLUDE Irvine32.inc

; [매크로 정의] dest = dest * source (왼쪽 변수에 오른쪽 변수를 곱해서 다시 왼쪽에 저장)
mMult32 MACRO dest, source
    push eax                ; EAX 보호
    mov eax, dest           ; 첫 번째 숫자를 EAX에 가져옴
    imul eax, source        ; EAX = EAX * source (부호 있는 32비트 곱셈)
    mov dest, eax           ; 계산된 결과를 왼쪽(dest) 변수에 덮어씀
    pop eax                 ; EAX 복구
ENDM

.data
num1 SDWORD 250             ; 부호 있는 32비트 정수 250
num2 SDWORD -4              ; 부호 있는 32비트 정수 -4

.code
main PROC
    mMult32 num1, num2      ; num1 = 250 * (-4) = -1000

    mov eax, num1           ; 결과 확인을 위해 EAX로 이동
    call WriteInt           ; 화면에 -1000 출력
    call Crlf
    exit
main ENDP
END main
```

---

### 5. `mReadInt` Macro (난이도 ★★)
> **영어 원문:** Create a macro named `mReadInt` that reads a 16- or 32-bit signed integer from standard input and returns the value in an argument. Use conditional operators to allow the macro to adapt to the size of the desired result. Write a program that tests the macro, passing it operands of various sizes.
>
> **한국어 번역:** 표준 입력(키보드)으로부터 16비트 또는 32비트 부호 있는 정수를 읽어 매개변수에 반환하는 `mReadInt` 매크로를 작성하시오. 조건부 연산자를 사용하여 원하는 결과의 크기(16비트 또는 32비트)에 맞게 매크로가 적응하도록 하시오. 다양한 크기의 피연산자를 전달하여 매크로를 테스트하는 프로그램을 작성하시오.

* **정답 및 테스트 코드:**
```masm
INCLUDE Irvine32.inc

; [매크로 정의] 입력받은 변수의 크기(2바이트 vs 4바이트)를 스스로 판단해 저장
mReadInt MACRO intVal
    push eax                ; EAX 보호
    call ReadInt            ; 키보드로 정수 입력받음 -> 결과는 무조건 EAX(32비트)에 들어옴

    IF TYPE intVal EQ 2     ; 만약 intVal의 크기(TYPE)가 2바이트(WORD/SWORD)와 같다면(EQ)
        mov intVal, ax      ; EAX의 하위 절반인 16비트 AX만 변수에 저장!
    ELSEIF TYPE intVal EQ 4 ; 그렇지 않고 만약 크기가 4바이트(DWORD/SDWORD)와 같다면(EQ)
        mov intVal, eax     ; 32비트 EAX 전체를 변수에 저장!
    ELSE                    ; 둘 다 아니면 어셈블(빌드) 콘솔에 에러 메시지 출력
        ECHO Error: mReadInt requires a 16-bit or 32-bit operand!
    ENDIF

    pop eax                 ; EAX 복구
ENDM

.data
val16 SWORD ?               ; 16비트(2바이트) 부호 있는 정수 변수
val32 SDWORD ?              ; 32비트(4바이트) 부호 있는 정수 변수
prompt1 BYTE "Enter a 16-bit signed integer: ", 0
prompt2 BYTE "Enter a 32-bit signed integer: ", 0

.code
main PROC
    mov edx, OFFSET prompt1
    call WriteString
    mReadInt val16          ; 2바이트 변수를 넘겨줌 -> 매크로가 알아서 AX로 처리!

    mov edx, OFFSET prompt2
    call WriteString
    mReadInt val32          ; 4바이트 변수를 넘겨줌 -> 매크로가 알아서 EAX로 처리!

    ; 입력받은 값 출력 확인
    movsx eax, val16        ; 16비트 음수 부호를 유지한 채 32비트 EAX로 확장
    call WriteInt
    call Crlf

    mov eax, val32
    call WriteInt
    call Crlf
    exit
main ENDP
END main
```

---

### 6. `mWriteInt` Macro (난이도 ★★)
> **영어 원문:** Create a macro named `mWriteInt` that writes a signed integer to standard output by calling the `WriteInt` library procedure. The argument passed to the macro can be a byte, word, or doubleword. Use conditional operators in the macro so it adapts to the size of the argument. Write a program that tests the macro, passing it arguments of different sizes.
>
> **한국어 번역:** `WriteInt` 라이브러리 프로시저를 호출하여 부호 있는 정수를 표준 출력(화면)에 출력하는 `mWriteInt` 매크로를 작성하시오. 매크로에 전달되는 인수는 바이트(`BYTE`), 워드(`WORD`), 또는 더블워드(`DWORD`)일 수 있습니다. 인수의 크기에 맞게 동작하도록 매크로 내에 조건부 연산자를 사용하시오. 서로 다른 크기의 인수를 전달하여 매크로를 테스트하는 프로그램을 작성하시오.

* **정답 및 테스트 코드:**
```masm
INCLUDE Irvine32.inc

; [매크로 정의] 1, 2, 4바이트 어떤 정수 변수가 오든 부호를 살려서 화면에 출력
mWriteInt MACRO intVal
    push eax                ; EAX 보호

    IF TYPE intVal EQ 1     ; 1바이트(SBYTE)라면
        movsx eax, intVal   ; 부호 확장(MOVSX)하여 32비트 EAX에 담음
    ELSEIF TYPE intVal EQ 2 ; 2바이트(SWORD)라면
        movsx eax, intVal   ; 부호 확장(MOVSX)하여 32비트 EAX에 담음
    ELSEIF TYPE intVal EQ 4 ; 4바이트(SDWORD)라면
        mov eax, intVal     ; 크기가 같으므로 그대로 EAX에 담음
    ELSE
        ECHO Error: Invalid operand size for mWriteInt!
        EXITM               ; 잘못된 크기면 매크로 확장 즉시 중단
    ENDIF

    call WriteInt           ; EAX에 담긴 부호 있는 정수를 화면에 출력!
    pop eax                 ; EAX 복구
ENDM

.data
bVal SBYTE -42              ; 1바이트 음수
wVal SWORD -1234            ; 2바이트 음수
dVal SDWORD 987654          ; 4바이트 양수

.code
main PROC
    mWriteInt bVal          ; 1바이트 출력 (-42)
    call Crlf
    mWriteInt wVal          ; 2바이트 출력 (-1234)
    call Crlf
    mWriteInt dVal          ; 4바이트 출력 (+987654)
    call Crlf
    exit
main ENDP
END main
```

---

### 7. The Professor’s Lost Phone (난이도 ★★★)
> **영어 원문:** When the professor took the drunkard's walk around campus in Section 10.1.6, we discovered that he lost his cell phone somewhere along the path. When you simulate the drunken walk, your program must drop the phone wherever the professor is standing at some random time interval. Each time you run the program, the cell phone will be lost at a different time interval (and location).
>
> **한국어 번역:** 10.1.6절에서 교수가 캠퍼스 주변을 취객처럼 비틀거리며 걸었을 때, 우리는 그가 길 어딘가에서 휴대폰을 잃어버렸다는 것을 알게 되었습니다. 취객의 산책을 시뮬레이션할 때, 프로그램은 임의의(랜덤한) 시간 간격(걸음 수)에 교수가 서 있는 위치에 휴대폰을 떨어뜨려야 합니다. 프로그램을 실행할 때마다 휴대폰을 잃어버리는 시점(및 위치)이 달라져야 합니다.

* **정답 및 테스트 코드:**
```masm
INCLUDE Irvine32.inc

WalkMax = 30                ; 총 걸음 수 (30걸음)
StartX = 25                 ; 시작 X 좌표
StartY = 25                 ; 시작 Y 좌표

DrunkardWalk STRUCT         ; 교수의 이동 경로를 기록할 구조체
    path COORD WalkMax DUP(<0,0>)
    pathsUsed WORD 0
DrunkardWalk ENDS

.data
aWalk DrunkardWalk <>
lostStep DWORD ?            ; 휴대폰을 떨어뜨릴 랜덤 걸음 번호 (0 ~ 29)
msgLost BYTE ">>> Professor dropped his phone at step ", 0
msgCoord BYTE " at coordinates: ", 0

.code
main PROC
    call Randomize          ; 실행할 때마다 다른 난수가 나오도록 시드 초기화
    mov eax, WalkMax        ; 난수 범위 설정 (30)
    call RandomRange        ; 0 ~ 29 사이의 랜덤 숫자 생성
    mov lostStep, eax       ; 이번 실행에서 휴대폰을 잃어버릴 걸음 번호 저장!

    mov esi, OFFSET aWalk
    call TakeDrunkenWalk
    exit
main ENDP

TakeDrunkenWalk PROC
    pushad                  ; 모든 범용 레지스터 보호
    mov edi, esi
    add edi, OFFSET DrunkardWalk.path
    mov ecx, WalkMax        ; 30번 반복 설정
    mov ebx, StartX         ; 현재 X 좌표 (25)
    mov edx, StartY         ; 현재 Y 좌표 (25)
    mov esi, 0              ; 현재 몇 번째 걸음인지 세는 카운터 (0부터 시작)

Again:
    mov (COORD PTR [edi]).X, bx     ; 현재 X 좌표 구조체에 기록
    mov (COORD PTR [edi]).Y, dx     ; 현재 Y 좌표 구조체에 기록

    ; 현재 좌표 화면에 출력 (예: 25,25 )
    mov eax, ebx
    call WriteDec
    mov al, ','
    call WriteChar
    mov eax, edx
    call WriteDec
    mov al, ' '
    call WriteChar

    ; [핵심] 현재 걸음(ESI)이 휴대폰을 잃어버릴 시점(lostStep)인지 검사!
    cmp esi, lostStep
    jne ContinueWalk        ; 다르면 그냥 계속 걸음

    ; 같다면 휴대폰 분실 메시지 및 좌표 출력!
    call Crlf
    push edx                ; WriteString을 위해 EDX를 잠시 써야 하므로 Y좌표 보호
    mov edx, OFFSET msgLost
    call WriteString
    mov eax, esi
    call WriteDec           ; 몇 번째 걸음인지 출력
    mov edx, OFFSET msgCoord
    call WriteString
    pop edx                 ; Y좌표 복구

    mov eax, ebx
    call WriteDec           ; 분실 X좌표 출력
    mov al, ','
    call WriteChar
    mov eax, edx
    call WriteDec           ; 분실 Y좌표 출력
    call Crlf

ContinueWalk:
    mov eax, 4
    call RandomRange        ; 0~3 랜덤 방향 뽑기 (0:북, 1:남, 2:서, 3:동)
    .IF eax == 0
        dec edx             ; 북쪽 이동 (Y 감소)
    .ELSEIF eax == 1
        inc edx             ; 남쪽 이동 (Y 증가)
    .ELSEIF eax == 2
        dec ebx             ; 서쪽 이동 (X 감소)
    .ELSE
        inc ebx             ; 동쪽 이동 (X 증가)
    .ENDIF

    add edi, TYPE COORD     ; 다음 좌표 저장 공간으로 이동 (+4바이트)
    inc esi                 ; 걸음 수 +1 증가
    loop Again

    call Crlf
    popad
    ret
TakeDrunkenWalk ENDP
END main
```

---

### 8. Drunkard’s Walk with Probabilities (난이도 ★★★★)
> **영어 원문:** When testing the `DrunkardWalk` program, you may have noticed that the professor doesn’t seem to wander very far from the starting point. This is no doubt caused by an equal probability of the professor moving in any direction. Modify the program so there is a 50% probability the professor will continue to walk in the same direction as he or she did when taking the previous step. There should be a 10% probability that he or she will reverse direction and a 20% probability that he or she will turn either right or left. Assign a default starting direction before the loop begins.
>
> **한국어 번역:** `DrunkardWalk` 프로그램을 테스트할 때, 교수가 시작 지점에서 별로 멀리 벗어나지 않는 것을 눈치챘을 것입니다. 이는 교수가 어느 방향으로든 똑같은 확률로 움직이기 때문입니다. 이전 걸음과 **같은 방향으로 계속 걸을 확률 50%**, **반대 방향으로 되돌아갈 확률 10%**, **우회전할 확률 20%**, **좌회전할 확률 20%**가 되도록 프로그램을 수정하시오. 루프가 시작되기 전에 기본 시작 방향을 지정하시오.

* **정답 및 테스트 코드:**
```masm
INCLUDE Irvine32.inc

WalkMax = 30
StartX = 25
StartY = 25

; 시계 방향으로 방향 번호 정의 (0 -> 1 -> 2 -> 3)
DIR_NORTH = 0               ; 북쪽
DIR_EAST  = 1               ; 동쪽 (북쪽에서 우회전)
DIR_SOUTH = 2               ; 남쪽 (북쪽에서 반대 방향)
DIR_WEST  = 3               ; 서쪽 (북쪽에서 좌회전)

.data
currDir DWORD DIR_NORTH     ; 루프 시작 전 기본 시작 방향: 북쪽(0)

.code
main PROC
    call Randomize
    mov ecx, WalkMax        ; 30걸음 반복
    mov ebx, StartX         ; 현재 X 좌표
    mov edx, StartY         ; 현재 Y 좌표

WalkLoop:
    ; 현재 좌표 출력
    mov eax, ebx
    call WriteDec
    mov al, ','
    call WriteChar
    mov eax, edx
    call WriteDec
    mov al, ' '
    call WriteChar

    ; 0 ~ 99 사이의 난수를 뽑아 100% 확률 계산
    mov eax, 100
    call RandomRange        ; EAX에 0 ~ 99 들어옴

    .IF eax < 50            ; 0 ~ 49 (50% 확률): 직진 (currDir 그대로 유지)
        ; 아무것도 변경하지 않음
    .ELSEIF eax < 60        ; 50 ~ 59 (10% 확률): 유턴 (반대 방향 = +2)
        add currDir, 2
    .ELSEIF eax < 80        ; 60 ~ 79 (20% 확률): 우회전 (시계 방향 한 칸 = +1)
        inc currDir
    .ELSE                   ; 80 ~ 99 (20% 확률): 좌회전 (반시계 한 칸 = +3)
        add currDir, 3
    .ENDIF

    and currDir, 3          ; 4로 나눈 나머지를 구해 항상 0~3 범위로 순환시킴!

    ; 결정된 방향(currDir)에 따라 실제 좌표 이동
    .IF currDir == DIR_NORTH
        dec edx             ; 북쪽 (Y 감소)
    .ELSEIF currDir == DIR_EAST
        inc ebx             ; 동쪽 (X 증가)
    .ELSEIF currDir == DIR_SOUTH
        inc edx             ; 남쪽 (Y 증가)
    .ELSE                   ; DIR_WEST
        dec ebx             ; 서쪽 (X 감소)
    .ENDIF

    loop WalkLoop
    call Crlf
    exit
main ENDP
END main
```

---

### 9. Shifting Multiple Doublewords (난이도 ★★★)
> **영어 원문:** Create a macro that shifts an array of 32-bit integers a variable number of bits in either direction, using the `SHRD` and `SHLD` instructions. Write a test program that tests your macro by shifting the same array in both directions and displaying the resulting values. You can assume that the array is in little-endian order.
>
> **한국어 번역:** `SHRD`와 `SHLD` 명령어를 사용하여 32비트 정수 배열을 어느 방향으로든 가변적인 비트 수만큼 시프트(비트 이동)하는 매크로를 작성하시오. 같은 배열을 양방향으로 시프트하고 결과 값을 출력하여 매크로를 테스트하는 프로그램을 작성하시오. 배열은 리틀 엔디언(Little-endian) 순서라고 가정할 수 있습니다.

* **정답 및 테스트 코드:**
```masm
INCLUDE Irvine32.inc

; [매크로 정의] 배열 전체를 하나의 긴 비트열처럼 연결해 왼쪽(L) 또는 오른쪽(R)으로 시프트
mShiftDoublewords MACRO arrayName, direction, numberOfBits
    LOCAL L_Left, L_Right   ; 반복문 레이블 중복 방지
    pushad                  ; 모든 레지스터 보호

    IFIDNI <direction>, <L> ; 대소문자 무시(I)하고 direction이 'L'(왼쪽)인지 비교
        ; 왼쪽(L) 시프트: 가장 큰 인덱스(상위)부터 1번 인덱스까지 내려오며 처리
        mov esi, (LENGTHOF arrayName - 1) * TYPE arrayName
        mov ecx, LENGTHOF arrayName - 1
    L_Left:
        mov eax, arrayName[esi - TYPE arrayName]    ; 바로 아래 칸의 비트를 EAX에 준비
        push ecx
        mov cl, numberOfBits                        ; 시프트할 비트 수는 CL에 넣어야 함
        shld arrayName[esi], eax, cl                ; 왼쪽으로 밀고 빈자리를 EAX 비트로 채움!
        pop ecx
        sub esi, TYPE arrayName                     ; 이전 칸으로 인덱스 이동 (-4바이트)
        loop L_Left

        ; 가장 하위(0번) 칸은 채워줄 아래 칸이 없으므로 일반 SHL 사용
        mov cl, numberOfBits
        shl arrayName[0], cl

    ELSEIFIDNI <direction>, <R> ; direction이 'R'(오른쪽)인지 비교
        ; 오른쪽(R) 시프트: 0번 인덱스(하위)부터 올라가며 처리
        mov esi, 0
        mov ecx, LENGTHOF arrayName - 1
    L_Right:
        mov eax, arrayName[esi + TYPE arrayName]    ; 바로 위 칸의 비트를 EAX에 준비
        push ecx
        mov cl, numberOfBits
        shrd arrayName[esi], eax, cl                ; 오른쪽으로 밀고 빈자리를 EAX 비트로 채움!
        pop ecx
        add esi, TYPE arrayName                     ; 다음 칸으로 인덱스 이동 (+4바이트)
        loop L_Right

        ; 가장 상위(마지막) 칸은 채워줄 위 칸이 없으므로 일반 SHR 사용
        mov cl, numberOfBits
        shr arrayName[esi], cl
    ENDIF

    popad                   ; 모든 레지스터 복구
ENDM

.data
myArr DWORD 11111111h, 22222222h, 33333333h

.code
main PROC
    ; 1. 오른쪽(R)으로 4비트 시프트 후 출력
    mShiftDoublewords myArr, R, 4
    call DisplayArray

    ; 2. 다시 왼쪽(L)으로 4비트 시프트 후 출력 (원래대로 복구됨)
    mShiftDoublewords myArr, L, 4
    call DisplayArray
    exit
main ENDP

; 배열의 모든 원소를 16진수로 출력하는 보조 함수
DisplayArray PROC
    mov esi, 0
    mov ecx, LENGTHOF myArr
L1:
    mov eax, myArr[esi]
    call WriteHex
    mov al, ' '
    call WriteChar
    add esi, TYPE myArr
    loop L1
    call Crlf
    ret
DisplayArray ENDP
END main
```

---

### 10. Three-Operand Instructions (난이도 ★★★)
> **영어 원문:** In the following macros, assume `EAX` is reserved for macro operations and is not preserved. Other registers modified by the macro must be preserved. All parameters are signed memory doublewords. Write macros that simulate the following operations:
> `a. add3 destination, source1, source2`
> `b. sub3 destination, source1, source2`
> `c. mul3 destination, source1, source2`
> `d. div3 destination, source1, source2`
> Write a program that tests your macros by implementing four arithmetic expressions, each involving multiple operations.
>
> **한국어 번역:** 다음 매크로들에서 `EAX`는 매크로 연산용으로 예약되어 있으며 보존되지 않는다고 가정하시오. 매크로에 의해 수정되는 **다른 레지스터들은 반드시 보존되어야 합니다.** 모든 매개변수는 부호 있는 32비트 메모리 더블워드(`SDWORD`)입니다. 다음 연산들을 시뮬레이션하는 매크로(`add3`, `sub3`, `mul3`, `div3`)를 작성하고, 각각 여러 연산을 포함하는 4가지 산술 표현식을 구현하여 매크로를 테스트하는 프로그램을 작성하시오.

* **정답 및 테스트 코드:**
```masm
INCLUDE Irvine32.inc

; a. destination = source1 + source2
add3 MACRO destination, source1, source2
    mov eax, source1        ; EAX는 보존하지 않아도 된다고 문제에 명시됨
    add eax, source2        ; EAX = source1 + source2
    mov destination, eax    ; 결과를 왼쪽 destination 변수에 저장
ENDM

; b. destination = source1 - source2
sub3 MACRO destination, source1, source2
    mov eax, source1
    sub eax, source2        ; EAX = source1 - source2
    mov destination, eax
ENDM

; c. destination = source1 * source2
mul3 MACRO destination, source1, source2
    mov eax, source1
    imul eax, source2       ; EAX = source1 * source2 (부호 있는 곱셈)
    mov destination, eax
ENDM

; d. destination = source1 / source2
div3 MACRO destination, source1, source2
    push edx                ; [중요!] IDIV가 EDX를 덮어쓰므로 EDX는 반드시 보존해야 함!
    mov eax, source1
    cdq                     ; EAX의 부호 비트를 EDX로 확장 (부호 있는 나눗셈 필수 준비)
    idiv source2            ; EAX = (EDX:EAX) / source2 (몫은 EAX, 나머지는 EDX)
    mov destination, eax    ; 몫(EAX)을 destination에 저장
    pop edx                 ; 보호했던 EDX 원래대로 복구!
ENDM

.data
w SDWORD 10
x SDWORD ?
y SDWORD 20
z SDWORD 5
temp SDWORD ?               ; 중간 계산 결과를 임시로 담을 변수

.code
main PROC
    ; 수식 1: x = (w + y) * z  -> (10 + 20) * 5 = 150
    add3 temp, w, y         ; temp = 10 + 20 = 30
    mul3 x, temp, z         ; x = 30 * 5 = 150
    mov eax, x
    call WriteInt
    call Crlf

    ; 수식 2: x = (y - w) / z  -> (20 - 10) / 5 = 2
    sub3 temp, y, w         ; temp = 20 - 10 = 10
    div3 x, temp, z         ; x = 10 / 5 = 2
    mov eax, x
    call WriteInt
    call Crlf

    ; 수식 3: x = (w * y) + z  -> (10 * 20) + 5 = 205
    mul3 temp, w, y         ; temp = 10 * 20 = 200
    add3 x, temp, z         ; x = 200 + 5 = 205
    mov eax, x
    call WriteInt
    call Crlf

    ; 수식 4: x = (y / z) - w  -> (20 / 5) - 10 = -6
    div3 temp, y, z         ; temp = 20 / 5 = 4
    sub3 x, temp, w         ; x = 4 - 10 = -6
    mov eax, x
    call WriteInt
    call Crlf

    exit
main ENDP
END main
```