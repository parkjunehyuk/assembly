# 🧮 두 수의 사칙연산 및 레지스터(AX, BX, CX, DX) 출력 프로그램

x86 어셈블리어(Irvine32 라이브러리)를 사용하여 **사용자로부터 두 개의 정수를 입력받고, 사칙연산(덧셈, 뺄셈, 곱셈, 나눗셈)의 결과와 주요 레지스터(`EAX`, `EBX`, `ECX`, `EDX` 및 하위 16비트 `AX`, `BX`, `CX`, `DX`)의 값을 화면에 출력**하는 프로그램입니다.

---

## 1. 전체 소스 코드 (`arithmethic_regs.asm`)

```assembly
INCLUDE Irvine32.inc            ; Irvine32 기본 입출력 함수(WriteString, ReadInt, DumpRegs 등) 불러오기

.data                           ; [데이터 영역] 프로그램에서 쓸 문자열과 변수 선언
    prompt1 BYTE "First number: ", 0                  ; 0은 문자열의 끝(Null) 표시
    prompt2 BYTE "Second number (non-zero): ", 0

    msgAdd  BYTE "[1. Add] num1 + num2 = ", 0
    msgSub  BYTE "[2. Sub] num1 - num2 = ", 0
    msgMul  BYTE "[3. Mul] num1 * num2 = ", 0
    msgDivQ BYTE "[4. Div] Quotient = ", 0            ; 나눗셈의 '몫'
    msgDivR BYTE "  Remainder = ", 0                  ; 나눗셈의 '나머지'

    num1    SDWORD ?            ; 첫 번째 숫자를 저장할 32비트 변수 (?는 비어있다는 뜻)
    num2    SDWORD ?            ; 두 번째 숫자를 저장할 32비트 변수

.code                           ; [코드 영역] 실제 실행할 명령어 작성
main PROC                       ; 프로그램의 시작점 (main 함수)

    ; ==========================================================
    ; 0단계: 두 개의 숫자 입력받기
    ; ==========================================================
    mov  edx, OFFSET prompt1    ; 'prompt1' 문자열의 시작 주소를 EDX에 넣음 (출력 준비)
    call WriteString            ; EDX 주소에 있는 글자("First number: ")를 화면에 출력
    call ReadInt                ; 키보드로 숫자 입력받기 -> 입력된 숫자는 자동으로 EAX에 들어감!
    mov  num1, eax              ; EAX에 들어온 첫 번째 숫자를 num1 변수에 보관

    mov  edx, OFFSET prompt2    ; 'prompt2' 문자열의 시작 주소를 EDX에 넣음
    call WriteString            ; 화면에 "Second number (non-zero): " 출력
    call ReadInt                ; 두 번째 숫자 입력받기 -> 자동으로 EAX에 들어감!
    mov  num2, eax              ; EAX에 들어온 두 번째 숫자를 num2 변수에 보관
    call Crlf                   ; 엔터키(줄바꿈) 한 번 출력

    ; ==========================================================
    ; 1단계: 덧셈 (ADD) 및 레지스터 출력
    ; ==========================================================
    mov  eax, num1              ; num1을 EAX로 가져옴
    mov  ebx, num2              ; num2를 EBX로 가져옴
    add  eax, ebx               ; EAX = EAX + EBX (덧셈 결과가 EAX에 저장됨!)

    push eax                    ; WriteString이 EAX 값을 망가뜨리지 않게 스택에 잠시 숨김!
    mov  edx, OFFSET msgAdd     ; "[1. Add] num1 + num2 = " 문자열 주소를 EDX에 넣음
    call WriteString            ; 글자 출력
    pop  eax                    ; 숨겨뒀던 덧셈 결과값을 다시 EAX로 꺼내옴!
    call WriteInt               ; EAX에 있는 덧셈 결과 숫자 출력
    call Crlf                   ; 줄바꿈

    ; 덧셈 직후의 레지스터 상태(AX, BX, CX, DX) 한눈에 보기
    mov  eax, num1
    mov  ebx, num2
    add  eax, ebx               ; AX = 덧셈 결과
    mov  ecx, 0                 ; CX = 0 (초기화)
    mov  edx, 0                 ; DX = 0 (초기화)
    call DumpRegs               ; 현재 EAX, EBX, ECX, EDX 값을 화면에 한 번에 출력!
    call Crlf

    ; ==========================================================
    ; 2단계: 뺄셈 (SUB) 및 레지스터 출력
    ; ==========================================================
    mov  eax, num1              ; num1을 EAX로 가져옴
    mov  ebx, num2              ; num2를 EBX로 가져옴
    sub  eax, ebx               ; EAX = EAX - EBX (뺄셈 결과가 EAX에 저장됨!)

    push eax                    ; 뺄셈 결과값 스택에 잠시 숨김
    mov  edx, OFFSET msgSub     ; "[2. Sub] num1 - num2 = " 문자열 주소를 EDX에 넣음
    call WriteString            ; 글자 출력
    pop  eax                    ; 숨겨뒀던 뺄셈 결과값 복원
    call WriteInt               ; 뺄셈 결과 숫자 출력
    call Crlf                   ; 줄바꿈

    ; 뺄셈 직후의 레지스터 상태 출력
    mov  eax, num1
    mov  ebx, num2
    sub  eax, ebx               ; AX = 뺄셈 결과
    mov  ecx, 0                 ; CX = 0
    mov  edx, 0                 ; DX = 0
    call DumpRegs               ; 레지스터 상태 출력
    call Crlf

    ; ==========================================================
    ; 3단계: 곱셈 (IMUL) 및 레지스터 출력
    ; ==========================================================
    mov  eax, num1              ; [규칙] 곱해지는 수(피승수)는 무조건 EAX에 넣어야 함!
    mov  ebx, num2              ; 곱하는 수(승수)는 EBX에 넣음
    imul ebx                    ; 부호 있는 곱셈: EDX:EAX = EAX * EBX (결과는 EAX에 저장됨!)

    push eax                    ; 곱셈 결과값 스택에 잠시 숨김
    mov  edx, OFFSET msgMul     ; "[3. Mul] num1 * num2 = " 문자열 주소를 EDX에 넣음
    call WriteString            ; 글자 출력
    pop  eax                    ; 숨겨뒀던 곱셈 결과값 복원
    call WriteInt               ; 곱셈 결과 숫자 출력
    call Crlf                   ; 줄바꿈

    ; 곱셈 직후의 레지스터 상태 출력
    mov  eax, num1
    mov  ebx, num2
    mov  ecx, 0                 ; CX = 0
    imul ebx                    ; 곱셈 실행 (결과 하위 비트는 AX, 상위 비트는 DX에 들어감!)
    call DumpRegs               ; 레지스터 상태 출력
    call Crlf

    ; ==========================================================
    ; 4단계: 나눗셈 (IDIV) 및 레지스터 출력
    ; ==========================================================
    mov  eax, num1              ; [규칙1] 나눠지는 수(피제수)는 무조건 EAX에 넣어야 함!
    cdq                         ; [규칙2] 필수! EAX의 부호(+/-)를 EDX 전체로 확장해서 나눗셈 준비
    mov  ebx, num2              ; 나누는 수(제수)는 EBX에 넣음
    idiv ebx                    ; 부호 있는 나눗셈 실행 -> [몫은 EAX], [나머지는 EDX]에 자동 저장!

    push edx                    ; 나머지(EDX)가 아래 WriteString 때문에 지워지지 않게 먼저 스택에 숨김!
    push eax                    ; 몫(EAX)도 스택에 숨김!

    mov  edx, OFFSET msgDivQ    ; "[4. Div] Quotient = " 문자열 주소를 EDX에 넣음
    call WriteString            ; "Quotient = " 출력
    pop  eax                    ; 아까 숨긴 몫(EAX)을 꺼내옴
    call WriteInt               ; 몫 출력

    mov  edx, OFFSET msgDivR    ; "  Remainder = " 문자열 주소를 EDX에 넣음
    call WriteString            ; "Remainder = " 출력
    pop  eax                    ; 아까 숨긴 나머지(EDX)를 EAX로 꺼내옴 (WriteInt는 EAX를 출력하므로!)
    call WriteInt               ; 나머지 출력
    call Crlf                   ; 줄바꿈

    ; 나눗셈 직후의 레지스터 상태 출력 (AX = 몫, BX = 나누는 수, DX = 나머지)
    mov  eax, num1
    cdq
    mov  ebx, num2
    mov  ecx, 0                 ; CX = 0
    idiv ebx                    ; 나눗셈 실행 -> AX에는 몫, DX에는 나머지가 들어감!
    call DumpRegs               ; 레지스터 상태 출력

    exit                        ; 프로그램 정상 종료
main ENDP
END main
```

