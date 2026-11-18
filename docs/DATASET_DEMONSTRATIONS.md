# Dataset demonstrations

## Recorded task

ROBOTIS AI Worker FFW BG2 Rev4의 오른팔로 검정 박스를 집어 갈색 박스 위에 놓는 모방학습 시연입니다. 공개한 두 영상은 원본 작업 데이터셋의 오른쪽 헤드 카메라 기록에서 선정했습니다.

| Clip | Episode | Length | Source camera | Content |
|---|---:|---:|---|---|
| [Demonstration A](assets/dataset-demo-ep10.mp4) | 10 | 15.034 s | `cam_right_head` | 검정 박스 접근·파지·이송·목표 박스 위 놓기 |
| [Demonstration B](assets/dataset-demo-ep40.mp4) | 40 | 12.234 s | `cam_right_head` | 목표 박스 배치가 다른 시연의 집기·놓기 |

**Media category: recorded dataset demonstrations.** 학습 모델의 자율 실행 성공률이나 특정 체크포인트의 성능을 나타내는 영상이 아닙니다. 실기체 데이터 수집 장면과 학습된 정책의 실행 결과를 구분하여 제시합니다.

## Presentation format

- Full episode at original speed; no temporal cuts or speed changes.
- Original camera frame retained, with a caption strip added below.
- MP4: H.264, 672 × 408, 30 fps; source camera frame 672 × 376.
- README preview: looping animated WebP at 10 fps, preserving playback duration to frame-sampling precision.
- Audio is not included in the portfolio derivative.
- Original dataset files are unchanged; SHA256 before/after verification recorded during export.

## Provenance

Source dataset: `Task_1_blackbox_stack_real_MCAP_lerobot_v21`.

Camera stream: `observation.images.rgb.cam_right_head`.

The clips are selected from the 50-episode black-box stacking dataset used in this project. Episode 10 belongs to the 40-episode training partition; episode 40 belongs to the 10-episode validation partition in the controlled fine-tuning experiment.

Source identifiers and checksums are recorded in [media_manifest.json](../results/media_manifest.json). The manuscript and media selection emphasize completed task demonstrations; the source material is preserved independently of the publication derivatives.
