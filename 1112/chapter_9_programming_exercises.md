# 💻 [9장] 9.10 Programming Exercises (1~14번) 완벽 풀이 가이드

Kip Irvine의 *Assembly Language for x86 Processors* 9장 프로그래밍 실습 문제(1~14번)의 **문제 원문, 한글 번역, 상세 주석 어셈블리 코드(32비트 Irvine32), 그리고 실행 순서 및 핵심 포인트 해설**입니다.

---

## 📌 1~6번: 문자열 처리 및 빈도수 테이블

### 1. Improved `Str_copy` Procedure (`Str_copyN`)

* **문제 원문**: The `Str_copy` procedure shown in this chapter does not limit the number of characters to be copied. Create a new version (named `Str_copyN`) that receives an additional input parameter indicating the maximum number of characters to be copied.
* **번역**: 이 장에 소개된 `Str_copy` 프로시저는 복사할 문자 수를 제한하지 않는다. 복사할 최대 문자 수를 나타내는 추가 입력 매개변수를 받는 새 버전(`Str_copyN`)을 작성하시오.

```assembly
Str_copyN PROC USES eax ecx esi edi,
    source:PTR BYTE,            ; 원본 문자열 시작 주소
    target:PTR BYTE,            ; 대상 버퍼 시작 주소
    maxChars:DWORD              ; 복사할 최대 문자 개수

    mov ecx, maxChars           ; ECX = 남은 복사 가능 횟수 설정
    mov esi, source             ; ESI = 원본 읽기 포인터
    mov edi, target             ; EDI = 대상 쓰기 포인터
    cld                         ; DF = 0 (앞에서 뒤로 순방향 이동)

L1:
    cmp ecx, 0                  ; 최대 허용 개수만큼 다 복사했는가?
    je  L2                      ; 도달했으면 루프 종료
    lodsb                       ; AL = [ESI] 읽고, ESI 자동 +1 증가
    cmp al, 0                   ; 원본 문자열의 끝(Null)을 만났는가?
    je  L2                      ; 끝났으면 루프 종료
    stosb                       ; [EDI] = AL 쓰고, EDI 자동 +1 증가
    dec ecx                     ; 남은 허용 횟수 1 감소
    jmp L1

L2:
    mov BYTE PTR [edi], 0       ; ★ 중요: 최대 개수에 걸려 중단되더라도 끝에 반드시 Null(0) 기록
    ret
Str_copyN ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **초기화**: `ESI`에 원본 주소, `EDI`에 대상 주소를 넣고 `ECX`에 `maxChars`를 세팅합니다.
  2. **복사 루프 (`L1`)**: `ECX`가 0이 되거나 원본에서 `0(Null)`을 만날 때까지 `LODSB`와 `STOSB`로 한 바이트씩 복사합니다.
  3. **Null 종료 보장 (`L2`)**: **`maxChars` 제한으로 인해 중간에 복사가 멈춘 경우에도 대상 문자열이 정상 종료되도록 `[EDI]`에 `0`을 강제로 기록**합니다.

---

### 2. `Str_concat` Procedure

* **문제 원문**: Write a procedure named `Str_concat` that concatenates a source string to the end of a target string. Sufficient space must exist in the target string to accommodate the new characters. Pass pointers to the source and target strings.
* **번역**: 원본(`source`) 문자열을 대상(`target`) 문자열의 끝에 이어 붙이는 `Str_concat` 프로시저를 작성하시오. 대상 문자열에는 새 문자들을 수용할 충분한 공간이 있어야 하며, 원본과 대상 문자열의 포인터를 전달하시오.

```assembly
Str_concat PROC USES eax esi edi,
    target:PTR BYTE,            ; 뒤에 이어 붙일 대상 문자열 주소
    source:PTR BYTE             ; 가져올 원본 문자열 주소

    INVOKE Str_length, target   ; EAX = target 문자열의 현재 길이 (Null 제외)
    mov edi, target             ; EDI = target의 시작 주소
    add edi, eax                ; ★ EDI = target의 맨 끝 Null('\0') 위치로 이동
    mov esi, source             ; ESI = source의 시작 주소
    cld                         ; DF = 0 (순방향)

L1:
    lodsb                       ; source에서 한 글자 읽기 (AL = [ESI], ESI++)
    stosb                       ; target 끝에 쓰기 ([EDI] = AL, EDI++)
    cmp al, 0                   ; 방금 복사한 글자가 Null(0)이었는가?
    jne L1                      ; Null이 아니면 계속 다음 글자 복사 (Null까지 복사 후 종료)
    ret
Str_concat ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **끝 위치 탐색**: `Str_length`로 `target`의 길이를 구한 뒤 `target + 길이`를 계산하여 **`EDI`가 `target`의 기존 Null 문자(`0`)를 정확히 가리키게** 만듭니다.
  2. **덮어쓰기 복사**: `source`의 첫 글자가 `target`의 기존 Null 자리를 덮어쓰면서 복사가 시작되며, `source`의 마지막 `0(Null)`까지 복사한 뒤 루프를 마칩니다.