---

## 2. 콘솔 실행 결과 예시 (`num1 = 20`, `num2 = 6` 입력 시)

`DumpRegs`는 레지스터 값을 **16진수(Hex)**로 출력합니다. (`20` = `14h`, `6` = `06h`)

```text
First number: 20
Second number (non-zero): 6

[1. Add] num1 + num2 = +26
  EAX=0000001A  EBX=00000006  ECX=00000000  EDX=00000000
  ESI=00000000  EDI=00000000  EBP=0019FF78  ESP=0019FF6C
  EIP=0040105F  EFL=00000202  CF=0  SF=0  ZF=0  OF=0  AF=0  PF=0

[2. Sub] num1 - num2 = +14
  EAX=0000000E  EBX=00000006  ECX=00000000  EDX=00000000
  ESI=00000000  EDI=00000000  EBP=0019FF78  ESP=0019FF6C
  EIP=0040108C  EFL=00000202  CF=0  SF=0  ZF=0  OF=0  AF=0  PF=0

[3. Mul] num1 * num2 = +120
  EAX=00000078  EBX=00000006  ECX=00000000  EDX=00000000
  ESI=00000000  EDI=00000000  EBP=0019FF78  ESP=0019FF6C
  EIP=004010B7  EFL=00000202  CF=0  SF=0  ZF=0  OF=0  AF=0  PF=0

[4. Div] Quotient = +3  Remainder = +2
  EAX=00000003  EBX=00000006  ECX=00000000  EDX=00000002
  ESI=00000000  EDI=00000000  EBP=0019FF78  ESP=0019FF6C
  EIP=004010F4  EFL=00000202  CF=0  SF=0  ZF=0  OF=0  AF=0  PF=0
```

