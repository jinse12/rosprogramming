# ROS2 RCLCPP 실습과제 (Publisher 노드)

## 실습과제 1  
예제 1번을 수정하여 문자열 대신에 정수값을 0으로 초기화하고 1씩 증가시키면서 퍼블리시하는 패키지 pub1-1를 작성하시오.

### 패키지 생성

```bash
cd ~/ros2_ws/src
ros2 pkg create pub1_1 --build-type ament_cmake --dependencies rclcpp std_msgs
```

### 소스파일: `pub1_1/src/pub1_1.cpp`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/int32.hpp"
#include <memory>
#include <chrono>
int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("node_pub1_1");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    auto mypub = node->create_publisher<std_msgs::msg::Int32>("topic_pub1_1", qos_profile);
    std_msgs::msg::Int32 message;
    message.data = 0;
    rclcpp::WallRate loop_rate(1.0);
    while(rclcpp::ok())
    {
        RCLCPP_INFO(node->get_logger(), "Publish: %d", message.data);
        mypub->publish(message);
        message.data++;
        loop_rate.sleep();
    }
    rclcpp::shutdown();
    return 0;
}
```

### 라인단위 코드설명

| 코드 | 설명 |
|---|---|
| `#include "rclcpp/rclcpp.hpp"` | ROS2 C++ 라이브러리(노드, 퍼블리셔, 로깅 등)의 헤더를 포함한다. |
| `#include "std_msgs/msg/int32.hpp"` | `std_msgs/msg/Int32` 메시지 인터페이스의 헤더를 포함한다. 예제1의 `string.hpp`에서 바꾼 부분이다. |
| `#include <memory>` | `std::make_shared`, `std::shared_ptr`를 사용하기 위한 헤더이다. |
| `#include <chrono>` | 시간 관련 표준 라이브러리 헤더이다. |
| `int main(int argc, char * argv[])` | 메인 함수. `argc`는 명령행 인자 개수, `argv`는 인자 문자열 배열이다. |
| `rclcpp::init(argc, argv);` | ROS2를 초기화한다(통신, 시그널 핸들러 등). 가장 먼저 호출한다. |
| `auto node = std::make_shared<rclcpp::Node>("node_pub1_1");` | 이름이 `node_pub1_1`인 노드 객체를 힙에 동적 생성하고, 그 주소를 `shared_ptr`인 `node`에 저장한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | History를 KEEP_LAST, depth를 10으로 설정한 QoS 객체를 만든다. 나머지 옵션은 기본값이다. |
| `auto mypub = node->create_publisher<std_msgs::msg::Int32>("topic_pub1_1", qos_profile);` | `Int32` 타입 메시지를 `topic_pub1_1` 토픽으로 발행하는 퍼블리셔를 생성한다. 반환값은 `shared_ptr<Publisher<Int32>>`이다. |
| `std_msgs::msg::Int32 message;` | 전송할 `Int32` 메시지 객체를 선언한다. |
| `message.data = 0;` | 메시지의 `data` 필드(`int32`)를 0으로 초기화한다. |
| `rclcpp::WallRate loop_rate(1.0);` | 반복 주파수를 1Hz(1초 주기)로 저장하는 객체를 만든다. |
| `while(rclcpp::ok())` | Ctrl+C 같은 종료 조건이 발생할 때까지 반복한다. |
| `RCLCPP_INFO(node->get_logger(), "Publish: %d", message.data);` | 현재 발행하는 정수값을 터미널에 로그로 출력한다. 정수이므로 `%d`를 사용한다. |
| `mypub->publish(message);` | `message`를 `topic_pub1_1` 토픽으로 발행한다. |
| `message.data++;` | 다음 발행을 위해 값을 1 증가시킨다. |
| `loop_rate.sleep();` | 1초 주기에서 남은 시간만큼 대기한다. |
| `rclcpp::shutdown();` | 루프를 빠져나오면 ROS2를 종료한다. |
| `return 0;` | 프로그램을 정상 종료한다. |

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)
project(pub1_1)

# Default to C++14
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
add_executable(pub1_1 src/pub1_1.cpp)
ament_target_dependencies(pub1_1 rclcpp std_msgs)

