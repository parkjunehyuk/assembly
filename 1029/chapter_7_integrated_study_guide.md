# 📘 Chapter 7. 정수 연산 (Integer Arithmetic) 종합 스터디 가이드

이 문서는 Kip Irvine 교재 **Chapter 7 (Integer Arithmetic)**의 세 가지 핵심 파트(개념 및 단답형 퀴즈, 복습 연습문제, 프로그래밍 실습)를 하나로 종합한 통합 학습 노트입니다.

---

## 📂 문서 구성 및 목차

| 파트 | 주요 내용 | 주요 학습 목표 |
| :--- | :--- | :--- |
| **Part 1. 개념 및 단답형 퀴즈 (Q1~Q10)** | Shift/Rotate, 레지스터 분할(`EDX:EAX`), 나눗셈 오버플로우 | 어셈블리 정수 연산 지시어의 기본 작동 메커니즘 이해 |
| **Part 2. 복습 및 연습문제 (Review Q1~Q16)** | 비트 확장, 표현식 구현, ASCII/BCD 보정, 심화 알고리즘 | 조건문 없이 절댓값 구하기, Bitwise 나머지 연산 등 응용력 강화 |
| **Part 3. 프로그래밍 실습 (Exercise 1~8)** | `WriteScaled`, `Extended_Sub`, 에라토스테네스의 체, GCD | MASM 라이브러리와 루프를 활용한 실제 어셈블리 프로그램 작성 |

---

## 💡 Chapter 7 핵심 개념 요약

### 1. 비트 회전 지시어 (`ROL/ROR` vs `RCL/RCR`)
- **`ROL / ROR`**: 레지스터 내부 비트들끼리만 순환 회전합니다 (Carry Flag는 복사본만 저장).
- **`RCL / RCR`**: **Carry Flag(CF)를 하나의 비트 칸으로 포함**하여 회전합니다. 여러 레지스터를 징검다리처럼 연결하여 비트를 넘길 때 필수적입니다.

### 2. Double-Register 구조 (`EDX:EAX` / `DX:AX`)
- 곱셈(`MUL/IMUL`) 결과 자릿수가 2배로 확장되거나, 나눗셈(`DIV/IDIV`) 피나눗수의 통을 나누는 수보다 크게 확보해야 할 때 사용합니다.
- **주의사항**: 부호 없는 나눗셈 수행 전에는 상위 통인 **`EDX`를 반드시 `0`으로 초기화(`mov edx, 0`)**해야 오버플로우 및 계산 오류를 방지할 수 있습니다.

---

# Part 1. 개념 및 단답형 퀴즈 (Questions 1 ~ 10)

### 📌 Q1. Shift 및 Rotate 기본 연산
* **`shr al, 1`** (`0D4h` 기준) $\rightarrow$ `6Ah`
* **`sar al, 1`** $\rightarrow$ `EAh` (부호비트 1 유지)
* **`sar al, 4`** $\rightarrow$ `FDh`
* **`rol al, 1`** $\rightarrow$ `A9h`

### 📌 Q2. Carry 포함 Rotate 연산
* **`ror al, 3`** $\rightarrow$ `9Ah`
* **`rol al, 7`** $\rightarrow$ `6Ah` (오른쪽 1칸 회전과 동일)
* **`stc` + `rcl al, 1`** $\rightarrow$ `A9h` (CF 포함 9비트 회전)
* **`stc` + `rcr al, 3`** $\rightarrow$ `9Ah`

### 📌 Q3. 16비트 곱셈 (`mul cx`)
- 연산: `AX (0222h) * CX (0100h) = 00022200h`
- **결과 분할 저장**: `DX = 0002h` (상위 16비트), `AX = 2200h` (하위 16비트)

### 📌 Q4. 8비트 나눗셈 (`div bl`)
- 연산: `AX (63h = 99) / BL (10h = 16)`
- **결과 저장**: `AL = 06h` (몫), `AH = 03h` (나머지) $\rightarrow$ **`AX = 0306h`**

