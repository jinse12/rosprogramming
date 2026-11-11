# ROS2 RCLCPP 실습과제 (클래스를 이용한 패키지)

> 패키지 이름: ROS2 패키지 이름에는 `-`를 쓸 수 없습니다(소문자, 숫자, `_`만 가능). 그래서 `psub2-1`, `psub2-2` 대신 **`psub2_1`, `psub2_2`**로 만들었습니다.
>
> 파일 위치: 헤더파일은 `<패키지명>/include/<패키지명>/`, 소스파일은 `<패키지명>/src/`에 저장합니다.

---

## 실습과제 1. 50ms마다 정수를 1씩 증가시켜 발행·구독 (psub2_1)

### 패키지 생성 및 구성

```bash
cd ~/ros2_ws/src
ros2 pkg create psub2_1 --build-type ament_cmake --dependencies rclcpp std_msgs
```

```
psub2_1
├── CMakeLists.txt
├── package.xml
├── include/psub2_1
│   ├── pub.hpp      # Publisher 클래스 선언부
│   └── sub.hpp      # Subscriber 클래스 선언부
└── src
    ├── pub.cpp      # Publisher 클래스 구현부
    ├── pmain.cpp    # Publisher 메인함수
    ├── sub.cpp      # Subscriber 클래스 구현부
    └── smain.cpp    # Subscriber 메인함수
```

### `include/psub2_1/pub.hpp`

```cpp
#ifndef _PUB_HPP_
#define _PUB_HPP_
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/int32.hpp"
#include <memory>
#include <chrono>
using namespace std::chrono_literals;
class Pub : public rclcpp::Node
{
private:
    rclcpp::TimerBase::SharedPtr timer_;
    rclcpp::Publisher<std_msgs::msg::Int32>::SharedPtr pub_;
    int count_;
    void publish_msg();
public:
    Pub();
};
#endif //_PUB_HPP_
```

| 코드 | 설명 |
|---|---|
| `#ifndef _PUB_HPP_` / `#define _PUB_HPP_` / `#endif` | 헤더 가드. 같은 헤더가 여러 번 포함되어 클래스가 중복 정의되는 것을 막는다. |
| `#include "rclcpp/rclcpp.hpp"` | ROS2 C++ 라이브러리 헤더를 포함한다. |
| `#include "std_msgs/msg/int32.hpp"` | 예제의 `string.hpp` 대신 `Int32` 메시지 헤더를 포함한다. |
| `#include <memory>`, `<chrono>` | 스마트 포인터와 시간 단위를 위한 헤더이다. |
| `using namespace std::chrono_literals;` | `50ms` 같은 시간 리터럴을 쓸 수 있게 한다. |
| `class Pub : public rclcpp::Node` | `rclcpp::Node`를 public 상속받아 노드의 모든 기능(퍼블리셔·타이머 생성, 로거 등)을 그대로 쓰는 Publisher 노드 클래스를 선언한다. |
| `rclcpp::TimerBase::SharedPtr timer_;` | 타이머 객체를 가리키는 멤버변수. 멤버로 저장해야 생성자가 끝나도 타이머가 사라지지 않는다. |
| `rclcpp::Publisher<std_msgs::msg::Int32>::SharedPtr pub_;` | `Int32` 메시지를 발행하는 퍼블리셔를 가리키는 멤버변수이다. |
| `int count_;` | 발행할 정수값을 저장하는 멤버변수. 이전 과제의 `static int count`를 대신한다. |
| `void publish_msg();` | 타이머 콜백으로 쓸 private 멤버함수이다. |
| `Pub();` | public 생성자. 퍼블리셔와 타이머를 생성한다. |

### `src/pub.cpp`

```cpp
#include "psub2_1/pub.hpp"

Pub::Pub() : Node("mypub"), count_(0)
{
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    pub_ = this->create_publisher<std_msgs::msg::Int32>("mytopic", qos_profile);
    timer_ = this->create_wall_timer(50ms, std::bind(&Pub::publish_msg, this));
}

void Pub::publish_msg()
{
    auto msg = std_msgs::msg::Int32();
    msg.data = count_++;
    RCLCPP_INFO(this->get_logger(), "Published message: %d", msg.data);
    pub_->publish(msg);
}
```

