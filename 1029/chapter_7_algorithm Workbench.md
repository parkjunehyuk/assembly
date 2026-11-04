# 📘 Chapter 7. Review Questions and Exercises 풀이

x86 어셈블리 언어(Kip Irvine 교재 기준) Chapter 7 (Integer Arithmetic) Review Questions & Exercises (1~16번) 풀이 노트입니다.

---

### 📌 Question 1. Sign Extension without CWD

#### 1. English Problem
> Write a sequence of shift instructions that cause `AX` to be sign-extended into `EAX`. In other words, the sign bit of `AX` is copied into the upper 16 bits of `EAX`. Do not use the `CWD` instruction.

#### 2. 문제 번역
`CWD` 지시어를 사용하지 않고, `AX` 레지스터의 부호 비트(Sign Bit)를 `EAX` 레지스터의 상위 16비트로 복사(부호 확장)하는 쉬프트 지시어 시퀀스를 작성하시오.

#### 3. 풀이 및 코드 (Solution)
`EAX` 전체를 먼저 16비트 왼쪽으로 밀어 올린 후, **산술 오른쪽 쉬프트(`SAR`)**를 수행하면 최상위 부호 비트가 비어있는 상위 16비트로 자동으로 채워집니다.

```assembly
shl eax, 16    ; AX에 있던 값을 EAX의 상위 16비트 위치로 이동
sar eax, 16    ; 부호 비트를 유지하면서 다시 원위치로 당김 (상위 16비트가 부호 비트로 채워짐)
```

---

### 📌 Question 2. Simulating ROR with SHR and Conditional Jump

#### 1. English Problem
> Suppose the instruction set contained no rotate instructions. Show how you would use `SHR` and a conditional jump instruction to rotate the contents of the `AL` register 1 bit to the right.

#### 2. 문제 번역
회전(Rotate) 지시어가 없다고 가정할 때, `SHR`과 조건 점프 지시어를 사용하여 `AL` 레지스터의 내용을 오른쪽으로 1비트 순환 회전(`ROR al, 1`)시키는 코드를 작성하시오.

#### 3. 풀이 및 코드 (Solution)
`SHR`을 수행할 때 밀려나간 최하위 비트(LSB)는 **Carry Flag(`CF`)**에 저장됩니다. `CF`가 1이면 `AL`의 최상위 비트(MSB, `80h`)를 켜주고, 0이면 그대로 둡니다.

```assembly
shr al, 1       ; LSB가 Carry Flag(CF)로 들어가고 AL은 오른쪽으로 1비트 이동
jnc L_SKIP      ; Carry Flag가 0이면 점프 (아무 작업 안 함)
or  al, 80h     ; Carry Flag가 1이면 맨 왼쪽 비트(MSB)에 1을 채워줌

L_SKIP:
```

---

### 📌 Question 3. Multiplication by Shift Instruction

#### 1. English Problem
> Write a logical shift instruction that multiplies the contents of `EAX` by 16.

#### 2. 문제 번역
`EAX` 레지스터의 값에 16을 곱하는 논리 쉬프트 지시어를 작성하시오.

#### 3. 풀이 및 코드 (Solution)
2진수에서 왼쪽으로 1비트 쉬프트(`SHL`)할 때마다 2배씩 증가합니다. $16 = 2^4$ 이므로 **왼쪽으로 4비트 쉬프트**하면 16 곱셈이 됩니다.

```assembly
shl eax, 4      ; EAX = EAX * 16
```

---

### 📌 Question 4. Division by Shift Instruction

#### 1. English Problem
> Write a logical shift instruction that divides `EBX` by 4.

#### 2. 문제 번역
`EBX` 레지스터의 값을 4로 나누는 논리 쉬프트 지시어를 작성하시오.

#### 3. 풀이 및 코드 (Solution)
부호 없는(Unsigned) 수의 나눗셈은 오른쪽 논리 쉬프트(`SHR`)를 사용합니다. $4 = 2^2$ 이므로 **오른쪽으로 2비트 쉬프트**하면 4 나눗셈이 됩니다.

```assembly
shr ebx, 2      ; EBX = EBX / 4
```

---

### 📌 Question 5. Swapping Nibbles in DL Register

#### 1. English Problem
> Write a single rotate instruction that exchanges the high and low halves of the `DL` register.

