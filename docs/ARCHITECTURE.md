# System architecture

## Observation contract

작업 시연의 오른팔 관절 7개와 그리퍼를 동일한 순서의 8D State·Action으로 정의합니다. 이름 기반 채널 선택으로 로봇 전체 관절 배열과 정책 입력을 연결하며, 19D 원본과 오른팔 추출값의 전 프레임 동일성을 검증했습니다.

세 카메라는 오른쪽 헤드, 오른쪽 손목, 왼쪽 헤드 순으로 각각 `image`, `image2`, `image3`에 연결됩니다. 오른쪽 손목 영상의 배포 설정에는 270° 회전 메타데이터가 포함됩니다. 이러한 관절·영상 설정은 작업별 구성값이며 모델 구조나 공식 추론 엔진의 수정이 아닙니다.

## Policy execution

저장된 전처리기는 상태·영상을 모델 입력으로 변환하고 자연어 작업 지시를 토큰화합니다. X-VLA는 30개 시점의 8D 액션을 생성합니다. 저장된 후처리 통계로 복원된 출력은 오른팔 관절 이름 및 명령 대상에 대응됩니다.

- Robot type: `ffw_bg2_rev4`.
- Policy: X-VLA / BF16.
- Action target: `/leader/joint_trajectory_command_broadcaster_right/joint_trajectory`.
- Channels: `arm_r_joint1` through `arm_r_joint7`, followed by `gripper_r_joint1`.

`3D Sim Deploy`는 예측 관절 동작의 미리보기 경로이며, 실기체 명령 게이트와 구분합니다. 관측·정책·액션 대상 검증은 명령 발행 없이 수행할 수 있도록 구성했습니다.

## Compute architecture

| Component | Environment | Responsibility |
|---|---|---|
| Training workstation | x86_64 / RTX 3090 / Docker | 모델 학습, 기록 입력 검증, 지표 집계 |
| Robot computer | aarch64 / NVIDIA Orin / Docker | ARM64 모델 로딩, 추론 및 Cyclo Worker |
| Cyclo Core | ROS 2 Jazzy / Zenoh | 정책 상태 및 실행 경로 관리 |
| Robot interface | BG2 Rev4 / named joint interfaces | 관측 수신 및 오른팔 액션 연결 |

실험은 학습 PC의 GPU 한 개를 사용했습니다. Core와 Worker의 소스는 동일한 Cyclo 커밋에 맞춰 구성하고, 데이터 및 체크포인트는 읽기 전용 검증과 SHA256 비교로 식별했습니다.

## Version identity

- Cyclo: `4fa05374d61666c440ad0b646241e69ea5ce0fdb`.
- LeRobot: `83405ab5059297d34faff105bd95690c75953a1a`.
- Zenoh ROS 2 SDK: `be2c4d4595305a9c282fca09820a8b3bfb8076a3`.
- Base model: `lerobot/xvla-base`, cached revision `cdb7964e4fe842935d671bfab5a5ebe00a96648c`.

## Ownership of implementation

공식 프로젝트는 모델 아키텍처, 학습·추론 엔진, 런타임과 로봇 인터페이스를 제공합니다. 본 프로젝트에서는 데이터·관절·카메라 정합, 모델별 설정, 실험 실행 관리, 비구동 검증 및 결과 분석을 구성했습니다. 공개 문서는 구현 이력 대신 완성된 시스템의 책임과 검증 결과를 설명합니다.
