## 실습과제 1. `ros2 topic echo` 결과의 6개 숫자

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

## 실습과제 2. `ros2 topic bw` 결과의 4개 숫자

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

## 실습과제 3. `ros2 topic hz` 결과의 5개 숫자

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

## 실습과제 4.
<img width="709" height="146" alt="스크린샷 2026-09-22 100414" src="https://github.com/user-attachments/assets/88e964b0-7d5c-4288-b480-7a05ea4f07c4" />
<img width="621" height="62" alt="스크린샷 2026-09-22 100435" src="https://github.com/user-attachments/assets/b6a5bcfb-4f5a-4a29-b854-65a612593852" />

<img width="731" height="205" alt="스크린샷 2026-09-22 100511" src="https://github.com/user-attachments/assets/6ba4ed4c-3674-4717-8c98-6518b926cb25" />
<img width="737" height="161" alt="스크린샷 2026-09-22 100528" src="https://github.com/user-attachments/assets/b1b7b921-5b02-4301-bcc1-f8cadaeef166" />
<img width="744" height="122" alt="스크린샷 2026-09-22 100557" src="https://github.com/user-attachments/assets/469e44b1-2cb1-4ff1-9c3b-a46a7824866c" />
<img width="707" height="80" alt="스크린샷 2026-09-22 100614" src="https://github.com/user-attachments/assets/d9fdcd1d-ff58-416d-8b23-6434e8e20735" />
<img width="866" height="554" alt="스크린샷 2026-09-22 100905" src="https://github.com/user-attachments/assets/2328b0d8-d883-4a99-8f93-6ec7aaaa7908" />
<img width="587" height="377" alt="스크린샷 2026-09-22 101028" src="https://github.com/user-attachments/assets/bf2931de-b71d-413d-bce4-43327c8d5293" />
<img width="622" height="200" alt="스크린샷 2026-09-22 101119" src="https://github.com/user-attachments/assets/4e97eb67-f618-43f6-911c-52c38c4f5ed5" />
<img width="728" height="126" alt="스크린샷 2026-09-22 101214" src="https://github.com/user-attachments/assets/3e41b14c-1b6f-42f9-8246-dafe1a6e8fec" />
<img width="898" height="416" alt="스크린샷 2026-09-22 101447" src="https://github.com/user-attachments/assets/b3bac4f6-ead1-4154-999a-53c2e2ec2335" />
<img width="903" height="629" alt="스크린샷 2026-09-22 101634" src="https://github.com/user-attachments/assets/2676ef5c-a027-427a-9d78-c642b41436e5" />

