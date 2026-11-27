# Pixhawk 5X와 Raspberry Pi 5 ROS 2 연결

## 1. 목표

Pixhawk 5X와 Raspberry Pi 5를 연결하고, ROS 2에서 PX4의 토픽을 송수신할 수 있는 환경을 구성했다.

최종 구성은 다음과 같다.

```text
Pixhawk 5X
    │
    │ TELEM2 / UART
    ▼
USB-to-TTL
    │
    ▼
Raspberry Pi 5
    │
    ├── Micro XRCE-DDS Agent
    │
    └── ROS 2 Jazzy
          │
          └── px4_msgs
```

TELEM1은 기존 송수신기가 사용 중이기 때문에 Raspberry Pi 연결에는 TELEM2를 사용했다.

---

## 2. 하드웨어 연결

Pixhawk 5X의 TELEM2 포트와 USB-to-TTL 어댑터를 연결했다.

연결 방식은 다음과 같다.

```text
Pixhawk TX  → USB-TTL RX
Pixhawk RX  → USB-TTL TX
Pixhawk GND → USB-TTL GND
```

Pixhawk의 5V 핀은 연결하지 않고 Raspberry Pi는 별도 전원을 사용했다.

USB-to-TTL 어댑터는 Raspberry Pi의 USB 포트에 연결했다.

---

## 3. Raspberry Pi에서 USB 시리얼 장치 확인

Raspberry Pi에 SSH로 접속한 뒤 USB-to-TTL이 정상적으로 인식되는지 확인했다.

```bash
ls /dev/ttyUSB*
```

결과:

```text
/dev/ttyUSB0
```

따라서 이후 Pixhawk와의 시리얼 통신에는 `/dev/ttyUSB0`을 사용했다.

장치 권한도 확인했다.

```bash
ls -l /dev/ttyUSB0
```

결과:

```text
crw-rw---- 1 root dialout ...
```

현재 사용자 계정이 `dialout` 그룹에 포함되어 있지 않아 다음 명령으로 추가했다.

```bash
sudo usermod -aG dialout ubicomp
```

적용 후 SSH를 종료하고 다시 접속했다.

```bash
groups
```

`dialout` 그룹이 추가된 것을 확인했다.

---

## 4. MAVLink로 UART 통신 확인

ROS 2 연결을 진행하기 전에 TELEM2와 Raspberry Pi 사이의 UART 연결이 정상인지 먼저 MAVLink로 확인했다.

PX4에서는 TELEM2를 MAVLink용으로 설정하고 baud rate를 57600으로 맞췄다.

MAVProxy를 실행했다.

```bash
mavproxy.py --master=/dev/ttyUSB0 --baudrate 57600
```

출력:

```text
Waiting for heartbeat from /dev/ttyUSB0
Detected vehicle 1:1 on link 0
online system 1
POSCTL> Mode POSCTL
```

`Detected vehicle`과 `online system 1` 메시지가 출력되어 Pixhawk와 Raspberry Pi 사이의 MAVLink 통신이 정상임을 확인했다.

이 단계에서 다음 항목을 검증할 수 있었다.

- TELEM2 포트 동작
- TX/RX 배선
- USB-to-TTL 어댑터
- Raspberry Pi 시리얼 장치
- baud rate 설정

---

## 5. uXRCE-DDS 연결로 전환

최종 목적은 ROS 2에서 PX4 토픽을 직접 사용하는 것이므로 TELEM2를 MAVLink 대신 uXRCE-DDS 통신용으로 변경했다.

TELEM2는 uXRCE-DDS용으로 설정하고 baud rate는 921600으로 사용했다.

구조는 다음과 같다.

```text
PX4 uXRCE-DDS Client
        │
        │ TELEM2 / 921600
        ▼
USB-to-TTL
        │
        ▼
Micro XRCE-DDS Agent
        │
        ▼
ROS 2
```

---

## 6. Micro XRCE-DDS Agent 설치

Raspberry Pi에서 Micro XRCE-DDS Agent를 소스 빌드했다.

