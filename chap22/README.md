# ROS2 영상전송(cv_bridge) 실습과제

> 패키지 이름: ROS2 패키지 이름에는 `-`를 쓸 수 없습니다(소문자, 숫자, `_`만 가능). 그래서 `image_psub1-1`, `image_psub1-2`, `image_psub1-3` 대신 **`image_psub1_1`, `image_psub1_2`, `image_psub1_3`**으로 만들었습니다.
>
> 동영상 경로: 예제와 같이 `/home/linux/simulation/5_lt_cw_100rpm_out.mp4`를 사용합니다. (`git clone https://github.com/2sungryul/simulation.git`로 홈 디렉터리에 받아 둔 상태)

---

## 실습과제 1. 원본·그레이·이진 영상 3개 출력 (image_psub1_1)

### 패키지 생성

```bash
cd ~/ros2_ws/src
ros2 pkg create image_psub1_1 --build-type ament_cmake --dependencies rclcpp sensor_msgs cv_bridge
```

### Publisher 소스파일: `image_psub1_1/src/pub.cpp`

예제 1의 pub.cpp와 같다.

```cpp
#include "rclcpp/rclcpp.hpp"
#include "sensor_msgs/msg/compressed_image.hpp"
#include "cv_bridge/cv_bridge.hpp"
#include "opencv2/opencv.hpp"
#include <memory>
#include <chrono>

std::string src = "/home/linux/simulation/5_lt_cw_100rpm_out.mp4";

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("imagepub");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10)); //TCP
    //auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10)).best_effort(); //UDP
    auto mypub = node->create_publisher<sensor_msgs::msg::CompressedImage>("image/compressed", qos_profile);
    std_msgs::msg::Header hdr;
    sensor_msgs::msg::CompressedImage::SharedPtr msg;
    rclcpp::WallRate loop_rate(30.0);
    cv::VideoCapture cap(src, cv::CAP_GSTREAMER);
    if (!cap.isOpened()) {
        RCLCPP_ERROR(node->get_logger(), "Could not open video!");
        rclcpp::shutdown();
        return -1;
    }
    cv::Mat frame;
    while(rclcpp::ok())
    {
        cap >> frame;
        if(frame.empty()){RCLCPP_ERROR(node->get_logger(),"frame empty"); break;}
        msg = cv_bridge::CvImage(hdr,"bgr8",frame).toCompressedImageMsg();
        mypub->publish(*msg);
        RCLCPP_INFO(node->get_logger(), "Send Image");
        loop_rate.sleep();
    }
    rclcpp::shutdown();
    return 0;
}
```

### Publisher 라인단위 코드설명

| 코드 | 설명 |
|---|---|
| `#include "rclcpp/rclcpp.hpp"` | ROS2 C++ 라이브러리 헤더를 포함한다. |
| `#include "sensor_msgs/msg/compressed_image.hpp"` | 압축 영상 메시지 `sensor_msgs/msg/CompressedImage`의 헤더를 포함한다. |
| `#include "cv_bridge/cv_bridge.hpp"` | `cv::Mat`과 ROS2 영상 메시지 사이의 변환 기능(`cv_bridge::CvImage`)을 위한 헤더이다. |
| `#include "opencv2/opencv.hpp"` | `cv::VideoCapture`, `cv::Mat` 등 OpenCV 기능을 위한 헤더이다. |
| `#include <memory>`, `<chrono>` | 스마트 포인터, 시간 단위를 위한 헤더이다. |
| `std::string src = "...mp4";` | 읽어 올 동영상 파일의 경로이다. |
| `rclcpp::init(argc, argv);` | ROS2를 초기화한다. |
| `auto node = std::make_shared<rclcpp::Node>("imagepub");` | 이름이 `imagepub`인 노드를 동적 생성한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | KEEP_LAST, depth 10, 기본 RELIABLE 모드의 QoS를 만든다. 주석 처리된 `.best_effort()`를 쓰면 속도 우선 모드가 된다. Subscriber와 설정을 같게 해야 한다. |
| `auto mypub = node->create_publisher<sensor_msgs::msg::CompressedImage>("image/compressed", qos_profile);` | `image/compressed` 토픽으로 압축 영상을 발행하는 퍼블리셔를 생성한다. |
| `std_msgs::msg::Header hdr;` | 메시지 헤더(타임스탬프, frame_id) 객체. CvImage 생성에 필요하다. |
| `sensor_msgs::msg::CompressedImage::SharedPtr msg;` | 변환된 압축 영상 메시지를 가리킬 스마트 포인터이다. |
| `rclcpp::WallRate loop_rate(30.0);` | 30Hz, 즉 초당 30장을 발행하도록 주기를 정한다. |
| `cv::VideoCapture cap(src, cv::CAP_GSTREAMER);` | GStreamer 백엔드로 동영상 파일을 연다. |
| `if (!cap.isOpened()) {...}` | 파일을 열지 못하면 에러 로그를 출력하고 ROS2를 종료한 뒤 -1을 반환한다. |
| `cv::Mat frame;` | 캡처한 한 장의 영상을 저장할 Mat 객체이다. |
| `while(rclcpp::ok())` | Ctrl+C 등 종료 조건이 생길 때까지 반복한다. |
| `cap >> frame;` | 동영상에서 다음 프레임을 읽어 `frame`에 저장한다. |
| `if(frame.empty()){... break;}` | 동영상이 끝나 빈 프레임이 오면 에러 로그를 출력하고 루프를 빠져나간다. |
| `msg = cv_bridge::CvImage(hdr,"bgr8",frame).toCompressedImageMsg();` | `cv::Mat` → `CvImage`(인코딩 bgr8) → `CompressedImage`(기본 JPG 압축) 메시지로 변환한다. |
| `mypub->publish(*msg);` | 스마트 포인터가 가리키는 메시지 객체를 발행한다. |
| `RCLCPP_INFO(node->get_logger(), "Send Image");` | 발행했다는 로그를 출력한다. |
| `loop_rate.sleep();` | 30Hz 주기에서 남은 시간만큼 대기한다. |
| `rclcpp::shutdown(); return 0;` | ROS2를 종료하고 프로그램을 끝낸다. |

