# 🛠️ 8.10.2 Algorithm Workbench 초쉬운 풀이 가이드

어셈블리 언어를 처음 접하는 입문자의 눈높이에 맞춰 **8.10.2 Algorithm Workbench (1번~11번)**의 모든 문제와 정답 코드를 실생활 비유 및 쉬운 해설로 풀었습니다.

---

### 1. 📦 스택에 상자 쌓기 (스택 프레임 그림)

**문제:** `push 10h`, `push 20h`, `push 30h`, `call AddThree`를 실행하고 함수 내부 첫 줄인 `push ebp`까지 실행되었을 때 스택 메모리의 상태를 그림으로 나타내시오.

#### 📍 메모리 구조도 (스택 상자 쌓기)

```
        +-----------------------+ 
        |        10h            |  [EBP + 16] 👈 3번째 넣은 재료 (Param 3)
        +-----------------------+ 
        |        20h            |  [EBP + 12] 👈 2번째 넣은 재료 (Param 2)
        +-----------------------+ 
        |        30h            |  [EBP + 8]  👈 1번째 넣은 재료 (Param 1)
        +-----------------------+ 
        |    Return Address     |  [EBP + 4]  👈 CALL 명령어가 찍어둔 "돌아갈 집 주소"
        +-----------------------+ 
 ESP -> |       Saved EBP       |  [EBP]      👈 이전 주인의 기준점 깃발 (push ebp)
        +-----------------------+ <-- EBP가 가리키는 고정 깃발 위치
```

* **💡 한 줄 비유 설명**:
  1. `STDCALL` 규약은 오른쪽 숫점부터 차곡차곡 쌓기 때문에 `10h` ➔ `20h` ➔ `30h` 순으로 상자가 쌓입니다.
  2. `CALL`이 실행되면 일을 마치고 복귀할 **'돌아갈 위치 주소'**가 상자 위에 올라갑니다.
  3. 함수 안에 들어와서 `push ebp`를 실행하면 원래 일하던 사람의 기준점(Saved EBP)을 맨 위에 얹고, 그 위치에 현재 기준 깃발(`EBP`)을 꽂습니다.

---

### 2. ➕ 세 숫자 더하기 함수 (`AddThree`)

**문제:** 정수 3개를 매개변수로 받아 모두 더한 뒤 결과값을 `EAX`에 담아 반환하는 `AddThree` 함수를 작성하시오.

#### 💻 정답 코드

```assembly
AddThree PROC,
    val1:SDWORD,    ; 첫 번째 정수
    val2:SDWORD,    ; 두 번째 정수
    val3:SDWORD     ; 세 번째 정수

    mov eax, val1   ; EAX = val1
    add eax, val2   ; EAX += val2
    add eax, val3   ; EAX += val3
    ret             ; EAX에 최종 결과값을 가지고 돌아감 (STDCALL 청소 자동)
AddThree ENDP
```

* **💡 한 줄 비유 설명**:
  결과물은 컴퓨터 세상의 **공식 반환 상자인 `EAX`**에 모아 담기만 하면 됩니다.

---

### 3 ~ 7. 🗄️ 지역 변수 선언하기 (`LOCAL` 사용법)

함수 안에서만 잠시 쓰고 버릴 '임시 변수(지역 변수)'를 만들어내는 지시어 `LOCAL` 사용법입니다.

| 문제 번호 | 요구 내용 | MASM 작성 코드 | 💡 쉽게 이해하기 |
|:---:|:---|:---|:---|
| **3** | DWORD 배열 포인터 `pArray` | `LOCAL pArray:PTR DWORD` | 32비트 배열의 **"집 주소"**를 보관할 상자 |
| **4** | 20바이트 크기의 배열 `buffer` | `LOCAL buffer[20]:BYTE` | 글자 20개를 담을 수 있는 **"20칸짜리 서랍"** |
| **5** | 16비트 정수 포인터 `pwArray` | `LOCAL pwArray:PTR WORD` | 16비트 숫자가 든 **"집 주소"**를 보관할 상자 |
| **6** | 8비트 부호 있는 정수 `myByte` | `LOCAL myByte:SBYTE` | -128~127 범위의 소형 정수 1개 보관함 |
| **7** | 20개의 DWORD 정수 배열 `myArray` | `LOCAL myArray[20]:DWORD` | 32비트 큰 숫자 20개를 꽂는 **"큰 서랍장"** |

---

### 8. 🎨 텍스트 색상 변경하기 (`SetColor`)

