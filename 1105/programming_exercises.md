# 💻 8.11 Programming Exercises & 어셈블리 코드 읽기 가이드

---

## 📖 Part 0. 어셈블리 코드 쉽게 읽는 5가지 핵심 법칙

어셈블리 코드가 빽빽하게 느껴질 때, 아래 5가지 포인트만 기억하고 읽으면 훨씬 눈에 잘 들어옵니다!

### 1. 명령어 읽기 규칙: `명령어 목적지(Dest), 원본(Src)`
* 예: `mov eax, [esi]` $\rightarrow$ **"`[esi]` 주소에 있는 값을 `eax`에 넣어라!"** (오른쪽에서 왼쪽으로 이동)
* 예: `sub eax, [edi]` $\rightarrow$ **"`eax`에서 `[edi]` 주소의 값을 빼서 `eax`에 남겨라!"**

### 2. 레지스터 역할 분담표 (x86 32비트)
| 레지스터 | 핵심 역할 및 비유 | 주로 쓰이는 용도 |
| :--- | :--- | :--- |
| **`EAX`** | 🎁 **공식 반환 상자 / 산술 계산기** | 함수의 최종 리턴값, 덧셈/뺄셈/곱셈/나눗셈 결과 |
| **`ECX`** | 🔄 **반복문 카운터** | `loop` 명령어 사용 시 남은 반복 횟수 자동 차감 |
| **`ESI` / `EDI`** | 📍 **배열 주소 가리키는 손가락** | `ESI`: 입력 배열 주소, `EDI`: 비교/출력 배열 주소 |
| **`EDX`** | 🧮 **보조 계산 / 출력용** | 나눗셈 나머지 저장, 문자열 주소 전달, 조건 개수 세기 |
| **`EBP`** | 🚩 **스택 고정 깃발** | 매개변수(`[EBP+n]`)와 지역 변수(`[EBP-n]`) 찾아갈 때의 기준점 |

### 3. 대괄호 `[...]`의 의미: "주소가 아닌 내용물!"
* `mov eax, esi` $\rightarrow$ `esi`에 들어있는 **주소값 숫자 자체**를 `eax`로 복사.
* `mov eax, [esi]` $\rightarrow$ `esi`가 가리키는 **메모리 번지 안의 진짜 알맹이(값)**를 `eax`로 읽어옴.
* `add esi, 4` $\rightarrow$ 32비트(4바이트) 정수 배열에서 **다음 칸(다음 원소)으로 이동**.

### 4. 조건문 패턴 읽기 (`cmp`와 점프 명령어)
* `cmp A, B`는 항상 **`A - B`를 내부적으로 계산**해 보고 깃발(Flag)을 올립니다.
* **부호 없는 정수 비교**: `JA` (Above, 크면), `JB` (Below, 작으면)
* **부호 있는 정수 비교**: `JG` (Greater, 크면), `JL` (Less, 작으면), `JE` (Equal, 같으면)

### 5. `USES` 지시어: "기존 락커룸 백업 기능"
* `FindLargest PROC USES esi ecx edx`
  * 함수 시작 시 `ESI`, `ECX`, `EDX`를 스택에 자동 `PUSH`하여 원래 값을 보존해 두고,
  * 함수가 끝날 때(`ret`) 알아서 `POP`하여 복원해 줍니다. (`EAX`는 반환값이므로 보존하지 않음!)

---

## 1. 🔍 FindLargest Procedure (최댓값 찾기)

부호 있는 32비트 정수(SDWORD) 배열과 배열 길이를 전달받아 **가장 큰 값을 `EAX` 레지스터로 반환**하는 함수입니다.

### 💡 로직 해석 및 코드 읽기 포인트
1. 첫 번째 원소 `[esi]`를 일단 `EAX`에 담아 "현재 최댓값"으로 삼습니다.
2. `add esi, 4`로 다음 칸을 가리키고, 반복 횟수 `ECX`를 1 차감합니다.
3. `cmp edx, eax` 후 `jle L_next` (현재 값 $\le$ 최댓값) 이면 스킵, 더 크면 `mov eax, edx`로 갱신합니다.

