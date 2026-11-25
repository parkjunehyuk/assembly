# Assembly Language Chapter 10.7.2 Algorithm Workbench 해설

이 문서는 Kip Irvine의 *Assembly Language for x86 Processors* 10장(Structures and Macros)의 **10.7.2 Algorithm Workbench** 16문제에 대한 원문, 한국어 번역, 정답 코드 및 상세 풀이, 그리고 **초보자를 위한 핵심 문법 가이드**를 정리한 실전 코딩 노트입니다.

## 📌 시작 전 필수 핵심 문법 가이드 (Algorithm Workbench용)

코딩 문제(1\~16번)를 풀기 전, 책의 라이브러리 매크로와 필수 연산자들을 먼저 훑어보세요!

### 1. Irvine32 라이브러리 핵심 함수 & 매크로 요약

문제에서 자주 등장하는 Irvine 교재 내장 프로시저(`call`)와 매크로(`m...`)의 역할입니다.

| 이름 | 종류 | 역할 | 사용하는 레지스터 / 인수 | 
| ----- | ----- | ----- | ----- | 
| **`WriteChar`** | 프로시저 (`call`) | 문자 1개 화면 출력 | `AL` = 출력할 문자 | 
| **`RandomRange`** | 프로시저 (`call`) | $0 \sim (\text{EAX}-1)$ 난수 생성 | 입력: `EAX` (범위 $n$), 출력: `EAX` (생성된 난수) | 
| **`ReadInt`** | 프로시저 (`call`) | 키보드로 부호 있는 정수 입력받기 | 출력: `EAX` (입력받은 숫자) | 
| **`mWrite`** | 매크로 | 문자열 리터럴 화면 출력 | `mWrite "Hello"` | 
| **`mWriteStr`** | 매크로 | 문자열 **변수** 화면 출력 | `mWriteStr myStrVar` | 
| **`mReadStr`** | 매크로 | 문자열 입력받아 변수에 저장 | `mReadStr bufferVar` | 
| **`mGotoxy`** | 매크로 | 콘솔 커서 위치 이동 | `mGotoxy x좌표, y좌표` (`DL`=X, `DH`=Y) | 
| **`mDumpMem`** | 매크로 | 메모리 내용을 16진수로 출력 | `mDumpMem 시작주소, 개수, 단위크기` | 

### 2. 변수 크기와 주소를 알아내는 3대 연산자

배열이나 구조체를 다룰 때 숫자 대신 아래 연산자를 쓰면 어셈블러가 알아서 크기를 계산해 줍니다.

* **`OFFSET 변수명`**: 변수가 시작하는 **메모리 주소**를 가져옵니다.

* **`TYPE 변수명`**: 변수 원소 1개의 크기(바이트)를 알려줍니다. (`BYTE`=1, `WORD`=2, `DWORD`=4, `Triangle`=12)

* **`LENGTHOF 변수명`**: 배열에 들어있는 **원소의 총 개수**를 알려줍니다.

### 3. 중첩 구조체(구조체 안의 구조체) 점(`.`) 접근법

큰 상자(`Triangle`) 안에 작은 상자(`COORD`)가 들어있을 때는 점(`.`)을 두 번 찍어서 안으로 들어갑니다.

* `myTri.Vertex1.X` 👉 `myTri` 상자 안의 $\rightarrow$ `Vertex1` 상자 안의 $\rightarrow$ `X` 값!

## 💻 10.7.2 Algorithm Workbench 문제 및 해설

### 1. Create a structure named `SampleStruct` containing two fields: `field1`, a single 16-bit `WORD`, and `field2`, an array of 20 32-bit `DWORD`s. The initial values of the fields may be left undefined.

* **한국어 번역:** 두 개의 필드를 포함하는 `SampleStruct`라는 이름의 구조체를 작성하시오: 첫 번째는 16비트 `WORD` 1개인 `field1`이고, 두 번째는 32비트 `DWORD` 20개로 이루어진 배열 `field2`입니다. 필드의 초기값은 정의되지 않은 상태(`?`)로 두어도 됩니다.

