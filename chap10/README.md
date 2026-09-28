## 실습과제(인터페이스)
---

## 1. 메시지, 토픽, 서비스, 액션, 인터페이스 용어를 명확히 구분하여 설명하라.

### 1-1. 용어 설명

| 용어 | 뜻 | 예 (turtlesim) |
|---|---|---|
| **메시지 (Message)** | 노드와 노드 사이에 실제로 주고받는 데이터 | 키보드 ↑를 눌렀을 때 보내는 `linear.x = 2.0` 값 |
| **토픽 (Topic)** | 발행자(Publisher)가 구독자(Subscriber)에게 메시지를 단방향·연속적으로 보내는 통신 방식, 또는 그 통로의 이름 | `/turtle1/cmd_vel` |
| **서비스 (Service)** | 클라이언트가 요청(Request)을 보내면 서버가 처리 후 응답(Response)을 한 번 돌려주는 양방향·일회성 통신 방식 | `/spawn` (거북이 생성) |
| **액션 (Action)** | 클라이언트가 목표(Goal)를 보내면 서버가 수행 중 피드백(Feedback)을 계속 보내고, 끝나면 결과(Result)를 돌려주는 통신 방식 (토픽 + 서비스의 복합형) | `/turtle1/rotate_absolute` (목표 각도로 회전) |
| **인터페이스 (Interface)** | 노드 사이에 주고받는 메시지의 자료형(type), 즉 데이터의 형식 정의 | `geometry_msgs/msg/Twist` |

**인터페이스의 종류**

| 인터페이스 | 파일 | 사용하는 통신 | 구성 |
|---|---|---|---|
| msg 인터페이스 | `*.msg` | 토픽 | 데이터(data) |
| srv 인터페이스 | `*.srv` | 서비스 | 요청(request) `---` 응답(response) |
| action 인터페이스 | `*.action` | 액션 | 목표(goal) `---` 결과(result) `---` 피드백(feedback) |

**핵심 구분**

- **메시지**는 실제 데이터(값)이고, **인터페이스**는 그 데이터의 형식(틀)이다.
  C언어에 비유하면 인터페이스는 `struct` 정의이고, 메시지는 그 구조체로 만든 변수에 값을 채운 것이다.
- **토픽·서비스·액션**은 데이터를 주고받는 **통신 방식**이고, 각각 msg, srv, action 인터페이스를 사용한다.
- 한 문장으로 정리하면 "`/turtle1/cmd_vel` **토픽**으로 `geometry_msgs/msg/Twist` **인터페이스** 형식의 **메시지**를 보낸다."

### 1-2. 동작 원리

**(1) 인터페이스의 동작 원리**

```
msg / srv / action 파일  →  빌드 시스템(colcon build)  →  C / C++ / Python 소스코드
```

- 인터페이스는 프로그래밍 언어에 독립적인 텍스트 파일(`fieldtype fieldname` 형식)로 정의된다.
- 빌드 과정에서 각 언어의 소스코드(C++ 헤더, Python 클래스)로 자동 변환된다.
- 노드는 생성된 타입에 값을 채워 메시지를 만들고, 메시지는 바이트로 직렬화되어 DDS를 통해 전송된다. 받는 쪽은 같은 인터페이스를 기준으로 역직렬화한다.
- 보내는 쪽과 받는 쪽이 같은 인터페이스를 사용하므로, C++ 노드와 Python 노드도 서로 통신할 수 있다.
- 인터페이스는 단순 자료형(C 기본 자료형), 메시지 안에 메시지가 포함된 구조(C 구조체), 메시지를 나열한 배열 구조(C 배열)로 구성할 수 있다.
  - 예: `Twist`는 `Vector3 linear`, `Vector3 angular`로 구성되고, `Vector3`는 다시 `float64 x, y, z`로 구성된다.

**(2) 토픽 통신의 동작 원리**

```
[teleop_turtle]  ──  /turtle1/cmd_vel (Twist 메시지)  ──▶  [turtlesim]
   Publisher                                              Subscriber
```

1. 발행자 노드가 특정 토픽 이름으로 메시지를 발행한다.
2. 같은 토픽을 구독하는 노드들이 메시지를 받는다. 발행자는 응답을 기다리지 않는다(비동기, 단방향).
3. 발행자와 구독자는 1:1, 1:N, N:1, N:N으로 연결될 수 있다.
4. 센서 데이터, 로봇 상태, 속도 명령처럼 **연속적으로** 흐르는 데이터에 사용한다.

**(3) 서비스 통신의 동작 원리**

```
[Client] ── Request (x, y, theta, name) ──▶ [Server]
[Client] ◀── Response (name) ────────────── [Server]
```

1. 클라이언트가 서버에 요청 메시지를 보낸다.
2. 서버가 요청을 처리한 뒤 응답 메시지를 돌려준다(동기, 양방향, 일회성).
3. 예를 들어 `/spawn` 서비스는 위치(x, y, theta)와 이름을 요청으로 받아 새 거북이를 만들고, 그 이름을 응답으로 돌려준다.
4. LED 제어, 모터 On/Off, 경로 계산처럼 **한 번 요청하고 결과를 받는** 작업에 사용한다.

