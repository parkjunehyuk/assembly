# 📘 Assembly Language for x86 Processors (7th Edition)
> **Chapter 6: Conditional Processing — Review Questions & Exercises Solution**

![Assembly Language](https://img.shields.io/badge/Language-x86_Assembly-blue.svg)
![Book](https://img.shields.io/badge/Author-Kip_Irvine-green.svg)
![License](https://img.shields.io/badge/License-MIT-orange.svg)

This repository contains full problem statements, Korean translations, step-by-step explanations, and answers for **Chapter 6 (Section 6.10 Review Questions and Exercises)** of Kip Irvine's *Assembly Language for x86 Processors (7th Edition)*.

---

## 📌 Table of Contents
- [Q1 ~ Q6: Bitwise Operations](#-q1--q6-bitwise-operations)
- [Q7 ~ Q9: Instruction Sequences & Flags](#-q7--q9-instruction-sequences--flags)
- [Q10 ~ Q16: Conditional Jumps & Control Flow](#-q10--q16-conditional-jumps--control-flow)
- [Q17 ~ Q19: 64-Bit Bitwise Operations](#-q17--q19-64-bit-bitwise-operations)

---

## 🔹 Q1 ~ Q6: Bitwise Operations

<details open>
<summary><b>Q1. Value of BX after <code>AND</code> operation</b></summary>

* **Problem:**
  ```assembly
  mov  bx, 0FFFFh
  and  bx, 6Bh
  ```
  What will be the value of `BX` after the instructions execute?
* **해석:** 다음 지시어들이 실행된 후 `BX` 레지스터의 값은 무엇입니까?
* **풀이:**
  * `0FFFFh`는 모든 비트가 `1`인 상태입니다.
  * 어떤 값과 `1`을 `AND` 연산하면 자기 자신의 값이 그대로 유지됩니다 ($X \land 1 = X$).
* **Answer:** `BX = 006Bh`
</details>

<details>
<summary><b>Q2. Value of BX after <code>AND</code> operation</b></summary>

* **Problem:**
  ```assembly
  mov  bx, 91BAh
  and  bx, 92h
  ```
  What will be the value of `BX` after the instructions execute?
* **해석:** 다음 지시어들이 실행된 후 `BX` 레지스터의 값은 무엇입니까?
* **풀이:**
  * 상위 바이트: `91h AND 00h` = `00h`
  * 하위 바이트: `BAh (1011 1010b) AND 92h (1001 0100b)` = `92h (1001 0010b)`
* **Answer:** `BX = 0092h`
</details>

<details>
<summary><b>Q3. Value of BX after <code>OR</code> operation</b></summary>

* **Problem:**
  ```assembly
  mov  bx, 0649Bh
  or   bx, 3Ah
  ```
  What will be the value of `BX` after the instructions execute?
* **해석:** 다음 지시어들이 실행된 후 `BX` 레지스터의 값은 무엇입니까?
* **풀이:**
  * 상위 바이트: `64h OR 00h` = `64h`
  * 하위 바이트: `9Bh (1001 1011b) OR 3Ah (0011 1010b)` = `BBh (1011 1011b)`
* **Answer:** `BX = 64BBh`
</details>

<details>
<summary><b>Q4. Value of BX after <code>XOR</code> operation</b></summary>

* **Problem:**
  ```assembly
  mov  bx, 029D6h
  xor  bx, 8181h
  ```
  What will be the value of `BX` after the instructions execute?
* **해석:** 다음 지시어들이 실행된 후 `BX` 레지스터의 값은 무엇입니까?
* **풀이:**
  ```text
    0010 1001 1101 0110b (29D6h)
  XOR 1000 0001 1000 0001b (8181h)
  --------------------------------
      1010 1000 0101 0111b (A857h)
  ```
* **Answer:** `BX = A857h`
</details>

<details>
<summary><b>Q5. Value of EBX after <code>OR</code> operation</b></summary>

* **Problem:**
  ```assembly
  mov  ebx, 0AFAF649Bh
  or   ebx, 3A219604h
  ```
  What will be the value of `EBX` after the instructions execute?
* **해석:** 다음 지시어들이 실행된 후 `EBX` 레지스터의 값은 무엇입니까?
* **풀이:**
  * `AFh OR 3Ah` = `BFh`
  * `AFh OR 21h` = `AFh`
  * `64h OR 96h` = `F6h`
  * `9Bh OR 04h` = `9Fh`
* **Answer:** `EBX = BFAF649Fh`
</details>

<details>
<summary><b>Q6. Value of RBX after 64-bit <code>XOR</code> operation</b></summary>

* **Problem:**
  ```assembly
  mov  rbx, 0AFAF649Bh
  xor  rbx, 0FFFFFFFFh
  ```
  What will be the value of `RBX` after the instructions execute?
* **해석:** 다음 지시어들이 실행된 후 `RBX` 레지스터의 값은 무엇입니까?
* **풀이:**
  * 하위 32비트인 `0AFAF649Bh`를 `0FFFFFFFFh`로 XOR 연산하면 전체 비트가 반전(NOT)되어 `50509B64h`가 됩니다.
  * 상위 32비트는 `0`으로 채워집니다.
* **Answer:** `RBX = 0000000050509B64h`
</details>

---

## 🔸 Q7 ~ Q9: Instruction Sequences & Flags

### 📍 Q7. Expressing AL values in Binary

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

| Label | Calculation | Binary Result |
| :---: | :--- | :---: |
| **a** | `01101111b AND 00101101b` | `00101101b` |
| **b** | `01101101b (6Dh) AND 01001010b (4Ah)` | `00001000b` |
| **c** | `00001111b OR 01100001b (61h)` | `01101111b` |
| **d** | `10010100b (94h) XOR 00110111b (37h)` | `10100111b` |

---

### 📍 Q8. Expressing AL values in Hexadecimal

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

| Label | Calculation | Hex Result |
| :---: | :--- | :---: |
| **a** | `NOT 7Ah (0111 1010b)` | `85h` |
| **b** | `3Dh (0011 1101b) AND 74h (0111 0100b)` | `34h` |
| **c** | `9Bh (1001 1011b) OR 35h (0011 0101b)` | `BFh` |
| **d** | `72h (0111 0010b) XOR 0DCh (1101 1100b)` | `AEh` |

---

### 📍 Q9. Flag States (CF, ZF, SF)

In the following instruction sequence, show the values of the Carry, Zero, and Sign flags where indicated:

```assembly
mov  al, 00001111b
test al, 00000010b ; a.
mov  al, 00000110b
cmp  al, 00000101b ; b.
mov  al, 00000101b
cmp  al, 00000111b ; c.
```

| Label | Instruction | Explanation | Flag Values |
| :---: | :--- | :--- | :---: |
| **a** | `TEST al, 00000010b` | Result is `00000010b` (Non-zero). `TEST` clears CF. | **`CF = 0, ZF = 0, SF = 0`** |
| **b** | `CMP 6, 5` | $6 - 5 = 1$ (Positive, Non-zero, No borrow). | **`CF = 0, ZF = 0, SF = 0`** |
| **c** | `CMP 5, 7` | $5 - 7 = -2$ (Negative, Borrow occurred). | **`CF = 1, ZF = 0, SF = 1`** |

---

## 🔹 Q10 ~ Q16: Conditional Jumps & Control Flow

#### 10. Conditional jump based on ECX
* **Question:** Which conditional jump instruction executes a branch based on the contents of `ECX`?
* **Answer:** **`JECXZ`** *(Jump if ECX is Zero)*

#### 11. Flags affecting JA and JNBE
* **Question:** How are `JA` and `JNBE` affected by the Zero and Carry flags?
* **Answer:** Both instructions jump when **`CF = 0` and `ZF = 0`**.

---

<details open>
<summary><b>12. Final value of EDX (Signed comparison)</b></summary>

```assembly
mov  edx, 1
mov  eax, 7FFFh
cmp  eax, 8000h
jl   L1
mov  edx, 0
L1:
```
* **Explanation:** `JL` (Jump if Less) uses signed comparison. Here $+32,767 < +32,768$, which is **true**. It jumps to `L1`, skipping `mov edx, 0`.
* **Answer:** `EDX = 1`
</details>

<details>
<summary><b>13. Final value of EDX (Unsigned comparison)</b></summary>

```assembly
mov  edx, 1
mov  eax, 7FFFh
cmp  eax, 8000h
jb   L1
mov  edx, 0
L1:
```
* **Explanation:** `JB` (Jump if Below) uses unsigned comparison. $32,767 < 32,768$ is **true**. It jumps to `L1`.
* **Answer:** `EDX = 1`
</details>

<details>
<summary><b>14. Final value of EDX (Signed comparison with negative value)</b></summary>

```assembly
mov  edx, 1
mov  eax, 7FFFh
cmp  eax, 0FFFF8000h
jl   L2
mov  edx, 0
L2:
```
* **Explanation:** `0FFFF8000h` represents `-32,768` in 32-bit signed integer. Since $+32,767 < -32,768$ is **false**, it does not jump and executes `mov edx, 0`.
* **Answer:** `EDX = 0`
</details>

<details>
<summary><b>15. Conditional Jump True/False check</b></summary>

```assembly
mov  eax, -30
cmp  eax, -50
jg   Target
```
* **Question:** Will the code jump to `Target`?
* **Explanation:** `JG` uses signed comparison. Since $-30 > -50$ is **true**, it will jump.
* **Answer:** **True**
</details>

<details>
<summary><b>16. Conditional Jump True/False check</b></summary>

```assembly
mov  eax, -42
cmp  eax, 26
ja   Target
```
* **Question:** Will the code jump to `Target`?
* **Explanation:** `JA` uses unsigned comparison. `-42` in unsigned 32-bit is `0FFFFFFD6h` ($4,294,967,254$). Since $4,294,967,254 > 26$ is **true**, it will jump.
* **Answer:** **True**
</details>

---

## 🔸 Q17 ~ Q19: 64-Bit Bitwise Operations

| No. | Code | Step-by-Step Logic | Final Value of RBX |
| :---: | :--- | :--- | :---: |
| **17** | `mov rbx, 0FFFFFFFFFFFFFFFFh`<br>`and rbx, 80h` | `80h` padded to 64-bit is `0000000000000080h`. | **`0000000000000080h`** |
| **18** | `mov rbx, 0FFFFFFFFFFFFFFFFh`<br>`and rbx, 808080h` | `808080h` padded to 64-bit is `0000000000808080h`. | **`0000000000808080h`** |
| **19** | `mov rbx, 0FFFFFFFFFFFFFFFFh`<br>`and rbx, 80808080h` | `80808080h` padded to 64-bit is `0000000080808080h`. | **`0000000080808080h`** |

---

> 💡 **Tip:** Feel free to open an issue or pull request if you notice any typos or have suggestions for clearer explanations!