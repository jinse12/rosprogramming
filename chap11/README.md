# 실습과제 1

## 1. CMake와 GNU Make의 차이점을 설명하라.

| 구분 | GNU Make | CMake |
|---|---|---|
| 정체 | **빌드 도구**: Makefile을 읽어 컴파일러를 직접 호출하고 실행파일을 만든다 | **빌드 시스템 생성기 (빌드 전과정의 지휘자)**: 컴파일러나 빌드 도구가 아니고, 빌드 도구가 사용할 파일을 만들어 준다 |
| 입력 파일 | Makefile | CMakeLists.txt |
| 출력 결과 | 실행파일, 라이브러리 | Makefile(GNU Make용), build.ninja(Ninja용), Solution file(Visual Studio용), project file(Xcode용) |
| 플랫폼 | 주로 Linux/Unix | Windows, Linux, macOS 등 다양한 플랫폼 지원 (Cross Platform Make) |
| 작성 난이도 | 컴파일러, 옵션, 목적파일, 의존관계를 개발자가 직접 작성해야 한다 | 타겟, 소스파일, 라이브러리 등 최소한의 정보만 작성하면 나머지를 자동으로 처리한다 |
| 환경 탐색 | 없음 (경로를 직접 지정) | 시스템 정보, 컴파일러, 라이브러리 경로를 자동으로 검색한다 |

**정리**

- Make를 이용한 개발 과정: `소스코드 → Makefile(직접 작성) → make → 실행파일`
- CMake를 이용한 개발 과정: `소스코드 → CMakeLists.txt(작성) → cmake → Makefile(자동 생성) → make → 실행파일`
- **Make는 실제로 빌드하는 도구**, **CMake는 그 빌드 도구가 사용할 Makefile을 자동으로 만들어 주는 도구**. CMake를 사용하면 운영체제마다 다른 빌드 도구를 따로 다룰 필요 없이 CMake 하나로 빌드 전과정을 수행할 수 있다.

## 2. CMakeLists.txt의 역할을 설명하라.

- CMakeLists.txt는 **CMake의 프로젝트 설정 파일**이다.
- 프로젝트 빌드에 필요한 최소한의 정보(**타겟, 소스파일, 라이브러리 등**)를 CMake 문법으로 작성한다.
- Configure 단계에서 CMake가 가장 먼저 읽고 해석·실행하는 **입력 파일**이며, 이 내용을 바탕으로 Makefile이 생성된다.

**예제1의 CMakeLists.txt**

```cmake
cmake_minimum_required(VERSION 3.16.3)   # 필요한 CMake 최소 버전 지정
project(Hello)                           # 프로젝트 이름을 Hello로 지정
add_executable(Hello main.cpp)           # main.cpp를 컴파일하여 Hello 실행파일 생성
```

## 3. CMakeCache.txt의 역할을 설명하라.

- CMakeCache.txt는 Configure 단계에서 **자동으로 생성되는 환경변수 저장 파일**이다. (`build` 디렉토리 안에 생성됨)
- CMake가 자동으로 검색한 **시스템 정보, 컴파일러, 라이브러리, 빌드 툴의 경로**를 `KEY:TYPE=VALUE` 형식으로 저장한다.
- Generate 단계는 이 파일의 정보를 바탕으로 Makefile을 생성한다.
- 다음 빌드 때 CMake가 이 파일을 먼저 읽어서(Read CMakeCache.txt) 환경을 다시 검색하지 않고 재사용하므로 빌드 속도가 빨라진다.

## 4. CMake의 각 단계별(configure → generate → build) 결과물을 자세히 설명하라.

```
CMakeLists.txt ─▶ [Configure] ─▶ CMakeCache.txt ─▶ [Generate] ─▶ Makefile ─▶ [Build] ─▶ 실행파일
```

`cmake -S src -B build` 명령이 Configure와 Generate를 수행하고, `cmake --build build` 명령이 Build를 수행한다.

### (1) Configure 단계

- **입력:** `CMakeLists.txt` (기존 `CMakeCache.txt`가 있으면 먼저 읽음)
- **하는 일:**
  - CMakeLists.txt의 내용을 해석하고 실행한다.
  - 시스템 정보, C/C++ 컴파일러, 라이브러리 정보를 자동으로 검색하고, 컴파일러가 정상 동작하는지 검사한다.
- **결과물:**
  - `build/CMakeCache.txt`: 수집한 환경변수(시스템 정보, 컴파일러, 라이브러리 경로) 저장
  - `build/CMakeFiles/`: 검사 과정의 로그(`CMakeOutput.log`), 컴파일러 정보(`3.28.3/`) 등 초기 빌드 트리

<img width="935" height="318" alt="image" src="https://github.com/user-attachments/assets/f9182dd2-54a2-4da8-a815-d7001fd2c43b" />

> **설명:** `cmake -S src -B build`를 실행한 화면이다. C/C++ 컴파일러(GNU)를 찾아 동작을 검사하고(`Check for working C/CXX compiler ... works`), `Configuring done`이 출력되면 Configure 단계가 끝난 것이다.

### (2) Generate 단계

- **입력:** `CMakeCache.txt` (Configure 단계의 결과물)
- **하는 일:** 개발 환경에서 사용 중인 빌드 툴에 맞는 빌드 파일을 자동 생성한다. Linux에서는 GNU Make용 Makefile을 만든다.
- **결과물:**
  - `build/Makefile`: make가 사용할 빌드 파일
  - `build/CMakeFiles/Hello.dir/`: Hello 타겟을 빌드하기 위한 세부 규칙
  - `build/cmake_install.cmake`: 설치(install) 단계에서 사용할 스크립트
 
