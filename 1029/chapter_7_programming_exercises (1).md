# 📘 Chapter 7. Programming Exercises 초보자 완벽 가이드

x86 어셈블리 언어(Kip Irvine 교재 기준) Chapter 7 (Integer Arithmetic) Programming Exercises (1~8번) 실습 문제 풀이 노트입니다. 어셈블리를 처음 접하는 분들도 쉽게 이해할 수 있도록 **기초 레지스터 역할, 비주얼 실행 흐름도, 한 줄 단위 한국어 상세 주석**을 포함했습니다.

---

## 🐣 어셈블리 입문자를 위한 필수 개념 요약

### 1. 레지스터(Register)란?
CPU 내부에 존재하는 **가장 빠른 변수/바구니**입니다. 메모리(RAM)에 있는 데이터는 CPU가 직접 계산할 수 없으므로, 레지스터로 끌고 올라와서 연산을 처리합니다.

* **`EAX`**: 연산의 중심이 되는 메인 통 (곱셈/나눗셈/출력의 기본값)
* **`EBX` / `ECX` / `EDX`**: 보조 데이터 및 매개변수를 담는 통
* **`ECX`**: **카운터(Counter) 전용 레지스터.** `loop` 명령어를 쓰면 CPU는 무조건 `ECX`를 쳐다보고 숫자를 1씩 깎으면서 반복합니다.
* **`ESI` / `EDI`**: **메모리 주소를 가리키는 손가락(Pointer).** 데이터 값 그 자체가 아니라 "데이터가 저장된 메모리의 주소 번지"를 들고 있습니다. `[esi]`처럼 대괄호를 씌워야 "그 주소 문을 열고 내용물 1개 가져오기"가 됩니다.

### 2. 핵심 명령어 맵 (Cheat Sheet)

| 명령어 | 뜻 | 초보자용 비유 |
| :--- | :--- | :--- |
| **`pushad` / `popad`** | Push/Pop All Double | 함수 시작 전 **레지스터 8개 전체 임시 백업** / 끝나고 **원상복구** |
| **`mov a, b`** | Move | `a = b` (b의 값을 a에 복사) |
| **`add / sub`** | Add / Subtract | 덧셈 / 뺄셈 |
| **`adc / sbb`** | Add with Carry / Sub with Borrow | 앞선 연산에서 생긴 **올림수(Carry)/빌림수(Borrow)까지 합쳐서** 더하기/빼기 |
| **`shl / shr`** | Shift Left / Right | 비트를 좌/우로 밀기 (**2배 곱하기 / 2로 나누기**) |
| **`rol / ror`** | Rotate Left / Right | 비트를 밖으로 버리지 않고 **원형으로 순환 회전시키기** |
| **`loop 라벨`** | Loop | `ECX`를 1 감소시키고 0이 아니면 지정한 라벨로 점프하기 |

---

### 📌 Exercise 1. Display ASCII Decimal (`WriteScaled`)

#### 1. 문제 설명 (Problem)
소수점이 생략된 숫자 문자열(예: `"100123456789765"`)에서 지정된 위치(오른쪽에서 5번째)에 소수점(`.`)을 자동으로 삽입하여 `1001234567.89765` 형태로 출력하는 프로시저를 작성합니다.

* **매개변수 약속:** `EDX` = 문자열 주소, `ECX` = 문자열 전체 길이, `EBX` = 소수점 위치

#### 2. 비주얼 실행 흐름 (Visual Diagram)
```text
전체 문자열: "1 0 0 1 2 3 4 5 6 7 8 9 7 6 5" (길이 ECX = 15)
소수점 위치: EBX = 5
소수점 인덱스: EDI = (15 - 5) = 10번째 글자 직후

[인덱스 0~9 출력] ──> [ '.' 점 출력 ] ──> [인덱스 10~14 출력]
 1001234567               .                 89765
```

#### 3. 초보자용 한 줄 주석 코드 (Solution)

