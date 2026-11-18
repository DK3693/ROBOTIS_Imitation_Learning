# Selected engineering results

## Hardware demonstration

검정 박스 집기와 갈색 박스 위 놓기 동작을 실제 BG2 Rev4에서 확인했습니다. 해당 결과는 오른팔 8D X-VLA **8,000 step 배포 모델**에 대한 사용자 관찰입니다. 실기체 시연과 아래의 encoder freeze 비교 학습은 서로 다른 검증 범위입니다.

## Controlled encoder-freeze comparison

두 모델은 동일한 `xvla-base` 초기 가중치에서 각각 새로 학습했습니다. 학습 변수 차이는 `freeze_vision_encoder` 하나이며, 저장된 최종 `train_config.json`을 비교하여 확인했습니다.

- 6,000 steps; batch 8; seed 42; BF16; base LR `1e-4`.
- 40 train episodes / 10 validation episodes; 18,267 / 3,745 frames.
- Image augmentation OFF; action chunk size 30.
- Official scheduler: effective warmup 200 / decay 6,000 steps.
- Language encoder remains trainable in both conditions.

| Step | Baseline train | Freeze train | Baseline validation | Freeze validation |
|---:|---:|---:|---:|---:|
| 1,000 | 0.068 | 0.069 | 0.0829 | 0.0789 |
| 2,000 | 0.032 | 0.037 | 0.0615 | 0.0618 |
| 3,000 | 0.024 | 0.031 | 0.0567 | 0.0541 |
| 4,000 | 0.018 | 0.027 | 0.0569 | 0.0519 |
| 5,000 | 0.017 | 0.025 | 0.0535 | 0.0492 |
| 6,000 | 0.018 | 0.025 | 0.0533 | 0.0491 |

**Relative validation-loss reduction:** `(0.0533 - 0.0491) / 0.0533 × 100 = 7.88%`.

Train loss is the mean over the preceding 25 updates; validation loss is computed on the held-out episodes. These are logged learning objectives rather than robot task success rates. The comparison uses one paired seed and the same validation split.

## Training resource profile

| Quantity | Baseline | Vision freeze |
|---|---:|---:|
| Total model parameters | 879,482,456 | 879,482,456 |
| Trainable parameters | 879,482,456 | 518,850,136 |
| Trainable vision parameters | 360,632,320 | 0 |
| Logged run duration | 4,774 s | 3,373 s |

The vision tower contains 360,632,320 parameters. Language parameters remain trainable. Observed run duration decreased by 29.35% on the same GPU environment, including the training run's validation/checkpoint intervals; this is a run-level comparison, not an isolated kernel benchmark.

## ARM64 and I/O verification

ARM64 Orin 환경에서 저장된 가중치·전후처리기를 로드하고 기록 입력에 대한 `30 × 8` 유한 액션 출력을 확인했습니다. 비구동 검사에서는 관절 메시지 순서 변화에도 오른팔 이름 기반 선택을 확인했으며, 출력 액션 전체가 오른팔 명령 대상으로 구성되는 것을 검증했습니다.

25초 미리보기 수신 측정:

| Stream | Samples | Observed arrival rate |
|---|---:|---:|
| Preview | 2,500 | 100.0003 Hz |
| JointState | 2,500 | 99.9984 Hz |
| Right head | 749 | 29.9528 Hz |
| Right wrist | 751 | 30.0262 Hz |
| Left head | 749 | 29.9508 Hz |

Preview 최대 수신 간격은 17.56ms이며, 50ms 초과 공백은 0회입니다. 이 값은 구독 지점에서 측정한 전송·스케줄링 포함 도착 주기이며 GPU 추론 시간이나 실기체 제어 성능을 직접 나타내지 않습니다.

## Integrity and evidence

- 19D 원본과 오른팔 8D 추출 결과: 전체 22,012프레임의 해당 State·Action 값 동일.
- Freeze 실험 보호 대상: 데이터·기존 체크포인트·캐시·기준선 결과 250개 파일의 SHA256 전후 동일.
- 공식 Cyclo 및 LeRobot 저장소: 실험 종료 시 Git working tree clean.
- 확인한 공식 학습 관련 소스 7개: SHA256 전후 동일.
- 수치 출처: 저장된 비교 JSON, 학습 로그, ARM64 비구동 검사 및 미리보기 수신 로그.

공개 JSON에는 측정값과 설정만 수록하며 호스트 경로·로그인 정보·원본 시연·모델 가중치는 포함하지 않습니다.