### 📌 Q5. 32비트 나눗셈 (`div ebx`)
- 연산: `EDX:EAX (00000000_00123400h) / EBX (10h)`
- **결과 저장**: `EAX = 00012340h` (몫), `EDX = 00000000h` (나머지)

### 📌 Q6. 16비트 나눗셈 오버플로우
- 피나눗수 `05004000h` / `10h` = 몫 `500400h`
- 몫이 `AX` 레지스터의 한계치(`FFFFh`)를 초과하여 **Divide Overflow Exception** 발생.

### 📌 Q7. Carry 포함 가산 (`adc bx, ax`)
- 연산: `BX(5) + AX(60h) + CF(1) = 66h` $\rightarrow$ **`BX = 0066h`**

### 📌 Q8. 64비트 나눗셈 (`div divisor`)
- 피나눗수 `RDX:RAX (00000108_33300020h)` / `100h`
- **결과**: `RAX = 0000000108333000h` (몫), `RDX = 0000000000000020h` (나머지)

### 📌 Q9. 다중 바이트 뺄셈 오류 수정
- 하위 바이트부터 진행해야 하므로 포인터를 오프셋의 끝(`+7`)으로 설정하고 `sbb`를 사용해 차례로 차감.
```assembly
mov esi, OFFSET val1 + 7
mov edi, OFFSET val2 + 7
mov ebx, OFFSET result + 7
clc
top:
    mov al, BYTE PTR [esi]
    sbb al, BYTE PTR [edi]
    mov BYTE PTR [ebx], al
    dec esi
    dec edi
    dec ebx
    loop top
```

### 📌 Q10. 3-피연산자 `IMUL`
- `imul rax, multiplicand, 4` (`multiplicand = 0001020304050000h`)
- **결과**: **`RAX = 0004080C10140000h`**

---

# Part 2. 복습 및 연습문제 (Review Q1 ~ Q16)

### 📌 RQ 1. `CWD` 없이 부호 확장
```assembly
shl eax, 16    ; AX를 EAX 상위로 이동
sar eax, 16    ; 부호 유지 오른쪽 이동
```

### 📌 RQ 2. `ROR` 지시어 시뮬레이션
```assembly
shr al, 1
jnc L_SKIP
or  al, 80h    ; CF가 1이면 MSB 채움
L_SKIP:
```

### 📌 RQ 3 & 4. Shift 기반 곱셈/나눗셈
- 16 곱하기: `shl eax, 4` ($2^4 = 16$)
- 4 나누기: `shr ebx, 2` ($2^2 = 4$)

### 📌 RQ 5.  Nibble Swapping
- `DL` 상위/하위 4비트 교환: `ror dl, 4` (또는 `rol dl, 4`)

### 📌 RQ 6. `SHLD` 활용
- `AX` MSB를 `DX` LSB로 비트 이동: `shld dx, ax, 1`

### 📌 RQ 7 & 8. 다중 바이트/워드 쉬프트
- **오른쪽 쉬프트 (3 바이트)**:
  ```assembly
  shr byteArray + 2, 1
  rcr byteArray + 1, 1
  rcr byteArray, 1
  ```
- **왼쪽 쉬프트 (3 워드)**:
  ```assembly
  shl wordArray, 1
  rcl wordArray + 2, 1
  rcl wordArray + 4, 1
  ```

### 📌 RQ 9 & 10. 곱셈 및 나눗셈 변수 저장
- **$5 \times 3$**: `mov ax, 5` $\rightarrow$ `mov cx, 3` $\rightarrow$ `mul cx` $\rightarrow$ `mov val1, ax`
- **$276 \div 10$**: `mov dx, 0` $\rightarrow$ `mov ax, 276` $\rightarrow$ `mov cx, 10` $\rightarrow$ `div cx` $\rightarrow$ `mov val1, ax`