#### 2. 문제 번역
`DL` 레지스터의 상위 4비트(High Half)와 하위 4비트(Low Half)를 서로 교환하는 단일 회전 지시어를 작성하시오.

#### 3. 풀이 및 코드 (Solution)
8비트 레지스터에서 반통(4비트)을 교환하려면 **4칸 순환 회전(`ROL` 또는 `ROR`)**을 해주면 됩니다.

```assembly
ror dl, 4       ; 또는 rol dl, 4
```

---

### 📌 Question 6. Shift Bits Between Registers with SHLD

#### 1. English Problem
> Write a single `SHLD` instruction that shifts the highest bit of the `AX` register into the lowest bit position of `DX` and shifts `DX` one bit to the left.

#### 2. 문제 번역
`AX` 레지스터의 최상위 비트를 `DX` 레지스터의 최하위 비트 위치로 넣고, 기존 `DX`를 왼쪽으로 1비트 이동시키는 단일 `SHLD` 지시어를 작성하시오.

#### 3. 풀이 및 코드 (Solution)
`SHLD destination, source, count` 지시어는 `source`의 상위 비트들을 가져와 `destination`을 왼쪽으로 밀어 넣습니다.

```assembly
shld dx, ax, 1  ; DX를 왼쪽으로 1비트 밀면서, AX의 최상위 비트(MSB)를 DX의 LSB 위치로 들어옴
```

---

### 📌 Question 7. Multi-Byte Right Shift

#### 1. English Problem
> Write a sequence of instructions that shift three memory bytes to the right by 1 bit position. Use the following test data:  
> `byteArray BYTE 81h, 20h, 33h`

#### 2. 문제 번역
`byteArray BYTE 81h, 20h, 33h` 데이터 세트에 저장된 3바이트 메모리를 오른쪽으로 1비트 쉬프트하는 지시어 시퀀스를 작성하시오.

#### 3. 풀이 및 코드 (Solution)
여러 바이트 데이터의 오른쪽 쉬프트는 **가장 상위 바이트(오프셋 `+ 2`)에서 `SHR`로 밀고, 밀려나온 `CF`를 하위 바이트로 `RCR`을 이용해 전달**해야 합니다.

```assembly
shr byteArray + 2, 1   ; 가장 상위 바이트를 SHR (밀려난 LSB는 CF에 저장)
rcr byteArray + 1, 1   ; CF를 받아서 중간 바이트 RCR
rcr byteArray,     1   ; CF를 받아서 가장 하위 바이트 RCR
```

---

### 📌 Question 8. Multi-Word Left Shift

#### 1. English Problem
> Write a sequence of instructions that shift three memory words to the left by 1 bit position. Use the following test data:  
> `wordArray WORD 810Dh, 0C064h, 93ABh`

#### 2. 문제 번역
`wordArray WORD 810Dh, 0C064h, 93ABh` 데이터 세트에 저장된 3워드 데이터를 왼쪽으로 1비트 쉬프트하는 지시어 시퀀스를 작성하시오.

#### 3. 풀이 및 코드 (Solution)
왼쪽 쉬프트는 **가장 하위 워드(오프셋 0)부터 시작하여 `SHL`로 밀고, 밀려나온 `CF`를 `RCL`로 전달**해야 합니다.

```assembly
shl wordArray,     1   ; 가장 하위 워드 SHL (밀려난 MSB는 CF에 저장)
rcl wordArray + 2, 1   ; CF를 받아서 두 번째 워드 RCL
rcl wordArray + 4, 1   ; CF를 받아서 가장 상위 워드 RCL
```

---

### 📌 Question 9. Multiplication and Store Result

#### 1. English Problem
> Write instructions that multiply 5 by 3 and store the result in a 16-bit variable `val1`.

#### 2. 문제 번역
5와 3을 곱한 결과를 16비트 변수 `val1`에 저장하는 지시어를 작성하시오.

#### 3. 풀이 및 코드 (Solution)
`AX = 5`로 두고 `MOV CX, 3` 후 `MUL CX`를 실행하면 결과는 `DX:AX`에 들어가며, 결과가 작으므로 `AX` 값을 `val1`에 넣으면 됩니다.

```assembly
mov ax, 5
mov cx, 3
mul cx          ; DX:AX = 15 (AX = 000Fh, DX = 0000h)
mov val1, ax    ; val1에 결과 15 저장
```

