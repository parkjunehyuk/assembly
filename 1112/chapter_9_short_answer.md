# 📚 \[9장\] 문자열과 배열 (Strings and Arrays) 완벽 복습 가이드

Kip Irvine의 *Assembly Language for x86 Processors* 9장의 핵심 개념, 스트링 기본 명령어, Irvine32 라이브러리, 2차원 배열 주소 계산법, 그리고 연습문제(9.9.1 Short Answer) 전 문항(원문·번역·정답·상세 해설 및 어셈블리 코드 디테일)을 담은 종합 정리 문서입니다.

## 1. 스트링 기본 명령어 (String Primitive Instructions)

스트링 기본 명령어는 피연산자를 코드에 명시하지 않고 약속된 레지스터(`ESI`, `EDI`, `EAX/AX/AL`)를 자동으로 사용합니다.

| 명령어 | 이름 | 동작 원리 (`DF = 0` 기준) | 
| ----- | ----- | ----- | 
| **`MOVSB / MOVSW / MOVSD`** | Move String | `[ESI]`의 데이터를 `[EDI]`로 복사하고 `ESI`, `EDI`를 자동 증가 (`+1 / +2 / +4`) | 
| **`CMPSB / CMPSW / CMPSD`** | Compare String | `[ESI]`와 `[EDI]`의 값을 뺄셈(`[ESI] - [EDI]`)하여 플래그(`ZF`, `CF`) 설정 후 포인터 증가 | 
| **`SCASB / SCASW / SCASD`** | Scan String | `AL / AX / EAX`와 `[EDI]`의 값을 비교(`A - [EDI]`)하여 같은 값을 탐색 후 `EDI` 증가 | 
| **`STOSB / STOSW / STOSD`** | Store String | `AL / AX / EAX`의 값을 `[EDI]` 메모리에 저장하고 `EDI` 자동 증가 | 
| **`LODSB / LODSW / LODSD`** | Load String | `[ESI]` 메모리의 값을 `AL / AX / EAX`로 읽어오고 `ESI` 자동 증가 | 

### 💡 방향 플래그 (Direction Flag, DF)

* **`CLD`** (Clear Direction Flag, `DF = 0`): 주소가 증가(`INC`)하는 정방향 (앞 $\rightarrow$ 뒤)

* **`STD`** (Set Direction Flag, `DF = 1`): 주소가 감소(`DEC`)하는 역방향 (뒤 $\rightarrow$ 앞)

## 2. 반복 접두사 (Repeat Prefixes)

스트링 명령어 앞에 붙어 **`ECX` 레지스터가 0이 될 때까지** 명령어를 자동으로 반복 실행합니다.

* **`REP`**: `ECX > 0`인 동안 무조건 반복 (주로 `MOVS`, `STOS`와 함께 사용)

* **`REPZ` / `REPE`**: `ECX > 0` **이고** `ZF = 1`(두 값이 같음)인 동안 반복 (두 문자열이 어디까지 같은지 비교할 때 `CMPS`와 함께 사용)

* **`REPNZ` / `REPNE`**: `ECX > 0` **이고** `ZF = 0`(두 값이 다름)인 동안 반복 (특정 문자를 찾을 때까지 `SCAS`와 함께 사용)

## 3. Irvine32 문자열 처리 프로시저

| 프로시저 이름 | 주요 기능 | 전달 인자 (`INVOKE` 사용 시) | 
| ----- | ----- | ----- | 
| **`Str_compare`** | 두 문자열을 사전순으로 비교 (`ZF`, `CF` 플래그 변경) | `ADDR string1, ADDR string2` | 
| **`Str_length`** | Null(`0`)을 제외한 문자열의 길이를 구해 `EAX`에 반환 | `ADDR string` | 
| **`Str_copy`** | 원본 문자열을 대상 버퍼로 복사 | `ADDR source, ADDR target` | 
| **`Str_trim`** | 문자열 끝(오른쪽)에 있는 특정 문자(공백 등)를 제거 | `ADDR string, charToTrim` | 
| **`Str_ucase`** | 문자열의 모든 소문자를 대문자로 변환 | `ADDR string` | 

