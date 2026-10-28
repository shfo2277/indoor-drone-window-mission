# 실내 창고 자율비행 드론 – 장애물 회피 · 창문 통과 미션

**건국대학교 스마트운행체공학과 졸업 프로젝트** (2024.09 ~ 2025.07)

GPS가 없는 실내 물류창고에서 드론이 **스스로 지도를 만들고, 장애물을 피해 경로를 계획하고, 카메라로 창문을 찾아 통과한 뒤 착륙**하는 자율비행 미션입니다.
RGB-D 카메라로 지도를 작성하고(RTAB-Map), 포인트클라우드 기반 RRT\*로 경로를 계획하며, PX4 SITL + Gazebo Classic 시뮬레이션에서 ROS 2로 구현했습니다.

> 경로 계획 알고리즘 개선(RRT\* 재계획 · 벤치마크)은 졸업 논문 저장소 [indoor-drone-obstacle-avoidance](https://github.com/shfo2277/indoor-drone-obstacle-avoidance)에 정리했습니다.

## 시연 영상

![시연 영상 (8배속)](docs/demo.gif)

*8배속 요약. 왼쪽: Gazebo 시뮬레이션 / 오른쪽: RViz(지도·경로) + 카메라 창문 탐지 화면*

▶ [전체 영상 보기 (2분 17초, webm)](./%EC%B5%9C%EC%A2%85.webm)

## 미션 흐름

1. **이륙**: PX4 Offboard 모드로 전환 후 고도 1 m까지 상승
2. **장애물 회피**: RTAB-Map 포인트클라우드로 만든 점유 격자 위에서 RRT\* 경로를 계획하고, 새 지도가 들어오면 재계획
3. **창문 탐색**: 벽 앞에서 좌우로 이동하며 카메라로 사각형 개구부 탐지
4. **정렬**: 창문 중심이 화면 중앙에 오도록 좌우·상하 위치 보정
5. **통과 → 목적지 → 착륙**

## 시뮬레이션 환경

![Gazebo 시뮬레이션 환경](docs/images/sim_environment.jpg)

창고(smartfactory) 맵에 기둥과 창문이 뚫린 벽을 배치했습니다.

## 시스템 구성

| 구성 요소 | 역할 |
|---|---|
| **PX4 SITL + Gazebo Classic** | 드론(iris + depth 카메라) 비행 시뮬레이션 |
| **Micro XRCE-DDS Agent** | PX4 ↔ ROS 2 통신 (`/fmu/in/*`, `/fmu/out/*`) |
| **RTAB-Map** | RGB-D 영상으로 위치 추정 + 3D 지도 작성 (`/cloud_map`) |
| **포인트클라우드 필터** | 높이 필터 → Voxel 다운샘플링 → 이상치 제거 (`/filtering_map`) |
| **경로계획 노드 (planning)** | 2D 점유 격자 생성, RRT\* 경로계획, 비행 상태머신, PX4 setpoint 전송 |
| **비전 노드 (canny_edge_detector)** | Canny 에지로 창문(사각형) 탐지 → 중심·크기 발행 (`/hole_info`) |
| **IMU 브리지** | PX4 IMU → `sensor_msgs/Imu` 변환 (RTAB-Map 입력) |

![ROS 2 노드 그래프](docs/images/ros_node_graph.png)

### 지도 작성 (RTAB-Map)

![RTAB-Map 포인트클라우드 지도](docs/images/rtabmap_pointcloud.jpg)

### 경로계획 (RViz)

![RRT 트리와 경로, 창문 탐지](docs/images/rrt_planning_rviz.jpg)

*분홍 점: RRT 트리 노드 / 초록 선: 계획 경로 / 왼쪽 아래: 카메라 화면의 창문 탐지(초록 박스)*

## 개발 환경

- Ubuntu 22.04, ROS 2 Humble
- PX4-Autopilot (SITL), Gazebo Classic 11, Micro XRCE-DDS Agent
- RTAB-Map (rtabmap_ros), PCL, OpenCV

## 저장소 구성

```
├── src/
│   ├── planning/                    # 경로계획 + 비행 제어
│   │   └── src/planning.cpp         #   planning_node: 점유 격자, RRT* 경로계획, 재계획, 미션 상태머신
│   ├── cannyedge_test1/             # 비전: Canny 에지 기반 창문(사각형) 탐지 → /hole_info
│   ├── cannyedge_test1_interfaces/  # HoleInfo.msg (창문 중심, 크기, 통과 가능 여부)
│   ├── bridge_imu_ws/               # PX4 IMU → sensor_msgs/Imu 변환
│   └── px4_msgs/                    # PX4 ROS 2 메시지 정의
├── docs/                            # README용 GIF·이미지
└── 최종.webm                        # 시연 영상 원본
```

