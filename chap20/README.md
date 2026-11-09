# ROS2 RCLCPP 실습과제 (Publisher / Subscriber)

> 패키지 이름: ROS2 패키지 이름에는 `-`를 쓸 수 없습니다(소문자, 숫자, `_`만 가능). 그래서 `psub1-1`, `psub1-2` 대신 **`psub1_1`, `psub1_2`**로 만들었습니다.

---

## 실습과제 1. 타이머로 100ms마다 문자열 발행·구독 (psub1_1)

### 패키지 생성

```bash
cd ~/ros2_ws/src
ros2 pkg create psub1_1 --build-type ament_cmake --dependencies rclcpp std_msgs
```

### Publisher 소스파일: `psub1_1/src/pub.cpp`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
#include <memory>
#include <chrono>
#include <functional>
#include <string>
using namespace std::chrono_literals;
void callback(rclcpp::Node::SharedPtr node,
              rclcpp::Publisher<std_msgs::msg::String>::SharedPtr mypub)
{
    static int count;
    auto message = std_msgs::msg::String();
    message.data = "Hello world! " + std::to_string(count++);
    RCLCPP_INFO(node->get_logger(), "Publish: %s", message.data.c_str());
    mypub->publish(message);
}
int main(int argc, char* argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("node_pub1");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    auto mypub = node->create_publisher<std_msgs::msg::String>("topic_pub1", qos_profile);
    std::function<void()> fn = std::bind(callback, node, mypub);
    auto timer = node->create_wall_timer(100ms, fn);
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

### Publisher 라인단위 코드설명

| 코드 | 설명 |
|---|---|
| `#include "rclcpp/rclcpp.hpp"` | ROS2 C++ 라이브러리(노드, 퍼블리셔, 타이머, 로깅 등)의 헤더를 포함한다. |
| `#include "std_msgs/msg/string.hpp"` | `std_msgs/msg/String` 메시지 인터페이스의 헤더를 포함한다. |
| `#include <memory>` | `std::make_shared`, `std::shared_ptr`를 사용하기 위한 헤더이다. |
| `#include <chrono>` | 시간 단위를 사용하기 위한 헤더이다. |
| `#include <functional>` | `std::function`, `std::bind`를 사용하기 위한 헤더이다. |
| `#include <string>` | `std::to_string`을 사용하기 위한 헤더이다. |
| `using namespace std::chrono_literals;` | `100ms` 같은 시간 리터럴을 쓸 수 있게 한다. |
| `void callback(rclcpp::Node::SharedPtr node, rclcpp::Publisher<std_msgs::msg::String>::SharedPtr mypub)` | 타이머 이벤트마다 호출되는 콜백함수. 로그용 노드와 발행용 퍼블리셔를 매개변수로 받는다. |
| `static int count;` | 정적 지역변수. 0으로 자동 초기화되고 호출 사이에 값이 유지된다. |
| `auto message = std_msgs::msg::String();` | 전송할 문자열 메시지 객체를 만든다. |
| `message.data = "Hello world! " + std::to_string(count++);` | "Hello world! " 뒤에 번호를 붙여 저장하고, `count`를 1 증가시킨다. 메시지가 바뀌므로 수신 여부를 확인하기 쉽다. |
| `RCLCPP_INFO(node->get_logger(), "Publish: %s", message.data.c_str());` | 발행하는 문자열을 로그로 출력한다. `%s`에는 C 문자열이 필요하므로 `c_str()`을 쓴다. |
| `mypub->publish(message);` | 메시지를 `topic_pub1` 토픽으로 발행한다. |
| `rclcpp::init(argc, argv);` | ROS2를 초기화한다. |
| `auto node = std::make_shared<rclcpp::Node>("node_pub1");` | 이름이 `node_pub1`인 노드를 동적 생성한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | KEEP_LAST, depth 10인 QoS를 설정한다. |
| `auto mypub = node->create_publisher<std_msgs::msg::String>("topic_pub1", qos_profile);` | `String` 메시지를 `topic_pub1` 토픽으로 발행하는 퍼블리셔를 생성한다. |
| `std::function<void()> fn = std::bind(callback, node, mypub);` | 타이머는 매개변수 없는 `void()` 함수만 받으므로, `std::bind`로 `node`, `mypub`을 묶어 매개변수 없는 함수로 바꾼다. |
| `auto timer = node->create_wall_timer(100ms, fn);` | 100ms마다 `fn`을 호출하는 타이머를 생성한다. |
| `rclcpp::spin(node);` | 타이머 이벤트를 처리하며 노드를 계속 실행한다. Ctrl+C를 누르면 리턴한다. |
| `rclcpp::shutdown(); return 0;` | ROS2를 종료하고 프로그램을 끝낸다. |

### Subscriber 소스파일: `psub1_1/src/sub.cpp`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
#include <memory>
#include <functional>
void mysub_callback(rclcpp::Node::SharedPtr node,
                    const std_msgs::msg::String::SharedPtr msg)
{
    RCLCPP_INFO(node->get_logger(), "Received message: %s", msg->data.c_str());
}
int main(int argc, char* argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("node_sub1");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    std::function<void(const std_msgs::msg::String::SharedPtr)> fn =
        std::bind(mysub_callback, node, std::placeholders::_1);
    auto mysub = node->create_subscription<std_msgs::msg::String>("topic_pub1", qos_profile, fn);
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

### Subscriber 라인단위 코드설명

| 코드 | 설명 |
|---|---|
| `#include "rclcpp/rclcpp.hpp"` | ROS2 C++ 라이브러리 헤더를 포함한다. |
| `#include "std_msgs/msg/string.hpp"` | 구독할 `String` 메시지의 헤더를 포함한다. |
| `#include <memory>`, `<functional>` | 스마트 포인터와 `std::function`, `std::bind`, `std::placeholders`를 위한 헤더이다. |
| `void mysub_callback(rclcpp::Node::SharedPtr node, const std_msgs::msg::String::SharedPtr msg)` | 토픽 메시지를 수신할 때마다 호출되는 콜백함수. 수신한 메시지는 `shared_ptr`(`msg`)로 전달된다. |
| `RCLCPP_INFO(node->get_logger(), "Received message: %s", msg->data.c_str());` | 수신한 문자열을 출력한다. `msg`가 포인터이므로 `->`로 `data`에 접근한다. |
| `auto node = std::make_shared<rclcpp::Node>("node_sub1");` | 이름이 `node_sub1`인 노드를 동적 생성한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | KEEP_LAST, depth 10인 QoS를 설정한다. Publisher와 호환되어야 통신된다. |
| `std::function<void(const std_msgs::msg::String::SharedPtr)> fn = std::bind(mysub_callback, node, std::placeholders::_1);` | `create_subscription`의 콜백은 메시지 하나만 받는 `void(MessageT::SharedPtr)` 형태여야 한다. `node`는 미리 묶고(bind), 메시지 자리는 `_1`로 비워 둔다. 그러면 매개변수 2개인 함수가 1개인 함수 `fn`으로 바뀐다. `fn(msg)`를 호출하면 `mysub_callback(node, msg)`가 실행된다. |
| `auto mysub = node->create_subscription<std_msgs::msg::String>("topic_pub1", qos_profile, fn);` | `topic_pub1` 토픽을 구독하는 Subscription 객체를 생성하고, 메시지가 수신되면 `fn`이 호출되도록 등록한다. |
| `rclcpp::spin(node);` | 메시지 수신 이벤트를 기다리다가 수신되면 콜백을 호출한다. |
| `rclcpp::shutdown(); return 0;` | ROS2를 종료하고 프로그램을 끝낸다. |

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)
project(psub1_1)

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
add_executable(pub src/pub.cpp)
ament_target_dependencies(pub rclcpp std_msgs)

add_executable(sub src/sub.cpp)
ament_target_dependencies(sub rclcpp std_msgs)

# Install
install(TARGETS pub sub DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `cmake_minimum_required(VERSION 3.16)` | 필요한 CMake 최소 버전을 지정한다. |
| `project(psub1_1)` | 패키지 이름. `package.xml`의 `<name>`과 같아야 한다. |
| `set(CMAKE_CXX_STANDARD 14)` | C++14를 사용한다(`100ms` 리터럴은 C++14부터 지원). |
| `add_compile_options(-Wall -Wextra -Wpedantic)` | GCC/Clang에서 경고 옵션을 켠다. |
| `find_package(ament_cmake REQUIRED)` | `ament_target_dependencies()`, `ament_package()`의 정의를 가져온다. |
| `find_package(rclcpp / std_msgs REQUIRED)` | 소스코드에 필요한 ROS2 라이브러리와 메시지 라이브러리를 찾는다. |
| `add_executable(pub src/pub.cpp)` | `pub.cpp`로 실행파일 `pub`을 만든다. |
| `ament_target_dependencies(pub rclcpp std_msgs)` | `pub`에 rclcpp, std_msgs를 연결한다. |
| `add_executable(sub src/sub.cpp)` | `sub.cpp`로 실행파일 `sub`를 만든다. 한 패키지에 노드 2개가 들어간다. |
| `ament_target_dependencies(sub rclcpp std_msgs)` | `sub`에 rclcpp, std_msgs를 연결한다. |
| `install(TARGETS pub sub DESTINATION lib/${PROJECT_NAME})` | 두 실행파일을 `install/psub1_1/lib/psub1_1`에 설치한다. |
| `ament_package()` | 패키지를 ROS2 패키지로 등록한다. 파일 마지막에 둔다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select psub1_1
source install/local_setup.bash

# 터미널 1
ros2 run psub1_1 pub
# 터미널 2 (새 터미널)
ros2 run psub1_1 sub
```

### 실행결과

> 예상 출력입니다. 본인 터미널 실행 화면으로 교체하세요.

Publisher:
```
[INFO] [...] [node_pub1]: Publish: Hello world! 0
[INFO] [...] [node_pub1]: Publish: Hello world! 1
[INFO] [...] [node_pub1]: Publish: Hello world! 2
```

Subscriber:
```
[INFO] [...] [node_sub1]: Received message: Hello world! 0
[INFO] [...] [node_sub1]: Received message: Hello world! 1
[INFO] [...] [node_sub1]: Received message: Hello world! 2
```

ROS2 명령어로 확인 (새 터미널):

```bash
rqt_graph                       # /node_pub1 → /topic_pub1 → /node_sub1
ros2 pkg list | grep psub1_1
ros2 node list                  # /node_pub1, /node_sub1
ros2 topic list -t              # /topic_pub1 [std_msgs/msg/String]
ros2 topic echo /topic_pub1
ros2 topic hz /topic_pub1       # average rate: 약 10.000 (주기 0.1s)
ros2 topic bw /topic_pub1
```

---

## 실습과제 2. 타이머로 50ms마다 정수 발행·구독 (psub1_2)

### 패키지 생성

```bash
cd ~/ros2_ws/src
ros2 pkg create psub1_2 --build-type ament_cmake --dependencies rclcpp std_msgs
```

### Publisher 소스파일: `psub1_2/src/pub.cpp`

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
    auto node = std::make_shared<rclcpp::Node>("node_pub1");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    auto mypub = node->create_publisher<std_msgs::msg::Int32>("topic_pub1", qos_profile);
    std::function<void()> fn = std::bind(callback, node, mypub);
    auto timer = node->create_wall_timer(50ms, fn);
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

### Publisher 라인단위 코드설명

과제 1의 `pub.cpp`와 같은 구조이며, 바뀐 줄만 설명한다.

| 코드 | 설명 |
|---|---|
| `#include "std_msgs/msg/int32.hpp"` | `string.hpp` 대신 `std_msgs/msg/Int32` 메시지의 헤더를 포함한다. |
| `rclcpp::Publisher<std_msgs::msg::Int32>::SharedPtr mypub` | 콜백이 받는 퍼블리셔의 메시지 타입을 `Int32`로 바꾼다. |
| `static int count;` | 0으로 자동 초기화되는 정적 지역변수. 호출 사이에 값이 유지된다. |
| `auto message = std_msgs::msg::Int32();` | `int32` 필드 `data`를 가진 메시지 객체를 만든다. |
| `message.data = count++;` | 현재 값을 저장한 뒤 1 증가시킨다. 0, 1, 2, … 순서로 발행된다. |
| `RCLCPP_INFO(node->get_logger(), "Publish: %d", message.data);` | 정수이므로 `%d`로 출력한다. |
| `auto mypub = node->create_publisher<std_msgs::msg::Int32>("topic_pub1", qos_profile);` | `Int32` 메시지를 발행하는 퍼블리셔를 생성한다. |
| `auto timer = node->create_wall_timer(50ms, fn);` | 주기를 `50ms`로 설정해 1초에 20번(20Hz) 발행한다. |

나머지 줄(`init`, 노드 생성, QoS, `std::bind`, `spin`, `shutdown`)은 과제 1과 같다.

### Subscriber 소스파일: `psub1_2/src/sub.cpp`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/int32.hpp"
#include <memory>
#include <functional>
void mysub_callback(rclcpp::Node::SharedPtr node,
                    const std_msgs::msg::Int32::SharedPtr msg)
{
    RCLCPP_INFO(node->get_logger(), "Received message: %d", msg->data);
}
int main(int argc, char* argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("node_sub1");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    std::function<void(const std_msgs::msg::Int32::SharedPtr)> fn =
        std::bind(mysub_callback, node, std::placeholders::_1);
    auto mysub = node->create_subscription<std_msgs::msg::Int32>("topic_pub1", qos_profile, fn);
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

### Subscriber 라인단위 코드설명

과제 1의 `sub.cpp`와 같은 구조이며, 바뀐 줄만 설명한다.

| 코드 | 설명 |
|---|---|
| `#include "std_msgs/msg/int32.hpp"` | 구독할 `Int32` 메시지의 헤더를 포함한다. |
| `const std_msgs::msg::Int32::SharedPtr msg` | 콜백이 받는 메시지 타입을 `Int32`의 `shared_ptr`로 바꾼다. |
| `RCLCPP_INFO(..., "Received message: %d", msg->data);` | 수신한 정수를 `%d`로 출력한다. 정수이므로 `c_str()`이 필요 없다. |
| `std::function<void(const std_msgs::msg::Int32::SharedPtr)> fn = std::bind(...)` | 콜백 형태를 `void(Int32::SharedPtr)`로 맞추기 위해 `node`는 묶고 메시지 자리는 `_1`로 비운다. |
| `node->create_subscription<std_msgs::msg::Int32>("topic_pub1", qos_profile, fn);` | `Int32` 타입으로 `topic_pub1` 토픽을 구독한다. Publisher와 메시지 타입이 같아야 통신된다. |

### CMakeLists.txt

과제 1과 같고 이름만 바뀐다. 메시지 패키지가 그대로 `std_msgs`이므로 의존성은 수정하지 않는다.

```cmake
cmake_minimum_required(VERSION 3.16)
project(psub1_2)

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
add_executable(pub src/pub.cpp)
ament_target_dependencies(pub rclcpp std_msgs)

add_executable(sub src/sub.cpp)
ament_target_dependencies(sub rclcpp std_msgs)

# Install
install(TARGETS pub sub DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `project(psub1_2)` | 패키지 이름을 `psub1_2`로 지정한다. |
| `add_executable(pub ...)`, `add_executable(sub ...)` | Publisher, Subscriber 실행파일 2개를 만든다. |
| `install(TARGETS pub sub ...)` | 두 실행파일을 `install/psub1_2/lib/psub1_2`에 설치한다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select psub1_2
source install/local_setup.bash

# 터미널 1
ros2 run psub1_2 pub
# 터미널 2 (새 터미널)
ros2 run psub1_2 sub
```

### 실행결과

> 예상 출력입니다. 본인 터미널 실행 화면으로 교체하세요.

Publisher:
```
[INFO] [...] [node_pub1]: Publish: 0
[INFO] [...] [node_pub1]: Publish: 1
[INFO] [...] [node_pub1]: Publish: 2
```

Subscriber:
```
[INFO] [...] [node_sub1]: Received message: 0
[INFO] [...] [node_sub1]: Received message: 1
[INFO] [...] [node_sub1]: Received message: 2
```

Subscriber를 Publisher보다 늦게 실행하면, 수신이 0부터가 아니라 실행한 시점의 값부터 시작한다(Durability가 VOLATILE이라 지난 메시지는 받지 않음).

ROS2 명령어로 확인 (새 터미널):

```bash
rqt_graph                       # /node_pub1 → /topic_pub1 → /node_sub1
ros2 node list                  # /node_pub1, /node_sub1
ros2 topic list -t              # /topic_pub1 [std_msgs/msg/Int32]
ros2 topic echo /topic_pub1
ros2 topic hz /topic_pub1       # average rate: 약 20.000 (주기 0.05s)
ros2 topic bw /topic_pub1
```