### 💻 완벽 정답 코드

```assembly
INCLUDE Irvine32.inc

FindLargest PROTO, ptrArray:PTR SDWORD, arraySize:DWORD

.data
array1 SDWORD -10, 20, 5, -30, 15
array2 SDWORD -50, -20, -3, -100
array3 SDWORD 100, 500, 200

.code
main PROC
    ; 테스트 1
    INVOKE FindLargest, ADDR array1, LENGTHOF array1
    call WriteInt           ; 출력: +20
    call Crlf

    ; 테스트 2
    INVOKE FindLargest, ADDR array2, LENGTHOF array2
    call WriteInt           ; 출력: -3
    call Crlf

    ; 테스트 3
    INVOKE FindLargest, ADDR array3, LENGTHOF array3
    call WriteInt           ; 출력: +500
    call Crlf

    exit
main ENDP

FindLargest PROC USES esi ecx edx,
    ptrArray:PTR SDWORD,
    arraySize:DWORD

    mov esi, ptrArray       ; ESI = 배열의 시작 주소
    mov ecx, arraySize      ; ECX = 배열 원소 개수
    
    mov eax, [esi]          ; 첫 번째 요소를 일단 최댓값으로 설정
    add esi, 4              ; 다음 원소 주소로 이동 (+4바이트)
    dec ecx                 ; 1개 검사했으므로 카운트 1 감소
    cmp ecx, 0
    jle L_done              ; 배열 크기가 1 이하라면 바로 종료

L_loop:
    mov edx, [esi]          ; 현재 원소 값 읽기
    cmp edx, eax            ; 현재 원소(EDX)와 최댓값(EAX) 비교
    jle L_next              ; EDX <= EAX 이면 스킵 (부호 비교 JLE 사용)
    mov eax, edx            ; 더 큰 값을 발견했으므로 최댓값 갱신!

L_next:
    add esi, 4              ; 다음 원소로 이동
    loop L_loop             ; ECX 1 감소 후 0이 아니면 반복

L_done:
    ret
FindLargest ENDP
END main
```

---

## 2. 🏁 Chess Board (체스판 그리기)

회색(Gray)과 흰색(White) 칸이 교차하는 **8x8 체스판**을 그리는 프로그램입니다. (전역 변수 사용 금지)

### 💡 로직 해석 및 코드 읽기 포인트
* `and eax, 1`은 숫자를 2로 나눈 나머지를 구하는 테크닉입니다 (0: 짝수, 1: 홀수).
* `(행 + 열)` 이 짝수면 회색, 홀수면 흰색을 지정합니다.

### 💻 완벽 정답 코드

```assembly
INCLUDE Irvine32.inc

DrawSquare PROTO, row:DWORD, col:DWORD, color:DWORD
DrawBoard PROTO

.code
main PROC
    call DrawBoard
    exit
main ENDP

DrawBoard PROC USES ecx edx,
    rowIdx:DWORD, colIdx:DWORD
    
    LOCAL r:DWORD, c:DWORD
    mov r, 0

L_row:
    cmp r, 8
    jge L_endRow
    mov c, 0

L_col:
    cmp c, 8
    jge L_nextRow

    ; (r + c) % 2 계산
    mov eax, r
    add eax, c
    and eax, 1             ; 2로 나눈 나머지 (0: 짝수, 1: 홀수)
    
    cmp eax, 0
    jne L_white
    INVOKE DrawSquare, r, c, gray + (black SHL 4)
    jmp L_cont

L_white:
    INVOKE DrawSquare, r, c, white + (black SHL 4)

L_cont:
    inc c
    jmp L_col

L_nextRow:
    inc r
    jmp L_row

L_endRow:
    ret
DrawBoard ENDP

DrawSquare PROC USES eax edx,
    row:DWORD, col:DWORD, color:DWORD

    ; Gotoxy 위치 설정 (x = col * 2, y = row)
    mov eax, col
    mov dl, al
    shl dl, 1              ; 커서 X 위치 (콘솔 글자 폭 고려하여 2칸씩)
    mov eax, row
    mov dh, al             ; 커서 Y 위치
    call Gotoxy

    mov eax, color
    call SetTextColor

    mov al, ' '            ; 공백 문자로 칸 채우기
    call WriteChar
    call WriteChar
    ret
DrawSquare ENDP
END main
```