```bash
git clone https://github.com/eProsima/Micro-XRCE-DDS-Agent.git
cd Micro-XRCE-DDS-Agent
mkdir build
cd build
cmake ..
make
sudo make install
sudo ldconfig /usr/local/lib/
```

설치 후 Agent를 실행했다.

```bash
sudo MicroXRCEAgent serial --dev /dev/ttyUSB0 -b 921600
```

정상 연결 시 다음과 같은 로그가 출력됐다.

```text
create_client
session established
participant created
create_topic
create_subscriber
create_datareader
```

`session established`가 출력되어 Pixhawk의 uXRCE-DDS Client와 Raspberry Pi의 Agent가 정상적으로 연결된 것을 확인했다.

---

## 7. ROS 2 환경 확인

설치된 ROS 2 배포판을 확인했다.

```bash
printenv ROS_DISTRO
```

결과:

```text
jazzy
```

ROS 2 환경을 현재 터미널에 적용했다.

```bash
source /opt/ros/jazzy/setup.bash
```

PX4 관련 토픽을 확인했다.

```bash
ros2 topic list
```

예시:

```text
/fmu/out/vehicle_attitude
/fmu/out/vehicle_global_position
/fmu/out/vehicle_gps_position
/fmu/out/vehicle_odometry

/fmu/in/offboard_control_mode
/fmu/in/trajectory_setpoint
/fmu/in/vehicle_command
/fmu/in/vehicle_visual_odometry
```

`/fmu/out/...`과 `/fmu/in/...` 토픽이 확인되어 DDS 연결 자체는 정상임을 확인했다.

---

## 8. px4_msgs 문제 해결

처음에는 다음 명령이 정상적으로 동작하지 않았다.

```bash
ros2 topic echo /fmu/out/vehicle_attitude
```

오류:

```text
The message type 'px4_msgs/msg/VehicleAttitude' is invalid
```

확인 결과 ROS 2에 `px4_msgs` 패키지가 설치되어 있지 않았다.

```bash
ros2 interface show px4_msgs/msg/VehicleAttitude
```

결과:

```text
Unknown package 'px4_msgs'
```

PX4 메시지 타입을 ROS 2에서 해석하기 위해 별도의 workspace를 생성했다.

```bash
mkdir -p ~/ros2_px4_ws/src
cd ~/ros2_px4_ws/src
git clone -b release/1.17 https://github.com/PX4/px4_msgs.git
```

workspace를 빌드했다.

```bash
cd ~/ros2_px4_ws
source /opt/ros/jazzy/setup.bash
colcon build
```

빌드 후 workspace 환경을 적용했다.

```bash
source ~/ros2_px4_ws/install/setup.bash
```

패키지 인식 여부를 확인했다.

```bash
ros2 pkg list | grep px4_msgs
```

이후 다음 명령으로 PX4의 자세 데이터를 정상적으로 확인할 수 있었다.

```bash
ros2 topic echo /fmu/out/vehicle_attitude
```

---

## 9. 현재 상태

현재까지 다음 항목을 완료했다.

- Pixhawk 5X TELEM2와 Raspberry Pi 5 연결
- USB-to-TTL 장치 인식
- UART 통신 확인
- MAVLink 통신 확인
- uXRCE-DDS 연결
- Micro XRCE-DDS Agent 실행
- ROS 2 Jazzy에서 PX4 토픽 확인
- px4_msgs 설정
- PX4 데이터 수신 확인

현재 구성에서는 Raspberry Pi의 ROS 2에서 PX4 상태 데이터를 읽을 수 있고, `/fmu/in/...` 토픽을 이용해 PX4로 명령을 전달할 수 있는 기반이 구성된 상태다.

---

## 10. 다음 단계

다음 단계에서는 ROS 2 노드에서 PX4 데이터를 직접 subscribe하고, 이후 Offboard Control을 구현할 예정이다.

이후 단계적으로 다음 기능을 추가할 계획이다.

- ROS 2 Offboard Control
- RealSense D455 연동
- LiDAR 연동
- VIO 기반 위치 추정
- 장애물 회피
- 카메라 기반 객체 추적