### 📌 RQ 11 & 12. C++ 표현식 어셈블리 변환
- Unsigned `val1 = (val2 * val3) / (val4 - 3)`:
  ```assembly
  mov ebx, val4
  sub ebx, 3
  mov eax, val2
  mul val3
  div ebx
  mov val1, eax
  ```
- Signed `val1 = (val2 / val3) * (val1 + val2)`:
  ```assembly
  mov eax, val2
  cdq
  idiv val3
  mov ebx, val1
  add ebx, val2
  imul eax, ebx
  mov val1, eax
  ```

### 📌 RQ 13. `showDecimal8` 프로시저
```assembly
showDecimal8 PROC
    mov ah, 0
    mov bl, 10
    div bl          ; AL = 몫, AH = 나머지
    mov bh, ah
    add al, '0'
    call WriteChar
    mov al, bh
    add al, '0'
    call WriteChar
    ret
showDecimal8 ENDP
```

### 📌 RQ 14. [Challenge] `AAA` 지시어 분석
- `AX = 0072h`, `AC = 1` 상태에서 `AAA` 실행 시:
  - `AL = 72h + 6 = 78h`, `AH = 00h + 1 = 01h`, `AL = 78h AND 0Fh = 08h`
  - **최종 결과**: **`AX = 0108h`**

### 📌 RQ 15. [Challenge] 비트 연산 기반 Modulo ($x = n \bmod y$)
```assembly
mov ecx, ebx    ; ebx = y (2의 거듭제곱)
sub ecx, 1      ; 비트 마스크 생성
and eax, ecx    ; eax = n AND (y - 1)
```

### 📌 RQ 16. [Challenge] 분기문 없는 절댓값 계산
```assembly
mov ebx, eax
sar ebx, 31     ; 양수면 0, 음수면 -1(모든 비트 1)
xor eax, ebx    ; 음수일 경우 비트 반전
add eax, ebx    ; 음수일 경우 1을 더해 2의 보수 완성
```

---

# Part 3. 프로그래밍 실습 (Exercises 1 ~ 8)

### 📌 Ex 1. `WriteScaled` (소수점 삽입 출력)
- 문자열의 `(전체길이 - 소수점 오프셋)` 위치에 도달했을 때 `.` 문자를 선제 출력하도록 제어.

### 📌 Ex 2. `Extended_Sub` (임의 크기 뺄셈)
- 32비트 단위로 배열을 순회하며 첫 요소 이후는 빌림을 고려하여 `SBB` 연산 적용.

### 📌 Ex 3. `PackedToAsc` (팩트 십진수 $\rightarrow$ ASCII 변환)
- 각 바이트의 상위/하위 Nibble을 `AND` 및 `SHR`로 분리 후 `'0'`(`30h`)을 더해 ASCII 문자로 매핑.

### 📌 Ex 4. Rotate 기반 암호화
- 키 배열의 음수/양수 여부에 따라 `ROL`(음수) 및 `ROR`(양수) 지시어로 분기하여 비트 회전 암호화 수행.

### 📌 Ex 5. 에라토스테네스의 체 (소수 생성)
- 1000바이트 배열을 활용해 소수의 배수 위치를 1로 마킹하고, 0으로 남아있는 인덱스들을 소수로 출력.

### 📌 Ex 6. 최대공약수 (`GCD`)
- 유클리드 호제법을 어셈블리로 구현. 나머지가 0이 될 때까지 `DIV` 연산 반복.

### 📌 Ex 7. `BitwiseMultiply` (비트 쉬프트 곱셈)
- 승수(`EBX`)를 1비트씩 오른쪽으로 쉬프트(`SHR`)하면서 LSB가 1일 때만 피승수(`EAX`)를 결과에 누적.

### 📌 Ex 8. `AddPacked` (임의 크기 BCD 덧셈)
- `ADC`로 바이트 단위 덧셈 후 `DAA` 지시어를 호출하여 BCD 보정 처리.
```

세 파일의 핵심 내용을 정리한 통합 `Chapter7_Complete_Study_Guide.md` 문서 작성을 완료했습니다!