# 📘 Chapter 7. Programming Exercises 풀이

x86 어셈블리 언어(Kip Irvine 교재 기준) Chapter 7 (Integer Arithmetic) Programming Exercises (1~8번) 실습 문제 풀이 노트입니다.

---

### 📌 Programming Exercise 1. Display ASCII Decimal (`WriteScaled`)

#### 1. English Problem
> Write a procedure named `WriteScaled` that outputs a decimal ASCII number with an implied decimal point. Suppose the following number were defined as follows, where `DECIMAL_OFFSET` indicates that the decimal point must be inserted five positions from the right side of the number:
>
> `DECIMAL_OFFSET = 5`  
> `.data`  
> `decimal_one BYTE "100123456789765"`
>
> `WriteScaled` would display the number like this: `1001234567.89765`  
> When calling `WriteScaled`, pass the number’s offset in `EDX`, the number length in `ECX`, and the decimal offset in `EBX`. Write a test program that passes three numbers of different sizes to the `WriteScaled` procedure.

#### 2. 문제 번역
소수점이 생략된 ASCII 숫자 문자열에 지정된 위치(`EBX`)에 소수점(`.`)을 삽입하여 출력하는 `WriteScaled` 프로시저를 작성하시오.
- `EDX`: 문자열의 오프셋 (주소)
- `ECX`: 전체 문자열 길이
- `EBX`: 오른쪽 끝에서부터 소수점이 위치할 자릿수 (`DECIMAL_OFFSET`)

#### 3. 핵심 개념 및 코드 (Solution)
* 소수점이 들어갈 인덱스는 전체 길이에서 소수점 오프셋을 뺀 지점입니다 (`ECX - EBX`).
* 루프를 돌며 인덱스가 해당 지점에 도달했을 때 `.` 문자를 먼저 출력한 후 기존 숫자를 출력합니다.

```assembly
INCLUDE Irvine32.inc

.code
WriteScaled PROC
    ; EDX = string offset, ECX = length, EBX = decimal offset
    pushad
    
    mov esi, edx        ; ESI = 문자열 포인터
    mov eax, ecx        ; EAX = 전체 길이
    sub eax, ebx        ; EAX = 소수점이 들어갈 인덱스 (ECX - EBX)
    mov edi, eax        ; EDI = 소수점 위치
    
    mov ebx, 0          ; EBX = 현재 문자 카운터

L1:
    cmp ebx, edi
    jne PrintChar
    
    ; 소수점 출력
    mov al, '.'
    call WriteChar

PrintChar:
    mov al, [esi]
    call WriteChar
    inc esi
    inc ebx
    loop L1

    call Crlf
    popad
    ret
WriteScaled ENDP

main PROC
    .data
    num1 BYTE "100123456789765"
    num2 BYTE "123456"
    num3 BYTE "9876543210"

    .code
    mov edx, OFFSET num1
    mov ecx, LENGTHOF num1
    mov ebx, 5
    call WriteScaled      ; 출력: 1001234567.89765

    mov edx, OFFSET num2
    mov ecx, LENGTHOF num2
    mov ebx, 2
    call WriteScaled      ; 출력: 1234.56

    mov edx, OFFSET num3
    mov ecx, LENGTHOF num3
    mov ebx, 4
    call WriteScaled      ; 출력: 987654.3210

    exit
main ENDP
END main
```

---

### 📌 Programming Exercise 2. Extended Subtraction Procedure (`Extended_Sub`)

#### 1. English Problem
> Create a procedure named `Extended_Sub` that subtracts two binary integers of arbitrary size. The storage size of the two integers must be the same, and their size must be a multiple of 32 bits. Write a test program that passes several pairs of integers, each at least 10 bytes long.

#### 2. 문제 번역
임의의 크기를 가진 두 이진 정수를 빼는 `Extended_Sub` 프로시저를 작성하시오. 두 정수의 크기는 동일하며 32비트(4바이트)의 배수여야 합니다. 10바이트 이상의 데이터 세트로 테스트하는 프로그램을 작성하시오.

#### 3. 핵심 개념 및 코드 (Solution)
* 다중 바이트/워드/더블워드 뺄셈은 **가장 하위 바이트(오프셋 0)**부터 시작합니다.
* 첫 32비트 뺄셈은 `SUB`를 사용하고, 그 이후 바이트들은 **빌림(Borrow)을 포함하는 `SBB`**를 사용합니다.