---

## 3. 각 레지스터에 왜 그런 값이 들어갔을까? (단계별 추적)

`DumpRegs`가 출력하는 8자리 16진수 중 **오른쪽 끝 4자리**가 16비트 레지스터(`AX`, `BX`, `CX`, `DX`)의 값입니다.

### ① 덧셈 (`20 + 6 = 26`) 직후
* **`EAX = 0000001A` (`AX = 001Ah`)**: `add eax, ebx` 명령어는 앞쪽 피연산자(`EAX`)에 두 수의 합(`20 + 6 = 26`)을 덮어씁니다. 10진수 `26`을 16진수로 바꾸면 $16 \times 1 + 10 = \text{1Ah}$가 됩니다.
* **`EBX = 00000006` (`BX = 0006h`)**: `mov ebx, num2`로 넣은 두 번째 숫자 `6`이 그대로 유지됩니다.
* **`ECX = 00000000` (`CX = 0000h`)**: 코드에서 `mov ecx, 0`으로 직접 초기화했습니다.
* **`EDX = 00000000` (`DX = 0000h`)**: 코드에서 `mov edx, 0`으로 직접 초기화했습니다.

### ② 뺄셈 (`20 - 6 = 14`) 직후
* **`EAX = 0000000E` (`AX = 000Eh`)**: `sub eax, ebx` 명령어가 `EAX`에서 `EBX`를 뺀 결과(`20 - 6 = 14`)를 `EAX`에 저장합니다. 10진수 `14`는 16진수로 `0Eh`입니다.
* **`EBX = 00000006` (`BX = 0006h`)**: 빼는 수로 사용된 `num2`(`6`)가 그대로 남아 있습니다.
* **`ECX = 00000000`, `EDX = 00000000`**: 명시적으로 `0`으로 초기화한 값입니다.