---

### 3. `Str_remove` Procedure

* **문제 원문**: Write a procedure named `Str_remove` that removes `n` characters from a string. Pass a pointer to the position in the string where the characters are to be removed. Pass an integer specifying the number of characters to remove.
* **번역**: 문자열에서 `n`개의 문자를 제거하는 `Str_remove` 프로시저를 작성하시오. 문자를 제거할 위치의 포인터와 제거할 문자 수를 나타내는 정수를 전달하시오.

```assembly
Str_remove PROC USES eax ecx esi edi,
    pStart:PTR BYTE,            ; 삭제를 시작할 위치의 포인터
    nChars:DWORD                ; 삭제할 문자 개수 (n)

    INVOKE Str_length, pStart   ; EAX = 삭제 시작점부터 문자열 끝까지의 남은 길이
    cmp eax, nChars             ; 남은 길이 <= 삭제할 개수 인가?
    jbe L_Truncate              ; 그렇다면 뒤에서 당겨올 글자가 없으므로 바로 문자열 종료 처리

    mov ecx, eax                ; ECX = 남은 길이
    sub ecx, nChars             ; ECX = 앞으로 당겨올 뒤쪽 문자 개수
    inc ecx                     ; ★ 중요: 맨 끝의 Null(0) 문자까지 함께 당기기 위해 +1

    mov edi, pStart             ; EDI = 덮어쓸 목적지 (삭제 시작 위치)
    mov esi, pStart             ; ESI = 당겨올 원본 위치 계산 시작
    add esi, nChars             ; ESI = pStart + nChars (삭제할 구간 바로 다음 글자)
    cld                         ; DF = 0 (앞에서 뒤로 순방향 복사)
    rep movsb                   ; 뒤쪽 문자열(Null 포함)을 앞쪽으로 당겨 덮어씀
    jmp L_Done

L_Truncate:
    mov edi, pStart
    mov BYTE PTR [edi], 0       ; 삭제 시작 위치에 바로 Null(0) 기록
L_Done:
    ret
Str_remove ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **범위 체크**: 삭제할 개수(`nChars`)가 남은 전체 길이보다 크거나 같으면(`jbe`), `pStart` 위치에 바로 `0`을 써서 뒷부분을 통째로 날립니다.
  2. **앞으로 당기기 (`REP MOVSB`)**: `ESI`를 `pStart + nChars`에 두고 `EDI`를 `pStart`에 둔 뒤 복사합니다. 이때 **마지막 Null 문자(`0`)까지 같이 이동하도록 복사 횟수(`ECX`)에 `+1`을 해주는 것**이 핵심입니다.

---

### 4. `Str_find` Procedure

* **문제 원문**: Write a procedure named `Str_find` that searches for the first matching occurrence of a source string inside a target string and returns the matching position. If a match is found, the procedure sets the Zero flag and `EAX` points to the matching position in the target string. Otherwise, the Zero flag is clear and `EAX` is undefined.
* **번역**: 대상(`target`) 문자열 내부에서 원본(`source`) 문자열이 처음 일치하는 위치를 찾아 반환하는 `Str_find` 프로시저를 작성하시오. 일치하는 부분을 찾으면 Zero 플래그를 세트(`ZF=1`)하고 `EAX`가 대상 문자열 내 일치 위치를 가리키게 하며, 찾지 못하면 Zero 플래그를 클리어(`ZF=0`)하시오.

```assembly
Str_find PROC USES ebx ecx edx esi edi,
    pSource:PTR BYTE,           ; 찾을 부분 문자열 (예: "ABC")
    pTarget:PTR BYTE            ; 전체 대상 문자열 (예: "123ABC342432")

    INVOKE Str_length, pSource
    mov ebx, eax                ; EBX = source 길이
    INVOKE Str_length, pTarget
    cmp eax, ebx                ; target 길이 < source 길이 인가?
    jb  NotFound                ; 대상이 더 짧으면 절대 찾을 수 없음

    mov ecx, eax
    sub ecx, ebx
    inc ecx                     ; ★ 최대 비교 시도 횟수 = (target길이 - source길이 + 1)
    mov edi, pTarget            ; EDI = target에서 현재 비교를 시작할 기준 위치

L_Outer:
    push ecx                    ; 외부 루프 카운터 백업
    push edi                    ; 현재 비교 시작 위치 백업
    mov esi, pSource            ; ESI = source의 첫 글자
    mov ecx, ebx                ; ECX = source 길이만큼 글자 대조
    cld
    repe cmpsb                  ; 두 문자열이 같은 동안(ZF=1) 계속 비교
    pop edi                     ; 비교 시작 위치 복원
    pop ecx                     ; 외부 루프 카운터 복원
    je  Found                   ; ★ 모든 글자가 일치했다면(ZF=1) 탐색 성공!

    inc edi                     ; 불일치 시 target의 다음 칸으로 한 칸 전진
    loop L_Outer