```assembly
INCLUDE Irvine32.inc

.code
Extended_Sub PROC
    ; ESI = op1 pointer (결과 저장 위치)
    ; EDI = op2 pointer
    ; ECX = 4바이트(DWORD) 단위 개수
    pushad

    clc             ; Carry Flag 초기화

L1:
    mov eax, [edi]  ; op2의 32비트값
    sbb [esi], eax  ; op1 = op1 - op2 - Borrow
    add esi, 4      ; 다음 4바이트(32비트)로 이동
    add edi, 4
    loop L1

    popad
    ret
Extended_Sub ENDP

main PROC
    .data
    ; 12바이트 (96비트) 정수 데이터 (배열의 시작이 LSB)
    val1 BYTE 0A1h, 47h, 20h, 36h, 04h, 30h, 40h, 20h, 12h, 34h, 56h, 78h
    val2 BYTE 0B2h, 30h, 26h, 4Ah, 30h, 10h, 52h, 05h, 01h, 02h, 03h, 04h

    .code
    mov esi, OFFSET val1
    mov edi, OFFSET val2
    mov ecx, 3          ; 12바이트 = 3 * 4바이트 (32비트 단위 3개)

    call Extended_Sub
    exit
main ENDP
END main
```

---

### 📌 Programming Exercise 3. Packed Decimal Conversion (`PackedToAsc`)

#### 1. English Problem
> Write a procedure named `PackedToAsc` that converts a 4-byte packed decimal integer to a string of ASCII decimal digits. Pass the packed integer and the address of a buffer holding the ASCII digits to the procedure. Write a short test program that passes at least 5 packed decimal integers to your procedure.

#### 2. 문제 번역
4바이트 팩트 십진수(Packed Decimal, 예: `12345678h`)를 ASCII 숫자 문자열(`"12345678"`)로 변환하는 `PackedToAsc` 프로시저를 작성하시오.

#### 3. 핵심 개념 및 코드 (Solution)
* 1바이트 팩트 십진수는 2개의 16진수 숫자로 이루어져 있습니다 (상위 4비트 nibble, 하위 4비트 nibble).
* 최상위 바이트부터 역순으로 가며 **상위 4비트와 하위 4비트를 추출(AND/SHR)한 뒤 `'0'` (`30h`)을 더해 ASCII 문자로 전환**합니다.

```assembly
INCLUDE Irvine32.inc

.code
PackedToAsc PROC
    ; EAX = 4바이트 packed decimal (예: 12345678h)
    ; EDI = ASCII 문자열을 저장할 버퍼 주소
    pushad

    mov ecx, 4          ; 4바이트 처리
    add edi, 8          ; 버퍼의 맨 끝 위치 지정 (문자열 종료 NULL 문자 설정용)
    mov BYTE PTR [edi], 0 ; NULL termination

L1:
    dec edi
    mov dl, al
    and dl, 0Fh         ; 하위 4비트 추출
    add dl, '0'         ; ASCII 변환
    mov [edi], dl       ; 버퍼에 저장

    dec edi
    mov dl, al
    shr dl, 4           ; 상위 4비트 추출
    add dl, '0'         ; ASCII 변환
    mov [edi], dl       ; 버퍼에 저장

    shr eax, 8          ; 다음 바이트로 이동
    loop L1

    popad
    ret
PackedToAsc ENDP

main PROC
    .data
    packedNum DWORD 12345678h
    buffer BYTE 9 DUP(0)

    .code
    mov eax, packedNum
    mov edi, OFFSET buffer
    call PackedToAsc

    mov edx, OFFSET buffer
    call WriteString     ; 출력: 12345678
    call Crlf
    exit
main ENDP
END main
```

---

### 📌 Programming Exercise 4. Encryption Using Rotate Operations

#### 1. English Problem
> Write a procedure that performs simple encryption by rotating each plaintext byte a varying number of positions in different directions. For example:
> `key BYTE -2, 4, 1, 0, -3, 5, 2, -4, -4, 6`  
> Your procedure should loop through a plaintext message and align the key to the first 10 bytes of the message. Rotate each plaintext byte by the amount indicated by its matching key array value. Then, align the key to the next 10 bytes of the message and repeat the process.

#### 2. 문제 번역
평문(Plaintext)의 각 바이트를 키 배열(`key`)에 지정된 회전 수만큼 비트 회전(`ROL` 또는 `ROR`)시켜 암호화하는 프로시저를 작성하시오.
- 키 값이 **음수**이면 **왼쪽 회전(`ROL`)**
- 키 값이 **양수**이면 **오른쪽 회전(`ROR`)**
- 키 배열 길이가 평문보다 짧으면 키를 반복 적용함.

#### 3. 핵심 개념 및 코드 (Solution)
* 키의 부호를 확인하여 양수/음수에 따라 회전 명령어를 분기 처리합니다.
* 키 배열 오프셋은 `Index % KeySize`로 순환합니다.