# Install
install(TARGETS
  pub1_1
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `cmake_minimum_required(VERSION 3.16)` | 필요한 CMake 최소 버전을 지정한다. |
| `project(pub1_1)` | 프로젝트(패키지) 이름. `package.xml`의 `<name>`과 같아야 한다. |
| `set(CMAKE_CXX_STANDARD 14)` | C++ 표준이 지정되지 않았으면 C++14를 사용한다. |
| `add_compile_options(-Wall -Wextra -Wpedantic)` | GCC/Clang 컴파일러에서 경고 옵션을 켠다. |
| `find_package(ament_cmake / rclcpp / std_msgs REQUIRED)` | 빌드에 필요한 패키지를 찾는다. 없으면 빌드 에러가 난다. |
| `add_executable(pub1_1 src/pub1_1.cpp)` | `src/pub1_1.cpp`를 컴파일해 실행파일 `pub1_1`을 만든다. |
| `ament_target_dependencies(pub1_1 rclcpp std_msgs)` | 실행파일에 rclcpp, std_msgs의 include 경로와 라이브러리를 연결한다. |
| `install(TARGETS pub1_1 DESTINATION lib/${PROJECT_NAME})` | 실행파일을 `install/pub1_1/lib/pub1_1`에 설치해 `ros2 run`으로 실행할 수 있게 한다. |
| `ament_package()` | 패키지를 ROS2 패키지로 등록하고 정보를 export한다. 파일 마지막에 둔다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select pub1_1
source install/local_setup.bash
ros2 run pub1_1 pub1_1
```

### 실행결과

<img width="968" height="612" alt="image" src="https://github.com/user-attachments/assets/2cff0a0b-4c30-4399-860a-0f649cc26063" />


## 실습과제 2  
예제 1번을 수정하여 키보드로부터 실수값 3개를 입력 받아 퍼블리시하는 패키지 pub1-2를 작성하시오.

### 패키지 생성

```bash
cd ~/ros2_ws/src
ros2 pkg create pub1_2 --build-type ament_cmake --dependencies rclcpp geometry_msgs
```

### 소스파일: `pub1_2/src/pub1_2.cpp`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "geometry_msgs/msg/vector3.hpp"
#include <memory>
#include <iostream>
int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("node_pub1_2");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    auto mypub = node->create_publisher<geometry_msgs::msg::Vector3>("topic_pub1_2", qos_profile);
    geometry_msgs::msg::Vector3 message;
    while(rclcpp::ok())
    {
        std::cout << "실수값 3개 입력(x y z): ";
        if(!(std::cin >> message.x >> message.y >> message.z)) break;
        RCLCPP_INFO(node->get_logger(), "Publish: x=%.2f, y=%.2f, z=%.2f",
                    message.x, message.y, message.z);
        mypub->publish(message);
    }
    rclcpp::shutdown();
    return 0;
}
```

### 라인단위 코드설명

| 코드 | 설명 |
|---|---|
| `#include "rclcpp/rclcpp.hpp"` | ROS2 C++ 라이브러리 헤더를 포함한다. |
| `#include "geometry_msgs/msg/vector3.hpp"` | `geometry_msgs/msg/Vector3` 메시지 인터페이스의 헤더를 포함한다. |
| `#include <memory>` | `std::make_shared`를 사용하기 위한 헤더이다. |
| `#include <iostream>` | 키보드 입력(`std::cin`)과 화면 출력(`std::cout`)을 위한 헤더이다. |
| `rclcpp::init(argc, argv);` | ROS2를 초기화한다. |
| `auto node = std::make_shared<rclcpp::Node>("node_pub1_2");` | 이름이 `node_pub1_2`인 노드를 동적 생성한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | KEEP_LAST, depth 10인 QoS를 설정한다. |
| `auto mypub = node->create_publisher<geometry_msgs::msg::Vector3>("topic_pub1_2", qos_profile);` | `Vector3` 메시지를 `topic_pub1_2` 토픽으로 발행하는 퍼블리셔를 생성한다. |
| `geometry_msgs::msg::Vector3 message;` | `x`, `y`, `z`(모두 `float64`, C++에서는 `double`) 필드를 가진 메시지 객체를 선언한다. |
| `while(rclcpp::ok())` | 종료 조건이 생길 때까지 입력과 발행을 반복한다. |
| `std::cout << "실수값 3개 입력(x y z): ";` | 사용자에게 입력 안내 문구를 출력한다. |
| `if(!(std::cin >> message.x >> message.y >> message.z)) break;` | 실수 3개를 차례로 `x`, `y`, `z`에 입력받는다. 숫자가 아닌 값이나 EOF(Ctrl+D)가 들어오면 입력이 실패하므로 루프를 빠져나간다. |
| `RCLCPP_INFO(..., "Publish: x=%.2f, y=%.2f, z=%.2f", ...)` | 발행할 세 값을 소수점 둘째 자리까지 로그로 출력한다. |
| `mypub->publish(message);` | 입력받은 벡터를 `topic_pub1_2` 토픽으로 발행한다. |
| `rclcpp::shutdown();` | ROS2를 종료한다. |
| `return 0;` | 프로그램을 정상 종료한다. |

> 입력을 기다리는 동안 발행이 멈추므로, 1초 주기를 맞추는 `WallRate`는 사용하지 않았습니다. 입력할 때마다 한 번씩 발행합니다.

### CMakeLists.txt (예제1에서 바뀐 부분)

```cmake
project(pub1_2)
...
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(geometry_msgs REQUIRED)

#build
add_executable(pub1_2 src/pub1_2.cpp)
ament_target_dependencies(pub1_2 rclcpp geometry_msgs)

# Install
install(TARGETS
  pub1_2
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `project(pub1_2)` | 패키지 이름을 `pub1_2`로 지정한다. |
| `find_package(geometry_msgs REQUIRED)` | `std_msgs` 대신 `geometry_msgs` 패키지를 찾는다. |
| `add_executable(pub1_2 src/pub1_2.cpp)` | 실행파일 `pub1_2`를 만든다. |
| `ament_target_dependencies(pub1_2 rclcpp geometry_msgs)` | 의존 패키지를 `std_msgs`에서 `geometry_msgs`로 바꿔 연결한다. |
| `install(TARGETS pub1_2 ...)` | 실행파일을 `install/pub1_2/lib/pub1_2`에 설치한다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select pub1_2
source install/local_setup.bash
ros2 run pub1_2 pub1_2
```

### 실행결과

<img width="1601" height="259" alt="image" src="https://github.com/user-attachments/assets/4ca26458-3b33-495c-b832-4b6b6388231c" />


## 실습과제 3  
Turtlesim 패키지의 teleop_turtle 노드를 대신하는 패키지 pub1-3를 작성하라.

### 패키지 생성

```bash
cd ~/ros2_ws/src
ros2 pkg create pub1_3 --build-type ament_cmake --dependencies rclcpp geometry_msgs
```

### 소스파일: `pub1_3/src/pub1_3.cpp`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "geometry_msgs/msg/twist.hpp"
#include <memory>
#include <cstdio>
#include <termios.h>
#include <unistd.h>
int getch()
{
    struct termios oldt, newt;
    tcgetattr(STDIN_FILENO, &oldt);
    newt = oldt;
    newt.c_lflag &= ~(ICANON | ECHO);
    tcsetattr(STDIN_FILENO, TCSANOW, &newt);
    int ch = getchar();
    tcsetattr(STDIN_FILENO, TCSANOW, &oldt);
    return ch;
}
int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("node_pub1_3");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    auto mypub = node->create_publisher<geometry_msgs::msg::Twist>("/turtle1/cmd_vel", qos_profile);
    geometry_msgs::msg::Twist message;
    printf("f:전진 b:후진 l:좌회전 r:우회전 q:종료\n");
    while(rclcpp::ok())
    {
        int key = getch();
        message.linear.x = 0.0;
        message.angular.z = 0.0;
        if(key == 'f') message.linear.x = 2.0;
        else if(key == 'b') message.linear.x = -2.0;
        else if(key == 'l') message.angular.z = 2.0;
        else if(key == 'r') message.angular.z = -2.0;
        else if(key == 'q') break;
        else continue;
        RCLCPP_INFO(node->get_logger(), "Publish: linear.x=%.1f, angular.z=%.1f",
                    message.linear.x, message.angular.z);
        mypub->publish(message);
    }
    rclcpp::shutdown();
    return 0;
}
```

### 라인단위 코드설명

| 코드 | 설명 |
|---|---|
| `#include "rclcpp/rclcpp.hpp"` | ROS2 C++ 라이브러리 헤더를 포함한다. |
| `#include "geometry_msgs/msg/twist.hpp"` | `geometry_msgs/msg/Twist` 메시지 인터페이스의 헤더를 포함한다. |
| `#include <memory>` | `std::make_shared`를 사용하기 위한 헤더이다. |
| `#include <cstdio>` | `getchar`, `printf`를 사용하기 위한 헤더이다. |
| `#include <termios.h>` | 터미널 입력 모드를 바꾸는 `tcgetattr`, `tcsetattr`를 사용하기 위한 헤더이다. |
| `#include <unistd.h>` | 표준 입력 파일 디스크립터 `STDIN_FILENO`를 사용하기 위한 헤더이다. |
| `int getch()` | 엔터 없이 키 하나를 바로 읽어 반환하는 함수이다. |
| `struct termios oldt, newt;` | 기존 터미널 설정(`oldt`)과 바꿀 설정(`newt`)을 저장할 구조체이다. |
| `tcgetattr(STDIN_FILENO, &oldt);` | 현재 터미널 설정을 `oldt`에 저장한다. |
| `newt = oldt;` | 기존 설정을 복사한다. |
| `newt.c_lflag &= ~(ICANON \| ECHO);` | 줄 단위 입력(ICANON)과 입력 문자 표시(ECHO)를 끈다. 키를 누르는 즉시 읽히고 화면에 표시되지 않는다. |
| `tcsetattr(STDIN_FILENO, TCSANOW, &newt);` | 바꾼 설정을 즉시 적용한다. |
| `int ch = getchar();` | 키 하나를 읽는다. |
| `tcsetattr(STDIN_FILENO, TCSANOW, &oldt);` | 터미널 설정을 원래대로 되돌린다. |
| `return ch;` | 읽은 키 값을 반환한다. |
| `rclcpp::init(argc, argv);` | ROS2를 초기화한다. |
| `auto node = std::make_shared<rclcpp::Node>("node_pub1_3");` | 이름이 `node_pub1_3`인 노드를 동적 생성한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | KEEP_LAST, depth 10인 QoS를 설정한다. |
| `auto mypub = node->create_publisher<geometry_msgs::msg::Twist>("/turtle1/cmd_vel", qos_profile);` | turtlesim이 구독하는 `/turtle1/cmd_vel` 토픽으로 `Twist` 메시지를 발행하는 퍼블리셔를 생성한다. |
| `geometry_msgs::msg::Twist message;` | 선속도 `linear`(x, y, z)와 각속도 `angular`(x, y, z)를 가진 메시지 객체를 선언한다. 생성 시 모든 값은 0이다. |
| `printf("f:전진 b:후진 l:좌회전 r:우회전 q:종료\n");` | 사용법을 출력한다. |
| `while(rclcpp::ok())` | 종료 조건이 생길 때까지 키 입력과 발행을 반복한다. |
| `int key = getch();` | 키 하나를 입력받는다. |
| `message.linear.x = 0.0; message.angular.z = 0.0;` | 이전 명령이 남지 않도록 속도를 0으로 초기화한다. |
| `if(key == 'f') message.linear.x = 2.0;` | `f`: 거북이 앞 방향(x축)으로 2.0의 속도를 준다(전진). |
| `else if(key == 'b') message.linear.x = -2.0;` | `b`: 음수 속도를 준다(후진). |
| `else if(key == 'l') message.angular.z = 2.0;` | `l`: z축 기준 양의 각속도를 준다(반시계 방향, 좌회전). |
| `else if(key == 'r') message.angular.z = -2.0;` | `r`: 음의 각속도를 준다(시계 방향, 우회전). |
| `else if(key == 'q') break;` | `q`: 루프를 빠져나가 프로그램을 종료한다. |
| `else continue;` | 다른 키는 무시하고 다음 입력을 기다린다. |
| `RCLCPP_INFO(...)` | 발행할 선속도와 각속도를 로그로 출력한다. |
| `mypub->publish(message);` | 속도 명령을 `/turtle1/cmd_vel` 토픽으로 발행한다. turtlesim은 이 메시지를 받아 약 1초 동안 그 속도로 움직인다. |
| `rclcpp::shutdown();` | ROS2를 종료한다. |
| `return 0;` | 프로그램을 정상 종료한다. |