```assembly
INCLUDE Irvine32.inc

.code
WriteScaled PROC
    pushad              ; [1] 기존 레지스터 8개의 상태를 스택에 안전하게 백업합니다.

    mov esi, edx        ; [2] ESI에 EDX(문자열 시작 주소)를 복사해 메모리를 가리키게 합니다.
    mov eax, ecx        ; [3] EAX = 전체 문자열 길이 (예: 15)
    sub eax, ebx        ; [4] EAX = (전체 길이 - 소수점 오프셋). 소수점이 찍혀야 할 글자 순번 계산.
    mov edi, eax        ; [5] 소수점 출력 위치(인덱스)를 EDI에 보관합니다.
    
    mov ebx, 0          ; [6] EBX = 현재 출력한 글자 수를 세는 카운터 (0부터 시작)

L1:
    cmp ebx, edi        ; [7] 현재 글자 위치(EBX)가 소수점 찍을 위치(EDI)와 같은지 비교합니다.
    jne PrintChar       ; [8] 다르면 소수점을 찍지 않고 바로 숫자 출력 라벨로 점프합니다.
    
    ; --- 소수점 출력 구간 ---
    mov al, '.'         ; [9] AL 레지스터에 소수점 문자('.')를 넣습니다.
    call WriteChar      ; [10] 화면에 '.' 문자를 출력합니다.

PrintChar:
    mov al, [esi]       ; [11] ESI 주소 문을 열고 실제 숫자 문자 1개를 AL로 가져옵니다.
    call WriteChar      ; [12] 숫자 문자를 화면에 출력합니다.
    inc esi             ; [13] 다음 문자를 가리키도록 메모리 포인터 1 증가 (+1)
    inc ebx             ; [14] 출력한 글자 수 카운터 1 증가 (+1)
    loop L1             ; [15] ECX(전체 길이)만큼 반복합니다 (돌 때마다 ECX가 1씩 자동 감소)

    call Crlf           ; [16] 출력이 다 끝나면 엔터(줄바꿈)를 칩니다.
    popad               ; [17] 백업해 두었던 레지스터 값들을 원래대로 복원합니다.
    ret                 ; [18] 프로시저 종료 (호출한 곳으로 복귀)
WriteScaled ENDP

main PROC
    .data
    num1 BYTE "100123456789765"
    num2 BYTE "123456"
    num3 BYTE "9876543210"

    .code
    mov edx, OFFSET num1    ; num1의 메모리 주소를 EDX에 전달
    mov ecx, LENGTHOF num1  ; num1의 전체 길이를 ECX에 전달
    mov ebx, 5              ; 오른쪽에서 5번째 자리에 소수점 지정
    call WriteScaled        ; 출력: 1001234567.89765

    mov edx, OFFSET num2
    mov ecx, LENGTHOF num2
    mov ebx, 2
    call WriteScaled        ; 출력: 1234.56

    mov edx, OFFSET num3
    mov ecx, LENGTHOF num3
    mov ebx, 4
    call WriteScaled        ; 출력: 987654.3210

    exit
main ENDP
END main
```

---

### 📌 Exercise 2. Extended Subtraction Procedure (`Extended_Sub`)

#### 1. 문제 설명 (Problem)
32비트(4바이트)를 넘어가는 거대한 정수의 뺄셈을 수행합니다. 필산(손으로 뺄셈)할 때와 똑같이 **가장 하위 4바이트부터 차례대로 빼되**, 이전 뺄셈에서 발생한 빌림(Borrow)을 다음 뺄셈에 반영하기 위해 `SBB` 지시어를 사용합니다.

#### 2. 초보자용 한 줄 주석 코드 (Solution)

```assembly
INCLUDE Irvine32.inc

.code
Extended_Sub PROC
    ; 매개변수: ESI = op1 주소(결과 저장), EDI = op2 주소, ECX = 4바이트 루프 횟수
    pushad              ; [1] 모든 레지스터 상태 백업

    clc                 ; [2] Clear Carry: 이전 계산 찌꺼기 깃발(Carry Flag)을 0으로 초기화

L1:
    mov eax, [edi]      ; [3] op2 메모리에서 4바이트(32비트) 데이터를 EAX로 읽어옵니다.
    sbb [esi], eax      ; [4] [esi] = [esi] - EAX - Carry(빌림값).
                        ;     ★ SBB(Subtract with Borrow)는 이전 턴의 빌림값까지 빼줍니다!
    add esi, 4          ; [5] op1의 다음 상위 4바이트 위치로 이동 (+4바이트)
    add edi, 4          ; [6] op2의 다음 상위 4바이트 위치로 이동 (+4바이트)
    loop L1             ; [7] ECX 횟수만큼 (32비트 덩어리 수만큼) 반복합니다.

    popad               ; [8] 백업된 레지스터 복원
    ret                 ; [9] 프로시저 종료
Extended_Sub ENDP

main PROC
    .data
    ; 12바이트(96비트) 거대 정수 (리틀 엔디안 방식: 배열의 앞쪽이 가장 작은 자릿수 LSB)
    val1 BYTE 0A1h, 47h, 20h, 36h, 04h, 30h, 40h, 20h, 12h, 34h, 56h, 78h
    val2 BYTE 0B2h, 30h, 26h, 4Ah, 30h, 10h, 52h, 05h, 01h, 02h, 03h, 04h

    .code
    mov esi, OFFSET val1    ; val1 메모리 시작 주소 전달
    mov edi, OFFSET val2    ; val2 메모리 시작 주소 전달
    mov ecx, 3              ; 12바이트 = 4바이트 x 3번 루프

    call Extended_Sub
    exit
main ENDP
END main
```