**문제:** 전경색(`forecolor`)과 배경색(`backcolor`)을 매개변수로 받아 라이브러리의 `SetTextColor`를 호출해 주는 함수를 작성하시오.

#### 💻 정답 코드

```assembly
SetColor PROC,
    forecolor:BYTE,
    backcolor:BYTE

    mov  al, backcolor     ; 배경색을 AL에 넣음
    shl  al, 4             ; 배경색을 상위 4비트로 밀어 올림 (* 16 효과)
    add  al, forecolor     ; 하위 4비트에 전경색을 합침
    movzx eax, al          ; EAX 레지스터 크기(32비트)로 깔끔하게 맞춤
    call SetTextColor      ; 색상 적용 라이브러리 함수 호출
    ret
SetColor ENDP
```

* **💡 한 줄 비유 설명**:
  컴퓨터는 색상을 바꿀 때 `(배경색 × 16) + 전경색` 조합을 원합니다. 배경색을 4비트 왼쪽으로 밀어(`shl al, 4`) 상위 칸에 두고, 하위 칸에 전경색을 더해 결합한 뒤 색상 함수를 부릅니다.

---

### 9. 🔤 색상을 입혀 문자 1개 출력하기 (`WriteColorChar`)

**문제:** 문자 1개, 전경색, 배경색을 받아서 해당 색상으로 문자를 출력하는 함수를 작성하시오.

#### 💻 정답 코드

```assembly
WriteColorChar PROC,
    char:BYTE,
    forecolor:BYTE,
    backcolor:BYTE

    ; 1. 8번에서 만든 SetColor를 불러 색상을 먼저 칠함
    INVOKE SetColor, forecolor, backcolor

    ; 2. 출력할 문자를 AL에 담아 WriteChar 실행
    mov  al, char
    call WriteChar
    ret
WriteColorChar ENDP
```

* **💡 한 줄 비유 설명**:
  앞서 만든 `SetColor`를 재사용해서 도화지 색을 먼저 세팅한 뒤, `AL` 레지스터에 출력할 문자를 넣고 찍어내는 방식입니다.

---

### 10. 💾 메모리 내용 덤프(출력)하기 (`DumpMemory`)

**문제:** Irvine32의 `DumpMem` 함수를 감싸는 `DumpMemory` 프로시저를 작성하시오. (단, 기존 레지스터 보호를 위해 `USES` 지시어를 사용할 것)

#### 💻 정답 코드

```assembly
DumpMemory PROC USES esi ebx ecx,   ; 사용 전 ESI, EBX, ECX를 백업(PUSH)하고 나갈 때 복원(POP)해 줌!
    address:PTR DWORD,
    elementCount:DWORD,
    elementSize:DWORD

    mov esi, address       ; 시작 주소를 ESI에 넣음
    mov ecx, elementCount  ; 출력할 개수를 ECX에 넣음
    mov ebx, elementSize   ; 단위 크기(1, 2, 4바이트)를 EBX에 넣음
    call DumpMem           ; 메모리 덤프 출력 함수 실행
    ret
DumpMemory ENDP
```

* **💡 한 줄 비유 설명**:
  `USES` 구문은 일하러 들어가기 전 **"기존 레지스터 값 보존용 임시 락커"**입니다. 내 코드가 `ESI`, `EBX`, `ECX`를 써도 외부 코드에 영향이 가지 않도록 백업 및 복원을 자동 처리합니다.

---

### 11. 📋 배열 곱셈 프로시저 및 선언부 (`MultArray`)

**문제:** 두 개의 DWORD 배열 주소 포인터와 배열 크기를 받는 `MultArray` 프로시저 정의와 `PROTO`(프로토타입) 선언문을 작성하시오.

#### 💻 정답 코드

```assembly
; 1. 미리 함수 모양을 알려주는 PROTO (선언부)
MultArray PROTO,
    ptrArray1:PTR DWORD,
    ptrArray2:PTR DWORD,
    count:DWORD

.code
; 2. 실제 일을 하는 본체 (정의부)
MultArray PROC,
    ptrArray1:PTR DWORD,
    ptrArray2:PTR DWORD,
    count:DWORD

    ; 배열 곱셈 계산 로직 위치
    ret
MultArray ENDP
```

* **💡 한 줄 비유 설명**:
  `PROTO`는 C언어의 `header` 선언처럼 "이 함수는 이렇게 생겼다"라고 컴파일러에게 미리 명함을 내미는 서식입니다.