---

## 3. 🎨 Chess Board with Alternating Colors (색상이 바뀌는 체스판)

500ms(0.5초)마다 배경 색상을 바꾸며 총 16번 체스판을 재출력하는 프로그램입니다.

### 💻 완벽 정답 코드

```assembly
INCLUDE Irvine32.inc

DrawSquare PROTO, row:DWORD, col:DWORD, color:DWORD
DrawBoardColor PROTO, colorIndex:DWORD

.code
main PROC
    LOCAL curColor:DWORD
    mov curColor, 0
    mov ecx, 16            ; 총 16가지 색상 순회

L_mainLoop:
    push ecx               ; loop 카운터 백업
    INVOKE DrawBoardColor, curColor
    mov eax, 500
    call Delay             ; 500ms 지연
    inc curColor
    pop ecx                ; loop 카운터 복원
    loop L_mainLoop

    exit
main ENDP

DrawBoardColor PROC USES ecx edx, colorIndex:DWORD
    LOCAL r:DWORD, c:DWORD
    mov r, 0

L_row:
    cmp r, 8
    jge L_endRow
    mov c, 0

L_col:
    cmp c, 8
    jge L_nextRow

    mov eax, r
    add eax, c
    and eax, 1
    
    cmp eax, 0
    jne L_white
    ; 변하는 색상 적용 (colorIndex << 4 는 배경색 비트 조합)
    mov eax, colorIndex
    shl eax, 4             
    add eax, white         
    INVOKE DrawSquare, r, c, eax
    jmp L_cont

L_white:
    INVOKE DrawSquare, r, c, (white SHL 4) + black

L_cont:
    inc c
    jmp L_col

L_nextRow:
    inc r
    jmp L_row

L_endRow:
    ret
DrawBoardColor ENDP
```

---

## 4. 🎯 FindThrees Procedure (연속된 3 3 3 찾기)

배열 안에 **숫자 3이 연속으로 3번(3, 3, 3)** 등장하면 `EAX = 1`, 아니면 `EAX = 0`을 반환합니다.

### 💡 로직 해석 및 코드 읽기 포인트
* `edx` 레지스터를 "연속 성공 콤보 카운터"로 활용합니다.
* `3`을 만날 때마다 `inc edx` 하고, 다른 숫자를 만나면 `mov edx, 0`으로 콤보를 초기화합니다.

### 💻 완벽 정답 코드

```assembly
FindThrees PROC USES esi ecx edx,
    ptrArray:PTR DWORD,
    arraySize:DWORD

    mov esi, ptrArray
    mov ecx, arraySize
    mov edx, 0             ; 연속 3 카운터 초기화

L_loop:
    cmp DWORD PTR [esi], 3 ; 현재 위치의 값이 3인가?
    jne L_reset            ; 3이 아니면 콤보 리셋

    inc edx                ; 3을 찾았으므로 콤보 +1
    cmp edx, 3
    je L_found             ; 3연속 달성 시 즉시 성공 처리
    jmp L_next

L_reset:
    mov edx, 0             ; 콤보 초기화

L_next:
    add esi, 4             ; 다음 4바이트 정수로 이동
    loop L_loop

    mov eax, 0             ; 끝까지 못 찾음 (실패 = 0)
    jmp L_done

L_found:
    mov eax, 1             ; 3연속 찾음 (성공 = 1)

L_done:
    ret
FindThrees ENDP
```

---

## 5. 🔀 DifferentInputs Procedure (서로 다른 3개의 입력값)

전달받은 3개의 정수 매개변수가 **모두 다르면 `EAX = 1`**, 하나라도 같은 값이 존재하면 `EAX = 0`을 반환합니다.