### Subscriber 소스파일: `image_psub1_1/src/sub.cpp`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "sensor_msgs/msg/compressed_image.hpp"
#include "opencv2/opencv.hpp"
#include <memory>
#include <functional>
#include <iostream>

void mysub_callback(rclcpp::Node::SharedPtr node, const sensor_msgs::msg::CompressedImage::SharedPtr msg)
{
    cv::Mat frame = cv::imdecode(cv::Mat(msg->data), cv::IMREAD_COLOR);
    cv::Mat gray, binary;
    cv::cvtColor(frame, gray, cv::COLOR_BGR2GRAY);
    cv::threshold(gray, binary, 128, 255, cv::THRESH_BINARY);
    cv::imshow("color", frame);
    cv::imshow("gray", gray);
    cv::imshow("binary", binary);
    cv::waitKey(1);
    RCLCPP_INFO(node->get_logger(), "Received Image : %s,%d,%d", msg->format.c_str(), frame.rows, frame.cols);
}

int main(int argc, char* argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("imagesub");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10)); //TCP
    //auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10)).best_effort(); //UDP
    std::function<void(const sensor_msgs::msg::CompressedImage::SharedPtr msg)> fn;
    fn = std::bind(mysub_callback, node, std::placeholders::_1);
    auto mysub = node->create_subscription<sensor_msgs::msg::CompressedImage>("image/compressed", qos_profile, fn);
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

### Subscriber 라인단위 코드설명