## 4. 2차원 배열 주소 계산 (Base-Index Addressing)

행 우선(Row-Major) 방식의 2차원 배열 접근 공식입니다.

$$
\text{주소} = \text{배열 시작 주소} + (\text{행 인덱스} \times \text{한 행의 바이트 크기}) + (\text{열 인덱스} \times \text{원소 자료형 크기})
$$

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

## 5. 심층 이해: 3번 문제 "`CMPS` 명령어는 왜 모호(Ambiguous)한가?"

`CMPS` 명령어는 **'겉으로 보이는 코드'와 'CPU가 실제로 하는 행동'이 서로 엇갈리는 이중성** 때문에 모호하다고 부릅니다.

### ① 크기의 모호함 (몇 바이트인지 안 적혀 있다!)

* 피연산자 없이 `CMPS`라고만 적으면 1바이트(`CMPSB`), 2바이트(`CMPSW`), 4바이트(`CMPSD`) 중 어떤 단위로 비교할지 알 수 없습니다.

### ② 명시적 피연산자의 함정 (변수 이름을 적어도 CPU가 무시한다!)

* 문법상 `cmps str1, str2`처럼 변수 이름을 적을 수 있지만, 어셈블러(번역기)는 이를 보고 **자료형 크기(BYTE/WORD/DWORD)만 파악한 뒤 변수 이름은 버립니다.**

* 실제 CPU는 코드에 적힌 `str1`, `str2`를 쳐다보지도 않고, **무조건 `DS:[ESI]`와 `ES:[EDI]`가 가리키는 메모리 주소를 찾아가 비교**합니다.

* **🚕 택시 기사님 비유**: 코드에 `cmps str1, str2`라고 적은 것은 승객이 말로 *"강남역이랑 홍대 비교해 주세요"*라고 중얼거린 것뿐이고, 실제 기사님(CPU)은 무조건 내비게이션에 찍힌 출발지(`ESI`)와 도착지(`EDI`)로만 달립니다.

### ③ 방향(비교 대상)의 비가시성

* 일반 `CMP A, B`는 코드에 `A`와 `B`가 보이지만, `cmpsb`는 코드에 아무것도 안 적혀 있어 머릿속으로 `[ESI] - [EDI]`를 계산한다는 사실을 외워서 해석해야 하므로 어느 쪽이 Source(`ESI`)이고 Target(`EDI`)인지 한눈에 파악하기 어렵습니다.

> **결론 (실전 정석):** 오해를 막기 위해 `cmps var1, var2` 형태는 쓰지 않고, **`ESI`와 `EDI`를 먼저 세팅한 뒤 크기를 명시한 `cmpsb`, `cmpsw`, `cmpsd`를 단독으로 사용하는 것**이 가장 안전합니다.

## 6. 연습문제 9.9.1 Short Answer (1~14번) 전체 원문·번역·정답·상세 코드 디테일

### 📌 1~5번: 문자열 기본 명령어 (`MOVS`, `STOS`, `CMPS`, `SCAS`)

#### 1. Which Direction flag setting causes index registers to move backward through memory when executing string primitives?

* **번역**: 문자열 기본 명령어를 실행할 때 인덱스 레지스터(`ESI`, `EDI`)가 메모리에서 역방향(뒤쪽 주소)으로 이동하게 만드는 Direction Flag(DF) 설정은 무엇인가?
* **정답**: **Direction Flag = 1 (`STD` 명령어 실행)**
* **해설 및 코드 디테일**: `STD`(Set Direction Flag)를 실행해 `DF = 1`로 세팅하면 주소를 감소(`DEC`)시키며 역방향으로 진행합니다. 단, 작업을 마친 후에는 반드시 `CLD`로 원상복구하는 것이 안전합니다.
  ```assembly
  std                     ; DF = 1 설정 (역방향 이동: 뒤 -> 앞)
  ; ... 역방향 문자열 처리 ...
  cld                     ; 작업 완료 후 기본 상태(DF = 0)로 복구
  ```

