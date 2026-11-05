# ROS2 RCLCPP 실습과제 (타이머 콜백 Publisher)

## 실습과제 1. 예제 1번을 수정하여 문자열 대신에 정수값을 0으로 초기화하고 1씩 증가시키면서 퍼블리시하는 패키지 pub2-1를 작성하시오

### 패키지 생성

```bash
cd ~/ros2_ws/src
ros2 pkg create pub2_1 --build-type ament_cmake --dependencies rclcpp std_msgs
```

### 소스파일: `pub2_1/src/pub2_1.cpp`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/int32.hpp"
#include <memory>
#include <chrono>
#include <functional>
using namespace std::chrono_literals;
void callback(rclcpp::Node::SharedPtr node,
              rclcpp::Publisher<std_msgs::msg::Int32>::SharedPtr mypub)
{
    static int count;
    auto message = std_msgs::msg::Int32();
    message.data = count++;
    RCLCPP_INFO(node->get_logger(), "Publish: %d", message.data);
    mypub->publish(message);
}
int main(int argc, char* argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("node_pub2_1");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    auto pub = node->create_publisher<std_msgs::msg::Int32>("topic_pub2_1", qos_profile);
    std::function<void()> fn = std::bind(callback, node, pub);
    auto timer = node->create_wall_timer(100ms, fn);
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

### 라인단위 코드설명

| 코드 | 설명 |
|---|---|
| `#include "rclcpp/rclcpp.hpp"` | ROS2 C++ 라이브러리(노드, 퍼블리셔, 타이머, 로깅 등)의 헤더를 포함한다. |
| `#include "std_msgs/msg/int32.hpp"` | `std_msgs/msg/Int32` 메시지 인터페이스의 헤더를 포함한다. 예제의 `string.hpp`에서 바꾼 부분이다. |
| `#include <memory>` | `std::make_shared`, `std::shared_ptr`를 사용하기 위한 헤더이다. |
| `#include <chrono>` | 시간 단위(`std::chrono::duration`)를 사용하기 위한 헤더이다. |
| `#include <functional>` | `std::function`, `std::bind`를 사용하기 위한 헤더이다. |
| `using namespace std::chrono_literals;` | `100ms`, `1s` 같은 시간 사용자 정의 리터럴을 쓸 수 있게 한다. |
| `void callback(rclcpp::Node::SharedPtr node, rclcpp::Publisher<std_msgs::msg::Int32>::SharedPtr mypub)` | 타이머 이벤트마다 실행할 콜백함수. 로그 출력에 쓸 노드와 발행에 쓸 퍼블리셔를 매개변수로 받는다. |
| `static int count;` | 정적 지역변수. 0으로 자동 초기화되고, 함수가 끝나도 값이 유지되어 호출할 때마다 이어서 증가한다. |
| `auto message = std_msgs::msg::Int32();` | 전송할 `Int32` 메시지 객체를 만든다. |
| `message.data = count++;` | 현재 `count` 값을 메시지에 저장한 뒤 `count`를 1 증가시킨다. 0, 1, 2, … 순서로 발행된다. |
| `RCLCPP_INFO(node->get_logger(), "Publish: %d", message.data);` | 발행하는 정수값을 로그로 출력한다. 정수이므로 `%d`를 쓴다. |
| `mypub->publish(message);` | 메시지를 `topic_pub2_1` 토픽으로 발행한다. |
| `rclcpp::init(argc, argv);` | ROS2를 초기화한다. |
| `auto node = std::make_shared<rclcpp::Node>("node_pub2_1");` | 이름이 `node_pub2_1`인 노드를 동적 생성한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | History를 KEEP_LAST, depth를 10으로 설정한 QoS를 만든다. |
| `auto pub = node->create_publisher<std_msgs::msg::Int32>("topic_pub2_1", qos_profile);` | `Int32` 메시지를 `topic_pub2_1` 토픽으로 발행하는 퍼블리셔를 생성한다. |
| `std::function<void()> fn = std::bind(callback, node, pub);` | `create_wall_timer`는 매개변수 없는 `void()` 함수만 받는다. 그래서 `std::bind`로 `node`, `pub`을 미리 묶어 매개변수 없는 함수 `fn`으로 바꾼다. |
| `auto timer = node->create_wall_timer(100ms, fn);` | 100ms마다 `fn`(결국 `callback(node, pub)`)을 호출하는 타이머를 생성한다. |
| `rclcpp::spin(node);` | 노드를 계속 실행하면서 타이머 이벤트가 발생할 때마다 콜백을 호출한다. Ctrl+C를 누르면 리턴한다. |
| `rclcpp::shutdown();` | ROS2를 종료한다. |
| `return 0;` | 프로그램을 정상 종료한다. |

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)
project(pub2_1)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)

