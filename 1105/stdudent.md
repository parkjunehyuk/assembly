# 📘 어셈블리어 학번 & 이름 출력 프로그램 완전 입문 가이드

x86 어셈블리 언어(Irvine32 라이브러리)를 이용해 화면에 학번과 이름을 출력하는 기초 프로그램 노트입니다.

## 📌 1. 전체 소스 코드

```assembly
INCLUDE Irvine32.inc

; Windows API 함수 선언 (콘솔 인코딩 변경용)
SetConsoleOutputCP PROTO STDCALL :DWORD

.data
    studentID   BYTE "학번: 20261234", 0Dh, 0Ah, 0
    studentName BYTE "이름: 홍길동", 0Dh, 0Ah, 0

.code
main PROC
    ; 콘솔 출력을 UTF-8(65001)로 설정하여 한글 깨짐 방지
    INVOKE SetConsoleOutputCP, 65001

    ; 1. 학번 출력
    mov edx, OFFSET studentID   ; studentID가 저장된 메모리 주소를 EDX에 저장
    call WriteString            ; EDX가 가리키는 주소의 문자열을 화면에 출력

    ; 2. 이름 출력
    mov edx, OFFSET studentName ; studentName이 저장된 메모리 주소를 EDX에 저장
    call WriteString            ; EDX가 가리키는 주소의 문자열을 화면에 출력

    exit                        ; 프로그램 종료
main ENDP

END main
```

## 📌 2. 핵심 어셈블리 개념 해설

### ① 왜 `call WriteString` 줄에는 `EDX`라는 말이 없을까? (어바인 교수님의 비밀 약속)

코드의 `call WriteString`에는 `EDX`라는 단어가 어디에도 보이지 않아 헷갈릴 수 있습니다. 하지만 이 명령어는 무조건 `EDX`에 들어있는 주소를 읽어서 출력합니다.

* **어바인(Kip Irvine) 교수님의 약속:**  
  교재 라이브러리를 만드신 어바인 교수님이 `WriteString` 함수를 제작할 때 내부에 **"누군가 나를 부르면(`call WriteString`), 나는 무조건 `EDX` 레지스터 쟁반부터 먼저 열어볼 거야!"**라는 규칙을 고정으로 작성해 두셨기 때문입니다.
* **사용법:**  
  따라서 우리는 `call WriteString`을 부르기 전에 반드시 **`mov edx, OFFSET 변수명`**을 통해 출력하고 싶은 문자열의 메모리 주소를 `EDX` 쟁반에 미리 준비해 두어야 합니다.

---

### ② `EDX` 레지스터와 `OFFSET`

* **`EDX` 레지스터:** 문자열 출력 명령(`WriteString`)을 실행할 때 출력할 데이터가 위치한 **메모리 주소를 보관하는 주소 전용 바구니**입니다.
* **`OFFSET`:** 변수 이름 앞에 붙어서 실제 글자 데이터가 아닌 "변수가 저장된 메모리 주소 번지수"를 뜻합니다.
* **작동 원리:** `mov edx, OFFSET studentID`를 통해 `studentID`의 주소를 `EDX`에 넣고 `call WriteString`을 부르면, `WriteString` 내부에서 `EDX` 주소를 찾아가 글자들을 출력합니다.

---

### ③ 특수 제어 문자의 의미 (`0Dh, 0Ah, 0`)

| 특수 문자 | 정식 명칭 | 역할 및 비유 |
| :--- | :--- | :--- |
| **`0Dh`** | Carriage Return (`CR`) | 커서를 줄의 **맨 왼쪽 끝으로 이동**시킵니다. (ASCII 13) |
| **`0Ah`** | Line Feed (`LF`) | 커서를 **한 줄 아래로 내려줍니다.** (ASCII 10) |
| **`0Dh, 0Ah`** | CRLF | 타자기의 원리로 **[맨 왼쪽 이동 + 한 줄 아래]**가 합쳐져 완벽한 **Enter (줄바꿈)** 역할을 합니다. |
| **`, 0`** | NULL Termination | `WriteString` 함수에게 **"여기서 문자열이 끝났다"**는 지점을 알려주는 마침표 역할을 합니다. |

---

### ④ 한글 깨짐 방지 (`SetConsoleOutputCP`)

* Windows 콘솔(cmd) 창은 기본 인코딩(CP949)을 사용하는 경우가 많아 UTF-8 기반으로 작성된 한글 코드가 깨져 보일 수 있습니다.
* `INVOKE SetConsoleOutputCP, 65001` 구문을 실행하여 콘솔 인코딩을 UTF-8 (코드 페이지 65001)로 즉시 변경하면 한글이 깨지지 않고 정상 출력됩니다.

---

## 📌 3. 한 줄 명령어 종합 해설표

| 코드 라인 | 명령어 해석 | 초보자용 기초 풀이 |
| :--- | :--- | :--- |
| **`SetConsoleOutputCP PROTO ...`** | Windows API 함수 선언 | 콘솔 인코딩 변경 함수를 사용하기 위한 사전 선언 |
| **`.data`** | Data Segment | 변수를 선언하는 메모리 영역 시작 지정 |
| **`studentID BYTE "...", 0`** | 바이트 배열 변수 선언 | 문자열 데이터와 줄바꿈(`0Dh, 0Ah`), 종료 기호(`0`) 저장 |
| **`.code`** | Code Segment | 실제로 실행될 어셈블리 명령어가 들어가는 영역 |
| **`INVOKE SetConsoleOutputCP, 65001`** | API 함수 호출 | 콘솔 화면 출력을 UTF-8(65001) 인코딩으로 설정 |
| **`mov edx, OFFSET studentID`** | `EDX = &studentID` | `studentID`의 메모리 **주소**를 `EDX` 바구니에 보관 |
| **`call WriteString`** | `WriteString()` 실행 | `EDX` 주소로 찾아가 `0`을 만날 때까지 문자를 화면에 출력 |
| **`exit`** | `ExitProcess(0)` | 프로그램을 정상 종료하고 OS로 제어권 반환 |