NotFound:
    or  eax, 1                  ; ★ 강제로 ZF = 0 설정 (0이 아닌 값과 OR 연산 -> 탐색 실패 표시)
    ret

Found:
    mov eax, edi                ; EAX = 일치하는 시작 위치의 메모리 주소
    cmp eax, eax                ; ★ 강제로 ZF = 1 설정 (자기 자신과 비교 -> 탐색 성공 표시)
    ret
Str_find ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **이중 루프 구조**: 바깥 루프(`L_Outer`)는 `target`의 시작점을 1바이트씩 이동시키고, 안쪽 루프(`REPE CMPSB`)는 그 위치에서 `source` 길이만큼 글자가 모두 일치하는지 검사합니다.
  2. **플래그 수동 제어**: 호출자가 `jnz notFound`로 결과를 판별할 수 있도록, **성공 시 `cmp eax, eax`로 `ZF = 1`을 만들고, 실패 시 `or eax, 1`로 `ZF = 0`을 강제 설정**합니다.

---

### 5. `Str_nextWord` Procedure

* **문제 원문**: Write a procedure called `Str_nextWord` that scans a string for the first occurrence of a certain delimiter character and replaces the delimiter with a null byte. After the call, if the delimiter was found, the Zero flag is set and `EAX` contains the offset of the next character beyond the delimiter. Otherwise, the Zero flag is clear and `EAX` is undefined.
* **번역**: 문자열에서 특정 구분자(delimiter) 문자가 처음 나타나는 위치를 스캔하여 그 구분자를 Null 바이트(`0`)로 교체하는 `Str_nextWord` 프로시저를 작성하시오. 호출 후 구분자를 찾았으면 Zero 플래그가 세트(`ZF=1`)되고 `EAX`에는 구분자 바로 다음 문자의 오프셋이 담기며, 찾지 못했으면 Zero 플래그가 클리어(`ZF=0`)되어야 한다.

```assembly
Str_nextWord PROC USES edi,
    pString:PTR BYTE,           ; 대상 문자열 주소 (예: "Johnson,Calvin")
    delimiter:BYTE              ; 찾을 구분자 문자 (예: ',')

    mov edi, pString            ; EDI = 문자열 탐색 포인터
    mov al, delimiter           ; AL = 찾을 구분자
    cld

L1:
    cmp BYTE PTR [edi], 0       ; 문자열 끝(Null)에 도달했는가?
    je  NotFound                ; 끝까지 구분자를 못 찾음
    cmp [edi], al               ; 현재 문자가 구분자와 일치하는가?
    je  Found                   ; 찾았으면 Found로 이동
    inc edi                     ; 다음 문자로 이동
    jmp L1

Found:
    mov BYTE PTR [edi], 0       ; ★ 구분자 위치에 Null(0)을 덮어써서 앞 단어를 분리
    inc edi                     ; ★ 구분자 바로 다음 문자를 가리키도록 +1 이동
    mov eax, edi                ; EAX = 다음 단어의 시작 주소 반환
    cmp eax, eax                ; ★ ZF = 1 설정 (성공 표시)
    ret

NotFound:
    or  eax, 1                  ; ★ ZF = 0 설정 (실패 표시)
    ret
Str_nextWord ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **탐색 및 분리**: 문자열을 순회하다 구분자(`','`)를 만나면 그 자리에 `0(Null)`을 기록해 앞 단어(`"Johnson"`)를 독립된 문자열로 만듭니다.
  2. **포인터 반환**: **`inc edi`를 수행하여 `EAX`가 Null로 바뀐 자리가 아닌 그 다음 단어(`"Calvin"`)의 첫 글자를 가리키도록 반환**합니다.

---

### 6. Constructing a Frequency Table (`Get_frequencies`)

* **문제 원문**: Write a procedure named `Get_frequencies` that constructs a character frequency table. Input to the procedure should be a pointer to a string and a pointer to an array of 256 doublewords initialized to all zeros. Each array position is indexed by its corresponding ASCII code.
* **번역**: 문자 빈도수 테이블을 생성하는 `Get_frequencies` 프로시저를 작성하시오. 입력은 문자열 포인터와 모두 0으로 초기화된 256개의 `DWORD` 배열 포인터이며, 배열의 각 위치는 해당 문자의 ASCII 코드로 인덱싱된다.

```assembly
Get_frequencies PROC USES eax esi edi,
    pString:PTR BYTE,           ; 입력 문자열 주소
    pTable:PTR DWORD            ; 256개 DWORD 빈도수 배열 주소

    mov esi, pString            ; ESI = 문자열 읽기 포인터
    mov edi, pTable             ; EDI = 빈도수 테이블 시작 주소

L1:
    movzx eax, BYTE PTR [esi]   ; ★ 현재 문자의 1바이트 ASCII 코드를 32비트 EAX로 확장
    cmp eax, 0                  ; 문자열 끝(Null)이면 종료
    je  L2
    inc DWORD PTR [edi + eax*4] ; ★ freqTable[ASCII]++ (DWORD는 4바이트이므로 인덱스 * 4)
    inc esi                     ; 다음 문자로 이동
    jmp L1