### 💻 완벽 정답 코드

```assembly
DifferentInputs PROC,
    v1:DWORD, v2:DWORD, v3:DWORD

    mov eax, v1
    cmp eax, v2
    je L_same              ; v1 == v2 이면 실패 (0 반환)

    cmp eax, v3
    je L_same              ; v1 == v3 이면 실패 (0 반환)

    mov eax, v2
    cmp eax, v3
    je L_same              ; v2 == v3 이면 실패 (0 반환)

    mov eax, 1             ; 세 값이 모두 다름! (1 반환)
    ret

L_same:
    mov eax, 0             ; 중복 값 존재
    ret
DifferentInputs ENDP
```

---

## 6. 🔄 Exchanging Integers (짝지어 값 교환하기)

배열에서 인접한 두 원소끼리 **짝을 지어 값(`Swap`)을 교환**합니다. (예: `[1, 2, 3, 4]` $\rightarrow$ `[2, 1, 4, 3]`)

### 💡 로직 해석 및 코드 읽기 포인트
* 2개씩 짝지어 바꾸므로 전체 반복 횟수 `ECX`는 `arraySize / 2` (`shr ecx, 1`)로 설정합니다.
* 2개 원소를 처리한 후에는 주소를 8바이트(`add esi, 8`) 이동합니다.

### 💻 완벽 정답 코드

```assembly
Swap PROTO, pValX:PTR DWORD, pValY:PTR DWORD

ExchangePairs PROC USES esi ecx,
    ptrArray:PTR DWORD,
    arraySize:DWORD

    mov esi, ptrArray
    mov ecx, arraySize
    shr ecx, 1             ; 크기를 2로 나눔 (원소 2개당 1번 반복)

L_loop:
    mov eax, esi
    add eax, 4             ; EAX = 바로 옆 원소의 주소 (ESI + 4)
    INVOKE Swap, esi, eax  ; [ESI]와 [ESI+4]의 값을 스왑

    add esi, 8             ; 다음 짝으로 이동 (+8바이트)
    loop L_loop

    ret
ExchangePairs ENDP
```

---

## 7. 🌀 Greatest Common Divisor (유클리드 호제법 재귀 구현)

두 정수의 최대공약수(GCD)를 **재귀(Recursion) 함수**로 구합니다.

$$
\text{gcd}(a, b) = \begin{cases} a & \text{if } b = 0 \\ \text{gcd}(b, a \pmod b) & \text{if } b > 0 \end{cases}
$$

### 💻 완벽 정답 코드

```assembly
GetGCD PROTO, n1:DWORD, n2:DWORD

GetGCD PROC,
    n1:DWORD, n2:DWORD

    mov edx, 0
    mov eax, n1
    mov ebx, n2
    
    cmp ebx, 0
    je L_baseCase          ; 탈출 조건: n2 == 0 이면 n1(EAX)이 최종 답!

    div ebx                ; EDX = n1 % n2 (나머지), EAX = n1 / n2 (몫)
    INVOKE GetGCD, n2, edx ; 재귀 호출: GetGCD(n2, n1 % n2)
    ret

L_baseCase:
    mov eax, n1            ; 기저 조건: EAX에 n1 담아 반환
    ret
GetGCD ENDP
```

---

## 8. 👯 Counting Matching Elements (일치하는 원소 개수 세기)

크기가 같은 두 배열을 순회하며 **같은 위치의 값이 완벽히 일치하는 개수**를 세어 `EAX`로 반환합니다.

### 💻 완벽 정답 코드

```assembly
CountMatches PROC USES esi edi ecx edx,
    ptrArray1:PTR SDWORD,
    ptrArray2:PTR SDWORD,
    arrayLength:DWORD

    mov esi, ptrArray1     ; 첫 번째 배열 주소
    mov edi, ptrArray2     ; 두 번째 배열 주소
    mov ecx, arrayLength   ; 순회할 크기
    mov edx, 0             ; 일치 개수 카운터

L_loop:
    mov eax, [esi]
    cmp eax, [edi]         ; 두 배열의 원소 비교
    jne L_next             ; 다르면 카운트 없이 스킵
    inc edx                ; 일치하면 카운트 +1

L_next:
    add esi, 4             ; 첫 번째 배열 다음 칸
    add edi, 4             ; 두 번째 배열 다음 칸
    loop L_loop

    mov eax, edx           ; 최종 개수를 EAX에 담아 반환
    ret
CountMatches ENDP
```

