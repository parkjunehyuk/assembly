# 📚 [9장] 문자열과 배열 (Strings and Arrays) 강의 총정리

Kip Irvine의 *Assembly Language for x86 Processors* **9장(Strings and Arrays)**의 핵심 개념, 스트링 기본 명령어 5대장, 반복 접두사(Repeat Prefixes), Irvine32 문자열 라이브러리, 그리고 2차원 배열 주소 계산법을 정리한 복습 문서입니다.

---

## 1. 스트링 기본 명령어 (String Primitive Instructions)

스트링 기본 명령어는 피연산자를 코드에 명시하지 않고 **약속된 레지스터(`ESI`, `EDI`, `EAX/AX/AL`)**를 자동으로 사용합니다.

| 명령어 | 이름 | 동작 원리 (`DF = 0` 기준) |
| :--- | :--- | :--- |
| **`MOVSB / MOVSW / MOVSD`** | Move String | `[ESI]`의 데이터를 `[EDI]`로 복사하고 `ESI`, `EDI`를 자동 증가 (`+1 / +2 / +4`) |
| **`CMPSB / CMPSW / CMPSD`** | Compare String | `[ESI]`와 `[EDI]`의 값을 뺄셈(`ESI - EDI`)하여 플래그(`ZF`, `CF`) 설정 후 포인터 증가 |
| **`SCASB / SCASW / SCASD`** | Scan String | `AL / AX / EAX`와 `[EDI]`의 값을 비교(`A - [EDI]`)하여 같은 값을 탐색 후 `EDI` 증가 |
| **`STOSB / STOSW / STOSD`** | Store String | `AL / AX / EAX`의 값을 `[EDI]` 메모리에 저장하고 `EDI` 자동 증가 |
| **`LODSB / LODSW / LODSD`** | Load String | `[ESI]` 메모리의 값을 `AL / AX / EAX`로 읽어오고 `ESI` 자동 증가 |

### 💡 방향 플래그 (Direction Flag, DF)
* **`CLD`** (Clear Direction Flag, `DF = 0`): 주소가 **증가**하는 방향 (앞 $\rightarrow$ 뒤, 정방향)
* **`STD`** (Set Direction Flag, `DF = 1`): 주소가 **감소**하는 방향 (뒤 $\rightarrow$ 앞, 역방향)

---

## 2. 반복 접두사 (Repeat Prefixes)

스트링 명령어 앞에 붙어 **`ECX` 레지스터가 0이 될 때까지** 명령어를 자동으로 반복 실행합니다.

* **`REP`**: `ECX > 0`인 동안 무조건 반복 (주로 `MOVS`, `STOS`와 함께 사용)
* **`REPZ` / `REPE`**: `ECX > 0` **이고** `ZF = 1`(두 값이 같음)인 동안 반복 (두 문자열이 어디까지 같은지 비교할 때 `CMPS`와 함께 사용)
* **`REPNZ` / `REPNE`**: `ECX > 0` **이고** `ZF = 0`(두 값이 다름)인 동안 반복 (특정 문자를 찾을 때까지 `SCAS`와 함께 사용)

---

## 3. Irvine32 문자열 처리 프로시저

교재 9.3절에서 다루는 Irvine32 내장 문자열 유틸리티 함수들입니다.

| 프로시저 이름 | 주요 기능 | 전달 인자 (`INVOKE` 사용 시) |
| :--- | :--- | :--- |
| **`Str_compare`** | 두 문자열을 사전순으로 비교 (`ZF`, `CF` 플래그 변경) | `ADDR string1, ADDR string2` |
| **`Str_length`** | Null(`0`)을 제외한 문자열의 길이를 구해 **`EAX`**에 반환 | `ADDR string` |
| **`Str_copy`** | 원본 문자열을 대상 버퍼로 복사 | `ADDR source, ADDR target` |
| **`Str_trim`** | 문자열 끝(오른쪽)에 있는 특정 문자(공백 등)를 제거 | `ADDR string, charToTrim` |
| **`Str_ucase`** | 문자열의 모든 소문자를 대문자로 변환 | `ADDR string` |

---

## 4. 2차원 배열 주소 계산 (Base-Index Addressing)

9.4절에서 다루는 행 우선(Row-Major) 방식의 2차원 배열 접근 공식입니다.

* **주소 계산 공식:**
  $$\text{주소} = \text{배열 시작 주소} + (\text{행 인덱스} \times \text{한 행의 바이트 크기}) + (\text{열 인덱스} \times \text{원소 자료형 크기})$$