* **정답 코드:**

  ```
  SampleStruct STRUCT
      field1 WORD ?
      field2 DWORD 20 DUP(?)
  SampleStruct ENDS
  
  ```

* **상세 풀이:**

  * 구조체는 `[구조체명] STRUCT`로 시작해서 `[구조체명] ENDS`로 끝납니다.

  * 16비트는 `WORD` (2바이트), 32비트는 `DWORD` (4바이트)입니다.

  * 20개짜리 배열을 초기화 없이  공간만 잡을 때는 `20 DUP(?)`를 사용합니다.

> **💡 쉬운 용어 & 문법 돋보기**
>
> * **`20 DUP(?)`:** "비어 있는(`?`) `DWORD` 상자 20개를 연속으로 복제(`DUP`)해서 배열을 만들어라"라는 뜻입니다. C언어의 `int field2[20];`와 같습니다.

### 2. Write a statement that retrieves the `wHour` field of a `SYSTEMTIME` structure.

* **한국어 번역:** `SYSTEMTIME` 구조체의 `wHour` 필드를 가져오는(조회하는) 명령문을 작성하시오.

* **정답 코드:**

  ```
  .data
  sysTime SYSTEMTIME <>
  
  .code
  mov ax, sysTime.wHour
  
  ```

* **상세 풀이:**

  * 윈도우 API에 내장된 `SYSTEMTIME` 구조체 내부의 `wHour`(현재 시간) 필드는 `WORD`(16비트) 크기입니다.

  * 따라서 16비트 레지스터인 `AX` (또는 `BX`, `CX`, `DX` 등)로 값을 복사(`MOV`)해 옵니다. (단, 먼저 `SYSTEMTIME` 타입의 변수 `sysTime`이 선언되어 있어야 합니다.)

> **💡 쉬운 용어 & 문법 돋보기**
>
> * **왜 `EAX`가 아니라 `AX`를 쓰나요?** `SYSTEMTIME`의 모든 시간/날짜 필드(`wYear`, `wMonth`, `wHour` 등)는 앞에 `w`가 붙어있는데, 이는 **W**ORD(16비트) 타입이라는 뜻입니다. 16비트 데이터는 16비트 그릇(`AX`)에 담아야 크기가 딱 맞습니다!

### 3. Using the following `Triangle` structure, declare a structure variable and initialize its vertices to `(0,0)`, `(5, 0)`, and `(7,6)`:

```
Triangle STRUCT
    Vertex1 COORD <>
    Vertex2 COORD <>
    Vertex3 COORD <>
Triangle ENDS

```

* **한국어 번역:** 위의 `Triangle` 구조체를 사용하여 구조체 변수를 선언하고, 각 꼭짓점(vertices)을 `(0,0)`, `(5, 0)`, `(7,6)`으로 초기화하시오.

* **정답 코드:**

  ```
  .data
  myTriangle Triangle < <0,0>, <5,0>, <7,6> >
  
  ```

* **상세 풀이:**

  * `Triangle` 구조체 안에는 이미 구조체인 `COORD`(X, Y 좌표를 가진 구조체) 3개가 들어있습니다.

  * 구조체 안의 구조체를 초기화할 때는 바깥쪽 꺾쇠 `< >` 안에, 각 `COORD`를 초기화하는 안쪽 꺾쇠 `<X, Y>`를 쉼표로 구분해 넣어줍니다.

> **💡 쉬운 용어 & 문법 돋보기**
>
> * **중첩 꺾쇠 `< <0,0>, <5,0>, <7,6> >`:** 큰 선물 상자(`Triangle`) 안에 작은 보석 상자(`COORD`) 3개가 들어있는 모양을 그대로 기호로 표현한 것입니다.

### 4. Declare an array of `Triangle` structures. Write a loop that initializes `Vertex1` of each triangle to random coordinates in the range `(0...10, 0...10)`.

* **한국어 번역:** `Triangle` 구조체의 배열을 선언하시오. 그리고 반복문(Loop)을 작성하여 각 삼각형의 `Vertex1`을 `(0...10, 0...10)` 범위의 난수(랜덤 좌표)로 초기화하시오.