L2:
    ret
Get_frequencies ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **ASCII 코드의 인덱스화**: `movzx eax, BYTE PTR [esi]`를 통해 문자의 ASCII 값(예: `'A'` = `41h` = `65`)을 배열의 인덱스로 바로 사용합니다.
  2. **4바이트 스케일링**: 테이블 원소가 `DWORD`(4바이트)이므로 **`[edi + eax*4]` 주소 지정 방식을 사용해 해당 문자 칸의 카운트를 1씩 증가**시킵니다.

---

## 📌 7~9번: 알고리즘 구현 및 최적화

### 7. Sieve of Eratosthenes (에라토스테네스의 체)

* **문제 원문**: Create a 65,000-element array and display all primes between 2 and 65,000. Declare the array in an uninitialized data segment and use `STOSB` to fill it with zeros.
* **번역**: 65,000개 원소 배열을 생성하고 2와 65,000 사이의 모든 소수를 출력하시오. 배열은 초기화되지 않은 데이터 세그먼트(`.data?`)에 선언하고 `STOSB`를 사용하여 0으로 채우시오.

```assembly
.data?
    sieve BYTE 65000 DUP(?)     ; 초기화되지 않은 데이터 세그먼트에 65,000바이트 선언

.code
SieveOfEratosthenes PROC
    ; 1단계: STOSB를 이용해 배열 전체를 0(소수 후보)으로 초기화
    cld
    mov al, 0
    mov edi, OFFSET sieve
    mov ecx, 65000
    rep stosb

    ; 2단계: 2부터 시작하여 소수의 배수들을 1(합성수)로 마킹
    mov esi, 2                  ; ESI = 현재 검사 중인 숫자 (2부터 시작)
L_Outer:
    cmp esi, 65000
    jae L_Print                 ; 65,000에 도달하면 출력 단계로 이동
    cmp sieve[esi], 0           ; 현재 숫자가 0(소수)인가?
    jne L_Next                  ; 이미 1로 지워진 합성수면 건너뜀

    mov edi, esi
    add edi, esi                ; ★ EDI = 2 * ESI (자기 자신을 제외한 첫 번째 배수부터 시작)
L_Mark:
    cmp edi, 65000
    jae L_Next                  ; 배열 범위를 벗어나면 다음 소수로 이동
    mov sieve[edi], 1           ; 배수 위치에 1 기록 (소수가 아님을 표시)
    add edi, esi                ; 다음 배수로 점프 (EDI += ESI)
    jmp L_Mark

L_Next:
    inc esi
    jmp L_Outer

    ; 3단계: 배열 값이 여전히 0인 인덱스만 소수로 출력
L_Print:
    mov esi, 2
L_PrintLoop:
    cmp esi, 65000
    jae L_Done
    cmp sieve[esi], 0           ; 마킹되지 않은(0) 인덱스인가?
    jne L_SkipPrint
    mov eax, esi
    call WriteDec               ; 소수 출력
    mov al, ' '
    call WriteChar
L_SkipPrint:
    inc esi
    jmp L_PrintLoop
L_Done:
    ret
SieveOfEratosthenes ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **`.data?` 및 `REP STOSB`**: 실행 파일 크기를 줄이기 위해 `.data?`에 배열을 선언한 뒤, 실행 시점에 `REP STOSB`로 빠르게 `0`을 채웁니다.
  2. **배수 제거 (`L_Mark`)**: `sieve[esi] == 0`인 소수를 발견할 때마다 **`2 * esi`부터 시작해 `esi`씩 더해가며(`add edi, esi`) 배수 칸에 `1`을 마킹**합니다.

---

### 8. Improved `BubbleSort` (Exchange Flag 적용)

* **문제 원문**: Add a variable to the `BubbleSort` procedure in Section 9.5.1 that is set to 1 whenever a pair of values is exchanged within the inner loop. Use this variable to exit the sort before its normal completion if you discover that no exchanges took place during a complete pass through the array.
* **번역**: 9.5.1절의 `BubbleSort` 프로시저에 내부 루프에서 두 값이 교환될 때마다 1로 설정되는 변수를 추가하시오. 배열 전체를 한 번 순회하는 동안 교환이 전혀 일어나지 않았음이 확인되면 정상 종료 전에 정렬을 조기 종료하도록 이 변수를 사용하시오.

```assembly
BubbleSort PROC USES eax ebx ecx esi,
    pArray:PTR DWORD,
    Count:DWORD
    LOCAL exchangeFlag:BYTE     ; 교환 발생 여부를 기록할 지역 변수

    mov ecx, Count
    dec ecx                     ; 외부 루프 반복 횟수 = N - 1

L1:
    push ecx                    ; 외부 루프 카운터 저장
    mov esi, pArray             ; 매 패스마다 배열 맨 앞부터 시작
    mov exchangeFlag, 0         ; ★ 패스 시작 전 교환 플래그를 0으로 초기화