**(4) 액션 통신의 동작 원리**

```
[Client] ── Goal (theta) ─────────────────▶ [Server]
[Client] ◀── Feedback (remaining) … 반복 ── [Server]
[Client] ◀── Result (delta) ─────────────── [Server]
```

1. 클라이언트가 목표(Goal)를 서버에 보낸다(서비스 방식).
2. 서버는 작업을 수행하면서 진행 상황을 피드백으로 계속 보낸다(토픽 방식).
3. 작업이 끝나면 최종 결과(Result)를 보낸다(서비스 방식).
4. 예를 들어 `/turtle1/rotate_absolute`는 목표 각도(theta)를 받아 회전하는 동안 남은 회전량(remaining)을 피드백으로 보내고, 끝나면 회전한 각도(delta)를 결과로 보낸다.
5. 목적지 이동, 물건 집기처럼 **시간이 오래 걸리는 복합 작업**에 사용한다.

**(5) 토픽·서비스·액션 비교**

| 구분 | 토픽 | 서비스 | 액션 |
|---|---|---|---|
| 연속성 | 연속성 | 일회성 | 복합(토픽 + 서비스) |
| 방향성 | 단방향 | 양방향 | 양방향 |
| 동기성 | 비동기 | 동기 | 동기 + 비동기 |
| 다자간 연결 | 1:1, 1:N, N:1, N:N | 1:1 | 1:1 |
| 노드 역할 | 발행자 / 구독자 | 서버 / 클라이언트 | 서버 / 클라이언트 |
| 동작 트리거 | 발행자 | 클라이언트 | 클라이언트 |
| 인터페이스 | msg | srv | action |
| CLI 명령어 | `ros2 topic` | `ros2 service` | `ros2 action` |

---

## 2. ros2 명령어를 이용하여 turtlesim과 teleop_turtle 노드를 각각 실행하고, 현재 실행 중인 토픽 메시지와 메시지 인터페이스를 출력하시오.

### 2-1. turtlesim 노드 실행 (터미널 1)

```bash
ros2 run turtlesim turtlesim_node
```

<img width="833" height="546" alt="스크린샷 2026-09-28 133416" src="https://github.com/user-attachments/assets/f2bbce26-2216-4274-938b-badefa6012aa" />

> **설명:** `turtlesim` 패키지의 `turtlesim_node`를 실행한 화면. 파란 배경의 창에 거북이(turtle1)가 생성되고, 터미널에는 거북이의 이름과 초기 위치(x, y, theta)가 출력된다.

### 2-2. teleop_turtle 노드 실행 (터미널 2)

```bash
ros2 run turtlesim turtle_teleop_key
```

<img width="834" height="171" alt="스크린샷 2026-09-28 133533" src="https://github.com/user-attachments/assets/870ae0f9-5515-4858-9a59-a1f2ce7621c7" />

> **설명:** `turtle_teleop_key`를 실행한 화면. 방향키로 거북이를 움직일 수 있다는 안내가 출력되며, 키를 누르면 `/turtle1/cmd_vel` 토픽으로 속도 메시지가 발행되어 거북이가 이동한다.

### 2-3. 실행 중인 토픽과 메시지 인터페이스 출력 (터미널 3)

```bash
ros2 topic list -t
```

<img width="783" height="122" alt="스크린샷 2026-09-28 133654" src="https://github.com/user-attachments/assets/ffce29df-a6f1-4820-b311-451bf9d48f9a" />


> **설명:** `-t` 옵션을 사용하면 토픽 이름 옆 대괄호 안에 해당 토픽의 메시지 인터페이스가 함께 출력된다. 출력 형식은 `토픽 이름 [패키지명/msg/타입명]`이다.

| 토픽 이름 | 메시지 인터페이스 | 의미 |
|---|---|---|
| `/parameter_events` | `rcl_interfaces/msg/ParameterEvent` | 노드 파라미터 변경 이벤트 |
| `/rosout` | `rcl_interfaces/msg/Log` | 노드의 로그 메시지 |
| `/turtle1/cmd_vel` | `geometry_msgs/msg/Twist` | 거북이 속도 명령 (teleop_turtle → turtlesim) |
| `/turtle1/color_sensor` | `turtlesim/msg/Color` | 거북이 아래 배경 색상 |
| `/turtle1/pose` | `turtlesim/msg/Pose` | 거북이 위치와 자세 |

- 예를 들어 `/turtle1/cmd_vel [geometry_msgs/msg/Twist]`는 "`/turtle1/cmd_vel` 토픽으로 `geometry_msgs` 패키지의 `msg` 분류에 있는 `Twist` 형식의 메시지가 오간다"는 뜻이다.
- `/parameter_events`, `/rosout`은 ROS2 노드가 기본으로 사용하는 토픽이다.

---

## 3. 앞에서 출력한 메시지 인터페이스의 정의를 ros2 명령어를 이용하여 각각 출력하시오.