| 코드 | 설명 |
|---|---|
| `#include "sensor_msgs/msg/compressed_image.hpp"` | 구독할 압축 영상 메시지의 헤더를 포함한다. |
| `#include "opencv2/opencv.hpp"` | `imdecode`, `cvtColor`, `threshold`, `imshow` 등 OpenCV 기능을 위한 헤더이다. |
| `#include <memory>`, `<functional>`, `<iostream>` | 스마트 포인터, `std::function`/`std::bind`, 입출력을 위한 헤더이다. |
| `void mysub_callback(rclcpp::Node::SharedPtr node, const sensor_msgs::msg::CompressedImage::SharedPtr msg)` | 영상 메시지를 수신할 때마다 호출되는 콜백함수이다. |
| `cv::Mat frame = cv::imdecode(cv::Mat(msg->data), cv::IMREAD_COLOR);` | 메시지의 JPG 압축 데이터(`uint8[] data`)를 Mat으로 감싸고, `imdecode`로 압축을 풀어 3채널 컬러 영상을 만든다. |
| `cv::Mat gray, binary;` | 그레이 영상과 이진 영상을 저장할 Mat 객체를 선언한다. |
| `cv::cvtColor(frame, gray, cv::COLOR_BGR2GRAY);` | BGR 컬러 영상을 1채널 그레이 영상으로 변환한다. |
| `cv::threshold(gray, binary, 128, 255, cv::THRESH_BINARY);` | 그레이 영상의 각 화소가 임계값 128보다 크면 255(흰색), 아니면 0(검은색)으로 바꿔 이진 영상을 만든다. |
| `cv::imshow("color", frame);` | 원본 컬러 영상을 `color` 창에 출력한다. |
| `cv::imshow("gray", gray);` | 그레이 영상을 `gray` 창에 출력한다. |
| `cv::imshow("binary", binary);` | 이진 영상을 `binary` 창에 출력한다. |
| `cv::waitKey(1);` | 1ms 대기하며 창을 갱신한다. 이 호출이 없으면 창이 그려지지 않는다. |
| `RCLCPP_INFO(..., msg->format.c_str(), frame.rows, frame.cols);` | 압축 형식(jpeg)과 영상의 행·열 크기를 로그로 출력한다. |
| `rclcpp::init(argc, argv);` | ROS2를 초기화한다. |
| `auto node = std::make_shared<rclcpp::Node>("imagesub");` | 이름이 `imagesub`인 노드를 동적 생성한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | Publisher와 같은 QoS(KEEP_LAST 10, RELIABLE)를 설정한다. |
| `std::function<void(const sensor_msgs::msg::CompressedImage::SharedPtr msg)> fn;` | 구독 콜백 형태인 `void(MessageT::SharedPtr)` 함수 객체를 선언한다. |
| `fn = std::bind(mysub_callback, node, std::placeholders::_1);` | `node`는 묶고 메시지 자리는 `_1`로 비워, 매개변수 2개인 콜백을 1개짜리로 바꾼다. |
| `auto mysub = node->create_subscription<...>("image/compressed", qos_profile, fn);` | `image/compressed` 토픽을 구독하고, 수신 시 `fn`이 호출되도록 등록한다. |
| `rclcpp::spin(node);` | 메시지 수신을 기다리며 콜백을 처리한다. |
| `rclcpp::shutdown(); return 0;` | ROS2를 종료하고 프로그램을 끝낸다. |

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)
project(image_psub1_1)

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
find_package(sensor_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(OpenCV REQUIRED)

#build
add_executable(pub src/pub.cpp)
ament_target_dependencies(pub rclcpp sensor_msgs cv_bridge OpenCV)

add_executable(sub src/sub.cpp)
ament_target_dependencies(sub rclcpp sensor_msgs OpenCV)

# Install
install(TARGETS pub sub DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `project(image_psub1_1)` | 패키지 이름. `package.xml`의 `<name>`과 같아야 한다. |
| `set(CMAKE_CXX_STANDARD 14)` | C++14를 사용한다. |
| `add_compile_options(-Wall -Wextra -Wpedantic)` | GCC/Clang 경고 옵션을 켠다. |
| `find_package(ament_cmake / rclcpp REQUIRED)` | ament 명령 정의와 ROS2 C++ 라이브러리를 찾는다. |
| `find_package(sensor_msgs REQUIRED)` | `CompressedImage` 메시지가 들어 있는 패키지를 찾는다. |
| `find_package(cv_bridge REQUIRED)` | Mat ↔ ROS2 영상 메시지 변환 라이브러리를 찾는다. |
| `find_package(OpenCV REQUIRED)` | OpenCV 라이브러리를 찾는다. |
| `add_executable(pub src/pub.cpp)` | 실행파일 `pub`을 만든다. |
| `ament_target_dependencies(pub rclcpp sensor_msgs cv_bridge OpenCV)` | Publisher는 cv_bridge로 메시지를 만들므로 cv_bridge까지 연결한다. |
| `add_executable(sub src/sub.cpp)` | 실행파일 `sub`을 만든다. |
| `ament_target_dependencies(sub rclcpp sensor_msgs OpenCV)` | Subscriber는 `imdecode`로 직접 변환하므로 cv_bridge 없이 OpenCV만 연결한다. |
| `install(TARGETS pub sub DESTINATION lib/${PROJECT_NAME})` | 두 실행파일을 `install/image_psub1_1/lib/image_psub1_1`에 설치한다. |
| `ament_package()` | 패키지를 ROS2 패키지로 등록한다. 마지막 줄에 둔다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select image_psub1_1
source install/local_setup.bash

# 터미널 1
ros2 run image_psub1_1 pub
# 터미널 2 (새 터미널, source install/local_setup.bash 후)
ros2 run image_psub1_1 sub
```

### 실행결과

> 예상 출력입니다. 터미널 화면과 `color`, `gray`, `binary` 영상 창 3개를 캡처해서 교체하세요.

```
[INFO] [...] [imagepub]: Send Image
[INFO] [...] [imagepub]: Send Image
```

```
[INFO] [...] [imagesub]: Received Image : jpeg,360,640
[INFO] [...] [imagesub]: Received Image : jpeg,360,640
```

(영상 크기는 동영상 파일에 따라 다르게 나온다.)

### 토픽 전송상태 확인 (새 터미널)

```bash
rqt_graph                                # /imagepub → /image/compressed → /imagesub
ros2 node list                           # /imagepub, /imagesub
ros2 topic list -t                       # /image/compressed [sensor_msgs/msg/CompressedImage]
ros2 topic hz /image/compressed          # average rate: 약 30.000
ros2 topic bw /image/compressed          # 메시지 1개당 약 40KB
ros2 topic info /image/compressed -v     # 퍼블리셔·섭스크라이버의 QoS(RELIABLE, KEEP_LAST 10) 확인
```

---

## 실습과제 2. 구독 영상 출력 + mp4 저장 (image_psub1_2)

### 패키지 생성

```bash
cd ~/ros2_ws/src
ros2 pkg create image_psub1_2 --build-type ament_cmake --dependencies rclcpp sensor_msgs cv_bridge
```

### Publisher 소스파일: `image_psub1_2/src/pub.cpp`

실습과제 1(예제 1)의 pub.cpp와 같다.

### Subscriber 소스파일: `image_psub1_2/src/sub.cpp`

```cpp
#include "rclcpp/rclcpp.hpp"
#include "sensor_msgs/msg/compressed_image.hpp"
#include "opencv2/opencv.hpp"
#include <memory>
#include <functional>
#include <iostream>

std::string dst = "/home/linux/output.mp4";
cv::VideoWriter writer;

void mysub_callback(rclcpp::Node::SharedPtr node, const sensor_msgs::msg::CompressedImage::SharedPtr msg)
{
    cv::Mat frame = cv::imdecode(cv::Mat(msg->data), cv::IMREAD_COLOR);
    if(!writer.isOpened())
    {
        writer.open(dst, cv::VideoWriter::fourcc('m','p','4','v'), 30.0, frame.size(), true);
        if(!writer.isOpened()) {
            RCLCPP_ERROR(node->get_logger(), "Could not open video writer!");
            return;
        }
        RCLCPP_INFO(node->get_logger(), "Start recording : %s", dst.c_str());
    }
    writer << frame;
    cv::imshow("wsl", frame);
    cv::waitKey(1);
    RCLCPP_INFO(node->get_logger(), "Received Image : %s,%d,%d", msg->format.c_str(), frame.rows, frame.cols);
}

int main(int argc, char* argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<rclcpp::Node>("imagesub");
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10)); //TCP
    //auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10)).best_effort(); //UDP
    std::function<void(const sensor_msgs::msg::CompressedImage::SharedPtr msg)> fn;
    fn = std::bind(mysub_callback, node, std::placeholders::_1);
    auto mysub = node->create_subscription<sensor_msgs::msg::CompressedImage>("image/compressed", qos_profile, fn);
    rclcpp::spin(node);
    writer.release();
    RCLCPP_INFO(node->get_logger(), "Stop recording");
    rclcpp::shutdown();
    return 0;
}
```

### Subscriber 라인단위 코드설명

실습과제 1과 같은 줄(헤더, 노드 생성, QoS, `std::bind`, `create_subscription`)은 생략하고, 바뀐 줄만 설명한다.

| 코드 | 설명 |
|---|---|
| `std::string dst = "/home/linux/output.mp4";` | 저장할 동영상 파일의 경로이다. |
| `cv::VideoWriter writer;` | 동영상 저장 객체. 콜백과 main에서 함께 써야 하므로 전역변수로 선언한다. |
| `cv::Mat frame = cv::imdecode(...);` | 수신한 압축 영상을 Mat으로 변환한다. |
| `if(!writer.isOpened())` | 아직 저장 파일을 열지 않았으면(첫 프레임 수신 시) 한 번만 실행된다. |
| `writer.open(dst, cv::VideoWriter::fourcc('m','p','4','v'), 30.0, frame.size(), true);` | 저장 파일을 연다. 코덱은 mp4v(MPEG-4), 초당 30프레임(Publisher 발행 주기와 같음), 크기는 수신 영상 크기, 마지막 `true`는 컬러 영상이라는 뜻이다. 첫 프레임의 크기를 알아야 열 수 있어서 콜백 안에서 연다. |
| `if(!writer.isOpened()) {... return;}` | 파일을 열지 못하면 에러를 출력하고 콜백을 끝낸다. |
| `RCLCPP_INFO(..., "Start recording : %s", dst.c_str());` | 저장 시작과 파일 경로를 출력한다. |
| `writer << frame;` | 수신한 프레임을 동영상 파일에 한 장씩 기록한다. |
| `cv::imshow("wsl", frame); cv::waitKey(1);` | 수신 영상을 화면에 출력한다. |
| `rclcpp::spin(node);` | 영상을 수신하며 저장을 계속한다. Ctrl+C를 누르면 ROS2 시그널 핸들러가 종료 처리를 하여 `spin`이 리턴한다. |
| `writer.release();` | 저장 파일을 닫는다. 이 호출로 mp4 파일 정보가 마무리되어야 정상 재생된다. |
| `RCLCPP_INFO(..., "Stop recording");` | 저장 종료를 출력한다. |
| `rclcpp::shutdown(); return 0;` | ROS2를 종료하고 프로그램을 끝낸다. |

### CMakeLists.txt

실습과제 1과 같고, 패키지 이름만 바뀐다.

```cmake
project(image_psub1_2)
...
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(OpenCV REQUIRED)