| 코드 | 설명 |
|---|---|
| `#include "psub2_1/pub.hpp"` | 클래스 선언부를 포함한다. CMake의 `include_directories(include)` 덕분에 `include/` 아래 경로로 찾는다. |
| `Pub::Pub() : Node("mypub"), count_(0)` | 생성자. 초기화 리스트에서 기본 클래스 `Node`의 생성자를 호출해 노드 이름을 `mypub`으로 정하고, `count_`를 0으로 초기화한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | KEEP_LAST, depth 10인 QoS를 설정한다. |
| `pub_ = this->create_publisher<std_msgs::msg::Int32>("mytopic", qos_profile);` | 자기 자신(노드)의 `create_publisher`로 `mytopic` 토픽에 `Int32`를 발행하는 퍼블리셔를 만들어 멤버 `pub_`에 저장한다. |
| `timer_ = this->create_wall_timer(50ms, std::bind(&Pub::publish_msg, this));` | 50ms마다 콜백을 호출하는 타이머를 만든다. 멤버함수는 호출할 객체가 필요하므로 `std::bind`로 `this`를 묶어 매개변수 없는 `void()` 함수로 바꿔 전달한다. |
| `void Pub::publish_msg()` | 타이머 콜백 멤버함수의 구현이다. |
| `auto msg = std_msgs::msg::Int32();` | 발행할 `Int32` 메시지를 만든다. |
| `msg.data = count_++;` | 현재 값을 저장하고 `count_`를 1 증가시킨다. 0, 1, 2, … 순서로 발행된다. |
| `RCLCPP_INFO(this->get_logger(), "Published message: %d", msg.data);` | 상속받은 `get_logger()`로 발행 값을 출력한다. 정수이므로 `%d`를 쓴다. |
| `pub_->publish(msg);` | 메시지를 `mytopic` 토픽으로 발행한다. |

### `src/pmain.cpp`

```cpp
#include "psub2_1/pub.hpp"

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<Pub>();
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

| 코드 | 설명 |
|---|---|
| `#include "psub2_1/pub.hpp"` | `Pub` 클래스 선언부를 포함한다. |
| `rclcpp::init(argc, argv);` | ROS2를 초기화한다. |
| `auto node = std::make_shared<Pub>();` | `Pub` 노드 객체를 동적 생성한다. 생성자에서 퍼블리셔와 타이머가 만들어진다. `Pub`이 `Node`를 상속하므로 `spin`에 그대로 넘길 수 있다. |
| `rclcpp::spin(node);` | 타이머 이벤트를 처리하며 노드를 계속 실행한다. |
| `rclcpp::shutdown(); return 0;` | ROS2를 종료하고 프로그램을 끝낸다. |

### `include/psub2_1/sub.hpp`

```cpp
#ifndef _SUB_HPP_
#define _SUB_HPP_
#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/int32.hpp"
#include <functional>
#include <memory>
using std::placeholders::_1;
class Sub : public rclcpp::Node
{
private:
    rclcpp::Subscription<std_msgs::msg::Int32>::SharedPtr sub_;
    void subscribe_msg(const std_msgs::msg::Int32::SharedPtr msg) const;
public:
    Sub();
};
#endif //_SUB_HPP_
```

| 코드 | 설명 |
|---|---|
| `#ifndef _SUB_HPP_` / `#define` / `#endif` | 헤더 가드. |
| `#include "std_msgs/msg/int32.hpp"` | 구독할 `Int32` 메시지 헤더를 포함한다. |
| `#include <functional>`, `<memory>` | `std::bind`, `std::placeholders`, 스마트 포인터를 위한 헤더이다. |
| `using std::placeholders::_1;` | `std::bind`에서 첫 번째 인자 자리를 `_1`로 짧게 쓸 수 있게 한다. |
| `class Sub : public rclcpp::Node` | `rclcpp::Node`를 상속받은 Subscriber 노드 클래스이다. |
| `rclcpp::Subscription<std_msgs::msg::Int32>::SharedPtr sub_;` | 구독 객체를 저장하는 멤버변수. 멤버로 저장해야 구독이 유지된다. |
| `void subscribe_msg(const std_msgs::msg::Int32::SharedPtr msg) const;` | 메시지 수신 콜백 멤버함수. 멤버변수를 바꾸지 않으므로 `const` 멤버함수로 선언한다. |
| `Sub();` | 생성자. 구독 객체를 생성한다. |

### `src/sub.cpp`

```cpp
#include "psub2_1/sub.hpp"

void Sub::subscribe_msg(const std_msgs::msg::Int32::SharedPtr msg) const
{
    RCLCPP_INFO(this->get_logger(), "Received message: %d", msg->data);
}

Sub::Sub() : Node("mysub")
{
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    sub_ = this->create_subscription<std_msgs::msg::Int32>("mytopic", qos_profile,
        std::bind(&Sub::subscribe_msg, this, _1));
}
```