<img width="844" height="423" alt="image" src="https://github.com/user-attachments/assets/bbb426af-c446-4fc0-b257-c26740558efe" />

> **설명:** `tree -L 3`으로 확인한 화면이다. 처음에는 `src` 폴더만 있었지만 `build` 폴더가 자동 생성되었고, 그 안에 `CMakeCache.txt`(Configure 결과)와 `Makefile`(Generate 결과)이 만들어졌다.

### (3) Build 단계

- **입력:** `Makefile` (Generate 단계의 결과물)
- **하는 일:** CMake가 빌드 툴(make)을 호출하여 컴파일 → 링크를 수행한다.
  - 컴파일: `main.cpp` → `CMakeFiles/Hello.dir/main.cpp.o` (목적파일)
  - 링크: 목적파일 → `Hello` (실행파일)
- **결과물:** `build/Hello` 실행파일 (라이브러리 타겟이면 라이브러리 파일)

<img width="859" height="209" alt="image" src="https://github.com/user-attachments/assets/a0176437-c893-4347-ba30-b0da6e27f9ed" />

> **설명:** `cmake --build build`를 실행하면 `Building CXX object`(컴파일) → `Linking CXX executable`(링크) 순서로 진행되고, `build` 폴더에 `Hello` 실행파일이 생성된다. `./Hello`로 실행하면 `Hello World!`가 출력된다.

### 단계별 정리

| 단계 | 명령어 | 입력 | 결과물 |
|---|---|---|---|
| Configure | `cmake -S src -B build` | CMakeLists.txt | CMakeCache.txt, CMakeFiles/ |
| Generate | (위 명령에 포함) | CMakeCache.txt | Makefile, cmake_install.cmake |
| Build | `cmake --build build` | Makefile | 실행파일(Hello) 또는 라이브러리 |

## 5. 예제 1에서 cmake --build build 명령어 대신에 make 명령어를 이용하여 빌드해보시오.

Generate 단계 후 `hello/build/Makefile`이 생성되므로, `build` 디렉토리로 이동하여 `make`를 실행하면 된다.

```bash
cd ~/hello
rm -rf build              # 이전에 빌드한 결과 삭제 (이미 빌드되어 있으면 make가 다시 빌드하지 않음)
cmake -S src -B build     # Configure & Generate → build/Makefile 생성
cd build
make                      # Makefile을 이용해 빌드
./Hello                   # 실행
```

<img width="883" height="534" alt="image" src="https://github.com/user-attachments/assets/4f9ca91e-1951-46e1-b6ac-26d7f530af3c" />

> **설명:** `build` 디렉토리에서 `make`를 실행하면 `cmake --build build`와 똑같이 컴파일 → 링크가 진행되어 `Hello` 실행파일이 생성된다. `cmake --build`는 내부적으로 make를 호출하는 것이므로 결과가 같다. `./Hello`를 실행하면 `Hello World!`가 출력된다.

---

# 실습과제 2

CMake를 이용하여 2개의 정수를 입력받아 합을 출력하는 C++ 프로그램을 작성하시오.

## tree

```
adder
├── src
│   ├── CMakeLists.txt
│   └── main.cpp
└── build          (자동 생성)
```

## 소스파일

### src/main.cpp

<img width="776" height="189" alt="image" src="https://github.com/user-attachments/assets/1da2c2d0-16cf-424f-bb20-2fa37255dd0b" />

### src/CMakeLists.txt

<img width="698" height="55" alt="스크린샷 2026-09-29 100221" src="https://github.com/user-attachments/assets/2ff991d4-b463-42ce-b623-0512decc1f2f" />

## 실행 과정

```bash
mkdir -p adder/src
cd adder/src
vi main.cpp               # 소스 작성
vi CMakeLists.txt         # cmake 설정파일 작성
cd ..                     # cmake 실행위치(adder)로 이동
cmake -S src -B build     # Configure & Generate
cmake --build build       # Build
cd build
./Adder                   # 실행
```

## 실행 결과

### (1) 프로젝트 디렉토리 및 소스파일 작성

<img width="725" height="421" alt="image" src="https://github.com/user-attachments/assets/2667749d-b228-46b4-b421-d4c55f704e21" />

> **설명:** tree로 확인한 프로젝트 구조이다. adder/src 폴더 안에 `CMakeLists.txt`와 `main.cpp`가 작성되어 있다.

### (2) Configure & Generate

<img width="855" height="303" alt="스크린샷 2026-09-29 100956" src="https://github.com/user-attachments/assets/39103d59-5c54-462f-9ae1-e376c4c9c4f9" />

> **설명:** `cmake -S src -B build`를 실행하여 컴파일러를 검사하고(Configure), `build` 폴더에 `CMakeCache.txt`와 `Makefile`을 생성했다(Generate). `Build files have been written to: .../adder/build`가 출력된다.

### (3) Build & Execute

<img width="843" height="74" alt="스크린샷 2026-09-29 101021" src="https://github.com/user-attachments/assets/85ee0a8b-1abe-4c95-9e66-a273f721017f" />
<img width="733" height="74" alt="image" src="https://github.com/user-attachments/assets/fd3718b1-fd4c-4920-bf0c-3abb79169b26" />

> **설명:** `cmake --build build`로 컴파일과 링크를 수행하여 Adder 실행파일을 만들었다. ./Adder를 실행하고 결과를 확인했다.

---