L2:
    mov eax, [esi]              ; 앞 원소 읽기
    cmp [esi + 4], eax          ; 뒤 원소와 크기 비교
    jge L3                      ; 앞 <= 뒤 이면 교환 불필요
    xchg eax, [esi + 4]         ; 앞 > 뒤 이면 두 원소 위치 교환
    mov [esi], eax
    mov exchangeFlag, 1         ; ★ 교환이 발생했으므로 플래그를 1로 설정

L3:
    add esi, 4                  ; 다음 원소 쌍으로 이동 (DWORD = 4바이트)
    loop L2

    pop ecx                     ; 외부 루프 카운터 복원
    cmp exchangeFlag, 0         ; ★ 이번 패스에서 교환이 한 번도 없었는가?
    je  L4                      ; 교환이 0번이면 이미 정렬 완료 상태이므로 즉시 조기 종료!
    loop L1

L4:
    ret
BubbleSort ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **플래그 초기화**: 외부 루프(`L1`)가 시작될 때마다 `exchangeFlag`를 `0`으로 리셋합니다.
  2. **조기 종료 판별**: 내부 루프(`L2`)에서 `xchg`가 일어나면 `exchangeFlag = 1`로 바꾸고, 한 패스가 끝난 뒤 **`exchangeFlag == 0`이면 남은 패스를 돌지 않고 `L4`로 점프해 불필요한 연산을 생략**합니다.

---

### 9. Register-Based `BinarySearch`

* **문제 원문**: Rewrite the binary search procedure shown in this chapter by using registers for `mid`, `first`, and `last`. Add comments to clarify the registers’ usage.
* **번역**: 이 장에 소개된 이진 탐색 프로시저를 `mid`, `first`, `last`에 메모리 변수 대신 레지스터를 사용하도록 다시 작성하고, 레지스터 용도를 명확히 설명하는 주석을 추가하시오.

```assembly
BinarySearch PROC USES ebx ecx edx esi edi,
    pArray:PTR DWORD,
    Count:DWORD,
    searchVal:DWORD

    ; [레지스터 용도 정의]
    ; EBX = first (현재 탐색 구간의 시작 인덱스)
    ; EDX = last  (현재 탐색 구간의 끝 인덱스)
    ; ESI = mid   (현재 탐색 구간의 중간 인덱스)

    mov ebx, 0                  ; first = 0
    mov edx, Count
    dec edx                     ; last = Count - 1
    mov edi, pArray             ; EDI = 정렬된 배열의 시작 주소

L1:
    cmp ebx, edx                ; while (first <= last) 검사
    jg  NotFound                ; first > last 이면 탐색 실패

    mov esi, ebx
    add esi, edx
    shr esi, 1                  ; ★ mid = (first + last) / 2 (우측 1비트 시프트로 빠른 나누기 2)

    mov eax, [edi + esi*4]      ; EAX = array[mid] 값 읽기
    cmp eax, searchVal          ; 중간값과 찾는 값 비교
    je  Found                   ; 일치하면 찾은 인덱스(mid) 반환
    jl  IsLess                  ; array[mid] < searchVal 이면 오른쪽 절반 탐색

    ; array[mid] > searchVal 인 경우: 왼쪽 절반 탐색
    mov edx, esi
    dec edx                     ; last = mid - 1
    jmp L1

IsLess:
    ; array[mid] < searchVal 인 경우: 오른쪽 절반 탐색
    mov ebx, esi
    inc ebx                     ; first = mid + 1
    jmp L1

Found:
    mov eax, esi                ; ★ 찾은 위치의 인덱스(mid)를 EAX에 담아 반환
    ret

NotFound:
    mov eax, -1                 ; ★ 탐색 실패 시 -1 반환
    ret
BinarySearch ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **메모리 접근 제거**: `first(EBX)`, `last(EDX)`, `mid(ESI)`를 모두 CPU 내부 레지스터에 할당하여 루프 내 메모리 읽기/쓰기 병목을 없앴습니다.
  2. **중간값 계산**: `(first + last) / 2`를 나눗셈(`DIV`) 대신 **`shr esi, 1` 명령어로 처리하여 실행 속도를 최적화**했습니다.

---

## 📌 10~14번: 2차원 행렬 생성/순회 및 문자열 변형

### 10. Letter Matrix (50% 모음 확률 4x4 랜덤 알파벳 행렬)

* **문제 원문**: Create a procedure that generates a four-by-four matrix of randomly chosen capital letters. When choosing the letters, there must be a 50% probability that the chosen letter is a vowel. Write a test program with a loop that calls your procedure five times and displays each matrix in the console window.
* **번역**: 무작위로 선택된 대문자로 구성된 4x4 행렬을 생성하는 프로시저를 작성하시오. 문자를 선택할 때 선택된 문자가 모음(vowel)일 확률이 50%여야 한다. 이 프로시저를 5번 호출하는 루프를 가진 테스트 프로그램을 작성하고 각 행렬을 콘솔 창에 출력하시오.

```assembly
.data
    vowels     BYTE "AEIOU", 0                  ; 모음 5개
    consonants BYTE "BCDFGHJKLMNPQRSTVWXYZ", 0  ; 자음 21개
    matrix     BYTE 16 DUP(?)                   ; 4x4 문자 행렬 저장 버퍼

