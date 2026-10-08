## 실습과제 1

### 1. 빌드 시스템과 빌드 툴의 차이

| 구분 | 빌드 시스템 (build system) | 빌드 툴 (build tool) |
|---|---|---|
| 대상 | **단일 패키지** | **시스템 전체(여러 패키지)** |
| 역할 | 한 패키지의 의존성을 해결하고 빌드하여 실행 파일 생성 | 각 패키지에 기술된 종속성 그래프를 해석하고, 토폴로지 순서대로 각 패키지에 맞는 빌드 시스템을 호출 |
| 예 | `ament_cmake`(C++), `ament_python`(Python) | `colcon` |

패키지마다 사용하는 언어가 다르면 빌드 시스템도 달라지는데, 이를 하나로 통합 관리하는 것이 빌드 툴이다.

```
build tool (colcon)
 ├─ build system (ament_cmake)  → ros2 package (C++)
 ├─ build system (ament_cmake)  → ros2 package (C++)
 └─ build system (ament_python) → ros2 package (python)
```

### 2. 패키지 생성 명령어의 실행 위치와 이동 명령어

- 실행 위치: **`~/ros2_ws/src`** (사용자 작업 폴더의 src 폴더)
- 이동 명령어:

```bash
cd ~/ros2_ws/src
```

### 3. 패키지 생성 명령어의 사용법

```bash
ros2 pkg create <pkgname> --build-type <buildtype> --dependencies <deppkg1> ... <deppkgn>
```

- `<pkgname>`: 생성할 패키지 이름
- `--build-type <buildtype>`: 빌드 타입. C++ 패키지는 `ament_cmake`, Python 패키지는 `ament_python`
- `--dependencies <deppkg1> ... <deppkgn>`: 패키지를 빌드하는 데 필요한 의존 라이브러리(패키지)들

예: `ros2 pkg create first_pkg --build-type ament_cmake --dependencies rclcpp std_msgs`

### 4. 패키지 빌드 명령어의 실행 위치와 이동 명령어

- 실행 위치: **`~/ros2_ws`** (작업 폴더의 최상위)
- 이동 명령어:

```bash
cd ~/ros2_ws
```

### 5. 패키지 빌드 명령어의 사용법

```bash
colcon build --symlink-install --packages-select <pkgname>
```

- `colcon build`: colcon 빌드 툴로 특정 패키지 또는 전체 패키지를 빌드
- `--packages-select <pkgname>`: 지정한 패키지만 선택하여 빌드 (생략하면 전체 패키지 빌드)
- `--symlink-install`: 설치 시 실행 파일을 복사하는 대신 **심볼릭 링크**를 설치 (소스/설정 파일 수정 시 재빌드 없이 반영되는 경우가 많음)

예: `colcon build --symlink-install --packages-select first_pkg`

---

## 실습과제 2

### 1. 작업 폴더 생성 및 패키지 생성 · 빌드

```bash
# 작업 폴더 생성
mkdir -p ~/ros2_ws/src

# 패키지 생성
cd ~/ros2_ws/src
ros2 pkg create first_pkg --build-type ament_cmake --dependencies rclcpp std_msgs

# 패키지 빌드
cd ~/ros2_ws
colcon build --symlink-install --packages-select first_pkg
```

### 2. 실행 결과

패키지 생성 결과:

```
going to create a new package
package name: first_pkg
destination directory: /home/<사용자명>/ros2_ws/src
package format: 3
version: 0.0.0
description: TODO: Package description
maintainer: ['<사용자명> <<사용자명>@todo.todo>']
licenses: ['TODO: License declaration']
build type: ament_cmake
dependencies: ['rclcpp', 'std_msgs']
creating folder ./first_pkg
creating ./first_pkg/package.xml
creating source and include folder
creating folder ./first_pkg/src
creating folder ./first_pkg/include/first_pkg
creating ./first_pkg/CMakeLists.txt
```

빌드 결과:

```
Starting >>> first_pkg
Finished <<< first_pkg [8.91s]

Summary: 1 package finished [9.48s]
```

### 3. 자동 생성되는 파일과 디렉터리

`tree -L 1` 결과 (`~/ros2_ws/src/first_pkg`):

```
.
├── CMakeLists.txt
├── include
├── package.xml
└── src
```

| 이름 | 종류 | 설명 |
|---|---|---|
| `CMakeLists.txt` | 파일 | C/C++ 빌드 설정 파일. ament_cmake가 CMake 기반이므로 빌드 설정(`find_package`, 실행 파일 등록 등)을 여기에 기술 |
| `package.xml` | 파일 | 패키지 정보 파일(XML). 패키지 이름, 버전, 설명, 관리자, 라이선스, 의존성 패키지, 빌드 타입 등을 기술 |
| `include/first_pkg/` | 디렉터리 | C/C++ 헤더 파일용 폴더. 패키지 이름별 하위 폴더로 헤더를 구분 |
| `src/` | 디렉터리 | C/C++ 소스 코드용 폴더 (현재는 비어 있음) |

빌드 후 `tree -L 1` 결과 (`~/ros2_ws`):

```
.
├── build
├── install
├── log
└── src
```

| 이름 | 설명 |
|---|---|
| `build/` | 빌드 설정 및 중간 산출물 폴더 |
| `install/` | msg, srv 헤더 파일과 사용자 패키지의 라이브러리·실행 파일이 설치되는 폴더 (환경 설정용 `setup.bash` 등도 포함) |
| `log/` | 빌드 로그 파일 폴더 |
| `src/` | 사용자 패키지 소스 폴더 |

---