### ③ 곱셈 (`20 * 6 = 120`) 직후
* **`EAX = 00000078` (`AX = 0078h`)**: `imul ebx` 명령어는 `EAX`와 `EBX`를 곱한 뒤 **64비트 크기(`EDX:EAX`)**로 결과를 저장합니다. `20 * 6 = 120`은 32비트 안에 충분히 들어가므로 하위 32비트인 `EAX`에 $16 \times 7 + 8 = \text{78h}$가 저장됩니다.
* **`EBX = 00000006` (`BX = 0006h`)**: 곱하는 수로 사용된 `6`이 유지됩니다.
* **`ECX = 00000000` (`CX = 0000h`)**: `mov ecx, 0`으로 초기화한 값입니다.
* **`EDX = 00000000` (`DX = 0000h`)**: 코드에서 `mov edx, 0`을 쓰지 않았는데도 `0`이 나온 이유! `imul ebx`가 실행될 때 결과값의 **상위 32비트**가 자동으로 `EDX`에 덮어씌워지기 때문입니다. (결과가 양수이고 32비트를 넘지 않으므로 상위 비트는 모두 `0`이 됩니다.)

### ④ 나눗셈 (`20 / 6` $\rightarrow$ 몫 `3`, 나머지 `2`) 직후
* **`EAX = 00000003` (`AX = 0003h`)**: `idiv ebx` 명령어는 나눗셈의 **몫(Quotient)**을 무조건 `EAX`에 저장합니다. (`20 / 6`의 몫 = `3`)
* **`EBX = 00000006` (`BX = 0006h`)**: 나누는 수(제수)로 전달된 `6`이 그대로 유지됩니다.
* **`ECX = 00000000` (`CX = 0000h`)**: `mov ecx, 0`으로 초기화한 값입니다.
* **`EDX = 00000002` (`DX = 0002h`)**: `idiv ebx` 명령어는 나눗셈의 **나머지(Remainder)**를 무조건 `EDX`에 자동으로 저장합니다. (`20 % 6`의 나머지 = `2`)

---

## 4. 한눈에 보는 Irvine32 핵심 입출력 공식

| 목적 | 사용하는 레지스터 | 호출하는 함수 | 기억할 핵심 |
| :--- | :--- | :--- | :--- |
| **글자 출력** | `EDX`에 문자열 주소(`OFFSET`) 넣기 | `call WriteString` | `EDX` = 출력할 글자의 시작 번지 |
| **숫자 입력** | 입력된 값이 자동으로 `EAX`에 들어옴 | `call ReadInt` | 결과는 무조건 `EAX`에 도착 |
| **숫자 출력** | 출력할 숫자를 `EAX`에 넣기 | `call WriteInt` | `EAX`에 있는 숫자를 화면에 출력 |
| **줄바꿈** | 없음 | `call Crlf` | 엔터키 한 번 치기 (`\n`) |
| **레지스터 확인** | 없음 | `call DumpRegs` | 현재 `EAX`~`EDI` 값과 플래그 한 번에 출력 |

---

## 5. 반드시 기억할 시험 & 실습 포인트

1. **스택(`PUSH` / `POP`)을 이용한 값 보호:**
   * `WriteString` 같은 라이브러리 함수를 호출하면 내부 동작 과정에서 `EAX` 값이 덮어씌워질 수 있습니다.
   * 따라서 연산 직후 `push eax`로 계산 결과를 스택에 대피시켰다가, 문자열 출력이 끝난 뒤 `pop eax`로 복원해 `WriteInt`로 출력합니다.
   * 나눗셈(`IDIV`)의 경우 스택이 **후입선출(LIFO)** 구조이므로 `push edx`(나머지) $\rightarrow$ `push eax`(몫) 순서로 넣고, 꺼낼 때는 `pop eax`(몫) $\rightarrow$ `pop eax`(나머지) 순서로 꺼냅니다.
2. **나눗셈 직전의 `CDQ` 명령어:**
   * 32비트 부호 있는 나눗셈(`IDIV`)은 64비트 쌍인 `EDX:EAX`를 피제수로 사용합니다.
   * `IDIV` 실행 직전에 반드시 `CDQ`를 실행해 `EAX`의 부호 비트를 `EDX` 전체로 확장해야 오버플로우 에러가 발생하지 않습니다.