---

### 📌 Exercise 3. Packed Decimal Conversion (`PackedToAsc`)

#### 1. 문제 설명 (Problem)
1바이트 안에 2개의 10진수 숫자가 압축(Packed BCD)되어 있는 데이터(예: `12345678h`)를 비트 쪼개기를 통해 사람이 읽을 수 있는 ASCII 문자열(`"12345678"`)로 변환합니다.

#### 2. 비주얼 비트 분리 개념
```text
1 바이트 = [ 상위 4비트 (High Nibble) ] [ 하위 4비트 (Low Nibble) ]
예: 0x12  =        0001 (1)                  0010 (2)
                     │                         │
            SHR 4로 당겨서 + '0'        AND 0Fh 마스크 후 + '0'
                     ▼                         ▼
                 문자 '1'                   문자 '2'
```

#### 3. 초보자용 한 줄 주석 코드 (Solution)

```assembly
INCLUDE Irvine32.inc

.code
PackedToAsc PROC
    ; 매개변수: EAX = 4바이트 팩트 십진수 (예: 12345678h), EDI = 출력 버퍼 주소
    pushad

    mov ecx, 4          ; [1] 4바이트 데이터를 처리할 것이므로 ECX = 4
    add edi, 8          ; [2] 총 8개 숫자가 들어가므로 문자열 맨 끝 위치로 EDI 포인터 이동
    mov BYTE PTR [edi], 0 ; [3] 문자열 종료를 알리는 NULL 문자(0) 세팅

L1:
    ; --- 1. 하위 4비트 (오른쪽 1진수 숫자) 추출 ---
    dec edi             ; [4] 버퍼 저장 위치를 한 칸 왼쪽으로 이동
    mov dl, al          ; [5] AL 값을 DL로 복사
    and dl, 0Fh         ; [6] AND 비트 마스킹: 상위 4비트를 날리고 하위 4비트만 남김
    add dl, '0'         ; [7] 십진수 숫자에 '0'(30h)을 더해 ASCII 문자로 변환 (예: 2 -> '2')
    mov [edi], dl       ; [8] 변환된 문자를 버퍼에 저장

    ; --- 2. 상위 4비트 (왼쪽 1진수 숫자) 추출 ---
    dec edi             ; [9] 버퍼 저장 위치를 한 칸 더 왼쪽으로 이동
    mov dl, al          ; [10] AL 값을 다시 DL로 복사
    shr dl, 4           ; [11] 오른쪽으로 4비트 밀어 상위 4비트를 하위 위치로 내림
    add dl, '0'         ; [12] ASCII 문자로 변환
    mov [edi], dl       ; [13] 변환된 문자를 버퍼에 저장

    shr eax, 8          ; [14] 다음 바이트 처리를 위해 EAX를 오른쪽으로 8비트(1바이트) 밉니다.
    loop L1             ; [15] 4바이트 모두 처리할 때까지 4번 반복!

    popad
    ret
PackedToAsc ENDP

main PROC
    .data
    packedNum DWORD 12345678h
    buffer BYTE 9 DUP(0)    ; 8글자 + NULL문자(1바이트) = 9바이트 버퍼

    .code
    mov eax, packedNum
    mov edi, OFFSET buffer
    call PackedToAsc

    mov edx, OFFSET buffer
    call WriteString        ; 출력: 12345678
    call Crlf
    exit
main ENDP
END main
```

---

### 📌 Exercise 4. Encryption Using Rotate Operations (`RotateEncrypt`)