* **정답 코드:**

  ```
  NUM_TRIANGLES = 5               ; 예시로 삼각형 5개 배열 선언
  
  .data
  triArray Triangle NUM_TRIANGLES DUP(<>)
  
  .code
  mov ecx, LENGTHOF triArray      ; 반복 횟수 설정 (5번)
  mov esi, 0                      ; 배열 인덱스(바이트 오프셋) 초기화
  
  L1:
      ; X 좌표 난수 생성 (0 ~ 10)
      mov eax, 11                 ; RandomRange는 0 ~ (EAX-1) 범위 생성
      call RandomRange            ; EAX에 0~10 난수 반환
      mov triArray[esi].Vertex1.X, ax
  
      ; Y 좌표 난수 생성 (0 ~ 10)
      mov eax, 11
      call RandomRange
      mov triArray[esi].Vertex1.Y, ax
  
      add esi, TYPE Triangle      ; 다음 Triangle 구조체 위치로 이동 (+12바이트)
      loop L1
  
  ```

* **상세 풀이:**

  * `RandomRange` 프로시저는 $0$부터 $\text{EAX}-1$까지의 난수를 생성하므로, $0 \sim 10$을 얻으려면 `EAX`에 `11`을 넣어야 합니다.

  * `COORD`의 `X`와 `Y`는 `SWORD`(16비트)이므로 난수 결과인 `EAX`의 하위 16비트인 `AX`를 저장합니다.

  * 배열의 다음 원소로 이동할 때는 `TYPE Triangle`만큼 `ESI`를 더해줍니다. (`COORD` 1개가 4바이트이므로 `Triangle` 1개는 $4 \times 3 = 12$바이트)

### 5. Write a macro named `mPrintChar` that displays a single character on the screen. It should have two parameters: the first specifies the character to be displayed and the second specifies how many times the character should be repeated. Here is a sample call: `mPrintChar 'X',20`

* **한국어 번역:** 화면에 단일 문자를 출력하는 `mPrintChar`라는 매크로를 작성하시오. 두 개의 매개변수를 가져야 합니다: 첫 번째는 출력할 문자를 지정하고, 두 번째는 문자를 몇 번 반복할지 지정합니다. 호출 예시는 다음과 같습니다: `mPrintChar 'X',20`

* **정답 코드:**

  ```
  mPrintChar MACRO char, count
      LOCAL L1
      push eax
      push ecx
  
      mov al, char        ; 출력할 문자를 AL에 저장
      mov ecx, count      ; 반복 횟수를 ECX에 저장
  L1:
      call WriteChar      ; AL의 문자 1개 출력
      loop L1
  
      pop ecx
      pop eax
  ENDM
  
  ```

* **상세 풀이:**

  * 매크로 내부에서 `L1`이라는 반복 레이블을 사용하므로, 매크로를 여러 번 호출해도 이름이 겹치지 않도록 반드시 `LOCAL L1`을 선언해야 합니다.

  * 매크로 실행 전후로 기존 레지스터 값이 망가지지 않도록 `push` / `pop`으로 `EAX`와 `ECX`를 보호해 주는 것이 좋은 습관입니다.

### 6. Write a macro named `mGenRandom` that generates a random integer between $0$ and $n - 1$. Let `n` be the only parameter.

* **한국어 번역:** $0$과 $n - 1$ 사이의 랜덤 정수를 생성하는 `mGenRandom`이라는 매크로를 작성하시오. `n`을 유일한 매개변수로 사용하시오.

* **정답 코드:**

  ```
  mGenRandom MACRO n
      mov eax, n
      call RandomRange
  ENDM
  
  ```

* **상세 풀이:**

  * Irvine32 라이브러리의 `RandomRange` 함수는 `EAX`에 들어있는 값 `n`을 기준으로 $0 \sim (n-1)$ 사이의 난수를 생성하여 다시 `EAX`에 담아 돌려줍니다.

### 7. Write a macro named `mPromptInteger` that displays a prompt and inputs an integer from the user. Pass it a string literal and the name of a doubleword variable.

```
.data
minVal DWORD ?
.code
mPromptInteger "Enter the minimum value", minVal

```