#### 2. When a repeat prefix is used with STOSW, what value is added to or subtracted from the index register?

* **번역**: `STOSW`에 반복 접두사(`REP`)를 함께 사용할 때 인덱스 레지스터(`EDI`)에 더해지거나 빼지는 값은 얼마인가?
* **정답**: **2 (2 바이트)**
* **해설 및 코드 디테일**: 끝 글자 `W`는 Word(2바이트)를 의미하므로 한 번 실행될 때마다 `DF = 0`이면 `+2`, `DF = 1`이면 `-2`씩 `EDI` 주소가 증감합니다. (`B`=1, `W`=2, `D`=4)
  ```assembly
  cld                     ; DF = 0 (증가 방향)
  mov ax, 0FFFFh          ; 채울 2바이트(WORD) 값
  mov edi, OFFSET wordArr ; WORD 배열 시작 주소
  mov ecx, 10             ; 10개 원소 반복
  rep stosw               ; 실행될 때마다 EDI가 +2씩 자동 증가 (총 20바이트 이동)
  ```

#### 3. In what way is the CMPS instruction ambiguous?

* **번역**: `CMPS` 명령어는 어떤 점에서 모호한가?
* **정답**: **피연산자의 크기(Byte, Word, Doubleword)가 명시되지 않았으며, 명시적 피연산자(변수 이름)를 적더라도 실제 비교는 변수 이름이 아닌 `[ESI]`와 `[EDI]` 주소를 기준으로 수행되어 어느 쪽이 Source이고 Target인지 헷갈리기 쉽다.**
* **해설 및 코드 디테일**: `CMPS`의 명시적 피연산자 형식은 자료형 크기와 의도만 표시할 뿐 실제 접근 주소를 지정하지 않습니다. 32비트 주소 크기에서는 무조건 `DS:[ESI]`와 `ES:[EDI]`를 비교(`CMP [esi], [edi]`와 동일하게 플래그 설정)하므로 `ESI`와 `EDI`를 올바르게 설정해야 합니다.
  ```assembly
  mov esi, OFFSET source  ; 비교할 첫 번째 문자열 주소 (Source)
  mov edi, OFFSET target  ; 비교할 두 번째 문자열 주소 (Target)
  cmpsb                   ; DS:[ESI]와 ES:[EDI]의 바이트를 비교 ([ESI] - [EDI])
  ```

#### 4. When the Direction flag is clear and SCASB has found a matching character, where does EDI point?

* **번역**: Direction Flag가 0(`CLD`)인 상태에서 `SCASB`가 일치하는 문자를 찾았을 때, `EDI`는 어디를 가리키고 있는가?
* **정답**: **찾은 문자의 바로 다음 바이트(1바이트 뒤의 위치)**
* **해설 및 코드 디테일**: `SCASB`는 문자를 비교한 직후 무조건 `EDI`를 1 증가(`+1`)시킨 뒤에 멈춥니다. 따라서 일치한 문자의 실제 주소를 얻으려면 종료 후 `EDI - 1`로 보정해야 합니다.
  ```assembly
  cld                     ; DF = 0 (정방향)
  mov al, 'X'             ; 찾을 문자
  mov edi, OFFSET strBuf  ; 검색 대상 문자열
  mov ecx, LENGTHOF strBuf
  repne scasb             ; 'X'를 찾으면 멈추지만 EDI는 이미 +1 지나간 상태!
  jnz NotFound            ; 못 찾았으면(ZF=0) 분기

  ; [주소 보정 디테일] 일치하는 문자를 찾은 직후 EDI - 1 보정
  lea eax, [edi - 1]      ; EAX = 일치한 문자의 실제 메모리 주소 (또는 dec edi 사용)
  ```