---

### 📌 Question 10. Division and Store Result

#### 1. English Problem
> Write instructions that divide 276 by 10 and store the result in a 16-bit variable `val1`.

#### 2. 문제 번역
276을 10으로 나눈 몫을 16비트 변수 `val1`에 저장하는 지시어를 작성하시오.

#### 3. 풀이 및 코드 (Solution)
16비트 나눗셈 전, 피나눗수를 담는 상위 그릇인 `DX`를 `0`으로 초기화해야 합니다 (`DX:AX = 276`).

```assembly
mov dx, 0
mov ax, 276
mov cx, 10
div cx          ; AX = 몫(27), DX = 나머지(6)
mov val1, ax    ; val1에 몫 저장
```

---

### 📌 Question 11. C++ Unsigned Expression Translation

#### 1. English Problem
> Implement the following C++ expression in assembly language, using 32-bit unsigned operands:  
> `val1 = (val2 * val3) / (val4 - 3)`

#### 2. 문제 번역
다음 C++ 식을 32비트 부호 없는(Unsigned) 피연산자를 사용하여 어셈블리로 구현하시오:  
`val1 = (val2 * val3) / (val4 - 3)`

#### 3. 풀이 및 코드 (Solution)
```assembly
; 1. 분모 계산 (val4 - 3)
mov ebx, val4
sub ebx, 3

; 2. 분자 계산 (val2 * val3)
mov eax, val2
mul val3        ; EDX:EAX = val2 * val3

; 3. 나눗셈 수행 (EDX:EAX / EBX)
div ebx         ; EAX = 몫, EDX = 나머지

; 4. 결과 저장
mov val1, eax
```

---

### 📌 Question 12. C++ Signed Expression Translation

#### 1. English Problem
> Implement the following C++ expression in assembly language, using 32-bit signed operands:  
> `val1 = (val2 / val3) * (val1 + val2)`

#### 2. 문제 번역
다음 C++ 식을 32비트 부호 있는(Signed) 피연산자를 사용하여 어셈블리로 구현하시오:  
`val1 = (val2 / val3) * (val1 + val2)`

#### 3. 풀이 및 코드 (Solution)
부호 있는 나눗셈은 `IDIV`를 사용하며, 피나눗수를 확장할 때 `CDQ` 지시어로 `EAX`를 `EDX:EAX`로 부호 확장해야 합니다.

```assembly
; 1. 나눗셈 수행 (val2 / val3)
mov eax, val2
cdq             ; EAX 부호 확장 -> EDX:EAX
idiv val3       ; EAX = 몫 (val2 / val3)

; 2. 덧셈 수행 (val1 + val2)
mov ebx, val1
add ebx, val2

; 3. 곱셈 수행
imul eax, ebx   ; EAX = EAX * EBX

; 4. 결과 저장
mov val1, eax
```

---

### 📌 Question 13. Procedure `showDecimal8` Implementation

#### 1. English Problem
> Write a procedure that displays an unsigned 8-bit binary value in decimal format. Pass the binary value in `AL`. The input range is limited to 0 to 99, decimal. The only procedure you can call from the book’s link library is `WriteChar`. The procedure should contain approximately eight instructions. Here is a sample call:  
> ```assembly
> mov al, 65 ; range limit: 0 to 99
> call showDecimal8
> ```

#### 2. 문제 번역
`AL` 레지스터로 전달된 8비트 부호 없는 정수(범위: 0~99)를 10진수로 출력하는 프로시저를 작성하시오. (도서 라이브러리 함수는 `WriteChar`만 호출 가능)

#### 3. 풀이 및 코드 (Solution)
10으로 나누어 십의 자리(몫)와 일의 자리(나머지)를 구한 뒤, 각각 `'0'` (`30h`)을 더해 ASCII 문자로 변환하여 출력합니다.

```assembly
showDecimal8 PROC
    mov ah, 0       ; AX = AL (0 ~ 99)
    mov bl, 10
    div bl          ; AL = 몫(십의 자리), AH = 나머지(일의 자리)

    mov bh, ah      ; 일의 자리(나머지)를 BH에 잠시 보관

    ; 십의 자리 출력
    add al, '0'     ; ASCII 문자로 변환
    call WriteChar

    ; 일의 자리 출력
    mov al, bh
    add al, '0'     ; ASCII 문자로 변환
    call WriteChar

    ret
showDecimal8 ENDP
```