* **한국어 번역:** 안내 문구(프롬프트)를 화면에 출력하고 사용자로부터 정수를 입력받는 `mPromptInteger` 매크로를 작성하시오. 문자열 리터럴과 더블워드(`DWORD`) 변수의 이름을 인수로 전달하시오.

* **정답 코드:**

  ```
  mPromptInteger MACRO promptString, intVal
      mWrite promptString     ; 안내 문자열 출력 (책의 매크로 활용)
      call ReadInt            ; 키보드로 정수 입력받아 EAX에 저장
      mov intVal, eax         ; 입력받은 정수를 변수에 저장
  ENDM
  
  ```

* **상세 풀이:**

  * 문자열 리터럴(`"Enter the minimum value"`)을 직접 출력하기 위해 책의 `mWrite` 매크로를 내부에서 호출합니다.

  * `ReadInt` 프로시저를 호출하면 사용자가 입력한 정수가 `EAX` 레지스터에 저장되므로, 이를 두 번째 매개변수인 `intVal` 변수로 옮겨줍니다.

### 8. Write a macro named `mWriteAt` that locates the cursor and writes a string literal to the console window. Suggestion: Invoke the `mGotoxy` and `mWrite` macros from the book’s macro library.

* **한국어 번역:** 커서의 위치를 이동시키고 콘솔 창에 문자열 리터럴을 출력하는 `mWriteAt` 매크로를 작성하시오. (제안: 책의 매크로 라이브러리에 있는 `mGotoxy`와 `mWrite` 매크로를 호출하시오.)

* **정답 코드:**

  ```
  mWriteAt MACRO xPos, yPos, literalStr
      mGotoxy xPos, yPos
      mWrite literalStr
  ENDM
  
  ```

* **상세 풀이:**

  * X좌표(`xPos`), Y좌표(`yPos`), 그리고 출력할 문자열(`literalStr`) 총 3개의 매개변수를 받아 기존 매크로 2개를 순서대로 연결해 주는 래퍼(Wrapper) 매크로입니다.

### 9. Show the expanded code produced by the following statement that invokes the `mWriteString` macro from Section 10.2.5: `mWriteStr namePrompt`

* **한국어 번역:** 10.2.5절의 `mWriteString`(`mWriteStr`) 매크로를 호출하는 `mWriteStr namePrompt` 문장에 의해 생성되는 확장된(번역된) 코드를 보이시오.

* **정답 코드:**

  ```
  push edx
  mov edx, OFFSET namePrompt
  call WriteString
  pop edx
  
  ```

* **상세 풀이:**

  * 교재의 `mWriteStr` 매크로 정의는 다음과 같습니다:

    ```
    mWriteStr MACRO buffer
        push edx
        mov edx, OFFSET buffer
        call WriteString
        pop edx
    ENDM
    
    ```

  * 어셈블러는 매크로 호출부를 위 코드로 그대로 치환하며, 매개변수 `buffer` 자리에 인수 `namePrompt`를 끼워 넣습니다.

### 10. Show the expanded code produced by the following statement that invokes the `mReadString` macro from Section 10.2.5: `mReadStr customerName`

* **한국어 번역:** 10.2.5절의 `mReadString`(`mReadStr`) 매크로를 호출하는 `mReadStr customerName` 문장에 의해 생성되는 확장된 코드를 보이시오.

* **정답 코드:**

  ```
  push ecx
  push edx
  mov edx, OFFSET customerName
  mov ecx, SIZEOF customerName
  call ReadString
  pop edx
  pop ecx
  
  ```

* **상세 풀이:**

  * `ReadString` 프로시저는 버퍼의 시작 주소를 `EDX`에, 버퍼의 최대 크기(`SIZEOF`)를 `ECX`에 필요로 합니다. `mReadStr` 매크로는 이를 자동으로 세팅한 후 호출하고 레지스터를 복구합니다.

### 11. Write a macro named `mDumpMemx` that receives a single parameter, the name of a variable. Your macro must call the `mDumpMem` macro from the book’s library, passing it the variable’s offset, number of units, and unit size. Demonstrate a call to the `mDumpMemx` macro.

