## 실습과제 1. `ros2 topic echo` 결과의 6개 숫자를 설명하라.

```
$ ros2 topic echo /turtle1/cmd_vel
linear:
  x: 2.0
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.0
---
```

`/turtle1/cmd_vel`은 `geometry_msgs/msg/Twist` 타입으로, 거북이에게 내리는 속도 명령입니다. 직선 속도 `linear` 3개와 회전 속도 `angular` 3개, 총 6개 값으로 구성됩니다.

| 값 | 결과 | 의미 |
|---|---|---|
| `linear.x` | 2.0 | x축 방향(거북이가 바라보는 앞쪽)으로 2 m/s 직진 |
| `linear.y` | 0.0 | 옆으로 이동하지 않음 |
| `linear.z` | 0.0 | 위아래로 이동하지 않음 |
| `angular.x` | 0.0 | x축 둘레 회전 없음 |
| `angular.y` | 0.0 | y축 둘레 회전 없음 |
| `angular.z` | 0.0 | z축 둘레 회전 없음 (방향을 바꾸지 않음) |

- `linear`는 각 축 방향으로의 이동 속도(m/s), `angular`는 각 축을 중심으로 한 회전 속도(rad/s)입니다.
- 거북이는 xy평면에서만 움직이므로 `linear.z`, `angular.x`, `angular.y`는 항상 0이고, 실제로 사용하는 값은 `linear.x`와 `angular.z` 두 개입니다.
- 따라서 이 메시지는 "방향을 바꾸지 않고 앞으로 2 m/s로 직진하라"는 명령입니다.

---

## 실습과제 2. `ros2 topic bw` 결과의 4개 숫자(42B/s, mean, min, max)의 의미를 설명하라.

```
$ ros2 topic bw /turtle1/cmd_vel
Subscribed to [/turtle1/cmd_vel]
42 B/s from 2 messages
        Message size mean: 52 B min: 52 B max: 52 B
```

`bw`는 토픽의 대역폭, 즉 초당 전송되는 바이트 수를 측정합니다.

| 값 | 의미 |
|---|---|
| `42 B/s` | 초당 전송량 (메시지 2개를 기준으로 계산) |
| `mean: 52 B` | 메시지 1개 크기의 평균 |
| `min: 52 B` | 메시지 1개 크기의 최소 |
| `max: 52 B` | 메시지 1개 크기의 최대 |

- 42 B/s는 시간당 전송량이고, 52 B는 메시지 하나의 크기입니다.
- mean, min, max가 모두 같은 이유는 `Twist` 메시지가 항상 크기가 고정된 메시지이기 때문입니다.

---

## 실습과제 3. `ros2 topic hz` 결과의 5개의 숫자(average rate, min, max, std dev, window)의 의미를 설명하라.

```
$ ros2 topic hz /turtle1/cmd_vel
average rate: 3.692
        min: 0.136s max: 0.775s std dev: 0.21953s window: 7
```

`hz`는 토픽이 초당 몇 번 발행되는지(전송률)를 측정합니다.

| 값 | 결과 | 의미 |
|---|---|---|
| `average rate` | 3.692 | 평균 발행 빈도 (초당 약 3.7번, Hz) |
| `min` | 0.136s | 메시지 사이 간격의 최소값 |
| `max` | 0.775s | 메시지 사이 간격의 최대값 |
| `std dev` | 0.21953s | 간격의 표준편차 (간격이 들쭉날쭉한 정도) |
| `window` | 7 | 계산에 사용된 샘플 개수 |

- `average rate`는 빈도(Hz), `min`/`max`는 간격(초)입니다.
- 평균 주기는 1 ÷ 3.692 ≈ 0.27초로, 약 0.27초에 한 번씩 발행된다는 뜻입니다.
- `std dev`가 큰 것은 teleop이 키를 누를 때만 발행해서 간격이 불규칙하기 때문입니다.

---