#build
add_executable(pub2_1 src/pub2_1.cpp)
ament_target_dependencies(pub2_1 rclcpp std_msgs)

# Install
install(TARGETS
  pub2_1
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `cmake_minimum_required(VERSION 3.16)` | 필요한 CMake 최소 버전을 지정한다. |
| `project(pub2_1)` | 패키지 이름. `package.xml`의 `<name>`과 같아야 한다. |
| `set(CMAKE_CXX_STANDARD 14)` | C++ 표준이 지정되지 않았으면 C++14를 사용한다(`100ms` 같은 chrono 리터럴은 C++14부터 지원). |
| `add_compile_options(-Wall -Wextra -Wpedantic)` | GCC/Clang에서 경고 옵션을 켠다. |
| `find_package(ament_cmake REQUIRED)` | `ament_target_dependencies()`, `ament_package()` 같은 ROS2 전용 명령의 정의를 가져온다. |
| `find_package(rclcpp / std_msgs REQUIRED)` | 소스코드에 필요한 ROS2 라이브러리와 메시지 인터페이스 라이브러리를 찾는다. |
| `add_executable(pub2_1 src/pub2_1.cpp)` | 소스파일을 컴파일해 실행파일 `pub2_1`을 만든다. |
| `ament_target_dependencies(pub2_1 rclcpp std_msgs)` | 실행파일에 의존 패키지의 include 경로와 라이브러리를 연결한다. |
| `install(TARGETS pub2_1 DESTINATION lib/${PROJECT_NAME})` | 실행파일을 `install/pub2_1/lib/pub2_1`에 설치한다. |
| `ament_package()` | 패키지를 ROS2 패키지로 등록한다. 파일 마지막에 둔다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select pub2_1
source install/local_setup.bash
ros2 run pub2_1 pub2_1
```

### 실행결과

<img width="1041" height="314" alt="image" src="https://github.com/user-attachments/assets/bcc2b338-ad5d-4e6a-8cc3-f68abc8b889d" />

ROS2 명령어로 확인 (새 터미널):

```bash
rqt_graph                       # /node_pub2_1 → /topic_pub2_1
ros2 node list                  # /node_pub2_1
ros2 topic list -t              # /topic_pub2_1 [std_msgs/msg/Int32]
ros2 topic echo /topic_pub2_1
ros2 topic hz /topic_pub2_1
ros2 topic bw /topic_pub2_1
```
<img width="1069" height="314" alt="image" src="https://github.com/user-attachments/assets/4a06af05-587c-4409-8b53-758c9ab126c7" />


### 토픽 메시지의 수신 주기와 주파수

- **주기:** 타이머를 `100ms`로 설정했으므로 메시지가 0.1초마다 발행되고, 같은 간격으로 수신된다.
- **주파수:** 주파수는 주기의 역수이므로 1 / 0.1s = **10Hz**, 즉 1초에 10개를 수신한다.
- `ros2 topic hz`의 `average rate` 값이 약 10.000, `min`/`max`가 약 0.100s로 나오는 것으로 확인할 수 있다. 값이 정확히 10이 아니고 약간 흔들리는 것은 OS 스케줄링과 통신 지연 때문이다.

---

## 실습과제 2. 예제 1번을 수정하여 키보드로부터 실수값 3개를 입력받아 퍼블리시하는 패키지 pub2-2를 작성하시오.

### 패키지 생성

```bash
cd ~/ros2_ws/src
ros2 pkg create pub2_2 --build-type ament_cmake --dependencies rclcpp geometry_msgs
```

### 소스파일: `pub2_2/src/pub2_2.cpp`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "geometry_msgs/msg/vector3.hpp"
#include <memory>
#include <chrono>
#include <functional>
#include <iostream>
using namespace std::chrono_literals;
void callback(rclcpp::Node::SharedPtr node,
              rclcpp::Publisher<geometry_msgs::msg::Vector3>::SharedPtr mypub)
{
    auto message = geometry_msgs::msg::Vector3();
    std::cout << "실수값 3개 입력(x y z): ";
    if(!(std::cin >> message.x >> message.y >> message.z))
    {
        rclcpp::shutdown();
        return;
    }
    RCLCPP_INFO(node->get_logger(), "Publish: x=%.2f, y=%.2f, z=%.2f",
                message.x, message.y, message.z);
    mypub->publish(message);
}
int main(int argc, char* argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("node_pub2_2");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    auto pub = node->create_publisher<geometry_msgs::msg::Vector3>("topic_pub2_2", qos_profile);
    std::function<void()> fn = std::bind(callback, node, pub);
    auto timer = node->create_wall_timer(100ms, fn);
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

### 라인단위 코드설명

| 코드 | 설명 |
|---|---|
| `#include "geometry_msgs/msg/vector3.hpp"` | `geometry_msgs/msg/Vector3` 메시지 인터페이스의 헤더를 포함한다. |
| `#include <memory>`, `<chrono>`, `<functional>` | 스마트 포인터, 시간 리터럴, `std::function`/`std::bind`를 위한 헤더이다. |
| `#include <iostream>` | 키보드 입력(`std::cin`)과 화면 출력(`std::cout`)을 위한 헤더이다. |
| `using namespace std::chrono_literals;` | `100ms` 리터럴을 쓸 수 있게 한다. |
| `void callback(..., rclcpp::Publisher<geometry_msgs::msg::Vector3>::SharedPtr mypub)` | 타이머마다 호출되어 입력을 받고 발행하는 콜백함수이다. |
| `auto message = geometry_msgs::msg::Vector3();` | `x`, `y`, `z`(모두 `double`) 필드를 가진 메시지 객체를 만든다. |
| `std::cout << "실수값 3개 입력(x y z): ";` | 입력 안내 문구를 출력한다. |
| `if(!(std::cin >> message.x >> message.y >> message.z))` | 실수 3개를 차례로 `x`, `y`, `z`에 입력받는다. 숫자가 아닌 값이나 EOF(Ctrl+D)로 입력이 실패하면 조건이 참이 된다. |
| `rclcpp::shutdown(); return;` | 입력이 실패하면 ROS2를 종료해 `spin`이 리턴하도록 하고 콜백을 끝낸다. |
| `RCLCPP_INFO(..., "Publish: x=%.2f, y=%.2f, z=%.2f", ...)` | 발행할 세 값을 소수점 둘째 자리까지 로그로 출력한다. |
| `mypub->publish(message);` | 입력받은 벡터를 `topic_pub2_2` 토픽으로 발행한다. |
| `auto node = std::make_shared<rclcpp::Node>("node_pub2_2");` | 이름이 `node_pub2_2`인 노드를 동적 생성한다. |
| `auto pub = node->create_publisher<geometry_msgs::msg::Vector3>("topic_pub2_2", qos_profile);` | `Vector3` 메시지를 `topic_pub2_2` 토픽으로 발행하는 퍼블리셔를 생성한다. |
| `std::function<void()> fn = std::bind(callback, node, pub);` | 매개변수 있는 콜백을 `void()` 형태로 바꾼다. |
| `auto timer = node->create_wall_timer(100ms, fn);` | 100ms마다 콜백을 호출하는 타이머를 생성한다. |
| `rclcpp::spin(node);` | 타이머 이벤트를 처리하며 노드를 계속 실행한다. |
| `rclcpp::shutdown(); return 0;` | ROS2를 종료하고 프로그램을 끝낸다. |

### CMakeLists.txt (예제에서 바뀐 부분)

```cmake
project(pub2_2)
...
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(geometry_msgs REQUIRED)

#build
add_executable(pub2_2 src/pub2_2.cpp)
ament_target_dependencies(pub2_2 rclcpp geometry_msgs)

# Install
install(TARGETS
  pub2_2
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `project(pub2_2)` | 패키지 이름을 `pub2_2`로 지정한다. |
| `find_package(geometry_msgs REQUIRED)` | `std_msgs` 대신 `geometry_msgs` 패키지를 찾는다. |
| `add_executable(pub2_2 src/pub2_2.cpp)` | 실행파일 `pub2_2`를 만든다. |
| `ament_target_dependencies(pub2_2 rclcpp geometry_msgs)` | 의존 패키지를 `geometry_msgs`로 바꿔 연결한다. |
| `install(TARGETS pub2_2 ...)` | 실행파일을 `install/pub2_2/lib/pub2_2`에 설치한다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select pub2_2
source install/local_setup.bash
ros2 run pub2_2 pub2_2
```

### 실행결과

<img width="1020" height="288" alt="image" src="https://github.com/user-attachments/assets/f198955e-f1b7-41bc-b189-a880886688d1" />


ROS2 명령어로 확인 (새 터미널):

```bash
ros2 node list                  # /node_pub2_2
ros2 topic list -t              # /topic_pub2_2 [geometry_msgs/msg/Vector3]
ros2 topic echo /topic_pub2_2
ros2 topic hz /topic_pub2_2
```

<img width="1051" height="293" alt="image" src="https://github.com/user-attachments/assets/eabec9a8-c8c6-4eac-b4be-a483c0c4166c" />


### 토픽 메시지의 수신 주기와 주파수

- 타이머 주기는 100ms(10Hz)지만, 콜백 안의 `std::cin`이 입력을 기다리는 동안 콜백이 끝나지 않는다. 그래서 다음 타이머 콜백도 실행되지 못한다.
- 따라서 **실제 수신 주기는 사용자가 값을 입력하는 간격**과 같다. 예를 들어 2초마다 입력하면 주기는 약 2초, 주파수는 1 / 2s = 약 0.5Hz이다.
- `ros2 topic hz` 결과도 입력 간격에 따라 `average rate`가 불규칙하게 나온다. 입력이 없으면 메시지가 발행되지 않아 rate가 출력되지 않는다.

---

## 실습과제 3. Turtlesim 패키지의 teleop_turtle 노드를 대신하는 패키지 pub2-3를 작성하라.

### 패키지 생성

```bash
cd ~/ros2_ws/src
ros2 pkg create pub2_3 --build-type ament_cmake --dependencies rclcpp geometry_msgs
```

### 소스파일: `pub2_3/src/pub2_3.cpp`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "geometry_msgs/msg/twist.hpp"
#include <memory>
#include <chrono>
#include <functional>
using namespace std::chrono_literals;
void callback(rclcpp::Node::SharedPtr node,
              rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr mypub)
{
    auto message = geometry_msgs::msg::Twist();
    message.linear.x = 2.0;
    message.angular.z = 1.0;
    RCLCPP_INFO(node->get_logger(), "Publish: linear.x=%.1f, angular.z=%.1f",
                message.linear.x, message.angular.z);
    mypub->publish(message);
}
int main(int argc, char* argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("node_pub2_3");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    auto pub = node->create_publisher<geometry_msgs::msg::Twist>("/turtle1/cmd_vel", qos_profile);
    std::function<void()> fn = std::bind(callback, node, pub);
    auto timer = node->create_wall_timer(100ms, fn);
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

### 라인단위 코드설명

| 코드 | 설명 |
|---|---|
| `#include "geometry_msgs/msg/twist.hpp"` | `geometry_msgs/msg/Twist` 메시지 인터페이스의 헤더를 포함한다. |
| `#include <memory>`, `<chrono>`, `<functional>` | 스마트 포인터, 시간 리터럴, `std::function`/`std::bind`를 위한 헤더이다. |
| `using namespace std::chrono_literals;` | `100ms` 리터럴을 쓸 수 있게 한다. |
| `void callback(..., rclcpp::Publisher<geometry_msgs::msg::Twist>::SharedPtr mypub)` | 타이머마다 속도 명령을 발행하는 콜백함수이다. |
| `auto message = geometry_msgs::msg::Twist();` | 선속도 `linear`(x, y, z)와 각속도 `angular`(x, y, z)를 가진 메시지를 만든다. 모든 값은 0으로 초기화된다. |
| `message.linear.x = 2.0;` | 거북이 앞 방향(x축)으로 2.0 m/s의 선속도를 준다. |
| `message.angular.z = 1.0;` | z축 기준 1.0 rad/s의 각속도를 준다(양수이므로 반시계 방향). |
| `RCLCPP_INFO(...)` | 발행하는 선속도와 각속도를 로그로 출력한다. |
| `mypub->publish(message);` | 속도 명령을 `/turtle1/cmd_vel` 토픽으로 발행한다. |
| `auto node = std::make_shared<rclcpp::Node>("node_pub2_3");` | 이름이 `node_pub2_3`인 노드를 동적 생성한다. |
| `auto pub = node->create_publisher<geometry_msgs::msg::Twist>("/turtle1/cmd_vel", qos_profile);` | turtlesim이 구독하는 `/turtle1/cmd_vel` 토픽으로 `Twist`를 발행하는 퍼블리셔를 생성한다. |
| `std::function<void()> fn = std::bind(callback, node, pub);` | 매개변수 있는 콜백을 `void()` 형태로 바꾼다. |
| `auto timer = node->create_wall_timer(100ms, fn);` | 100ms마다 속도 명령을 발행하는 타이머를 생성한다. |
| `rclcpp::spin(node);` | Ctrl+C를 누를 때까지 타이머 콜백을 계속 실행한다. 그래서 거북이가 무한히 회전한다. |
| `rclcpp::shutdown(); return 0;` | ROS2를 종료하고 프로그램을 끝낸다. |

**회전 반경:** 반경 r = 선속도 / 각속도 = 2.0 / 1.0 = **2.0**이다. 선속도와 각속도가 일정하므로 거북이는 반경 2.0의 원을 그린다. 한 바퀴 도는 데 2π / 1.0 ≈ 6.28초가 걸린다.

**타이머로 계속 발행하는 이유:** turtlesim은 마지막 `cmd_vel`을 받고 약 1초가 지나면 거북이를 멈춘다. 한 번만 발행하면 잠깐 움직이다 멈추므로, 타이머로 계속 발행해야 무한히 회전한다.

### CMakeLists.txt (예제에서 바뀐 부분)

```cmake
project(pub2_3)
...
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(geometry_msgs REQUIRED)

#build
add_executable(pub2_3 src/pub2_3.cpp)
ament_target_dependencies(pub2_3 rclcpp geometry_msgs)

# Install
install(TARGETS
  pub2_3
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `project(pub2_3)` | 패키지 이름을 `pub2_3`으로 지정한다. |
| `find_package(geometry_msgs REQUIRED)` | `Twist` 메시지가 들어 있는 `geometry_msgs` 패키지를 찾는다. |
| `add_executable(pub2_3 src/pub2_3.cpp)` | 실행파일 `pub2_3`을 만든다. |
| `ament_target_dependencies(pub2_3 rclcpp geometry_msgs)` | 의존 패키지를 `geometry_msgs`로 바꿔 연결한다. |
| `install(TARGETS pub2_3 ...)` | 실행파일을 `install/pub2_3/lib/pub2_3`에 설치한다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select pub2_3
source install/local_setup.bash

# 터미널 1: turtlesim 먼저 실행 (subscriber 역할)
ros2 run turtlesim turtlesim_node

# 터미널 2
ros2 run pub2_3 pub2_3
```

### 실행결과

<img width="1027" height="290" alt="image" src="https://github.com/user-attachments/assets/021c8866-7f36-4bab-ae73-c6b76826675a" />

ROS2 명령어로 확인 (새 터미널):

```bash
rqt_graph                       # /node_pub2_3 → /turtle1/cmd_vel → /turtlesim
ros2 node list                  # /node_pub2_3, /turtlesim
ros2 topic list -t              # /turtle1/cmd_vel [geometry_msgs/msg/Twist]
ros2 topic echo /turtle1/cmd_vel
ros2 topic hz /turtle1/cmd_vel
```
<img width="1084" height="300" alt="image" src="https://github.com/user-attachments/assets/9251e689-20d0-4444-8882-f751154d316d" />


### 토픽 메시지의 수신 주기와 주파수

- **주기:** 타이머를 `100ms`로 설정했으므로 turtlesim은 0.1초마다 속도 명령을 수신한다.
- **주파수:** 1 / 0.1s = **10Hz**이다. `ros2 topic hz /turtle1/cmd_vel`에서 `average rate`가 약 10.000으로 확인된다.
- 10Hz는 turtlesim의 정지 시간(약 1초)보다 훨씬 자주 명령을 보내는 것이므로, 거북이가 멈추지 않고 일정한 반경으로 계속 회전한다.