```assembly
INCLUDE Irvine32.inc

KEY_SIZE = 10

.code
RotateEncrypt PROC
    ; ESI = plaintext offset, ECX = plaintext length, EDX = key offset
    pushad
    mov ebx, 0          ; Key 인덱스

L1:
    mov al, [esi]       ; 평문 바이트
    mov dl, [edx + ebx] ; 키 값 가져오기
    
    cmp dl, 0
    jge RotateRight

RotateLeft:
    neg dl              ; 음수를 양수로 변환
    mov cl, dl
    rol al, cl
    jmp StoreByte

RotateRight:
    mov cl, dl
    ror al, cl

StoreByte:
    mov [esi], al       ; 암호화된 바이트 저장
    inc esi
    
    inc ebx             ; 다음 키로 이동
    cmp ebx, KEY_SIZE
    jl NextLoop
    mov ebx, 0          ; 키 배열 끝에 도달하면 0으로 리셋

NextLoop:
    pop ecx             ; loop 카운터 복원 패턴 사용
    dec ecx
    jnz L1

    popad
    ret
RotateEncrypt ENDP
```

---

### 📌 Programming Exercise 5. Prime Numbers (Sieve of Eratosthenes)

#### 1. English Problem
> Write a program that generates all prime numbers between 2 and 1000, using the Sieve of Eratosthenes method. Display all the prime values.

#### 2. 문제 번역
에라토스테네스의 체(Sieve of Eratosthenes) 알고리즘을 사용하여 2부터 1000 사이의 모든 소수(Prime Number)를 구하고 화면에 출력하는 프로그램을 작성하시오.

#### 3. 핵심 개념 및 코드 (Solution)
* 1000개의 바이트 배열을 생성하고 모든 요소를 0으로 초기화합니다.
* 2부터 시작하여 소수인 수의 배수(Composite Number)들을 모두 1로 지워나갑니다.

```assembly
INCLUDE Irvine32.inc

MAX_VAL = 1000

.data
sieve BYTE MAX_VAL DUP(0) ; 0 = 소수 후보, 1 = 합성수

.code
main PROC
    mov ecx, 2          ; 2부터 시작

FilterLoop:
    cmp ecx, MAX_VAL
    jge DisplayPrimes

    mov esi, OFFSET sieve
    cmp BYTE PTR [esi + ecx], 1
    je NextP             ; 이미 소수가 아니면 패스

    ; 배수들 지우기 (p*2, p*3, ...)
    mov ebx, ecx
    add ebx, ecx

MarkMultiples:
    cmp ebx, MAX_VAL
    jge NextP
    mov BYTE PTR [esi + ebx], 1
    add ebx, ecx
    jmp MarkMultiples

NextP:
    inc ecx
    jmp FilterLoop

DisplayPrimes:
    mov ecx, 2

PrintLoop:
    cmp ecx, MAX_VAL
    jge Done
    mov esi, OFFSET sieve
    cmp BYTE PTR [esi + ecx], 0
    jne SkipPrint

    mov eax, ecx
    call WriteDec
    mov al, ' '
    call WriteChar

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

### 📌 Programming Exercise 6. Greatest Common Divisor (`GCD`)

#### 1. English Problem
> Implement the Greatest Common Divisor algorithm in assembly language and write a test program that calls the function several times, passing it different values. Display all results on the screen.
> ```cpp
> int GCD(int x, int y) {
>     x = abs(x); y = abs(y);
>     do {
>         int n = x % y;
>         x = y;
>         y = n;
>     } while (y > 0);
>     return x;
> }
> ```

#### 2. 문제 번역
유클리드 호제법(Euclidean Algorithm)을 사용하여 두 정수의 최대공약수(GCD)를 구하는 프로시저를 작성하고, 여러 값으로 테스트하여 결과를 출력하시오.

#### 3. 핵심 개념 및 코드 (Solution)
* 나눗셈 후 **나머지($x \pmod y$)가 0이 될 때까지 $x \leftarrow y, y \leftarrow \text{나머지}$의 루프를 반복**합니다.

```assembly
INCLUDE Irvine32.inc

.code
GCD PROC
    ; EAX = x, EBX = y
    push edx

    ; 절대값 처리
    cmp eax, 0
    jge CheckY
    neg eax

CheckY:
    cmp ebx, 0
    jge GCD_Loop
    neg ebx