#### 1. 문제 설명 (Problem)
평문 글자들을 키 배열(`key`)의 회전수 값에 따라 비트 회전시켜 암호화합니다.
* 키 값이 **음수**이면 **왼쪽 회전 (`ROL`)**
* 키 값이 **양수**이면 **오른쪽 회전 (`ROR`)**
* 키 배열 길이(10개)보다 평문이 길면 키를 처음부터 다시 재사용(순환)합니다.

#### 2. 초보자용 한 줄 주석 코드 (Solution)

```assembly
INCLUDE Irvine32.inc

KEY_SIZE = 10           ; 키 배열 크기 상수 선언

.code
RotateEncrypt PROC
    ; 매개변수: ESI = 평문 주소, ECX = 평문 길이, EDX = 키 배열 주소
    pushad
    mov ebx, 0          ; [1] EBX = 키 배열의 인덱스 카운터 (0부터 시작)

L1:
    mov al, [esi]       ; [2] ESI 주소에서 암호화할 평문 1바이트를 AL로 가져옵니다.
    mov dl, [edx + ebx] ; [3] EDX(키 주소) + EBX(인덱스) 위치에서 키 값 1개를 DL로 가져옵니다.
    
    cmp dl, 0           ; [4] 가져온 키 값이 0 이상(양수/0)인지 확인합니다.
    jge RotateRight     ; [5] 0 이상이면 오른쪽 회전 처리 구문으로 점프합니다.

RotateLeft:             ; --- 음수 키 값 처리 (왼쪽 회전 ROL) ---
    neg dl              ; [6] -2 같은 음수를 +2 양수로 만듭니다 (회전 횟수용)
    mov cl, dl          ; [7] 회전 횟수는 무조건 CL 레지스터에 들어가야 합니다.
    rol al, cl          ; [8] AL의 비트를 CL만큼 왼쪽으로 순환 회전(ROL)시킵니다.
    jmp StoreByte       ; [9] 저장 구문으로 점프

RotateRight:            ; --- 양수 키 값 처리 (오른쪽 회전 ROR) ---
    mov cl, dl          ; [10] 회전 횟수를 CL에 저장
    ror al, cl          ; [11] AL의 비트를 CL만큼 오른쪽으로 순환 회전(ROR)시킵니다.

StoreByte:
    mov [esi], al       ; [12] 암호화된 AL 바이트를 메모리에 덮어씁니다.
    inc esi             ; [13] 다음 평문 바이트 위치로 포인터 이동 (+1)
    
    inc ebx             ; [14] 다음 키 오프셋으로 이동 (+1)
    cmp ebx, KEY_SIZE   ; [15] 키 배열 끝(10번째)에 도달했는지 검사
    jl NextLoop
    mov ebx, 0          ; [16] 키 10개를 다 썼다면 0으로 리셋하여 키를 다시 처음부터 돌려씀!

NextLoop:
    loop L1             ; [17] ECX(평문 길이)만큼 반복

    popad
    ret
RotateEncrypt ENDP
```

---

### 📌 Exercise 5. Prime Numbers (Sieve of Eratosthenes)

#### 1. 문제 설명 (Problem)
'에라토스테네스의 체' 알고리즘을 사용하여 2부터 1000 사이의 모든 소수(Prime Number)를 구합니다. 배열에서 소수의 배수 위치(4, 6, 8, 10...)들을 전부 1(합성수)로 마킹하여 지운 후, 0으로 깨끗이 남아있는 숫자만 소수로 출력합니다.

#### 2. 초보자용 한 줄 주석 코드 (Solution)