#build
add_executable(pub src/pub.cpp)
ament_target_dependencies(pub rclcpp sensor_msgs cv_bridge OpenCV)

add_executable(sub src/sub.cpp)
ament_target_dependencies(sub rclcpp sensor_msgs OpenCV)

# Install
install(TARGETS pub sub DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `project(image_psub1_2)` | 패키지 이름을 `image_psub1_2`로 지정한다. |
| `ament_target_dependencies(sub rclcpp sensor_msgs OpenCV)` | `cv::VideoWriter`도 OpenCV 기능이므로 의존성은 그대로이다. |
| 나머지 | 실습과제 1과 같다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select image_psub1_2
source install/local_setup.bash

# 터미널 1
ros2 run image_psub1_2 pub
# 터미널 2
ros2 run image_psub1_2 sub
# 저장을 끝내려면 터미널 2에서 Ctrl+C
```

저장 확인:

```bash
ls -lh ~/output.mp4
```

### 실행결과

> 예상 출력입니다. 터미널 화면, 영상 창, 저장된 mp4 파일 정보를 캡처해서 교체하세요.

```
[INFO] [...] [imagesub]: Start recording : /home/linux/output.mp4
[INFO] [...] [imagesub]: Received Image : jpeg,360,640
[INFO] [...] [imagesub]: Received Image : jpeg,360,640
^C[INFO] [...] [rclcpp]: signal_handler(signum=2)
[INFO] [...] [imagesub]: Stop recording
```

### 토픽 전송상태 확인 (새 터미널)

```bash
rqt_graph                                # /imagepub → /image/compressed → /imagesub
ros2 node list                           # /imagepub, /imagesub
ros2 topic list -t                       # /image/compressed [sensor_msgs/msg/CompressedImage]
ros2 topic hz /image/compressed          # average rate: 약 30.000
ros2 topic bw /image/compressed
ros2 topic info /image/compressed -v
```

---

## 실습과제 3. 클래스를 이용한 영상전송 패키지 (image_psub1_3)

### 패키지 생성 및 구성

```bash
cd ~/ros2_ws/src
ros2 pkg create image_psub1_3 --build-type ament_cmake --dependencies rclcpp sensor_msgs cv_bridge
```

```
image_psub1_3
├── CMakeLists.txt
├── package.xml
├── include/image_psub1_3
│   ├── pub.hpp
│   └── sub.hpp
└── src
    ├── pub.cpp
    ├── pmain.cpp
    ├── sub.cpp
    └── smain.cpp
```

예제 1의 `while` + `WallRate` 반복을 클래스 방식에 맞게 **타이머 콜백**(33ms ≈ 30Hz)으로 바꿨다.

### `include/image_psub1_3/pub.hpp`

```cpp
#ifndef _PUB_HPP_
#define _PUB_HPP_
#include "rclcpp/rclcpp.hpp"
#include "sensor_msgs/msg/compressed_image.hpp"
#include "cv_bridge/cv_bridge.hpp"
#include "opencv2/opencv.hpp"
#include <memory>
#include <chrono>
#include <string>
using namespace std::chrono_literals;
class Pub : public rclcpp::Node
{
private:
    rclcpp::TimerBase::SharedPtr timer_;
    rclcpp::Publisher<sensor_msgs::msg::CompressedImage>::SharedPtr pub_;
    cv::VideoCapture cap_;
    std_msgs::msg::Header hdr_;
    void publish_msg();
public:
    Pub(const std::string& src);
    bool is_opened() const;
};
#endif //_PUB_HPP_
```

| 코드 | 설명 |
|---|---|
| `#ifndef _PUB_HPP_` / `#define` / `#endif` | 헤더 가드. 헤더가 중복 포함되는 것을 막는다. |
| `#include "sensor_msgs/msg/compressed_image.hpp"` | 압축 영상 메시지 헤더를 포함한다. |
| `#include "cv_bridge/cv_bridge.hpp"`, `"opencv2/opencv.hpp"` | Mat → 메시지 변환과 OpenCV 기능을 위한 헤더이다. |
| `#include <memory>`, `<chrono>`, `<string>` | 스마트 포인터, 시간 리터럴, 문자열을 위한 헤더이다. |
| `using namespace std::chrono_literals;` | `33ms` 리터럴을 쓸 수 있게 한다. |
| `class Pub : public rclcpp::Node` | `rclcpp::Node`를 상속받은 영상 Publisher 노드 클래스이다. |
| `rclcpp::TimerBase::SharedPtr timer_;` | 30Hz로 프레임을 발행하는 타이머 멤버변수이다. |
| `rclcpp::Publisher<sensor_msgs::msg::CompressedImage>::SharedPtr pub_;` | 압축 영상을 발행하는 퍼블리셔 멤버변수이다. |
| `cv::VideoCapture cap_;` | 동영상 파일을 읽는 캡처 객체. 타이머 콜백마다 다음 프레임을 읽어야 하므로 멤버로 둔다. |
| `std_msgs::msg::Header hdr_;` | CvImage 생성에 쓸 메시지 헤더 멤버변수이다. |
| `void publish_msg();` | 타이머 콜백 멤버함수. 한 프레임을 읽어 발행한다. |
| `Pub(const std::string& src);` | 동영상 경로를 받아 파일을 열고 퍼블리셔·타이머를 만드는 생성자이다. |
| `bool is_opened() const;` | 동영상 파일이 정상적으로 열렸는지 main에서 확인하기 위한 멤버함수이다. |

### `src/pub.cpp`

```cpp
#include "image_psub1_3/pub.hpp"

Pub::Pub(const std::string& src) : Node("imagepub")
{
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10)); //TCP
    //auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10)).best_effort(); //UDP
    pub_ = this->create_publisher<sensor_msgs::msg::CompressedImage>("image/compressed", qos_profile);
    cap_.open(src, cv::CAP_GSTREAMER);
    if (!cap_.isOpened()) {
        RCLCPP_ERROR(this->get_logger(), "Could not open video!");
        return;
    }
    timer_ = this->create_wall_timer(33ms, std::bind(&Pub::publish_msg, this));
}

bool Pub::is_opened() const
{
    return cap_.isOpened();
}

void Pub::publish_msg()
{
    cv::Mat frame;
    cap_ >> frame;
    if(frame.empty()) {
        RCLCPP_ERROR(this->get_logger(), "frame empty");
        timer_->cancel();
        rclcpp::shutdown();
        return;
    }
    auto msg = cv_bridge::CvImage(hdr_, "bgr8", frame).toCompressedImageMsg();
    pub_->publish(*msg);
    RCLCPP_INFO(this->get_logger(), "Send Image");
}
```

| 코드 | 설명 |
|---|---|
| `#include "image_psub1_3/pub.hpp"` | `Pub` 클래스 선언부를 포함한다. |
| `Pub::Pub(const std::string& src) : Node("imagepub")` | 기본 클래스 생성자를 호출해 노드 이름을 `imagepub`으로 정한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | KEEP_LAST 10, RELIABLE QoS를 설정한다. |
| `pub_ = this->create_publisher<...>("image/compressed", qos_profile);` | `image/compressed` 토픽의 퍼블리셔를 만들어 멤버에 저장한다. |
| `cap_.open(src, cv::CAP_GSTREAMER);` | 전달받은 경로의 동영상 파일을 연다. |
| `if (!cap_.isOpened()) {... return;}` | 열지 못하면 에러를 출력하고 타이머를 만들지 않은 채 생성자를 끝낸다. |
| `timer_ = this->create_wall_timer(33ms, std::bind(&Pub::publish_msg, this));` | 33ms(약 30Hz)마다 `publish_msg`를 호출하는 타이머를 만든다. 멤버함수이므로 `this`를 묶는다. |
| `bool Pub::is_opened() const { return cap_.isOpened(); }` | 동영상 파일이 열렸는지 반환한다. |
| `void Pub::publish_msg()` | 타이머 콜백의 구현이다. |
| `cv::Mat frame; cap_ >> frame;` | 동영상에서 다음 프레임을 읽는다. |
| `if(frame.empty()) {...}` | 동영상이 끝나면 에러를 출력하고, 타이머를 멈춘 뒤 `rclcpp::shutdown()`으로 `spin`을 끝낸다. 예제 1의 `break`에 해당한다. |
| `auto msg = cv_bridge::CvImage(hdr_, "bgr8", frame).toCompressedImageMsg();` | Mat → CvImage → CompressedImage(JPG)로 변환한다. |
| `pub_->publish(*msg);` | 압축 영상 메시지를 발행한다. |
| `RCLCPP_INFO(this->get_logger(), "Send Image");` | 발행 로그를 출력한다. |

### `src/pmain.cpp`

```cpp
#include "image_psub1_3/pub.hpp"

std::string src = "/home/linux/simulation/5_lt_cw_100rpm_out.mp4";

int main(int argc, char * argv[])
{
    rclcpp::init(argc, argv);
    auto node = std::make_shared<Pub>(src);
    if(!node->is_opened()) {
        rclcpp::shutdown();
        return -1;
    }
    rclcpp::spin(node);
    rclcpp::shutdown();
    return 0;
}
```

| 코드 | 설명 |
|---|---|
| `std::string src = "...mp4";` | 발행할 동영상 파일의 경로이다. |
| `rclcpp::init(argc, argv);` | ROS2를 초기화한다. |
| `auto node = std::make_shared<Pub>(src);` | 동영상 경로를 생성자에 넘겨 `Pub` 노드를 동적 생성한다. |
| `if(!node->is_opened()) {...}` | 동영상을 열지 못했으면 ROS2를 종료하고 -1을 반환한다. 예제 1의 `return -1`과 같은 처리이다. |
| `rclcpp::spin(node);` | 타이머 콜백을 처리하며 영상을 계속 발행한다. |
| `rclcpp::shutdown(); return 0;` | ROS2를 종료하고 프로그램을 끝낸다. |

### `include/image_psub1_3/sub.hpp`

```cpp
#ifndef _SUB_HPP_
#define _SUB_HPP_
#include "rclcpp/rclcpp.hpp"
#include "sensor_msgs/msg/compressed_image.hpp"
#include "opencv2/opencv.hpp"
#include <functional>
#include <memory>
using std::placeholders::_1;
class Sub : public rclcpp::Node
{
private:
    rclcpp::Subscription<sensor_msgs::msg::CompressedImage>::SharedPtr sub_;
    void subscribe_msg(const sensor_msgs::msg::CompressedImage::SharedPtr msg) const;
public:
    Sub();
};
#endif //_SUB_HPP_
```

| 코드 | 설명 |
|---|---|
| `#ifndef _SUB_HPP_` / `#define` / `#endif` | 헤더 가드. |
| `#include "sensor_msgs/msg/compressed_image.hpp"`, `"opencv2/opencv.hpp"` | 압축 영상 메시지와 OpenCV를 위한 헤더이다. |
| `using std::placeholders::_1;` | `std::bind`에서 메시지 자리를 `_1`로 쓸 수 있게 한다. |
| `class Sub : public rclcpp::Node` | `rclcpp::Node`를 상속받은 영상 Subscriber 노드 클래스이다. |
| `rclcpp::Subscription<sensor_msgs::msg::CompressedImage>::SharedPtr sub_;` | 구독 객체 멤버변수. 멤버로 저장해야 구독이 유지된다. |
| `void subscribe_msg(...) const;` | 영상 수신 콜백 멤버함수. 멤버변수를 바꾸지 않으므로 `const`로 선언한다. |
| `Sub();` | 구독을 생성하는 생성자이다. |

### `src/sub.cpp`

```cpp
#include "image_psub1_3/sub.hpp"

void Sub::subscribe_msg(const sensor_msgs::msg::CompressedImage::SharedPtr msg) const
{
    cv::Mat frame = cv::imdecode(cv::Mat(msg->data), cv::IMREAD_COLOR);
    cv::imshow("wsl", frame);
    cv::waitKey(1);
    RCLCPP_INFO(this->get_logger(), "Received Image : %s,%d,%d", msg->format.c_str(), frame.rows, frame.cols);
}

Sub::Sub() : Node("imagesub")
{
    auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10)); //TCP
    //auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10)).best_effort(); //UDP
    sub_ = this->create_subscription<sensor_msgs::msg::CompressedImage>("image/compressed", qos_profile,
        std::bind(&Sub::subscribe_msg, this, _1));
}
```

| 코드 | 설명 |
|---|---|
| `#include "image_psub1_3/sub.hpp"` | `Sub` 클래스 선언부를 포함한다. |
| `void Sub::subscribe_msg(...) const` | 수신 콜백의 구현이다. |
| `cv::Mat frame = cv::imdecode(cv::Mat(msg->data), cv::IMREAD_COLOR);` | 압축 데이터를 풀어 컬러 Mat으로 변환한다. |
| `cv::imshow("wsl", frame); cv::waitKey(1);` | 수신 영상을 `wsl` 창에 출력하고 창을 갱신한다. |
| `RCLCPP_INFO(this->get_logger(), ...)` | 상속받은 로거로 압축 형식과 영상 크기를 출력한다. |
| `Sub::Sub() : Node("imagesub")` | 노드 이름을 `imagesub`으로 정한다. |
| `auto qos_profile = rclcpp::QoS(rclcpp::KeepLast(10));` | Publisher와 같은 QoS를 설정한다. |
| `sub_ = this->create_subscription<...>("image/compressed", qos_profile, std::bind(&Sub::subscribe_msg, this, _1));` | `image/compressed`를 구독한다. `this`를 묶고 메시지 자리는 `_1`로 비워 `void(MessageT::SharedPtr)` 형태로 맞춘다. |

### `src/smain.cpp`

```cpp
#include "image_psub1_3/sub.hpp"

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
| `rclcpp::init(argc, argv);` | ROS2를 초기화한다. |
| `auto node = std::make_shared<Sub>();` | `Sub` 노드를 동적 생성한다. 생성자에서 구독이 시작된다. |
| `rclcpp::spin(node);` | 영상 수신을 기다리며 콜백을 처리한다. |
| `rclcpp::shutdown(); return 0;` | ROS2를 종료하고 프로그램을 끝낸다. |

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.16)
project(image_psub1_3)

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
find_package(sensor_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(OpenCV REQUIRED)

# 사용자 작성 헤더파일 경로 추가
include_directories(include)

#build
add_executable(pub src/pub.cpp src/pmain.cpp)
ament_target_dependencies(pub rclcpp sensor_msgs cv_bridge OpenCV)

add_executable(sub src/sub.cpp src/smain.cpp)
ament_target_dependencies(sub rclcpp sensor_msgs OpenCV)

# Install
install(TARGETS pub sub DESTINATION lib/${PROJECT_NAME})

ament_package()
```

| 코드 | 설명 |
|---|---|
| `project(image_psub1_3)` | 패키지 이름을 지정한다. |
| `find_package(sensor_msgs / cv_bridge / OpenCV REQUIRED)` | 영상 메시지, 변환 라이브러리, OpenCV를 찾는다. |
| `include_directories(include)` | `include/` 폴더를 헤더 검색 경로에 추가한다. 이 줄이 있어야 `#include "image_psub1_3/pub.hpp"`가 동작한다. |
| `add_executable(pub src/pub.cpp src/pmain.cpp)` | 클래스 구현부와 메인함수를 함께 컴파일해 실행파일 `pub`을 만든다. |
| `ament_target_dependencies(pub rclcpp sensor_msgs cv_bridge OpenCV)` | Publisher에 필요한 라이브러리를 연결한다. |
| `add_executable(sub src/sub.cpp src/smain.cpp)` | 실행파일 `sub`을 만든다. |
| `ament_target_dependencies(sub rclcpp sensor_msgs OpenCV)` | Subscriber에 필요한 라이브러리를 연결한다. |
| `install(TARGETS pub sub DESTINATION lib/${PROJECT_NAME})` | 두 실행파일을 설치한다. |
| `ament_package()` | 패키지를 ROS2 패키지로 등록한다. |

### 빌드 및 실행

```bash
cd ~/ros2_ws
colcon build --symlink-install --packages-select image_psub1_3
source install/local_setup.bash

# 터미널 1
ros2 run image_psub1_3 pub
# 터미널 2
ros2 run image_psub1_3 sub
```

### 실행결과

> 예상 출력입니다. 터미널 화면과 영상 창을 캡처해서 교체하세요.

```
[INFO] [...] [imagepub]: Send Image
[INFO] [...] [imagepub]: Send Image
```

```
[INFO] [...] [imagesub]: Received Image : jpeg,360,640
[INFO] [...] [imagesub]: Received Image : jpeg,360,640
```

### 토픽 전송상태 확인 (새 터미널)

```bash
rqt_graph                                # /imagepub → /image/compressed → /imagesub
ros2 node list                           # /imagepub, /imagesub
ros2 topic list -t                       # /image/compressed [sensor_msgs/msg/CompressedImage]
ros2 topic hz /image/compressed          # average rate: 약 30.000 (타이머 33ms이므로 약 30.3)
ros2 topic bw /image/compressed
ros2 topic info /image/compressed -v
```
