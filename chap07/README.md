## 실습과제 1. 모든 명령어의 사용예를 실습하고 결과를 캡쳐하여 제출하시오.

- turtlesim_node: turtlesim 패키지의 노드를 실행하면 노드명 /turtlesim으로 시작되며 거북이 한 마리가 생성됨
<img width="898" height="595" alt="스크린샷 2026-09-22 104124" src="https://github.com/user-attachments/assets/e8db6795-943b-437f-92c0-5425b503236d" />

- 실행중인 노드의 서비스 목록을 출력
<img width="651" height="400" alt="스크린샷 2026-09-22 104150" src="https://github.com/user-attachments/assets/d2687eb8-a478-44e6-9569-a78fc0747480" />

- 서비스에서 사용하는 메시지 형태(type)를 출력  
  서비스명은 /서비스명 형식으로 작성
<img width="714" height="161" alt="image" src="https://github.com/user-attachments/assets/5749caa0-8518-412a-9e15-63a636fb2e86" />

- 서비스 목록과 서비스 타입을 함께 출력
<img width="780" height="162" alt="스크린샷 2026-09-22 104346" src="https://github.com/user-attachments/assets/d7082d0d-5264-4787-ae11-198ddec3fc0c" />

- 지정한 형태(type)를 사용하는 서비스명을 출력
<img width="770" height="101" alt="스크린샷 2026-09-22 104455" src="https://github.com/user-attachments/assets/93980346-86d9-4d99-beea-83e6fa1fe5f6" />

- teleop_turtle 노드를 실행하고 방향키로 거북이를 이동시킴
<img width="879" height="507" alt="스크린샷 2026-09-22 104616" src="https://github.com/user-attachments/assets/841498c7-0db0-4450-8fe8-0d392292f057" />

- /clear 서비스 요청: 거북이가 움직이며 표시된 이동 궤적을 지움
<img width="762" height="578" alt="스크린샷 2026-09-22 104730" src="https://github.com/user-attachments/assets/e6271ea5-cab7-4216-a190-dcf82231abcc" />

- /kill 서비스 요청: 지정한 이름의 거북이를 화면에서 제거함
<img width="896" height="667" alt="스크린샷 2026-09-22 104847" src="https://github.com/user-attachments/assets/6cbb4e82-35f8-455a-b119-d896c6b1a139" />

- /reset 서비스 요청: 모든 거북이와 궤적을 지우고 거북이를 원점에 재배치함
<img width="759" height="624" alt="스크린샷 2026-09-22 115141" src="https://github.com/user-attachments/assets/0a33464d-6169-4311-b01a-29bb464dd176" />

- /turtle1/set_pen 서비스 요청: 지정한 거북이의 이동 궤적 색과 굵기를 변경함
<img width="910" height="676" alt="스크린샷 2026-09-22 115807" src="https://github.com/user-attachments/assets/f2a82e50-ae8d-4b4e-91b6-6bb189ed0a7c" />

- /spawn 서비스 요청: 지정한 위치와 방향에 지정한 이름으로 거북이를 추가함
<img width="910" height="756" alt="스크린샷 2026-09-22 120356" src="https://github.com/user-attachments/assets/1e1d0de2-f607-471b-94b0-e34afcbb061d" />

## 실습과제 2. 아래 서비스 리스트에서 /turtle1/teleport_absolute, /turtle1/teleport_relative 서비스를 설명하고 실습결과를 제출하라.

- /turtle1/teleport_absolute (turtlesim/srv/TeleportAbsolute) : 거북이를 화면의 절대좌표(x, y)와 절대각도(theta)로 순간이동시키는 서비스  
- ros2 service call /turtle1/teleport_absolute turtlesim/srv/TeleportAbsolute "{x: 5.5, y: 5.5, theta: 0.0}"

<img width="914" height="586" alt="스크린샷 2026-09-22 123257" src="https://github.com/user-attachments/assets/c2376f3e-e183-4aed-8eb3-40d249af4262" />

- 실행결과
<img width="909" height="680" alt="스크린샷 2026-09-22 123311" src="https://github.com/user-attachments/assets/134d069b-2c13-43ad-ae93-5832430cdf19" />

- /turtle1/teleport_relative (turtlesim/srv/TeleportRelative) : 거북이의 현재 위치를 기준으로 이동거리(linear)와 회전각도(angular)만큼 순간이동시키는 서비스  
- ros2 service call /turtle1/teleport_relative turtlesim/srv/TeleportRelative "{linear: 2.0, angular: 1.57}"
<img width="914" height="619" alt="스크린샷 2026-09-22 123526" src="https://github.com/user-attachments/assets/c8e43e38-f166-40ee-b473-12588e8887f5" />
