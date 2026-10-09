## 실습과제 1

### 1. 패키지를 구성하는 가장 중요한 필수 파일 2가지는 무엇인가?

| 파일 | 역할 |
|---|---|
| **`package.xml`** | 패키지 설정 파일. 패키지 이름, 버전, 설명, 관리자, 라이선스, 의존성 패키지, 빌드 타입 등 패키지 정보를 XML로 기술한다. 모든 ROS 패키지는 패키지당 정확히 1개를 반드시 포함해야 한다. |
| **`CMakeLists.txt`** | 빌드 설정 파일. C++ 패키지(`ament_cmake`)에서 CMake 빌드 환경(실행 파일 생성, 의존성 패키지 탐색, 설치 경로, 컴파일 옵션 등)을 기술한다. |

---

### 2. xml 파일 형식에 대하여 조사하시오.

**XML (eXtensible Markup Language)** 은 W3C가 표준화한 확장 가능한 마크업 언어로, 데이터를 **태그로 감싼 계층(트리) 구조의 텍스트**로 표현하는 형식이다. 사람이 읽을 수 있고 기계가 파싱하기 쉬우며 플랫폼·언어에 독립적이어서 설정 파일과 데이터 교환에 널리 쓰인다.

**기본 구성 요소**

| 요소 | 설명 | 예 |
|---|---|---|
| XML 선언 | 문서 맨 앞에서 XML 버전을 지정 | `<?xml version="1.0"?>` |
| 요소(element) | 여는 태그와 닫는 태그 쌍으로 이루어진 단위 | `<name>first_pkg</name>` |
| 속성(attribute) | 여는 태그 안의 `이름="값"` 형태 부가 정보 | `<maintainer email="a@b.com">` |
| 텍스트 | 요소 안의 내용 | `first_pkg` |
| 주석 | 처리되지 않는 설명 | `<!-- 설명 -->` |
| 루트 요소 | 문서 전체를 감싸는 최상위 요소 (하나만 존재) | `<package> ... </package>` |

**문법 규칙 (well-formed XML)**
- 문서에는 루트 요소가 하나만 있어야 한다.
- 모든 여는 태그는 닫는 태그로 닫아야 하고, 태그는 올바르게 중첩되어야 한다. (`<a><b></b></a>` 가능, `<a><b></a></b>` 불가)
- 태그 이름은 대소문자를 구분한다.
- 속성값은 반드시 따옴표(`"` 또는 `'`)로 감싼다.
- `<`, `&` 같은 특수문자는 `&lt;`, `&amp;` 로 표기한다.

**유효성 검증 (valid XML)**: DTD나 XSD 같은 스키마로 "어떤 태그가 어디에 올 수 있는지"를 정의하고 검증할 수 있다. ROS 2의 `package.xml`은 `package_format3.xsd` 스키마를 지정한다.

**ROS에서 XML이 쓰이는 곳**: `package.xml`(패키지 정보), launch 파일(XML 형식 지원), URDF(로봇 모델), `plugin.xml`(RQt 플러그인 설정) 등.

**예: ROS 2 `package.xml`**

```xml
<?xml version="1.0"?>                                   <!-- XML 선언 -->
<package format="3">                                    <!-- 루트 요소, 속성 format=3 -->
  <name>my_first_ros_rclcpp_pkg</name>                  <!-- 요소 + 텍스트 -->
  <version>0.0.0</version>
  <description>TODO: Package description</description>
  <maintainer email="pyo@robotis.com">pyo</maintainer> <!-- 속성 email -->
  <license>TODO: License declaration</license>

  <buildtool_depend>ament_cmake</buildtool_depend>
  <depend>rclcpp</depend>
  <depend>std_msgs</depend>

  <export>
    <build_type>ament_cmake</build_type>                <!-- 중첩된 요소 -->
  </export>
</package>
```

---

### 3. ament_cmake와 CMake의 차이를 설명하시오.

**CMake**는 ROS와 무관한 범용 **크로스 플랫폼 빌드 설정 도구**다. `CMakeLists.txt`에 빌드 규칙을 적으면 플랫폼에 맞는 빌드 파일을 생성해 준다. ROS가 CMake를 쓰는 이유는 패키지를 멀티 플랫폼(Linux, Windows, macOS)에서 빌드하기 위해서다. Make는 유닉스 계열만 지원하지만 CMake는 리눅스·BSD·macOS·윈도우를 모두 지원한다.

**ament_cmake**는 ROS 2의 빌드 시스템 `ament` 중 C++ 패키지용으로, **CMake 위에 ROS 2 전용 매크로와 규칙을 얹은 확장**이다. ROS 1 `catkin`의 업그레이드 버전에 해당하며, `CMakeLists.txt`에 기술된 CMake 설정을 기반으로 빌드를 수행한다.

| 구분 | CMake | ament_cmake |
|---|---|---|
| 성격 | 범용 빌드 설정 도구 (ROS와 무관) | CMake 위에 구현된 ROS 2 전용 빌드 시스템(확장 매크로 모음) |
| 사용 범위 | 모든 C/C++ 프로젝트 | ROS 2 C++ 패키지 |
| 필수 호출 | `project()`, `add_executable()` 등 | 추가로 `find_package(ament_cmake REQUIRED)`와 파일 마지막의 **`ament_package()`** 필수 |
| 의존성 처리 | `find_package()` + `target_link_libraries()` | `find_package()` + `ament_target_dependencies()` (include 경로·링크를 한 번에 설정) |
| 패키지 정보 | 없음 | `package.xml`과 연동, `ament_package()`가 패키지를 ament index에 등록 |
| 설치 규칙 | 임의의 경로 | ROS 2 관례: 실행 파일 `lib/${PROJECT_NAME}`, launch·param 등 `share/${PROJECT_NAME}` |
| 테스트·린트 | 직접 구성 | `ament_lint_auto` 등 제공 |
| 빌드 호출 | 직접 `cmake` / `make` | `colcon build`가 패키지별로 `ament_cmake`를 호출 |

**정리**: CMake는 도구 자체이고, ament_cmake는 "ROS 2 패키지가 따라야 할 규칙(의존성 선언, 설치 위치, 패키지 등록, 테스트)"을 CMake 매크로로 제공하는 계층이다. 빌드 설정 문법(`add_executable`, `install` 등)은 일반 CMake와 동일하다.

참고: [ament_cmake 문서](https://docs.ros.org/en/foxy/How-To-Guides/Ament-CMake-Documentation.html), [CMake 문서](https://cmake.org/cmake/help/latest/)