```assembly
INCLUDE Irvine32.inc

MAX_VAL = 1000

.data
sieve BYTE MAX_VAL DUP(0) ; 1000개 바이트 배열 선언 (0 = 소수 후보, 1 = 합성수/지워짐)

.code
main PROC
    mov ecx, 2          ; [1] 소수 검사는 2부터 시작합니다 (ECX = 2)

; --- 1단계: 체로 쳐서 배수들 지우기 ---
FilterLoop:
    cmp ecx, MAX_VAL    ; [2] 1000까지 다 검사했는지 확인
    jge DisplayPrimes   ; [3] 1000에 도달했으면 소수 출력 단계로 이동

    mov esi, OFFSET sieve
    cmp BYTE PTR [esi + ecx], 1 ; [4] 현재 숫자(ECX)가 이미 지워진 숫자(1)인지 확인
    je NextP                    ; [5] 이미 지워졌다면 다음 숫자로 넘어감

    ; --- 배수 지우기 시작 ---
    mov ebx, ecx        ; [6] EBX = 배수 지우기 카운터
    add ebx, ecx        ; [7] EBX = p * 2 (자기 자신의 2배 위치부터 지우기 시작)

MarkMultiples:
    cmp ebx, MAX_VAL    ; [8] 배수가 1000을 넘었는지 확인
    jge NextP
    mov BYTE PTR [esi + ebx], 1 ; [9] 배수 위치에 1을 적어 '소수 아님'으로 표시함
    add ebx, ecx        ; [10] 다음 배수로 이동 (예: 2의 배수 -> 4, 6, 8, 10...)
    jmp MarkMultiples

NextP:
    inc ecx             ; [11] 다음 검사할 숫자로 이동 (+1)
    jmp FilterLoop

; --- 2단계: 0으로 남은 소수들 화면에 출력 ---
DisplayPrimes:
    mov ecx, 2          ; 2부터 출력 검사 시작

PrintLoop:
    cmp ecx, MAX_VAL
    jge Done
    mov esi, OFFSET sieve
    cmp BYTE PTR [esi + ecx], 0 ; [12] 배열 값이 0(소수)으로 남아있는지 확인
    jne SkipPrint

    mov eax, ecx        ; [13] 소수인 인덱스 숫자를 EAX에 넣음
    call WriteDec       ; [14] 10진수로 화면 출력
    mov al, ' '
    call WriteChar      ; [15] 공백 한 칸 출력

SkipPrint:
    inc ecx
    jmp PrintLoop

Done:
    call Crlf
    exit
main ENDP
END main
```

---

### 📌 Exercise 6. Greatest Common Divisor (`GCD`)

#### 1. 문제 설명 (Problem)
유클리드 호제법을 사용하여 두 정수의 최대공약수(GCD)를 구합니다.
* 공식: $x \div y$ 의 **나머지($n$)가 0이 될 때까지** $x \leftarrow y$, $y \leftarrow n$ 교체를 반복합니다.

#### 2. 초보자용 한 줄 주석 코드 (Solution)

```assembly
INCLUDE Irvine32.inc

.code
GCD PROC
    ; 매개변수: EAX = x, EBX = y  /  반환: EAX = 최대공약수
    push edx

    ; --- 음수 입력 대비 절대값 만들기 ---
    cmp eax, 0
    jge CheckY
    neg eax             ; [1] x가 음수이면 neg 명령어로 양수 전환

CheckY:
    cmp ebx, 0
    jge GCD_Loop
    neg ebx             ; [2] y가 음수이면 양수 전환

GCD_Loop:
    mov edx, 0          ; [3] ★ 32비트 나눗셈 전 상위 통 EDX를 0으로 깨끗이 청소!
    div ebx             ; [4] EAX / EBX 수행 -> EAX = 몫, EDX = 나머지(n)

    mov eax, ebx        ; [5] x = y (다음 루프를 위해 y를 x에 복사)
    mov ebx, edx        ; [6] y = n (나머지 n을 y에 복사)

    cmp ebx, 0          ; [7] 새로운 y(나머지)가 0인지 확인
    jg GCD_Loop         ; [8] 0보다 크면 나머지가 남아있으므로 나눗셈 반복!

    ; [9] 나머지가 0이 되는 순간의 x(현재 EAX)가 바로 최대공약수입니다.
    pop edx
    ret                 ; [10] EAX에 결과값을 담고 리턴
GCD ENDP

main PROC
    mov eax, 48
    mov ebx, 18
    call GCD            ; EAX = 6 (48과 18의 최대공약수)
    call WriteDec       ; 출력: 6
    call Crlf
    exit
main ENDP
END main
```

---

### 📌 Exercise 7. Bitwise Multiplication (`BitwiseMultiply`)

#### 1. 문제 설명 (Problem)
곱셈 지시어(`MUL`/`IMUL`)를 사용하지 않고, **쉬프트(`SHL`/`SHR`)와 덧셈(`ADD`) 지시어만으로** 32비트 정수 곱셈을 구현합니다. (러시아 농부 곱셈법 / 비트 곱셈 알고리즘)

#### 2. 비주얼 비트 곱셈 개념
```text
  25 (11001b) × 10 (1010b)
  10을 2진수로 쪼개면: 1010b = (1 × 2^3) + (0 × 2^2) + (1 × 2^1) + (0 × 2^0)
  
  -> 1비트씩 오른쪽으로 깎아내며(SHR), 비트가 1일 때만 2배씩 키운(SHL) 피승수를 합산!
```

