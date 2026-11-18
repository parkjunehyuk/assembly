# 🛠️ [9장] 9.9.2 Algorithm Workbench (1~12번) 핵심 풀이

Kip Irvine의 *Assembly Language for x86 Processors* 9장 연습문제(Algorithm Workbench)의 **원문 해석, 표준 답안, 그리고 한눈에 읽히는 핵심 해설**입니다.

---

### 📌 1~3번: 주소 지정 방식과 2차원 배열 접근

#### 1. Show an example of a base-index operand in 32-bit mode.
* **번역**: 32비트 모드에서 **베이스-인덱스(Base-Index)** 피연산자의 예를 쓰시오.
* **정답**:
  ```assembly
  [ebx + esi]
  ; 사용 예: mov eax, [ebx + esi]
  ```
* **핵심 정리**:
  * **구조**: `[기준 주소 레지스터 + 인덱스 레지스터]`
  * **원리**: 두 레지스터(`EBX` + `ESI`)의 값을 더해 메모리 주소를 지정합니다.

---

#### 2. Show an example of a base-index-displacement operand in 32-bit mode.
* **번역**: 32비트 모드에서 **베이스-인덱스-변위(Base-Index-Displacement)** 피연산자의 예를 쓰시오.
* **정답**:
  ```assembly
  array[ebx + esi]
  ; 같은 표현: [array + ebx + esi] 또는 [ebx + esi + 4]
  ```
* **핵심 정리**:
  * **구조**: `배열이름[기준 레지스터 + 인덱스 레지스터]`
  * **원리**: 두 레지스터 합(`EBX + ESI`)에 배열 시작 주소(`array`)나 고정 숫자(`+ 4`)를 추가로 더해 2차원 배열의 특정 칸을 찾습니다.

---

#### 3. Suppose a two-dimensional array of doublewords has three logical rows and four logical columns. Write an expression using ESI and EDI that addresses the third column in the second row. (Numbering for rows and columns starts at zero.)
* **번역**: 3행 4열짜리 `DWORD`(4바이트) 2차원 배열이 있다. `ESI`와 `EDI`를 사용하여 2번째 행(Row)의 3번째 열(Column)에 접근하는 표현식을 쓰시오. (행과 열 번호는 0부터 시작)
* **정답**:
  ```assembly
  array[esi + edi*4]
  ```
* **핵심 정리 (계산 과정)**:
  1. **인덱스 확인**: 0부터 시작하므로 2번째 행 = `Row 1`, 3번째 열 = `Col 2`
  2. **행 이동 (`ESI`)**: 한 행당 4바이트 원소가 4개($4 \times 4 = 16\text{B}$)이므로 `ESI = 16`
  3. **열 이동 (`EDI * 4`)**: 원소 1개 크기가 4바이트이므로 `EDI(2) * 4 = 8`
  4. **최종 위치**: `array` 시작점에서 총 `16 + 8 = 24`바이트 떨어진 칸을 가리킵니다.

---

### 📌 4~8번: 문자열 비교(`CMPSW`), 탐색(`SCASW`) 및 라이브러리 함수

#### 4. Write instructions using CMPSW that compare two arrays of 16-bit values named `sourcew` and `targetw`.
* **번역**: `CMPSW`를 사용하여 16비트(`WORD`) 배열 `sourcew`와 `targetw`를 비교하는 명령어들을 쓰시오.
* **정답**:
  ```assembly
  cld                             ; DF = 0 (앞에서 뒤로 주소 증가)
  mov esi, OFFSET sourcew         ; 첫 번째 배열 시작 주소
  mov edi, OFFSET targetw         ; 두 번째 배열 시작 주소
  mov ecx, LENGTHOF sourcew       ; 비교할 WORD 원소 개수
  repe cmpsw                      ; 값이 같은 동안(ZF = 1) 2바이트씩 계속 비교
  ```
* **핵심 정리**:
  * **동작**: `[ESI]`와 `[EDI]`의 2바이트 값을 비교한 뒤 `ESI`, `EDI`를 자동으로 `+2`씩 증가시킵니다.
  * **종료 조건**: 서로 다른 값을 만나거나(`ZF = 0`) 끝까지 다 비교하면(`ECX = 0`) 멈춥니다.

---