.code
GenerateMatrix PROC USES eax ecx esi
    mov ecx, 16                 ; 총 16개 칸(4x4) 채우기
    mov esi, OFFSET matrix

L1:
    mov eax, 2
    call RandomRange            ; EAX = 0 또는 1 생성 (각각 50% 확률)
    cmp eax, 0
    je  PickVowel               ; 0이면 모음 선택

    ; 1이면 자음(21개 중 1개) 선택
    mov eax, 21
    call RandomRange            ; EAX = 0 ~ 20
    mov al, consonants[eax]
    jmp StoreChar

PickVowel:
    ; 0이면 모음(5개 중 1개) 선택
    mov eax, 5
    call RandomRange            ; EAX = 0 ~ 4
    mov al, vowels[eax]

StoreChar:
    mov [esi], al               ; 선택된 대문자를 행렬에 저장
    inc esi
    loop L1
    ret
GenerateMatrix ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **50% 확률 분기**: `RandomRange`에 `2`를 전달해 `0` 또는 `1`을 뽑음으로써 모음과 자음이 정확히 **50% 대 50% 확률**로 선택되도록 보장합니다.
  2. **문자 추출**: 모음은 길이 `5` 배열에서, 자음은 길이 `21` 배열에서 다시 랜덤 인덱스를 뽑아 `4x4` 버퍼(`16바이트`)에 순차 저장합니다.

---

### 11. Letter Matrix / Sets with Vowels (모음 2개 포함 4글자 집합 출력)

* **문제 원문**: Use the letter matrix generated in the previous programming exercise as a starting point for this program. Traverse each matrix row, column, and diagonal, generating sets of letters. Display only four-letter sets containing exactly two vowels.
* **번역**: 이전 프로그래밍 실습에서 생성한 문자 행렬을 시작점으로 사용하시오. 행렬의 각 행(row), 열(column), 그리고 대각선(diagonal)을 순회하며 문자 집합을 생성하고, 정확히 두 개의 모음을 포함하는 4글자 집합만 출력하시오.

```assembly
; [핵심 설계] 4x4 행렬(인덱스 0~15)의 가로 4줄, 세로 4줄, 대각선 2줄 = 총 10개 라인 검사
.data
    ; 각 라인을 구성하는 4개 칸의 인덱스 테이블 (총 10개 라인 * 4바이트)
    lines BYTE  0,  1,  2,  3   ; 0행 (가로)
          BYTE  4,  5,  6,  7   ; 1행 (가로)
          BYTE  8,  9, 10, 11   ; 2행 (가로)
          BYTE 12, 13, 14, 15   ; 3행 (가로)
          BYTE  0,  4,  8, 12   ; 0열 (세로)
          BYTE  1,  5,  9, 13   ; 1열 (세로)
          BYTE  2,  6, 10, 14   ; 2열 (세로)
          BYTE  3,  7, 11, 15   ; 3열 (세로)
          BYTE  0,  5, 10, 15   ; 주대각선 (\)
          BYTE  3,  6,  9, 12   ; 부대각선 (/)
    tempSet BYTE 5 DUP(0)       ; 4글자 + Null 저장용 임시 버퍼

.code
CheckMatrixSets PROC USES eax ebx ecx edx esi edi
    mov ecx, 10                 ; 총 10개 라인 순회
    mov esi, OFFSET lines

L_LineLoop:
    push ecx
    xor ebx, ebx                ; EBX = 현재 라인의 모음 개수 카운터 (0 초기화)
    mov edi, 0                  ; EDI = 라인 내 문자 위치 (0 ~ 3)

L_CharLoop:
    movzx eax, BYTE PTR [esi + edi] ; 행렬 인덱스 가져오기
    mov al, matrix[eax]             ; 행렬에서 실제 알파벳 읽기
    mov tempSet[edi], al            ; 출력용 버퍼에 복사

    ; 모음('A','E','I','O','U') 여부 판별
    cmp al, 'A'
    je  IsVowel
    cmp al, 'E'
    je  IsVowel
    cmp al, 'I'
    je  IsVowel
    cmp al, 'O'
    je  IsVowel
    cmp al, 'U'
    je  IsVowel
    jmp NextChar

IsVowel:
    inc ebx                     ; 모음이면 카운트 +1

NextChar:
    inc edi
    cmp edi, 4
    jb  L_CharLoop

    cmp ebx, 2                  ; ★ 모음 개수가 정확히 2개인가?
    jne SkipDisplay
    mov edx, OFFSET tempSet     ; 정확히 2개일 때만 4글자 집합 출력
    call WriteString
    call Crlf

SkipDisplay:
    add esi, 4                  ; 다음 라인 인덱스 테이블로 이동
    pop ecx
    loop L_LineLoop
    ret
CheckMatrixSets ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **인덱스 테이블 매핑**: 가로(4), 세로(4), 대각선(2) 총 **10개 라인의 1차원 오프셋을 테이블(`lines`)로 정의**하면 복잡한 다중 루프 없이 단일 루프로 깔끔하게 순회할 수 있습니다.
  2. **모음 카운팅**: 각 라인의 4글자를 추출하면서 모음 개수(`EBX`)를 세고, **`EBX == 2`일 때만 해당 4글자 문자열을 출력**합니다.

---

### 12. Calculating the Sum of an Array Row (`calc_row_sum`)

* **문제 원문**: Write a procedure named `calc_row_sum` that calculates the sum of a single row in a two-dimensional array of bytes, words, or doublewords. The procedure should have the following stack parameters: array offset, row size, array type, row index. It must return the sum in `EAX`. Use explicit stack parameters, not `INVOKE` or extended `PROC`.
* **번역**: `BYTE`, `WORD`, 또는 `DWORD` 2차원 배열에서 단일 행의 합계를 계산하는 `calc_row_sum` 프로시저를 작성하시오. 프로시저는 배열 오프셋, 행 크기(열 개수), 배열 타입(1/2/4), 행 인덱스를 스택 매개변수로 받아야 하며 합계를 `EAX`에 반환해야 한다. `INVOKE`나 확장 `PROC` 대신 명시적 스택 매개변수를 사용하시오.

```assembly
; [호출 규약: 호출자가 역순으로 push 후 call calc_row_sum]
; [ebp + 8]  = arrayOffset (배열 시작 주소)
; [ebp + 12] = rowSize     (한 행의 원소 개수)
; [ebp + 16] = arrayType   (원소 자료형 크기: 1=BYTE, 2=WORD, 4=DWORD)
; [ebp + 20] = rowIndex    (합계를 구할 행 번호, 0부터 시작)

