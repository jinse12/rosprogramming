## 실습과제 1

CMake를 이용하여 레나 영상을 그레이 영상, 이진 영상으로 변환하여 출력하는 프로그램을 작성하시오.

## 1. 프로젝트 구조

```
lennacv
├── src
│   ├── CMakeLists.txt
│   ├── main.cpp
│   └── lenna.bmp
└── build          (자동 생성됨)
```

## 2. 소스파일

### 2-1. src/CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.28.3)
project(LennaCV)
find_package(OpenCV REQUIRED)
add_executable(LennaCV main.cpp)
target_include_directories(LennaCV PUBLIC ${OpenCV_INCLUDE_DIRS})
target_link_libraries(LennaCV PUBLIC ${OpenCV_LIBS})
```

| 줄 | 코드 | 설명 |
|---|---|---|
| 1 | `cmake_minimum_required(VERSION 3.28.3)` | 사용 가능한 CMake의 최소 버전을 3.28.3으로 설정한다. 실행되는 CMake 버전이 이보다 낮으면 실행이 중단되며, 반드시 첫 줄에 작성해야 한다 |
| 2 | `project(LennaCV)` | 프로젝트 이름을 LennaCV로 설정한다. 언어를 지정하지 않았으므로 기본값인 C, C++로 설정된다 |
| 3 | `find_package(OpenCV REQUIRED)` | 시스템에서 OpenCV 패키지를 자동으로 검색하여 헤더파일 경로(`OpenCV_INCLUDE_DIRS`), 라이브러리 목록(`OpenCV_LIBS`) 등을 환경변수에 저장한다. `REQUIRED`는 찾지 못하면 실행을 중단하라는 의미이다 |
| 4 | `add_executable(LennaCV main.cpp)` | main.cpp를 컴파일하여 LennaCV라는 실행파일(target)을 만들도록 지정한다 |
| 5 | `target_include_directories(LennaCV PUBLIC ${OpenCV_INCLUDE_DIRS})` | LennaCV를 빌드할 때 OpenCV 헤더파일 경로를 사용하도록 지정한다 (gcc의 `-I` 옵션에 해당) |
| 6 | `target_link_libraries(LennaCV PUBLIC ${OpenCV_LIBS})` | LennaCV를 빌드할 때 OpenCV 라이브러리를 링크하도록 지정한다 (gcc의 `-l` 옵션에 해당) |

### 2-2. src/main.cpp

```cpp
#include "opencv2/opencv.hpp"
#include <iostream>
using namespace cv;
using namespace std;
int main()
{
    cout << "Hello OpenCV " << CV_VERSION << endl;
    Mat img = imread("../src/lenna.bmp");
    if (img.empty()) { cerr << "Image load failed!" << endl; return -1; }

    Mat gray, binary;
    cvtColor(img, gray, COLOR_BGR2GRAY);
    threshold(gray, binary, 128, 255, THRESH_BINARY);

    imshow("image", img);
    imshow("gray", gray);
    imshow("binary", binary);
    waitKey(0);
    return 0;
}
```

## 3. 실행 과정

```bash
mkdir -p lennacv/src
cd lennacv/src
vi CMakeLists.txt         # cmake 설정파일 작성
vi main.cpp               # 소스 작성
cd ..                     # cmake 실행위치(lennacv)로 이동
cmake -S src -B build     # Configure & Generate
cmake --build build       # Build
cd build
./LennaCV                 # 실행
```

## 4. 실행 결과

### 4-1. 프로젝트 디렉토리 및 소스파일 작성

<img width="726" height="152" alt="image" src="https://github.com/user-attachments/assets/c8111c30-d896-492c-9ebf-de7f5804cdb4" />

> **설명:** `tree`로 확인한 프로젝트 구조이다. `lennacv/src` 폴더 안에 `CMakeLists.txt`, `main.cpp`, `lenna.bmp`가 있다.

### 4-2. Configure & Generate

<img width="864" height="322" alt="image" src="https://github.com/user-attachments/assets/c2ca9a50-ff80-4c96-8ead-0121ec82b784" />

> **설명:** `cmake -S src -B build`를 실행한 화면이다. 컴파일러를 검사한 뒤 `find_package()`에 의해 `Found OpenCV: ... (found version "x.x.x")`가 출력되어 OpenCV를 찾은 것을 확인할 수 있다. 이어서 `Configuring done`, `Generating done`이 출력되고 build 폴더에 `CMakeCache.txt`와 `Makefile`이 생성된다.

### 4-3. Build

<img width="800" height="78" alt="image" src="https://github.com/user-attachments/assets/f22d83c3-e97e-45d3-9439-cf56e7153afa" />

> **설명:** `cmake --build build`를 실행하면 `Building CXX object`(컴파일) → `Linking CXX executable LennaCV`(링크) 순서로 진행되어 build 폴더에 `LennaCV` 실행파일이 생성된다.

### 4-4. 실행 결과

<img width="609" height="20" alt="image" src="https://github.com/user-attachments/assets/759b070b-fab0-417f-acb6-061f923c53e9" />
<img width="1581" height="589" alt="image" src="https://github.com/user-attachments/assets/faa193b2-8a8b-488a-8e9b-710860f64eab" />

> **설명:** `./LennaCV`를 실행하면 터미널에 OpenCV 버전이 출력되고 창 3개가 열린다.
> - **image:** 원본 컬러 레나 영상
> - **gray:** `cvtColor()`로 변환한 그레이 영상 (색상 정보 없이 밝기만 표현)
> - **binary:** `threshold()`로 변환한 이진 영상 (밝기 128 초과는 흰색, 이하는 검은색)

---