#### 5. Write instructions that use SCASW to scan for the 16-bit value `0100h` in an array named `wordArray`, and copy the offset of the matching member into the EAX register.
* **번역**: `SCASW`를 사용하여 `wordArray` 배열에서 16비트 값 `0100h`를 찾고, 일치하는 원소의 주소(오프셋)를 `EAX`에 복사하는 명령어를 쓰시오.
* **정답**:
  ```assembly
  cld                             ; DF = 0 (정방향)
  mov ax, 0100h                   ; 찾을 값 (SCASW는 무조건 AX와 비교)
  mov edi, OFFSET wordArray       ; 검색할 배열 시작 주소
  mov ecx, LENGTHOF wordArray     ; 배열 전체 원소 개수
  repne scasw                     ; 다른 동안(ZF = 0) 계속 탐색
  jnz NotFound                    ; 끝까지 못 찾았으면 분기

  sub edi, 2                      ; 찾은 직후 EDI가 +2 지나가 있으므로 2바이트 보정!
  mov eax, edi                    ; 실제 찾은 원소의 주소를 EAX에 저장
  NotFound:
  ```
* **핵심 정리**:
  * **비교 대상**: `SCASW`는 항상 `AX`와 `[EDI]`를 비교합니다.
  * **주의점 (주소 보정)**: 일치하는 값을 찾은 직후에도 `EDI`는 다음 칸(`+2`바이트 뒤)을 가리키고 있으므로 반드시 `sub edi, 2`로 한 칸 되돌려야 합니다.

---

#### 6. Write a sequence of instructions that use the `Str_compare` procedure to determine the larger of two input strings and write it to the console window.
* **번역**: `Str_compare` 프로시저를 사용하여 두 문자열 중 사전순으로 더 큰(뒤에 나오는) 문자열을 찾아 콘솔 창에 출력하는 명령어들을 쓰시오.
* **정답**:
  ```assembly
  INVOKE Str_compare, ADDR string1, ADDR string2
  ja  String1Larger               ; string1 > string2 이면 점프

  mov edx, OFFSET string2         ; string2가 더 크거나 같음
  jmp Display

  String1Larger:
  mov edx, OFFSET string1         ; string1이 더 큼

  Display:
  call WriteString                ; 큰 문자열 출력
  call Crlf
  ```
* **핵심 정리**:
  * **비교 방식**: 아스키(ASCII) 코드는 부호 없는 양수(Unsigned)이므로 크기 비교 시 `JA`(더 크면 점프) 또는 `JB`(더 작으면 점프)를 사용합니다.

---

#### 7. Show how to call the `Str_trim` procedure and remove all trailing `"@"` characters from a string.
* **번역**: `Str_trim` 프로시저를 호출하여 문자열 맨 뒤에 붙은 모든 `"@"` 문자를 제거하는 방법을 보이시오.
* **정답**:
  ```assembly
  .data
  myString BYTE "HelloWorld@@@@", 0

  .code
  INVOKE Str_trim, ADDR myString, '@'
  ```
* **핵심 정리**:
  * **인자 구성**: `(문자열 주소, 제거할 문자)` 순서로 전달합니다.
  * **동작 원리**: 문자열 맨 끝에서부터 역방향(`STD`)으로 검사해 `'@'`가 아닌 문자가 나올 때까지 지운 뒤 끝 표시(`0`)를 기록합니다.

---

#### 8. Show how to modify the `Str_ucase` procedure from the Irvine32 library so it changes all characters to lower case.
* **번역**: Irvine32 라이브러리의 `Str_ucase`(대문자 변환) 프로시저를 수정하여 모든 영문자를 소문자(Lower case)로 바꾸도록 작성하시오.
* **정답**:
  ```assembly
  Str_lcase PROC USES eax esi,
      pString:PTR BYTE

      mov esi, pString            ; 문자열 시작 주소
  L1:
      mov al, [esi]               ; 현재 문자 읽기
      cmp al, 0                   ; 문자열 끝(0)이면 종료
      je  L3

      cmp al, 'A'                 ; 'A'~'Z' 범위가 아니면 건너뜀
      jb  L2
      cmp al, 'Z'
      ja  L2

      or  BYTE PTR [esi], 00100000b   ; 5번 비트(20h = 32)를 1로 켜서 소문자로 변환
  L2:
      inc esi                     ; 다음 문자로 이동
      jmp L1
  L3:
      ret
  Str_lcase ENDP
  ```
