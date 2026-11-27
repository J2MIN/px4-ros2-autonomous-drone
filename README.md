# PX4 + ROS 2 Autonomous Drone

PX4 and ROS 2 based autonomous drone development using Pixhawk 5X and Raspberry Pi 5.

Pixhawk 5X와 Raspberry Pi 5를 기반으로 ROS 2 환경에서 자율비행 기능을 개발하는 프로젝트입니다.

현재 Pixhawk와 Raspberry Pi 사이의 UART 통신, uXRCE-DDS 연결, ROS 2에서 PX4 토픽 송수신 환경 구성을 완료했습니다.

## System Architecture

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

## Hardware

- Pixhawk 5X
- Raspberry Pi 5
- USB-to-TTL adapter

### Planned Sensors

- Intel RealSense D455
- LiDAR

## Software

- PX4
- ROS 2 Jazzy
- Micro XRCE-DDS
- px4_msgs
- Python
- C++

## Progress

- [x] Pixhawk TELEM2와 Raspberry Pi UART 연결
- [x] MAVLink 통신 확인
- [x] uXRCE-DDS 통신 구성
- [x] ROS 2에서 PX4 토픽 확인
- [x] px4_msgs workspace 구성 및 데이터 수신 확인
- [ ] ROS 2 Offboard Control
- [ ] RealSense D455 연동
- [ ] LiDAR 연동
- [ ] 장애물 회피
- [ ] 카메라 기반 객체 추적

## Development Log

개발 과정과 트러블슈팅 내용은 `docs/` 디렉터리에 정리할 예정입니다.

- Pixhawk와 Raspberry Pi UART 연결
- MAVLink 통신 테스트
- uXRCE-DDS 설정
- ROS 2와 px4_msgs 설정
- PX4 토픽 수신 확인

## Goal

ROS 2에서 카메라와 LiDAR 데이터를 처리하고, PX4 Offboard Control을 이용해 장애물 회피 및 객체 추적 기반 자율비행 기능을 구현하는 것을 목표로 합니다.