calc_row_sum PROC
    push ebp
    mov  ebp, esp               ; 스택 프레임 기준점 설정
    push ebx
    push ecx
    push edx
    push esi

    ; 1단계: 선택한 행의 시작 메모리 주소 계산 = arrayOffset + (rowIndex * rowSize * arrayType)
    mov  eax, [ebp + 20]        ; EAX = rowIndex
    mul  DWORD PTR [ebp + 12]   ; EAX = rowIndex * rowSize (원소 개수 오프셋)
    mul  DWORD PTR [ebp + 16]   ; EAX = 바이트 단위 행 오프셋
    mov  esi, [ebp + 8]         ; ESI = arrayOffset
    add  esi, eax               ; ★ ESI = 목표 행의 첫 번째 원소 주소

    ; 2단계: 자료형 크기에 맞춰 행의 모든 원소 합산
    mov  ecx, [ebp + 12]        ; ECX = 반복 횟수 (한 행의 원소 개수)
    mov  ebx, [ebp + 16]        ; EBX = 자료형 크기 (1, 2, 4)
    xor  eax, eax               ; EAX = 누적 합계 (0 초기화)

L_SumLoop:
    cmp  ebx, 1
    je   AddByte
    cmp  ebx, 2
    je   AddWord

    add  eax, [esi]             ; DWORD(4바이트) 원소 합산
    jmp  NextElem

AddByte:
    movzx edx, BYTE PTR [esi]   ; BYTE(1바이트)를 32비트로 확장 후 합산
    add  eax, edx
    jmp  NextElem

AddWord:
    movzx edx, WORD PTR [esi]   ; WORD(2바이트)를 32비트로 확장 후 합산
    add  eax, edx

NextElem:
    add  esi, ebx               ; ★ 자료형 크기(1, 2, 4)만큼 포인터 전진
    loop L_SumLoop

    pop  esi
    pop  edx
    pop  ecx
    pop  ebx
    pop  ebp
    ret  16                     ; ★ 스택에 push된 매개변수 4개(4 * 4 = 16바이트) 정리 후 복귀
calc_row_sum ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **명시적 스택 프레임**: `EBP`를 기준으로 `[ebp + 8]`부터 `[ebp + 20]`까지 매개변수를 직접 참조하며, 종료 시 **`ret 16`으로 스택 매개변수 16바이트를 깔끔하게 정리**합니다.
  2. **범용 자료형 처리**: `arrayType`(`1, 2, 4`)에 따라 `movzx`로 안전하게 확장하여 더하고 포인터를 `add esi, ebx`만큼 이동시키므로 모든 정수형 2차원 배열에 동작합니다.

---

### 13. Trimming Leading Characters (`Str_trimLeading`)

* **문제 원문**: Create a variant of the `Str_trim` procedure that lets the caller remove all instances of a leading character from a string. For example, if you were to call it with a pointer to the string `"###ABC"` and pass it the `#` character, the resulting string would be `"ABC"`.
* **번역**: 호출자가 문자열의 맨 앞부분(leading)에 있는 특정 문자의 모든 인스턴스를 제거할 수 있도록 하는 `Str_trim`의 변형 프로시저를 작성하시오. 예를 들어 `"###ABC"`와 `'#'` 문자를 전달하면 결과 문자열은 `"ABC"`가 되어야 한다.