### CMakeLists.txt (예제1에서 바뀐 부분)

```cmake
project(pub1_3)
...
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(geometry_msgs REQUIRED)

#build
add_executable(pub1_3 src/pub1_3.cpp)
ament_target_dependencies(pub1_3 rclcpp geometry_msgs)

# Install
install(TARGETS
  pub1_3
  DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `project(pub1_3)` | 패키지 이름을 `pub1_3`으로 지정한다. |
| `find_package(geometry_msgs REQUIRED)` | `Twist` 메시지가 들어 있는 `geometry_msgs` 패키지를 찾는다. |
| `add_executable(pub1_3 src/pub1_3.cpp)` | 실행파일 `pub1_3`을 만든다. |
| `ament_target_dependencies(pub1_3 rclcpp geometry_msgs)` | 의존 패키지를 `std_msgs`에서 `geometry_msgs`로 바꿔 연결한다. |
| `install(TARGETS pub1_3 ...)` | 실행파일을 `install/pub1_3/lib/pub1_3`에 설치한다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select pub1_3
source install/local_setup.bash

# 터미널 1: subscriber 역할
ros2 run turtlesim turtlesim_node

# 터미널 2: 조종 노드
ros2 run pub1_3 pub1_3
```

### 실행결과

f:전진  
<img width="1522" height="581" alt="image" src="https://github.com/user-attachments/assets/20513cf5-3cb3-4ae2-8afd-a61fa76316cd" />

b:후진  
<img width="1503" height="586" alt="image" src="https://github.com/user-attachments/assets/d8c291e0-ba75-4425-9f53-5557be11264b" />

l:좌회전  
<img width="1517" height="586" alt="image" src="https://github.com/user-attachments/assets/c478caaa-b9ca-4aab-8414-ec4880e02f58" />

r:우회전  
<img width="1501" height="580" alt="image" src="https://github.com/user-attachments/assets/bf1f1c6a-2f11-4d68-b4f2-ee1bbae5a88d" />