## 4. turtlesim_node를 실행하고 새로운 창에서 강의노트의 명령어(rosbag 제외)를 모두 실습하고 결과를 캡쳐하여 제출하라. 명령과 출력 결과가 일치하는지 설명하라.

- turtlesim_node: turtle_teleop_key로부터 속도 값을 토픽으로 받아 움직이게 하는 간단 2D 시뮬레이터 노드
<img width="929" height="541" alt="스크린샷 2026-09-22 103422" src="https://github.com/user-attachments/assets/f7265cb9-3f25-4563-b9a7-6cbfe1ed1d63" />

- turtle_teleop_key: turtlesim_node를 움직이게 하는 속도 값(/turtle1/cmd_vel)을 퍼블리시하는 노드
<img width="709" height="146" alt="스크린샷 2026-09-22 100414" src="https://github.com/user-attachments/assets/3700aee3-92ac-4033-b4e8-ef2b6df0ed55" />

- 현재 실행중인 노드 목록을 출력
<img width="621" height="62" alt="스크린샷 2026-09-22 100435" src="https://github.com/user-attachments/assets/7ebad982-65f8-44e1-9cd0-c5a44d4ae481" />

- 지정된 노드의 Subscriber, Publisher 정보를 출력  
  노드명은 /노드명 형식으로 작성
<img width="731" height="205" alt="스크린샷 2026-09-22 100511" src="https://github.com/user-attachments/assets/7117c065-f8a6-4238-ba41-1fc0ea8725b2" />
<img width="737" height="161" alt="스크린샷 2026-09-22 100528" src="https://github.com/user-attachments/assets/b1da8729-7b5d-432c-9b2b-61a1d11a5e9f" />


- 현재 사용중인 토픽 목록을 출력, `-t` 옵션을 붙이면 각 토픽의 메시지 타입까지 함께 출력
<img width="744" height="122" alt="스크린샷 2026-09-22 100557" src="https://github.com/user-attachments/assets/d77c1b28-e472-45d6-9ff3-04788f8b3a7b" />

- rqt_graph: 현재 개발 환경에서의 모든 노드와 토픽의 연결 관계를 그래프 형식으로 확인할 수 있는 명령어
<img width="866" height="554" alt="스크린샷 2026-09-22 100905" src="https://github.com/user-attachments/assets/c90f3007-d683-40bc-9e1a-e5c2ed747df1" />

- 지정된 토픽의 메시지 타입, Publisher/Subscriber 개수를 출력  
  토픽명은 /토픽명 형식으로 작성
<img width="707" height="80" alt="스크린샷 2026-09-22 100614" src="https://github.com/user-attachments/assets/1aef2b6d-4d80-4385-af12-39942964dea9" />

- 지정된 토픽으로 오가는 메시지 내용을 실시간으로 출력
<img width="587" height="377" alt="스크린샷 2026-09-22 101028" src="https://github.com/user-attachments/assets/02873ae8-ff59-4c53-b0eb-ec175705cc20" />

- 지정된 토픽의 대역폭(초당 전송 바이트 수)을 측정
<img width="622" height="200" alt="스크린샷 2026-09-22 101119" src="https://github.com/user-attachments/assets/ada08b31-6f0c-463c-a2a7-ee85f1de821f" />

- 지정된 토픽의 발행 주기(Hz)를 측정
<img width="728" height="126" alt="스크린샷 2026-09-22 101214" src="https://github.com/user-attachments/assets/84494e58-91a4-4426-b481-9a796981c05a" />

- 지정된 토픽에 메시지를 1회만 발행 (`--once`)
<img width="898" height="416" alt="스크린샷 2026-09-22 101447" src="https://github.com/user-attachments/assets/73491c53-5ef4-4a15-b6ca-24cb9d4a2a68" />

- 지정된 토픽에 메시지를 지정한 주기(Hz)로 반복 발행 (`--rate 1`)
<img width="903" height="629" alt="스크린샷 2026-09-22 101634" src="https://github.com/user-attachments/assets/c56eaf27-f91c-4be9-8103-8a182e312171" />