#### 5. When scanning an array for the first occurrence of a particular character, which repeat prefix would be best?

* **번역**: 배열에서 특정 문자가 처음 나타나는 위치를 스캔할 때 가장 적합한 반복 접두사는 무엇인가?
* **정답**: **`REPNE` (또는 `REPNZ`)**
* **해설 및 코드 디테일**: 찾는 문자가 나오기 전까지는 계속 **다르므로(`Not Equal`, `ZF = 0`)** 반복해야 합니다. 찾는 순간 두 값이 같아져 `ZF = 1`이 되며 루프가 즉시 종료됩니다. 루프 직후에는 반드시 `JNZ`로 실제 발견 여부를 체크해야 합니다.
  ```assembly
  repne scasb             ; AL != [EDI] 인 동안 계속 스캔
  jnz CharNotFound        ; ECX가 0이 되어 끝났는데도 못 찾은 경우(ZF=0) 예외 처리!
  ```

### 📌 6~10번: Irvine32 문자열 라이브러리 (`Str_trim`, `Str_ucase`, `Str_length`)

#### 6. What Direction flag setting is used in the Str_trim procedure from Section 9.3?

* **번역**: 9.3절의 `Str_trim` 프로시저에서는 어떤 Direction Flag 설정을 사용하는가?
* **정답**: **Direction Flag = 1 (`STD`, 역방향 설정)**
* **해설 및 코드 디테일**: `Str_trim`은 문자열의 **맨 뒤(Null 문자 바로 앞)에서부터 앞쪽으로 거꾸로** 오면서 불필요한 문자를 잘라내야 하므로 `STD`로 주소를 감소시키며 탐색합니다.
  ```assembly
  std                     ; DF = 1 설정 (문자열 끝에서 시작점으로 역방향 스캔)
  ; ... 뒤에서부터 제거할 문자 검사 ...
  cld                     ; 프로시저 종료 전 반드시 DF = 0으로 복구!
  ```

#### 7. Why does the Str_trim procedure from Section 9.3 use the JNE instruction?

* **번역**: 9.3절의 `Str_trim` 프로시저는 왜 `JNE` 명령어를 사용하는가?
* **정답**: **제거할 대상 문자가 아닌(일치하지 않는) 정상 문자를 만났을 때 반복(트림)을 중단하고 빠져나오기 위해서**
* **해설 및 코드 디테일**: 뒤에서부터 지워오다가 제거 대상이 아닌 진짜 글자(`Not Equal`)를 발견하면 더 이상 지우면 안 되므로 루프를 탈출한 뒤, 그 글자 바로 다음 칸(`EDI + 2` 또는 보정된 위치)에 `0(Null)`을 기록하여 문자열을 자릅니다.
  ```assembly
  L1:
      scasb               ; 제거할 문자(AL)와 현재 문자([EDI]) 비교 후 EDI 자동 감소
      jne L2              ; 제거할 문자가 아니면(Not Equal) 트림 중단하고 L2로 탈출!
      loop L1
  L2:
      mov BYTE PTR [edi + 2], 0 ; 정상 문자 바로 다음 위치에 Null(0)을 써서 문자열 종료
  ```

#### 8. What happens in the Str_ucase procedure from Section 9.3 if the target string contains a digit?

* **번역**: 9.3절의 `Str_ucase`(대문자 변환) 프로시저에서 대상 문자열에 숫자(Digit)가 포함되어 있으면 어떻게 되는가?
* **정답**: **아무 변화 없이 원래 숫자 그대로 유지된다.**
* **해설 및 코드 디테일**: `Str_ucase`는 각 문자의 아스키 코드가 소문자 알파벳(`'a'` = 61h ~ `'z'` = 7Ah) 범위 안에 있는지 먼저 검사한 뒤, 해당 범위일 때만 `and BYTE PTR [esi], 11011111b` (또는 `sub al, 20h`)로 대문자 변환을 수행합니다. 숫자(`'0'`~`'9'`)는 범위 밖이므로 건너뜁니다.
  ```assembly
  cmp al, 'a'
  jb  NextChar            ; 'a'보다 작으면(숫자, 대문자, 기호 등) 변환하지 않고 건너뜀
  cmp al, 'z'
  ja  NextChar            ; 'z'보다 크면 건너뜀
  and BYTE PTR [esi], 0DFh ; 소문자일 때만 비트 마스크로 대문자 변환
  ```