| 코드 | 설명 |
|---|---|
| `#include "psub2_1/sub.hpp"` | `Sub` 클래스 선언부를 포함한다. |
| `void Sub::subscribe_msg(...) const` | 수신 콜백의 구현이다. |
| `RCLCPP_INFO(this->get_logger(), "Received message: %d", msg->data);` | 수신한 정수를 출력한다. `msg`가 `shared_ptr`이므로 `->`로 접근한다. |
| `Sub::Sub() : Node("mysub")` | 기본 클래스 생성자를 호출해 노드 이름을 `mysub`으로 정한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | KEEP_LAST, depth 10인 QoS를 설정한다. |
| `sub_ = this->create_subscription<std_msgs::msg::Int32>("mytopic", qos_profile, std::bind(&Sub::subscribe_msg, this, _1));` | `mytopic` 토픽을 `Int32` 타입으로 구독한다. 콜백은 `void(MessageT::SharedPtr)` 형태여야 하므로 `std::bind`로 `this`를 묶고 메시지 자리는 `_1`로 비워 둔다. |

### `src/smain.cpp`

```cpp
#include "psub2_1/sub.hpp"

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<Sub>();
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

| 코드 | 설명 |
|---|---|
| `auto node = std::make_shared<Sub>();` | `Sub` 노드 객체를 동적 생성한다. 생성자에서 구독이 시작된다. |
| `rclcpp::spin(node);` | 메시지 수신 이벤트를 기다리다가 수신되면 콜백을 호출한다. |
| 나머지 | `init`, `shutdown`, `return 0`은 pmain.cpp와 같다. |

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)
project(psub2_1)

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

# 사용자 작성 헤더파일 경로 추가
include_directories(include)

#build
add_executable(pub src/pub.cpp src/pmain.cpp)
ament_target_dependencies(pub rclcpp std_msgs)

add_executable(sub src/sub.cpp src/smain.cpp)
ament_target_dependencies(sub rclcpp std_msgs)

# Install
install(TARGETS pub sub DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `project(psub2_1)` | 패키지 이름. `package.xml`의 `<name>`과 같아야 한다. |
| `set(CMAKE_CXX_STANDARD 14)` | C++14를 사용한다. |
| `add_compile_options(...)` | GCC/Clang 경고 옵션을 켠다. |
| `find_package(ament_cmake / rclcpp / std_msgs REQUIRED)` | ament 명령 정의와 필요한 라이브러리를 찾는다. |
| `include_directories(include)` | `include/` 폴더를 헤더 검색 경로에 추가한다. 이 줄이 있어야 `#include "psub2_1/pub.hpp"`가 동작한다. |
| `add_executable(pub src/pub.cpp src/pmain.cpp)` | 클래스 구현부와 메인함수 두 소스를 함께 컴파일해 실행파일 `pub`을 만든다. |
| `add_executable(sub src/sub.cpp src/smain.cpp)` | 같은 방식으로 실행파일 `sub`을 만든다. |
| `ament_target_dependencies(pub/sub rclcpp std_msgs)` | 각 실행파일에 의존 라이브러리를 연결한다. |
| `install(TARGETS pub sub DESTINATION lib/${PROJECT_NAME})` | 두 실행파일을 `install/psub2_1/lib/psub2_1`에 설치한다. |
| `ament_package()` | 패키지를 ROS2 패키지로 등록한다. 마지막 줄에 둔다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select psub2_1
source install/local_setup.bash