#### 3. 초보자용 한 줄 주석 코드 (Solution)

```assembly
INCLUDE Irvine32.inc

.code
BitwiseMultiply PROC
    ; 매개변수: EAX = 피승수(곱해지는 수), EBX = 승수(곱하는 수)  /  반환: EAX = 결과값
    push edx
    push ecx

    mov edx, 0          ; [1] EDX = 곱셈 결과를 차곡차곡 더해나갈 누적 통 (0으로 초기화)

L1:
    test ebx, 1         ; [2] EBX의 맨 오른쪽 비트(LSB)가 1인지 0인지 비트 검사
    jz SkipAdd          ; [3] 0이면 덧셈을 건너뛰고 SkipAdd로 점프!

    add edx, eax        ; [4] 1이면 현재 EAX(2배씩 커지고 있는 값)를 누적 통(EDX)에 더함

SkipAdd:
    shl eax, 1          ; [5] 피승수 EAX를 왼쪽으로 1비트 밀어 2배로 확장 (EAX = EAX * 2)
    shr ebx, 1          ; [6] 승수 EBX를 오른쪽으로 1비트 밀어 다음 비트 검사 준비
    jnz L1              ; [7] EBX가 0이 될 때까지 (모든 비트 검사 완료 시까지) 반복!

    mov eax, edx        ; [8] 누적 결과가 담긴 EDX를 반환 레지스터 EAX로 옮김
    pop ecx
    pop edx
    ret
BitwiseMultiply ENDP

main PROC
    mov eax, 25
    mov ebx, 10
    call BitwiseMultiply ; EAX = 250
    call WriteDec        ; 출력: 250
    call Crlf
    exit
main ENDP
END main
```

---

### 📌 Exercise 8. Add Packed Integers (`AddPacked`)

#### 1. 문제 설명 (Problem)
임의의 바이트 크기를 가진 팩트 십진수(Packed BCD) 두 개를 더합니다. 2진수 덧셈(`ADC`)을 한 직후 **`DAA` (Decimal Adjust after Addition)** 지시어를 실행하면, CPU가 자동으로 10진수 덧셈 보정 및 올림 처리를 완료해 줍니다.

#### 2. 초보자용 한 줄 주석 코드 (Solution)

```assembly
INCLUDE Irvine32.inc

.code
AddPacked PROC
    ; 매개변수: ESI = 수1 주소, EDI = 수2 주소, EDX = 합계 저장 주소, ECX = 바이트 수
    pushad
    clc                 ; [1] 첫 바이트 덧셈 전 Carry Flag(올림수 깃발)를 0으로 리셋

L1:
    mov al, [esi]       ; [2] 수1에서 1바이트 가져오기
    adc al, [edi]       ; [3] ADC: 수2 1바이트 + 이전 턴에서 넘어온 올림수(Carry)까지 더함
    daa                 ; [4] ★ 핵심! 덧셈 결과를 10진수(BCD) 규칙에 맞게 자동 보정!
                        ;     (예: 9 + 8 = 17 이 될 때 11h가 아닌 17h로 보정 및 다음 Carry 세팅)

    mov [edx], al       ; [5] 보정 완료된 10진수 1바이트를 결과 메모리에 저장

    inc esi             ; [6] 수1의 다음 바이트 위치로 포인터 이동 (+1)
    inc edi             ; [7] 수2의 다음 바이트 위치로 포인터 이동 (+1)
    inc edx             ; [8] 결과의 다음 바이트 위치로 포인터 이동 (+1)
    loop L1             ; [9] 바이트 수(ECX)만큼 반복

    popad
    ret
AddPacked ENDP

main PROC
    .data
    val1 BYTE 34h, 12h, 98h, 56h ; 리틀엔디안 표현: 56981234h
    val2 BYTE 66h, 87h, 01h, 43h ; 리틀엔디안 표현: 43018766h
    sum  BYTE 4 DUP(0)

    .code
    mov esi, OFFSET val1    ; 수1 주소
    mov edi, OFFSET val2    ; 수2 주소
    mov edx, OFFSET sum     ; 합계 저장 주소
    mov ecx, SIZEOF val1    ; 4바이트 크기 전달

    call AddPacked
    exit
main ENDP
END main
```