* **한국어 번역:** 변수의 이름이라는 단일 매개변수를 받는 `mDumpMemx`라는 매크로를 작성하시오. 이 매크로는 책의 라이브러리에 있는 `mDumpMem` 매크로를 호출해야 하며, 변수의 오프셋(주소), 원소 개수, 그리고 원소 단위 크기를 전달해야 합니다. `mDumpMemx` 매크로를 호출하는 예시도 보여주시오.

* **정답 코드:**

  ```
  ; 매크로 정의
  mDumpMemx MACRO varName
      mDumpMem OFFSET varName, LENGTHOF varName, TYPE varName
  ENDM
  
  ; 호출 예시
  .data
  myArray DWORD 10, 20, 30, 40
  
  .code
  mDumpMemx myArray
  
  ```

* **상세 풀이:**

  * 변수의 이름만 넘겨받으면 `OFFSET`(시작 주소), `LENGTHOF`(원소 개수: 4), `TYPE`(단위 크기: 4바이트) 연산자를 이용해 `mDumpMem`이 요구하는 3개의 인수를 자동으로 채워 넣을 수 있습니다.

### 12. Show an example of a macro parameter having a default argument initializer.

* **한국어 번역:** 기본 인수 초기화 값(Default argument initializer)을 가지는 매크로 매개변수의 예시를 작성하시오.

* **정답 코드:**

  ```
  mPrintLine MACRO count:=<80>
      mov ecx, count
      ; ... (count번 만큼 선을 그리는 코드) ...
  ENDM
  
  ```

* **상세 풀이:**

  * 매크로 매개변수 뒤에 `:=<기본값>`을 붙이면, 매크로를 호출할 때 인수를 생략하더라도 자동으로 기본값(`80`)이 사용됩니다.

  * 예: `mPrintLine` (인수 없이 호출하면 `count`는 `80`이 됨), `mPrintLine 40` (인수를 주면 `40`이 됨)

### 13. Write a short example that uses the `IF`, `ELSE`, and `ENDIF` directives.

* **한국어 번역:** `IF`, `ELSE`, `ENDIF` 지시어를 사용하는 짧은 예시를 작성하시오.

* **정답 코드:**

  ```
  mSetRegister MACRO val
      IF val EQ 0
          xor eax, eax        ; val이 0이면 빠르게 0으로 초기화
      ELSE
          mov eax, val        ; 0이 아니면 해당 값을 대입
      ENDIF
  ENDM
  
  ```

* **상세 풀이:**

  * 조건부 어셈블리 지시어(`IF`, `ELSE`, `ENDIF`)는 프로그램 실행 중이 아니라 **코드를 번역(어셈블)하는 시점**에 상수 조건을 검사하여 어떤 기계어 코드를 생성할지 결정합니다.

### 14. Write a statement using the `IF` directive that checks the value of the constant macro parameter `Z`; if `Z` is less than zero, display a message during assembly indicating that `Z` is invalid.

* **한국어 번역:** `IF` 지시어를 사용하여 상수 매크로 매개변수 `Z`의 값을 확인하는 문장을 작성하시오; 만약 `Z`가 $0$보다 작으면, 어셈블(번역) 도중에 `Z`가 유효하지 않다는 메시지를 화면에 출력하시오.

* **정답 코드:**

  ```
  IF Z LT 0
      ECHO Error: Parameter Z is invalid (cannot be less than zero).
  ENDIF
  
  ```

* **상세 풀이:**

  * "$0$보다 작다($<$)"는 관계 연산자 **`LT`**(Less Than)를 사용합니다.

  * 어셈블(빌드) 도중 콘솔에 메시지를 출력하는 지시어는 `ECHO`입니다.

### 15. Write a short macro that demonstrates the use of the `&` operator when the macro parameter is embedded in a literal string.

* **한국어 번역:** 매크로 매개변수가 문자열 리터럴(따옴표 안)에 포함되어 있을 때 `&` 연산자를 사용하는 짧은 매크로 예시를 작성하시오.