# 터미널 1
ros2 run psub2_1 pub
# 터미널 2
ros2 run psub2_1 sub
```

### 실행결과

> 예상 출력입니다. 본인 터미널 실행 화면으로 교체하세요.

```
[INFO] [...] [mypub]: Published message: 0
[INFO] [...] [mypub]: Published message: 1
[INFO] [...] [mypub]: Published message: 2
```

```
[INFO] [...] [mysub]: Received message: 15
[INFO] [...] [mysub]: Received message: 16
[INFO] [...] [mysub]: Received message: 17
```

(sub를 나중에 실행하면 그 시점의 값부터 수신한다.)

```bash
rqt_graph                       # /mypub → /mytopic → /mysub
ros2 node list                  # /mypub, /mysub
ros2 topic list -t              # /mytopic [std_msgs/msg/Int32]
ros2 topic echo /mytopic
ros2 topic hz /mytopic          # average rate: 약 20.000 (주기 0.05s)
ros2 topic bw /mytopic
```

---

## 실습과제 2. 실수 3개를 1초마다 1씩 증가시켜 발행·구독 (psub2_2)

### 패키지 생성

```bash
cd ~/ros2_ws/src
ros2 pkg create psub2_2 --build-type ament_cmake --dependencies rclcpp geometry_msgs
```

파일 구성은 과제 1과 같다(`include/psub2_2/pub.hpp`, `sub.hpp`, `src/pub.cpp`, `pmain.cpp`, `sub.cpp`, `smain.cpp`).

### `include/psub2_2/pub.hpp`

```cpp
#ifndef _PUB_HPP_
#define _PUB_HPP_
#include "rclcpp/rclcpp.hpp"
#include "geometry_msgs/msg/vector3.hpp"
#include <memory>
#include <chrono>
using namespace std::chrono_literals;
class Pub : public rclcpp::Node
{
private:
    rclcpp::TimerBase::SharedPtr timer_;
    rclcpp::Publisher<geometry_msgs::msg::Vector3>::SharedPtr pub_;
    double x_, y_, z_;
    void publish_msg();
public:
    Pub();
};
#endif //_PUB_HPP_
```

| 코드 | 설명 |
|---|---|
| `#include "geometry_msgs/msg/vector3.hpp"` | `Vector3` 메시지 헤더를 포함한다. |
| `rclcpp::Publisher<geometry_msgs::msg::Vector3>::SharedPtr pub_;` | `Vector3`를 발행하는 퍼블리셔 멤버변수이다. |
| `double x_, y_, z_;` | 발행할 실수값 3개를 저장하는 멤버변수. `Vector3`의 x, y, z가 `float64`(double)이므로 같은 타입을 쓴다. |
| 나머지 | 헤더 가드, `Node` 상속, `timer_`, `publish_msg()`, 생성자는 과제 1과 같다. |

### `src/pub.cpp`

```cpp
#include "psub2_2/pub.hpp"

Pub::Pub() : Node("mypub"), x_(0.0), y_(0.0), z_(0.0)
{
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    pub_ = this->create_publisher<geometry_msgs::msg::Vector3>("mytopic", qos_profile);
    timer_ = this->create_wall_timer(1s, std::bind(&Pub::publish_msg, this));
}

void Pub::publish_msg()
{
    auto msg = geometry_msgs::msg::Vector3();
    msg.x = x_++;
    msg.y = y_++;
    msg.z = z_++;
    RCLCPP_INFO(this->get_logger(), "Published message: x=%.1f, y=%.1f, z=%.1f",
                msg.x, msg.y, msg.z);
    pub_->publish(msg);
}
```

| 코드 | 설명 |
|---|---|
| `Pub::Pub() : Node("mypub"), x_(0.0), y_(0.0), z_(0.0)` | 노드 이름을 `mypub`으로 정하고, 세 실수값을 0으로 초기화한다. |
| `pub_ = this->create_publisher<geometry_msgs::msg::Vector3>("mytopic", qos_profile);` | `mytopic` 토픽에 `Vector3`를 발행하는 퍼블리셔를 만든다. |
| `timer_ = this->create_wall_timer(1s, std::bind(&Pub::publish_msg, this));` | 1초마다 `publish_msg`를 호출하는 타이머를 만든다. |
| `auto msg = geometry_msgs::msg::Vector3();` | 발행할 `Vector3` 메시지를 만든다. |
| `msg.x = x_++; msg.y = y_++; msg.z = z_++;` | 현재 값을 메시지에 저장하고 각각 1씩 증가시킨다. 0.0, 1.0, 2.0, … 순서로 발행된다. |
| `RCLCPP_INFO(..., "x=%.1f, y=%.1f, z=%.1f", ...)` | 실수 세 개를 소수점 첫째 자리까지 출력한다. |
| `pub_->publish(msg);` | 메시지를 발행한다. |

### `src/pmain.cpp`