---

## 9. 🤏 Counting Nearly Matching Elements (비슷한 원소 개수 세기)

두 배열의 같은 위치 값이 **오차 범위(`diff`) 이내**로 비슷하면 개수를 세어 `EAX`로 반환합니다 ($\vert{}x_i - y_i\vert{} \le \text{diff}$).

### 💡 로직 해석 및 코드 읽기 포인트
* `sub eax, [edi]`로 두 값의 차이를 구한 뒤, 음수일 경우 `neg eax`를 실행해 **절댓값(Absolute Value)**으로 만듭니다.

### 💻 완벽 정답 코드

```assembly
CountNearMatches PROC USES esi edi ecx edx ebx,
    ptrArray1:PTR SDWORD,
    ptrArray2:PTR SDWORD,
    arrayLength:DWORD,
    diff:DWORD

    mov esi, ptrArray1
    mov edi, ptrArray2
    mov ecx, arrayLength
    mov edx, 0             ; 카운터 초기화

L_loop:
    mov eax, [esi]
    sub eax, [edi]         ; EAX = x_i - y_i
    
    ; 절댓값 구하기 (EAX < 0 이면 부호 반전)
    jge L_checkDiff
    neg eax                ; 음수 -> 양수로 반전

L_checkDiff:
    cmp eax, diff
    jg L_next              ; 차이 > diff 이면 인정 안 함 (스킵)
    inc edx                ; 오차 범위 안이면 카운트 +1

L_next:
    add esi, 4
    add edi, 4
    loop L_loop

    mov eax, edx           ; 결과값 반환
    ret
CountNearMatches ENDP
```

---

## 10. 📜 Show Procedure Parameters (스택 매개변수 들여다보기)

나를 부른 함수(Caller)가 스택 메모리에 쌓아둔 32비트 **매개변수들의 주소와 16진수 값을 직접 추적하여 출력**합니다.

### 💡 로직 해석 및 코드 읽기 포인트
* `ShowParams` 함수 내부의 스택 프레임 구조:
  * `[EBP + 0]` = 이전 EBP (Old EBP)
  * `[EBP + 4]` = 돌아갈 주소 (Return Address)
  * `[EBP + 8]` = 자기 자신의 인자 (`paramCount`)
  * `[EBP + 12]` $\rightarrow$ **나를 부른 상위 함수가 넣어준 첫 번째 매개변수 시작점!**
* 따라서 `mov esi, ebp` 후 `add esi, 12`를 해 주면 상위 매개변수의 주소부터 순서대로 읽을 수 있습니다.

### 💻 완벽 정답 코드

```assembly
ShowParams PROC USES esi ecx edx eax,
    paramCount:DWORD

    mov ecx, paramCount
    mov esi, ebp
    add esi, 12            ; [EBP+12] 위치부터 상위 매개변수 시작!

    mov edx, OFFSET msgHeader
    call WriteString
    call Crlf

L_loop:
    ; 1. 스택 메모리 주소 출력
    mov edx, OFFSET msgAddr
    call WriteString
    mov eax, esi
    call WriteHex

    ; 2. 해당 스택 위치에 저장된 실제 값 출력
    mov edx, OFFSET msgVal
    call WriteString
    mov eax, [esi]         ; 역참조로 스택에 든 값 읽기
    call WriteHex
    call Crlf

    add esi, 4             ; 낮은 주소에서 높은 주소로 (+4바이트씩 이동)
    loop L_loop

    ret

.data
msgHeader BYTE "Stack parameters:--------------------------", 0
msgAddr   BYTE "Address ", 0
msgVal    BYTE " = ", 0
ShowParams ENDP
```