* **핵심 정리**:
  * **아스키 비트 차이**: 대문자와 소문자는 5번 비트(`32 = 20h = 00100000b`) 하나만 다릅니다.
  * **변환 공식**: 대문자 $\rightarrow$ 소문자는 `OR 00100000b` (5번 비트 켜기), 소문자 $\rightarrow$ 대문자는 `AND 11011111b` (5번 비트 끄기)를 사용합니다.

---

### 📌 9~12번: 64비트 모드 확장 및 배열 접근

#### 9. Create a 64-bit version of the `Str_trim` procedure.
* **번역**: `Str_trim` 프로시저의 **64비트 버전**을 작성하시오.
* **정답**:
  ```assembly
  ; 64비트 인자 규약: 1번째 인자 = RCX(문자열 주소), 2번째 인자 = RDX(하위 DL = 제거할 문자)
  Str_trim64 PROC USES rax rcx rdi
      mov  rdi, rcx               ; RDI = 문자열 시작 주소
      call Str_length             ; RAX = 문자열 길이 반환
      cmp  rax, 0                 ; 빈 문자열이면 종료
      je   L3

      mov  rcx, rax               ; RCX = 반복 횟수(문자열 길이)
      dec  rax                    ; RAX = 마지막 문자 인덱스(길이 - 1)
      add  rdi, rax               ; RDI = 문자열 맨 끝 문자를 가리킴

  L1:
      mov  al, [rdi]              ; 뒤에서부터 한 글자씩 확인
      cmp  al, dl                 ; 제거할 문자(DL)와 비교
      jne  L2                     ; 다른 문자를 만나면 반복 종료
      dec  rdi                    ; 앞 칸으로 이동
      loop L1

  L2:
      mov  BYTE PTR [rdi + 1], 0  ; 정상 문자 바로 다음 칸에 문자열 끝(0) 기록
  L3:
      ret
  Str_trim64 ENDP
  ```
* **핵심 정리 (32비트 $\rightarrow$ 64비트 변경점)**:
  1. **레지스터 확장**: `EAX`, `ECX`, `EDI` $\rightarrow$ `RAX`, `RCX`, `RDI` (64비트 주소 사용)
  2. **인자 전달 방식**: 스택 대신 레지스터로 전달 (`1번 인자 = RCX`, `2번 인자 = RDX`)

---

#### 10. Show an example of a base-index operand in 64-bit mode.
* **번역**: 64비트 모드에서 **베이스-인덱스(Base-Index)** 피연산자의 예를 쓰시오.
* **정답**:
  ```assembly
  [rbx + rsi]
  ; 사용 예: mov rax, [rbx + rsi]
  ```
* **핵심 정리**:
  * 1번 문제와 원리는 동일하며, 64비트 메모리 주소를 다루기 위해 64비트 레지스터(`RBX`, `RSI`)를 사용합니다.

---

#### 11. Assuming that `EBX` contains a row index into a two-dimensional array of 32-bit integers named `myArray` and `EDI` contains the index of a column, write a single statement that moves the content of the given array element into the `EAX` register.
* **번역**: `EBX`에 32비트 정수(`DWORD`) 2차원 배열 `myArray`의 행 오프셋이 들어있고, `EDI`에 열 인덱스가 들어있을 때, 해당 원소의 값을 `EAX`로 가져오는 한 줄짜리 명령문을 쓰시오.
* **정답**:
  ```assembly
  mov eax, myArray[ebx + edi*4]
  ```
* **핵심 정리**:
  * **자료형 크기**: 32비트 정수(`DWORD`) = **4바이트**이므로 열 인덱스 `EDI`에 `*4`를 곱합니다.

---

#### 12. Assuming that `RBX` contains a row index into a two-dimensional array of 64-bit integers named `myArray` and `RDI` contains the index of a column, write a single statement that moves the content of the given array element into the `RAX` register.
* **번역**: `RBX`에 64비트 정수(`QWORD`) 2차원 배열 `myArray`의 행 오프셋이 들어있고, `RDI`에 열 인덱스가 들어있을 때, 해당 원소의 값을 `RAX`로 가져오는 한 줄짜리 명령문을 쓰시오.
* **정답**:
  ```assembly
  mov rax, myArray[rbx + rdi*8]
  ```
* **핵심 정리**:
  * **자료형 크기**: 64비트 정수(`QWORD`) = **8바이트**이므로 열 인덱스 `RDI`에 `*8`을 곱하고, 64비트 레지스터(`RBX`, `RDI`, `RAX`)를 사용합니다.