```cpp
#include "psub2_2/pub.hpp"

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<Pub>();
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

과제 1의 pmain.cpp와 같고, 포함하는 헤더 경로만 `psub2_2/pub.hpp`로 바뀐다.

### `include/psub2_2/sub.hpp`

```cpp
#ifndef _SUB_HPP_
#define _SUB_HPP_
#include "rclcpp/rclcpp.hpp"
#include "geometry_msgs/msg/vector3.hpp"
#include <functional>
#include <memory>
using std::placeholders::_1;
class Sub : public rclcpp::Node
{
private:
    rclcpp::Subscription<geometry_msgs::msg::Vector3>::SharedPtr sub_;
    void subscribe_msg(const geometry_msgs::msg::Vector3::SharedPtr msg) const;
public:
    Sub();
};
#endif //_SUB_HPP_
```

| 코드 | 설명 |
|---|---|
| `#include "geometry_msgs/msg/vector3.hpp"` | 구독할 `Vector3` 메시지 헤더를 포함한다. |
| `rclcpp::Subscription<geometry_msgs::msg::Vector3>::SharedPtr sub_;` | `Vector3` 구독 객체 멤버변수이다. |
| `void subscribe_msg(const geometry_msgs::msg::Vector3::SharedPtr msg) const;` | 콜백의 매개변수 타입을 `Vector3::SharedPtr`로 바꾼다. |

### `src/sub.cpp`

```cpp
#include "psub2_2/sub.hpp"

void Sub::subscribe_msg(const geometry_msgs::msg::Vector3::SharedPtr msg) const
{
    RCLCPP_INFO(this->get_logger(), "Received message: x=%.1f, y=%.1f, z=%.1f",
                msg->x, msg->y, msg->z);
}

Sub::Sub() : Node("mysub")
{
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));
    sub_ = this->create_subscription<geometry_msgs::msg::Vector3>("mytopic", qos_profile,
        std::bind(&Sub::subscribe_msg, this, _1));
}
```

| 코드 | 설명 |
|---|---|
| `RCLCPP_INFO(..., msg->x, msg->y, msg->z);` | 수신한 세 실수값을 출력한다. |
| `this->create_subscription<geometry_msgs::msg::Vector3>("mytopic", ...)` | `Vector3` 타입으로 `mytopic`을 구독한다. Publisher와 메시지 타입이 같아야 통신된다. |
| 나머지 | 노드 이름, QoS, `std::bind(&Sub::subscribe_msg, this, _1)`는 과제 1과 같다. |

### `src/smain.cpp`

```cpp
#include "psub2_2/sub.hpp"

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<Sub>();
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

과제 1의 smain.cpp와 같고, 헤더 경로만 `psub2_2/sub.hpp`로 바뀐다.

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)
project(psub2_2)

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
find_package(geometry_msgs REQUIRED)

# 사용자 작성 헤더파일 경로 추가
include_directories(include)

#build
add_executable(pub src/pub.cpp src/pmain.cpp)
ament_target_dependencies(pub rclcpp geometry_msgs)

add_executable(sub src/sub.cpp src/smain.cpp)
ament_target_dependencies(sub rclcpp geometry_msgs)

# Install
install(TARGETS pub sub DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `project(psub2_2)` | 패키지 이름을 `psub2_2`로 지정한다. |
| `find_package(geometry_msgs REQUIRED)` | `std_msgs` 대신 `geometry_msgs`를 찾는다. |
| `ament_target_dependencies(pub/sub rclcpp geometry_msgs)` | 의존 패키지를 `geometry_msgs`로 바꿔 연결한다. |
| 나머지 | `include_directories`, `add_executable`, `install`, `ament_package`는 과제 1과 같다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select psub2_2
source install/local_setup.bash

# 터미널 1
ros2 run psub2_2 pub
# 터미널 2
ros2 run psub2_2 sub
```

### 실행결과

> 예상 출력입니다. 본인 터미널 실행 화면으로 교체하세요.

```
[INFO] [...] [mypub]: Published message: x=0.0, y=0.0, z=0.0
[INFO] [...] [mypub]: Published message: x=1.0, y=1.0, z=1.0
[INFO] [...] [mypub]: Published message: x=2.0, y=2.0, z=2.0
```

```
[INFO] [...] [mysub]: Received message: x=1.0, y=1.0, z=1.0
[INFO] [...] [mysub]: Received message: x=2.0, y=2.0, z=2.0
```

```bash
rqt_graph                       # /mypub → /mytopic → /mysub
ros2 node list                  # /mypub, /mysub
ros2 topic list -t              # /mytopic [geometry_msgs/msg/Vector3]
ros2 topic echo /mytopic
ros2 topic hz /mytopic          # average rate: 약 1.000 (주기 1s)
ros2 topic bw /mytopic
```

```
x: 3.0
y: 3.0
z: 3.0
---
```