```assembly
Str_trimLeading PROC USES eax esi,
    pString:PTR BYTE,           ; 대상 문자열 주소 (예: "###ABC")
    charToTrim:BYTE             ; 제거할 선행 문자 (예: '#')

    mov esi, pString            ; ESI = 문자열 시작 주소
    mov al, charToTrim          ; AL = 제거 대상 문자

L1:
    cmp BYTE PTR [esi], 0       ; 문자열 끝(Null)에 도달했는가? (전부 '#'인 경우)
    je  L2
    cmp [esi], al               ; 현재 문자가 제거할 문자와 같은가?
    jne L2                      ; ★ 다른(정상) 문자를 만나면 스캔 중단!
    inc esi                     ; 제거 대상 문자면 건너뛰고 다음 칸으로 전진
    jmp L1

L2:
    ; ★ 이제 ESI는 제거되지 않을 첫 번째 정상 문자('A')의 주소를 가리킴
    INVOKE Str_copy, esi, pString ; 정상 문자부터 끝(Null)까지를 문자열 맨 앞으로 당겨 덮어씀
    ret
Str_trimLeading ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **선행 문자 건너뛰기**: 문자열 시작점부터 `charToTrim`과 일치하는 동안 `ESI`를 전진시켜 **첫 번째 정상 문자의 시작 위치**를 찾습니다.
  2. **앞으로 당겨 복사**: 찾은 위치(`ESI`)부터 문자열 끝까지를 `Str_copy`를 이용해 원래 시작 주소(`pString`)로 복사하면 앞쪽 문자가 깔끔하게 제거됩니다.

---

### 14. Trimming a Set of Characters (`Str_trimSet`)

* **문제 원문**: Create a variant of the `Str_trim` procedure that lets the caller remove all instances of a set of characters from the end of a string. For example, if you were to call it with a pointer to the string `"ABC#$&"` and pass it a pointer to an array of filter characters containing `"%#!;$&*"`, the resulting string would be `"ABC"`.
* **번역**: 호출자가 문자열의 끝부분에서 특정 문자 집합(set of characters)에 속한 모든 문자를 제거할 수 있도록 하는 `Str_trim`의 변형 프로시저를 작성하시오. 예를 들어 `"ABC#$&"`와 필터 문자 배열 `"%#!;$&*"`의 포인터를 전달하면 결과 문자열은 `"ABC"`가 되어야 한다.

```assembly
Str_trimSet PROC USES eax esi edi,
    pString:PTR BYTE,           ; 대상 문자열 주소 (예: "ABC#$&")
    pFilterSet:PTR BYTE         ; 제거할 문자 집합 주소 (예: "%#!;$&*")

    INVOKE Str_length, pString
    cmp eax, 0                  ; 빈 문자열이면 즉시 종료
    je  L_Done

    mov esi, pString
    add esi, eax
    dec esi                     ; ★ ESI = 문자열의 맨 마지막 글자('&') 주소

L_Backward:
    cmp esi, pString            ; 문자열 시작 주소보다 앞으로 넘어갔는가? (전부 필터 문자인 경우)
    jb  L_Terminate

    ; 현재 문자([ESI])가 필터 집합(pFilterSet) 안에 들어있는지 내부 루프로 검사
    mov edi, pFilterSet         ; EDI = 필터 문자열 시작 주소
    mov al, [esi]               ; AL = 현재 검사할 대상 문자

L_CheckFilter:
    cmp BYTE PTR [edi], 0       ; 필터 집합 끝까지 확인했는데 일치하는 게 없는가?
    je  L_Terminate             ; ★ 필터에 없는 정상 문자이므로 트림 작업 완전히 종료!
    cmp al, [edi]               ; 현재 문자가 필터 문자와 일치하는가?
    je  L_RemoveChar            ; 일치하면 제거 대상이므로 한 칸 앞으로 후진!
    inc edi                     ; 다음 필터 문자로 이동
    jmp L_CheckFilter

L_RemoveChar:
    dec esi                     ; ★ 뒤에서 앞쪽으로 한 칸 이동하여 계속 검사
    jmp L_Backward

L_Terminate:
    mov BYTE PTR [esi + 1], 0   ; ★ 마지막으로 확인된 정상 문자 바로 다음 칸에 Null(0) 기록
L_Done:
    ret
Str_trimSet ENDP
```

* **실행 순서 및 핵심 포인트**:
  1. **역방향 이중 탐색**: 문자열의 맨 끝 글자부터 시작(`L_Backward`)해, 각 글자가 필터 집합(`L_CheckFilter`)에 속하는지 확인합니다.
  2. **종료 및 절단 (`L_Terminate`)**: 필터 집합에 **없는 정상 문자를 만나는 즉시 탐색을 멈추고, 그 정상 문자 바로 다음 칸(`[esi + 1]`)에 `0(Null)`을 기록**하여 뒤쪽 특수문자들을 한 번에 잘라냅니다.