#### 9. If the Str_length procedure from Section 9.3 used SCASB, which repeat prefix would be most appropriate?

* **번역**: 9.3절의 `Str_length` 프로시저가 `SCASB`를 사용한다면 어떤 반복 접두사가 가장 적절할까?
* **정답**: **`REPNE` (또는 `REPNZ`)**
* **해설 및 코드 디테일**: C/어셈블리 문자열의 끝을 알리는 `0(Null)`을 찾을 때까지, 즉 **`AL(0)`과 다를 동안(`Not Equal`)** 계속 스캔해야 하므로 `REPNE`를 사용합니다.
  ```assembly
  mov al, 0               ; 찾을 목표값: 문자열 종료 문자 Null(0)
  mov ecx, 0FFFFFFFFh     ; 최대 탐색 횟수를 무한대에 가깝게 설정
  repne scasb             ; 0(Null)을 만날 때까지 계속 전진 스캔!
  ```

#### 10. If the Str_length procedure from Section 9.3 used SCASB, how would it calculate and return the string length?

* **번역**: `Str_length` 프로시저가 `SCASB`를 사용한다면 문자열의 길이를 어떻게 계산해서 반환하겠는가?
* **정답**: **최종 `EDI` 주소에서 원래 문자열의 시작 주소를 뺀 뒤 1을 뺀 값(`EDI - 시작주소 - 1`)을 `EAX`에 넣어 반환한다.**
* **해설 및 코드 디테일**: `SCASB`가 끝의 `0(Null)`을 찾은 뒤에도 `EDI`가 한 칸 더 전진(`+1`)해 있으므로, 전체 이동 거리(`EDI - 시작주소`)에는 실제 글자 수 외에 **Null 문자(1바이트)까지 포함**되어 있습니다. 따라서 `-1`을 해주어야 순수 문자열 길이가 됩니다.
  ```assembly
  ; 방법 1: 주소 차이 이용 (시작 주소가 pString에 저장된 경우)
  sub edi, pString        ; EDI = 문자열 길이 + 1 (Null 문자 포함)
  dec edi                 ; Null 문자 1바이트 제외 (-1 보정)
  mov eax, edi            ; EAX에 순수 문자열 길이 반환

  ; 방법 2: ECX 카운터 역산 이용 (초기 ECX = 0FFFFFFFFh였던 경우)
  not ecx                 ; 감소된 ECX 비트를 반전하면 (길이 + 1)이 됨
  dec ecx                 ; Null 문자 1바이트 제외 (-1 보정)
  mov eax, ecx            ; EAX에 순수 문자열 길이 반환
  ```

### 📌 11~14번: 이진 탐색 (`BinarySearch` 알고리즘과 코드 분석)

#### 11. What is the maximum number of comparisons needed by the binary search algorithm when an array contains 1,024 elements?

* **번역**: 배열에 1,024개의 원소가 있을 때 이진 탐색(Binary Search) 알고리즘이 필요로 하는 최대 비교 횟수는 몇 번인가?
* **정답**: **10번 (또는 교재 수식 $\lfloor \log_2(N) \rfloor + 1$ 기준 11번)**
* **해설 및 코드 디테일**: $2^{10} = 1024$이므로 매 단계마다 탐색 구간(`first` ~ `last`)을 절반씩 좁혀나가면 $\log_2(1024) = 10$번의 분할로 원소를 특정할 수 있습니다. (단, 마지막 남은 1개 원소가 찾는 값과 일치하는지 최종 확인하는 비교까지 포함하면 교재 공식에 따라 $10 + 1 = 11$번이 됩니다.)