---

### 📌 Question 14. [Challenge] AAA Instruction Output

#### 1. English Problem
> Challenge: Suppose `AX` contains `0072h` and the Auxiliary Carry flag is set as a result of adding two unknown ASCII decimal digits. Use the Intel 64 and IA-32 Instruction Set Reference to determine what output the `AAA` instruction would produce. Explain your answer.

#### 2. 문제 번역
`AX = 0072h`이고 덧셈 결과로 Auxiliary Carry Flag(`AC`)가 1로 설정되어 있을 때, `AAA` (ASCII Adjust after Addition) 지시어가 실행되면 출력 결과는 무엇인지 설명하시오.

#### 3. 풀이 및 분석 (Solution & Explanation)
* **`AAA` 동작 규칙:** `AL` 하위 4비트가 9보다 크거나 `AC=1`인 경우:
  1. `AL = AL + 6`
  2. `AH = AH + 1`
  3. `AC = 1`, `CF = 1`
  4. `AL`의 상위 4비트는 `0`으로 클리어됨 (`AL = AL AND 0FH`)
* **계산 과정:**
  * 현재 `AX = 0072h` (`AH = 00h`, `AL = 72h`), `AC = 1`
  * `AC = 1` 조건을 만족하므로:
    * `AL = 72h + 6 = 78h`
    * `AH = 00h + 1 = 01h`
    * `AL = 78h AND 0Fh = 08h`
* **최종 정답:** **`AX = 0108h`**

---

### 📌 Question 15. [Challenge] Modulo using Bitwise Logic

#### 1. English Problem
> Challenge: Using only `SUB`, `MOV`, and `AND` instructions, show how to calculate $x = n \bmod y$, assuming that you are given the values of $n$ and $y$. You can assume that $n$ is any 32-bit unsigned integer, and $y$ is a power of 2.

#### 2. 문제 번역
`SUB`, `MOV`, `AND` 지시어만을 사용하여 $x = n \bmod y$를 구하는 코드를 작성하시오. ($y$는 2의 거듭제곱 수)

#### 3. 풀이 및 코드 (Solution)
$y$가 2의 거듭제곱($2^k$)일 때, $n \bmod y$의 결과는 $n$의 하위 $k$개 비트의 값과 정확히 같습니다. 비트 마스크 값은 $y - 1$이 됩니다. (예: $y = 16(10000_2)$ 이면 $y - 1 = 15(01111_2)$)

```assembly
; n = EAX, y = EBX 라고 가정
mov ecx, ebx    ; ECX = y
sub ecx, 1      ; ECX = y - 1 (마스크 생성)
and eax, ecx    ; EAX = n AND (y - 1)  -> mod 연산 완료
```

---

### 📌 Question 16. [Challenge] Absolute Value without Conditional Jumps

#### 1. English Problem
> Challenge: Using only `SAR`, `ADD`, and `XOR` instructions (but no conditional jumps), write code that calculates the absolute value of the signed integer in the `EAX` register. Hints: A number can be negated by adding 1 to it and then forming its one’s complement. Also, if you XOR an integer with all 1s, its 1s are reversed. On the other hand, if you XOR an integer with all zeros, the integer is unchanged.

#### 2. 문제 번역
조건 점프(`JMP`) 없이 `SAR`, `ADD`, `XOR` 지시어만으로 `EAX`에 들어있는 부호 있는 정수의 절대값($|EAX|$)을 구하는 코드를 작성하시오.

#### 3. 풀이 및 코드 (Solution)
`sar eax, 31`을 수행하면:
* `EAX`가 양수이면 전체 비트가 **`00000000h` (`0`)**가 됨
* `EAX`가 음수이면 전체 비트가 **`FFFFFFFFh` (`-1`)**가 됨

```assembly
mov ebx, eax    ; 원본 값 복사
sar ebx, 31     ; EAX가 양수면 EBX = 0, 음수면 EBX = -1 (비트 전체 1)
xor eax, ebx    ; 양수면 (EAX XOR 0 = EAX), 음수면 (EAX 비트 반전)
add eax, ebx    ; 양수면 (EAX + 0 = EAX), 음수면 (EAX - 1, 즉 비트 반전 후 +1 효과)
```