GCD_Loop:
    mov edx, 0
    div ebx             ; EDX = EAX % EBX
    mov eax, ebx        ; x = y
    mov ebx, edx        ; y = n (나머지)

    cmp ebx, 0
    jg GCD_Loop         ; y > 0 동안 반복

    pop edx
    ret                 ; 결과 EAX 반환
GCD ENDP

main PROC
    mov eax, 48
    mov ebx, 18
    call GCD            ; EAX = 6
    call WriteDec
    call Crlf
    exit
main ENDP
END main
```

---

### 📌 Programming Exercise 7. Bitwise Multiplication (`BitwiseMultiply`)

#### 1. English Problem
> Write a procedure named `BitwiseMultiply` that multiplies any unsigned 32-bit integer by `EAX`, using only shifting and addition. Pass the integer to the procedure in the `EBX` register, and return the product in the `EAX` register.

#### 2. 문제 번역
곱셈 지시어(`MUL`/`IMUL`)를 전혀 사용하지 않고 **쉬프트(`SHL`/`SHR`)와 덧셈(`ADD`)만으로 두 32비트 정수를 곱하는** `BitwiseMultiply` 프로시저를 작성하시오.

#### 3. 핵심 개념 및 코드 (Solution)
* **러시아 농부 곱셈법(Russian Peasant Multiplication) / 비트 곱셈:**
  * 승수(`EBX`)를 오른쪽으로 1비트씩 쉬프트하면서 비트가 1일 때만 피승수(`EAX`)를 결과에 더합니다.
  * 피승수(`EAX`)는 매 루프마다 2배씩(`SHL 1`) 확장합니다.

```assembly
INCLUDE Irvine32.inc

.code
BitwiseMultiply PROC
    ; EAX = multiplicand, EBX = multiplier
    ; Return: EAX = product
    push edx
    push ecx

    mov edx, 0          ; EDX = 누적 합계 (Product)

L1:
    test ebx, 1         ; LSB가 1인지 확인
    jz SkipAdd
    add edx, eax        ; 1이면 현재 EAX 값을 누적 합계에 더함

SkipAdd:
    shl eax, 1          ; 피승수를 2배 확장
    shr ebx, 1          ; 승수를 오른쪽으로 1비트 이동
    jnz L1              ; 승수가 0이 될 때까지 반복

    mov eax, edx        ; 결과를 EAX에 반환
    pop ecx
    pop edx
    ret
BitwiseMultiply ENDP

main PROC
    mov eax, 25
    mov ebx, 10
    call BitwiseMultiply ; EAX = 250
    call WriteDec
    call Crlf
    exit
main ENDP
END main
```

---

### 📌 Programming Exercise 8. Add Packed Integers (`AddPacked`)

#### 1. English Problem
> Extend the `AddPacked` procedure from Section 7.6.1 so that it adds two packed decimal integers of arbitrary size (both lengths must be the same). Pass parameters using ESI, EDI, EDX, ECX.

#### 2. 문제 번역
임의 크기(4, 8, 16바이트)의 두 팩트 십진수(Packed Decimal)를 더하는 `AddPacked` 프로시저를 작성하시오.
- `ESI`: 첫 번째 숫자 주소
- `EDI`: 두 번째 숫자 주소
- `EDX`: 합계 저장 주소
- `ECX`: 바이트 크기

#### 3. 핵심 개념 및 코드 (Solution)
* 가장 하위 바이트부터 1바이트씩 가져와 **Carry를 포함하는 `ADC` 후, BCD 보정을 위한 `DAA` 지시어를 호출**합니다.

```assembly
INCLUDE Irvine32.inc

.code
AddPacked PROC
    ; ESI = ptr num1, EDI = ptr num2, EDX = ptr sum, ECX = byte count
    pushad
    clc                 ; Carry flag 초기화

L1:
    mov al, [esi]
    adc al, [edi]       ; Carry 포함 덧셈
    daa                 ; Decimal Adjust after Addition (BCD 보정)
    mov [edx], al       ; 결과 저장

    inc esi
    inc edi
    inc edx
    loop L1

    popad
    ret
AddPacked ENDP

main PROC
    .data
    val1 BYTE 34h, 12h, 98h, 56h ; 56981234h
    val2 BYTE 66h, 87h, 01h, 43h ; 43018766h
    sum  BYTE 4 DUP(0)

    .code
    mov esi, OFFSET val1
    mov edi, OFFSET val2
    mov edx, OFFSET sum
    mov ecx, SIZEOF val1

    call AddPacked
    exit
main ENDP
END main
```
```


`Chapter7_Programming_Exercises.md` 파일 생성을 완료했습니다. 우측 문서 에디터 화면에서 코드와 문제 설명을 편하게 확인하고 내보낼 수 있습니다.