#### 12. In the FillArray procedure from the Binary Search example in Section 9.5, why must the Direction flag be cleared by the CLD instruction?

* **번역**: 9.5절 이진 탐색 예제의 `FillArray` 프로시저에서 `CLD` 명령어로 Direction Flag를 반드시 클리어해야 하는 이유는 무엇인가?
* **정답**: **`STOSD` 명령어가 배열의 시작 주소에서부터 정방향(주소 증가 방향)으로 난수를 순서대로 채워 넣도록 보장하기 위해서**
* **해설 및 코드 디테일**: `FillArray`가 호출되기 전 다른 프로시저에서 `STD`를 실행해 `DF = 1` 상태로 남겨두었을 가능성이 있습니다. 만약 `CLD` 없이 `stosd`를 실행하면 배열 시작 주소 이전의 엉뚱한 메모리 영역을 덮어쓰는 치명적인 버그가 발생합니다.
  ```assembly
  cld                     ; 이전 코드의 DF 상태를 믿지 않고 반드시 정방향(0) 보장!
  mov edi, pArray         ; 배열의 첫 번째 원소 주소
  ; ... 루프 내에서 stosd 실행 시 EDI가 +4씩 안전하게 증가 ...
  ```

#### 13. In the BinarySearch procedure from Section 9.5, why could the statement at label L2 be removed without affecting the outcome?

* **번역**: 9.5절 `BinarySearch` 프로시저에서 라벨 `L2`에 있는 명령문을 제거해도 실행 결과에 아무 영향이 없는 이유는 무엇인가?
* **정답**: **직전 명령어(비교/분기)에서 이미 조건이 걸러졌기 때문에 `L2` 위치로 코드가 흘러 내려왔을 때는 해당 조건이 항상 참(불필요한 중복 검사/점프)이기 때문**
* **해설 및 코드 디테일**: `BinarySearch` 루프에서 중간값 `EDX`와 찾는 값 `searchVal`을 `cmp edx, searchVal`로 비교한 뒤, `jl L2`(작은 경우)와 `jg L3`(큰 경우)로 분기할 때, 두 분기 중 하나가 바로 다음 줄에 위치한다면 굳이 자기 바로 아래 줄로 `jmp`하거나 중복 조건을 검사할 필요 없이 자연스럽게 다음 명령어로 흘러내려가면(Fall-through) 됩니다.

#### 14. In the BinarySearch procedure from Section 9.5, how might the statement at label L4 be eliminated?

* **번역**: 9.5절 `BinarySearch` 프로시저에서 라벨 `L4`에 있는 명령문을 어떻게 없앨 수 있는가?
* **정답**: **루프 종료 조건이나 앞선 분기문의 목적지 라벨을 수정하여 `L4`를 거치지 않고 루프 시작점(`L1`)으로 직접 점프하도록 만들면 제거할 수 있다.**
* **해설 및 코드 디테일**: 교재 코드에서 `L4`는 단순히 `jmp L1` 하나만 가지고 있는 징검다리 라벨입니다. 따라서 `first = mid + 1` 처리 후 `L4`로 떨어져 `jmp L1`을 타는 대신, 각 분기 블록의 끝에서 직접 `jmp L1`을 수행하도록 수정하면 불필요한 라벨 `L4`와 점프 단계를 줄여 코드를 최적화할 수 있습니다.
  ```assembly
  ; [기존 방식]
  ; ... last = mid - 1 처리 ...
  ; jmp L4          <-- 불필요한 징검다리 점프
  ; L4: jmp L1

  ; [개선 방식: L4 제거]
  ; ... last = mid - 1 처리 ...
  jmp L1            ; 루프 시작점(L1)으로 직접 점프하여 L4 제거!
  ```