인터페이스 정의는 `ros2 interface show <인터페이스명>` 명령어로 확인한다.

### 3-1. geometry_msgs/msg/Twist

```bash
ros2 interface show geometry_msgs/msg/Twist
```

<img width="931" height="217" alt="스크린샷 2026-09-28 134813" src="https://github.com/user-attachments/assets/39434e5a-6aef-41f7-9780-326025b86ca1" />

> **설명:** `Twist`는 `Vector3 linear`(병진 속도)와 `Vector3 angular`(회전 속도)로 구성된, 메시지 안에 메시지를 품은 구조이다. 각 `Vector3`는 `float64 x, y, z`를 가지므로, Twist는 병진 속도 3개와 회전 속도 3개를 표현한다. `/turtle1/cmd_vel` 토픽에서 사용한다.

### 3-2. geometry_msgs/msg/Vector3 (Twist 내부 구조 확인)

```bash
ros2 interface show geometry_msgs/msg/Vector3
```

<img width="918" height="198" alt="스크린샷 2026-09-28 135538" src="https://github.com/user-attachments/assets/17bfc3f5-7093-41bc-8d0f-b87fc5d19f29" />

> **설명:** `Vector3`는 `float64 x`, `float64 y`, `float64 z` 세 개의 실수로 이루어진 3차원 벡터이다. Twist의 `linear`, `angular`가 이 형식을 사용한다.

### 3-3. turtlesim/msg/Color

```bash
ros2 interface show turtlesim/msg/Color
```

<img width="942" height="77" alt="image" src="https://github.com/user-attachments/assets/dd60b705-7f14-4fdc-b900-68ee33b993d8" />

> **설명:** `Color`는 `uint8 r`, `uint8 g`, `uint8 b`로 구성된 단순 자료형 인터페이스이다. 거북이 아래 배경의 RGB 색상값을 나타내며 `/turtle1/color_sensor` 토픽에서 사용한다.

### 3-4. turtlesim/msg/Pose

```bash
ros2 interface show turtlesim/msg/Pose
```

<img width="940" height="141" alt="스크린샷 2026-09-28 134125" src="https://github.com/user-attachments/assets/ba366a7d-5077-40cb-a2a9-289eb59b4ec0" />

> **설명:** `Pose`는 `float32` 형식의 `x`, `y`(위치), `theta`(방향 각도), `linear_velocity`(병진 속도), `angular_velocity`(회전 속도)로 구성된다. 거북이의 현재 상태를 나타내며 `/turtle1/pose` 토픽에서 사용한다.

### 3-5. rcl_interfaces/msg/Log

```bash
ros2 interface show rcl_interfaces/msg/Log
```

<img width="922" height="98" alt="image" src="https://github.com/user-attachments/assets/10963176-81d6-44ca-9462-5eb50a29f8e9" />
<img width="929" height="205" alt="스크린샷 2026-09-28 134412" src="https://github.com/user-attachments/assets/f69ddc72-63e0-4e9d-8d20-a705babc11d4" />
<img width="914" height="348" alt="스크린샷 2026-09-28 134430" src="https://github.com/user-attachments/assets/11ff8f8f-010d-4a1f-a615-17af7277ce5a" />  

> **설명:** `Log`는 로그 레벨 상수(DEBUG, INFO, WARN, ERROR, FATAL)와 함께 시간(`stamp`), 레벨(`level`), 노드 이름(`name`), 로그 내용(`msg`), 파일·함수·줄 번호 등의 필드로 구성된다. `/rosout` 토픽에서 사용한다.

### 3-6. rcl_interfaces/msg/ParameterEvent

```bash
ros2 interface show rcl_interfaces/msg/ParameterEvent
```

<img width="1040" height="558" alt="image" src="https://github.com/user-attachments/assets/969b7086-1297-4383-a0f0-2a1818392196" />


> **설명:** `ParameterEvent`는 시간(`stamp`), 노드 이름(`node`)과 새로 추가·변경·삭제된 파라미터 목록(`new_parameters`, `changed_parameters`, `deleted_parameters`)으로 구성된다. 파라미터 목록은 `Parameter[]` 형식으로, 메시지들이 나열된 배열 구조이다. `/parameter_events` 토픽에서 사용한다.

### 3-7. 정리

| 인터페이스 | 구조 유형 | 주요 필드 |
|---|---|---|
| `geometry_msgs/msg/Twist` | 메시지 안에 메시지 (구조체) | `linear`, `angular` (Vector3) |
| `turtlesim/msg/Color` | 단순 자료형 | `r`, `g`, `b` (uint8) |
| `turtlesim/msg/Pose` | 단순 자료형 | `x`, `y`, `theta`, `linear_velocity`, `angular_velocity` (float32) |
| `rcl_interfaces/msg/Log` | 단순 자료형 + 메시지 포함 | `stamp`, `level`, `name`, `msg` 등 |
| `rcl_interfaces/msg/ParameterEvent` | 배열 구조 포함 | `stamp`, `node`, `new/changed/deleted_parameters` |  

---
