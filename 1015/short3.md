# 📘 Assembly Language for x86 Processors (7th Edition)

> **Chapter 6: Conditional Processing — Review Questions & Exercises Solution**

This repository contains full problem statements, Korean translations, step-by-step explanations, and answers for **Chapter 6 (Section 6.10 Review Questions and Exercises, Q1–Q19)** of Kip Irvine's *Assembly Language for x86 Processors (7th Edition)*.

---

## 📌 Table of Contents

* [Q1 ~ Q6: Bitwise Operations](#-q1--q6-bitwise-operations)
* [Q7 ~ Q9: Instruction Sequences & Flags](#-q7--q9-instruction-sequences--flags)
* [Q10 ~ Q16: Conditional Jumps & Control Flow](#-q10--q16-conditional-jumps--control-flow)
* [Q17 ~ Q19: 64-Bit Bitwise Operations](#-q17--q19-64-bit-bitwise-operations)

---

## 🔹 Q1 ~ Q6: Bitwise Operations

### 📍 Question 1
* **Problem:**
  ```assembly
  mov  bx, 0FFFFh
  and  bx, 6Bh
  ```
  What will be the value of `BX` after the following instructions execute?
* **한국어 해석:** 다음 지시어들이 실행된 후 `BX` 레지스터의 값은 무엇입니까?
* **풀이:**
  * `0FFFFh`는 16비트의 모든 비트가 `1`인 상태입니다 (`1111 1111 1111 1111b`).
  * 어떤 값에 `1`을 `AND` 연산하면 자기 자신의 값이 그대로 유지됩니다 ($X \land 1 = X$).
* **Answer:** `BX = 006Bh`

---

### 📍 Question 2
* **Problem:**
  ```assembly
  mov  bx, 91BAh
  and  bx, 92h
  ```
  What will be the value of `BX` after the following instructions execute?
* **한국어 해석:** 다음 지시어들이 실행된 후 `BX` 레지스터의 값은 무엇입니까?
* **풀이:**
  * **상위 바이트:** `91h AND 00h` = `00h`
  * **하위 바이트:** `BAh (1011 1010b) AND 92h (1001 0010b)` = `92h (1001 0010b)`
* **Answer:** `BX = 0092h`

---

### 📍 Question 3
* **Problem:**
  ```assembly
  mov  bx, 0649Bh
  or   bx, 3Ah
  ```
  What will be the value of `BX` after the following instructions execute?
* **한국어 해석:** 다음 지시어들이 실행된 후 `BX` 레지스터의 값은 무엇입니까?
* **풀이:**
  * **상위 바이트:** `64h OR 00h` = `64h`
  * **하위 바이트:** `9Bh (1001 1011b) OR 3Ah (0011 1010b)` = `BBh (1011 1011b)`
* **Answer:** `BX = 64BBh`

---

### 📍 Question 4
* **Problem:**
  ```assembly
  mov  bx, 029D6h
  xor  bx, 8181h
  ```
  What will be the value of `BX` after the following instructions execute?
* **한국어 해석:** 다음 지시어들이 실행된 후 `BX` 레지스터의 값은 무엇입니까?
* **풀이:**
  ```text
    0010 1001 1101 0110b (29D6h)
  XOR 1000 0001 1000 0001b (8181h)
  --------------------------------
    1010 1000 0101 0111b (A857h)
  ```
* **Answer:** `BX = A857h`

---

### 📍 Question 5
* **Problem:**
  ```assembly
  mov  ebx, 0AFAF649Bh
  or   ebx, 3A219604h
  ```
  What will be the value of `EBX` after the following instructions execute?
* **한국어 해석:** 다음 지시어들이 실행된 후 `EBX` 레지스터의 값은 무엇입니까?
* **풀이:**
  * `AFh OR 3Ah` = `BFh`
  * `AFh OR 21h` = `AFh`
  * `64h OR 96h` = `F6h`
  * `9Bh OR 04h` = `9Fh`
* **Answer:** `EBX = BFAF649Fh`

---

### 📍 Question 6
* **Problem:**
  ```assembly
  mov  rbx, 0AFAF649Bh
  xor  rbx, 0FFFFFFFFh
  ```
  What will be the value of `RBX` after the following instructions execute?
* **한국어 해석:** 다음 지시어들이 실행된 후 `RBX` 레지스터의 값은 무엇입니까?
* **풀이:**
  * 하위 32비트인 `0AFAF649Bh`를 `0FFFFFFFFh`로 `XOR` 연산하면 전체 비트가 반전(NOT)되어 `50509B64h`가 됩니다.
  * 상위 32비트는 `0`으로 채워집니다.
* **Answer:** `RBX = 0000000050509B64h`

---

## 🔸 Q7 ~ Q9: Instruction Sequences & Flags

### 📍 Question 7
* **Problem:**
  In the following instruction sequence, show the resulting value of `AL` where indicated, in **binary**:
  ```assembly
  mov  al, 01101111b
  and  al, 00101101b ; a.
  mov  al, 6Dh
  and  al, 4Ah       ; b.
  mov  al, 00001111b
  or   al, 61h       ; c.
  mov  al, 94h
  xor  al, 37h       ; d.
  ```
* **한국어 해석:** 다음 지시어 시퀀스에서 표시된 위치의 `AL` 결과 값을 **이진수(binary)**로 나타내시오.
* **풀이 및 답:**

| Item | Calculation | Explanation | Binary Result |
| :---: | :--- | :--- | :---: |
| **a** | `01101111b AND 00101101b` | 둘 다 1인 비트만 1 | **`00101101b`** |
| **b** | `01101101b (6Dh) AND 01001010b (4Ah)` | 둘 다 1인 비트만 1 | **`00001000b`** |
| **c** | `00001111b OR 01100001b (61h)` | 둘 중 하나라도 1이면 1 | **`01101111b`** |
| **d** | `10010100b (94h) XOR 00110111b (37h)` | 서로 다른 비트만 1 | **`10100111b`** |

---

### 📍 Question 8
* **Problem:**
  In the following instruction sequence, show the resulting value of `AL` where indicated, in **hexadecimal**:
  ```assembly
  mov  al, 7Ah
  not  al           ; a.
  mov  al, 3Dh
  and  al, 74h       ; b.
  mov  al, 9Bh
  or   al, 35h       ; c.
  mov  al, 72h
  xor  al, 0DCh      ; d.
  ```
* **한국어 해석:** 다음 지시어 시퀀스에서 표시된 위치의 `AL` 결과 값을 **16진수(hexadecimal)**로 나타내시오.
* **풀이 및 답:**

| Item | Calculation | Explanation | Hex Result |
| :---: | :--- | :--- | :---: |
| **a** | `NOT 7Ah (0111 1010b)` | 전체 비트 반전 $\rightarrow$ `1000 0101b` | **`85h`** |
| **b** | `3Dh (0011 1101b) AND 74h (0111 0100b)` | 둘 다 1인 비트만 1 $\rightarrow$ `0011 0100b` | **`34h`** |
| **c** | `9Bh (1001 1011b) OR 35h (0011 0101b)` | 둘 중 하나라도 1이면 1 $\rightarrow$ `1011 1111b` | **`BFh`** |
| **d** | `72h (0111 0010b) XOR 0DCh (1101 1100b)` | 서로 다른 비트만 1 $\rightarrow$ `1010 1110b` | **`AEh`** |

---

### 📍 Question 9
* **Problem:**
  In the following instruction sequence, show the values of the Carry, Zero, and Sign flags where indicated:
  ```assembly
  mov  al, 00001111b
  test al, 00000010b ; a. CF=   ZF=   SF=
  mov  al, 00000110b
  cmp  al, 00000101b ; b. CF=   ZF=   SF=
  mov  al, 00000101b
  cmp  al, 00000111b ; c. CF=   ZF=   SF=
  ```
* **한국어 해석:** 다음 지시어 시퀀스에서 표시된 위치의 Carry(CF), Zero(ZF), Sign(SF) 플래그 값을 구하시오.
* **풀이 및 답:**

| Item | Instruction | Explanation | Flag Values |
| :---: | :--- | :--- | :---: |
| **a** | `TEST al, 00000010b` | 결과는 `00000010b` (0이 아님). `TEST`는 항시 `CF=0` 설정 | **`CF = 0, ZF = 0, SF = 0`** |
| **b** | `CMP 6, 5` | $6 - 5 = 1$ (양수, 0이 아님, 빌림 없음) | **`CF = 0, ZF = 0, SF = 0`** |
| **c** | `CMP 5, 7` | $5 - 7 = -2$ (음수, 0이 아님, 빌림 발생) | **`CF = 1, ZF = 0, SF = 1`** |

---

## 🔹 Q10 ~ Q16: Conditional Jumps & Control Flow

### 📍 Question 10
* **Problem:**
  Which conditional jump instruction executes a branch based on the contents of `ECX`?
* **한국어 해석:** `ECX`의 내용에 기반하여 분기(점프)를 실행하는 조건 점프 명령어는 무엇입니까?
* **풀이:** `ECX` 레지스터의 값이 0일 때 점프하는 전용 명령어가 존재합니다.
* **Answer:** **`JECXZ`** *(Jump if ECX is Zero)*

---

### 📍 Question 11
* **Problem:**
  How are `JA` and `JNBE` affected by the Zero and Carry flags?
* **한국어 해석:** `JA` 및 `JNBE` 명령어는 Zero 플래그와 Carry 플래그에 의해 어떻게 영향을 받습니까?
* **풀이:** `JA` (Jump if Above)와 `JNBE` (Jump if Not Below or Equal)는 부호 없는 비교에서 완전히 동일한 조건(초과)을 체크합니다.
* **Answer:** **`CF = 0` 이고 `ZF = 0` 일 때 점프 실행**

---

### 📍 Question 12
* **Problem:**
  ```assembly
  mov  edx, 1
  mov  eax, 7FFFh
  cmp  eax, 8000h
  jl   L1
  mov  edx, 0
  L1:
  ```
  What will be the final value in `EDX` after this code executes?
* **한국어 해석:** 이 코드가 실행된 후 `EDX`의 최종 값은 무엇입니까?
* **풀이:**
  * `JL` (Jump if Less)은 **부호 있는(signed) 정수 비교**를 수행합니다.
  * `EAX` = `7FFFh` (+32,767), 비교 대상 = `8000h` (+32,768).
  * $+32,767 < +32,768$ 조건이 **참**이므로 `L1`으로 점프하여 `mov edx, 0`을 건너뜁니다.
* **Answer:** `EDX = 1`

---

### 📍 Question 13
* **Problem:**
  ```assembly
  mov  edx, 1
  mov  eax, 7FFFh
  cmp  eax, 8000h
  jb   L1
  mov  edx, 0
  L1:
  ```
  What will be the final value in `EDX` after this code executes?
* **한국어 해석:** 이 코드가 실행된 후 `EDX`의 최종 값은 무엇입니까?
* **풀이:**
  * `JB` (Jump if Below)는 **부호 없는(unsigned) 정수 비교**를 수행합니다.
  * $32,767 < 32,768$ 이므로 조건이 **참**이 되어 `L1`으로 점프합니다.
* **Answer:** `EDX = 1`

---

### 📍 Question 14
* **Problem:**
  ```assembly
  mov  edx, 1
  mov  eax, 7FFFh
  cmp  eax, 0FFFF8000h
  jl   L2
  mov  edx, 0
  L2:
  ```
  What will be the final value in `EDX` after this code executes?
* **한국어 해석:** 이 코드가 실행된 후 `EDX`의 최종 값은 무엇입니까?
* **풀이:**
  * `JL`은 **부호 있는 정수 비교**를 수행합니다.
  * `EAX` = `7FFFh` (+32,767), `0FFFF8000h` = 32비트 부호 있는 정수로 **`-32,768`**.
  * $+32,767 < -32,768$ 조건은 **거짓**이므로 점프하지 않고 `mov edx, 0`을 실행합니다.
* **Answer:** `EDX = 0`

---

### 📍 Question 15
* **Problem:**
  ```assembly
  mov  eax, -30
  cmp  eax, -50
  jg   Target
  ```
  (True/False): The following code will jump to the label named `Target`.
* **한국어 해석:** (참/거짓): 다음 코드는 `Target`이라는 라벨로 점프할 것입니까?
* **풀이:**
  * `JG` (Jump if Greater)는 **부호 있는 정수 비교**를 수행합니다.
  * $-30 > -50$ 은 **참**이므로 점프합니다.
* **Answer:** **True (참)**

---

### 📍 Question 16
* **Problem:**
  ```assembly
  mov  eax, -42
  cmp  eax, 26
  ja   Target
  ```
  (True/False): The following code will jump to the label named `Target`.
* **한국어 해석:** (참/거짓): 다음 코드는 `Target`이라는 라벨로 점프할 것입니까?
* **풀이:**
  * `JA` (Jump if Above)는 **부호 없는 정수 비교**를 수행합니다.
  * `-42`를 부호 없는 32비트 수로 표현하면 `0FFFFFFD6h` ($4,294,967,254$)입니다.
  * $4,294,967,254 > 26$ 조건은 **참**이므로 점프합니다.
* **Answer:** **True (참)**

---

## 🔹 Q17 ~ Q19: 64-Bit Bitwise Operations

### 📍 Question 17
* **Problem:**
  ```assembly
  mov  rbx, 0FFFFFFFFFFFFFFFFh
  and  rbx, 80h
  ```
  What will be the value of `RBX` after the instructions execute?
* **한국어 해석:** 다음 지시어들이 실행된 후 `RBX` 레지스터의 값은 무엇입니까?
* **풀이:** `80h`를 64비트로 제로 확장하면 `0000000000000080h`입니다. `0FFFFFFFFFFFFFFFFh`와 AND 연산하면 값이 그대로 유지됩니다.
* **Answer:** `RBX = 0000000000000080h`

---

### 📍 Question 18
* **Problem:**
  ```assembly
  mov  rbx, 0FFFFFFFFFFFFFFFFh
  and  rbx, 808080h
  ```
  What will be the value of `RBX` after the instructions execute?
* **한국어 해석:** 다음 지시어들이 실행된 후 `RBX` 레지스터의 값은 무엇입니까?
* **풀이:** `808080h`를 64비트로 제로 확장하면 `0000000000808080h`입니다.
* **Answer:** `RBX = 0000000000808080h`

---

### 📍 Question 19
* **Problem:**
  ```assembly
  mov  rbx, 0FFFFFFFFFFFFFFFFh
  and  rbx, 80808080h
  ```
  What will be the value of `RBX` after the instructions execute?
* **한국어 해석:** 다음 지시어들이 실행된 후 `RBX` 레지스터의 값은 무엇입니까?
* **풀이:** `80808080h`를 64비트로 제로 확장하면 `0000000080808080h`입니다.
* **Answer:** `RBX = 0000000080808080h`