* **정답 코드:**

  ```
  mShowVar MACRO varName
      .data
      msg&varName BYTE "The value of &varName is: ", 0
      .code
      mov edx, OFFSET msg&varName
      call WriteString
  ENDM
  
  ```

* **상세 풀이:**

  * 따옴표(`"..."`) 안에 있는 글자는 원래 매개변수로 인식되지 않고 단순 문자열로 취급됩니다.

  * 하지만 앞에 **`&`**(치환 연산자)를 붙여 `"The value of &varName is: "`처럼 쓰면, 호출 시 넘겨준 실제 변수 이름으로 치환됩니다. (예: `mShowVar count` 호출 시 `"The value of count is: "`로 변환됨)

### 16. Assume the following `mLocate` macro definition:

```
mLocate MACRO xval,yval
    IF xval LT 0        ;; xval < 0?
        EXITM           ;; if so, exit
    ENDIF
    IF yval LT 0        ;; yval < 0?
        EXITM           ;; if so, exit
    ENDIF
    mov bx,0            ;; video page 0
    mov ah,2            ;; locate cursor
    mov dh,yval
    mov dl,xval
    int 10h             ;; call the BIOS
ENDM

```

> Show the source code generated by the preprocessor when the macro is expanded by each of the following statements:
>
> ```
> .data
> row BYTE 15
> col BYTE 60
> .code
> mLocate -2,20
> mLocate 10,20
> mLocate col,row
> 
> ```

* **한국어 번역:** 위와 같은 `mLocate` 매크로 정의가 주어졌을 때, 아래 3개의 호출문에 의해 전처리기(Preprocessor)가 확장하여 생성하는 소스 코드를 각각 보이시오.

* **정답 및 상세 풀이:**

  * **① `mLocate -2, 20` 의 확장 결과:**

    ```
    ; (생성되는 코드 없음 - 아무것도 출력되지 않음)
    
    ```

    * **이유:** `xval`이 `-2`이므로 첫 번째 조건 `IF -2 LT 0`이 참(True)이 됩니다. 즉시 **`EXITM`**(Exit Macro) 지시어가 실행되어 매크로 확장이 중단되므로, 아래의 `mov` 명령어들이 전혀 생성되지 않습니다.

  * **② `mLocate 10, 20` 의 확장 결과:**

    ```
    mov bx,0
    mov ah,2
    mov dh,20
    mov dl,10
    int 10h
    
    ```

    * **이유:** `10`과 `20` 모두 $0$보다 작지 않으므로 두 `IF`문을 무사히 통과합니다. 그 후 `yval` 자리에 `20`이, `xval` 자리에 `10`이 대입된 코드가 생성됩니다.

  * **③ `mLocate col, row` 의 확장 결과:**

    ```
    mov bx,0
    mov ah,2
    mov dh,row
    mov dl,col
    int 10h
    
    ```

    * **이유 (★함정 주의★):** 전처리기는 어셈블(번역) 시점에 텍스트만 보고 판단합니다. `col`과 `row`는 변수 이름(문자열)일 뿐 음수 기호(`-`)가 붙은 상수가 아니므로 `IF col LT 0` 조건을 거짓(False)으로 보고 통과시킵니다. 따라서 `yval`에 `row`, `xval`에 `col`이라는 변수 이름 자체가 그대로 들어간 코드가 생성됩니다.

> **💡 쉬운 용어 & 문법 돋보기**
>
> * **`EXITM` (Exit Macro):** 함수의 `return`처럼, 매크로를 코드로 번역하다가 특정 조건에서 "더 이상 아래 코드를 만들지 말고 여기서 매크로 확장을 즉시 종료해!"라고 명령하는 지시어입니다.
>
> * **3번(`mLocate col, row`)이 헷갈리는 이유:** 전처리기는 `col` 안에 `60`이 들어있다는 사실을 모릅니다(변수의 값은 프로그램이 실행될 때 메모리에 들어가기 때문). 전처리기 눈에는 그저 `'c','o','l'`이라는 글자로만 보이기 때문에 `0`보다 작지 않다고 판단하고 코드를 그대로 만들어 줍니다.