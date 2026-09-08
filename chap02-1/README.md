##  **WSL2의 기능은 무엇인가?**
  - WSL2(Windows Subsystem for Linux 2)는 실제 리눅스 커널을 경량 가상머신 위에서 돌려서 윈도우 안에서 리눅스 배포판(Ubuntu 등)을 거의 네이티브 성능으로 실행하게 해줍니다.    
    별도의 가상머신 소프트웨어 없이 윈도우 파일시스템과 리눅스 파일시스템을 서로 접근할 수 있고, 네트워크도 통합되어 있어서 ROS2처럼 리눅스 전용 개발환경이 필요한 프로그램을 윈도우 PC에서 그대로 설치 및 실행할 수 있습니다.

##  **윈도우에서 ipconfig 명령어를 실행하고 출력결과를 설명하시오.**

<img width="589" height="233" alt="스크린샷 2026-09-08 122740" src="https://github.com/user-attachments/assets/88239252-b8b4-43df-8762-70d00ee9d7aa" />
  
  - vEthernet (WSL): WSL2가 실제로는 Hyper-V 기반의 경량 가상머신으로 동작하기 때문에, 윈도우가 자동으로 만들어주는 가상 네트워크 어댑터입니다. 윈도우 호스트와 WSL2 안의 리눅스가 서로 통신할 수 있도록 연결해주는 다리 역할을 합니다.
  - IPv4 주소: WSL2 가상머신에 할당된 사설 IP 대역입니다. WSL2를 새로 켤 때마다 바뀔 수 있어서, 예를 들어 ROS2 노드끼리 통신할 때 이 주소를 참고하게 됩니다.
  - 서브넷 마스크: 이 가상 네트워크의 범위를 나타내며, 보통 /20(255.255.240.0) 정도로 넓게 잡혀 있습니다.
  - 기본 게이트웨이가 비어있는 경우: 이 어댑터는 외부 인터넷으로 나가는 용도가 아니라 윈도우 호스트 ↔ WSL2 간 내부 통신 전용이기 때문에 게이트웨이가 없는 게 정상입니다.
  - (Hyper-V firewall): 이 트래픽이 Hyper-V의 방화벽 규칙을 거친다는 표시로, WSL2와 윈도우 사이의 네트워크 트래픽이 Hyper-V 가상 스위치를 통해 필터링된다는 의미입니다.

##  **리눅스에서 ifconfig 명령어를 실행하고 출력결과를 설명하시오.**

<img width="556" height="188" alt="스크린샷 2026-09-08 120604" src="https://github.com/user-attachments/assets/92d53286-a98b-4bdb-af3c-40d49edbbd74" />

  - eth0: WSL2 안에서 리눅스가 보는 기본 네트워크 인터페이스 이름입니다. 물리 이더넷 카드가 아니라, 윈도우가 만들어준 가상 네트워크 어댑터를 리눅스 쪽에서 이렇게 인식하는 것입니다.
  - inet: 앞서 윈도우 ipconfig에서 보신 vEthernet (WSL)과 같은 대역(172.x.x.x)의 IP 주소가 할당됩니다. 즉 윈도우 호스트가 WSL2에게 이 대역 안에서 하나의 주소를 내부적으로 부여하는 것이고, 두 결과가 같은 네트워크 대역이면 정상적으로 연결되어 있다는 뜻입니다.
  - netmask: 앞서 윈도우 쪽 서브넷 마스크와 동일한 범위(보통 255.255.240.0)를 갖습니다.
  - ether (MAC 주소): WSL2 가상 어댑터에 부여된 가상 MAC 주소입니다.
  - RX/TX packets: 지금까지 이 인터페이스로 주고받은 패킷 수와 바이트 양으로, 네트워크 통신이 실제로 발생하고 있는지 확인할 수 있는 지표입니다. 둘 다 0이 아니라면 정상적으로 트래픽이 오간 것입니다.

##  **윈도우즈의 ssh client에서 wsl2-ubuntu 24.04의 ssh server에 원격 접속하시오.**  

<img width="751" height="369" alt="스크린샷 2026-09-08 120918" src="https://github.com/user-attachments/assets/f5fb782d-a575-4b38-84e3-dca91ee49f87" />

##  **윈도우즈의 vscode에서 wsl2-ubuntu 24.04의 ssh server에 원격 접속하시오.**  

<img width="944" height="275" alt="스크린샷 2026-09-08 121209" src="https://github.com/user-attachments/assets/33227469-e589-47dc-99a5-3de3679e7a8b" />
<img width="950" height="506" alt="스크린샷 2026-09-08 121309" src="https://github.com/user-attachments/assets/6ee6268f-a0d1-47c0-8940-961b289ae411" />