* **어셈블리 구현 예시 (`table`이 `DWORD` 2차원 배열일 때):**
  ```assembly
  ; table[row][col] 접근하기 (1행당 5개의 DWORD = 20바이트 가정)
  mov ebx, OFFSET table       ; 배열 시작 주소
  mov eax, row                ; 행 인덱스
  mov ecx, 20                 ; 한 행의 바이트 크기 (5 * 4)
  mul ecx                     ; EAX = row * 20
  add ebx, eax                ; EBX = 해당 행의 시작 주소
  mov esi, col                ; 열 인덱스
  mov eax, [ebx + esi*4]      ; Base-Index-Scale 주소 지정으로 값 읽기
  ```

---

## 5. 9장 핵심 종합 실습 예제 (`ch9_practice.asm`)

`MOVSD`, `CMPSB`, `Str_length`, `Str_ucase`, `Str_trim`의 동작을 한 번에 확인할 수 있는 코드입니다.

```assembly
INCLUDE Irvine32.inc

SetConsoleOutputCP PROTO, wCodePageID:DWORD

.data
    ; 1. MOVSD (배열 복사) 실습용
    sourceArr   DWORD 10, 20, 30, 40, 50
    destArr     DWORD 5 DUP(0)

    ; 2. CMPSB (문자열 일치 비교) 실습용
    codeA       BYTE  "AB123", 0
    codeB       BYTE  "AB129", 0

    ; 3. Irvine32 문자열 함수 실습용
    rawText     BYTE  "hello assembly###", 0
    
    msgCopy     BYTE  "[1] MOVSD 복사 완료: destArr[2] = ", 0
    msgDiff     BYTE  "[2] CMPSB 비교 결과: 두 코드가 다릅니다!", 0
    msgLen      BYTE  "[3] Str_trim 후 길이: ", 0
    msgUcase    BYTE  "[4] Str_ucase 결과  : ", 0

.code
main PROC
    INVOKE SetConsoleOutputCP, 65001

    ; -------------------------------------------------------------------------
    ; 1. REP MOVSD : sourceArr(5개 DWORD) -> destArr로 통째로 복사
    ; -------------------------------------------------------------------------
    cld                                  ; 정방향(주소 증가) 설정
    mov esi, OFFSET sourceArr            ; 원본 주소는 무조건 ESI!
    mov edi, OFFSET destArr              ; 목적지 주소는 무조건 EDI!
    mov ecx, LENGTHOF sourceArr          ; 반복 횟수 = 5
    rep movsd                            ; [ESI] -> [EDI] 4바이트씩 5번 복사

    mov edx, OFFSET msgCopy
    call WriteString
    mov eax, destArr[2 * TYPE DWORD]     ; 3번째 원소(30) 출력 확인
    call WriteDec
    call Crlf

    ; -------------------------------------------------------------------------
    ; 2. REPE CMPSB : 두 문자열이 어디까지 같은지 비교
    ; -------------------------------------------------------------------------
    cld
    mov esi, OFFSET codeA                ; 첫 번째 문자열 주소
    mov edi, OFFSET codeB                ; 두 번째 문자열 주소
    mov ecx, LENGTHOF codeA              ; 최대 비교 길이
    repe cmpsb                           ; 같은 동안(ZF=1) 계속 비교하며 전진
                                         ; '3'과 '9'를 만나는 순간 ZF=0이 되며 중단!
    jnz L_NotEqual
    jmp L_Section3

L_NotEqual:
    mov edx, OFFSET msgDiff
    call WriteString
    call Crlf

    ; -------------------------------------------------------------------------
    ; 3. Irvine32 라이브러리: Str_trim, Str_length, Str_ucase
    ; -------------------------------------------------------------------------
L_Section3:
    ; 문자열 끝의 '#' 문자 모두 제거 -> "hello assembly"
    INVOKE Str_trim, ADDR rawText, '#'

    ; 제거 후 문자열 길이 구하기 (결과는 EAX에 반환됨)
    INVOKE Str_length, ADDR rawText
    mov edx, OFFSET msgLen
    call WriteString
    call WriteDec                        ; 길이 14 출력
    call Crlf

    ; 소문자를 모두 대문자로 변환 -> "HELLO ASSEMBLY"
    INVOKE Str_ucase, ADDR rawText
    mov edx, OFFSET msgUcase
    call WriteString
    mov edx, OFFSET rawText
    call WriteString
    call Crlf

    exit
main ENDP
END main
```