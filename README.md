STM32F103C8T6 기반의 힘 및 3축 가속도 측정 센서입니다.

5 kg Load Cell과 HX711을 이용하여 물체에 작용하는 힘을 측정하고,
ADXL345 3축 가속도 센서를 이용하여 센서의 움직임 변화를 동시에 측정합니다.

**1. Overview**

본 프로젝트는 교육용 실험 및 센서 데이터 수집을 목적으로 개발한 힘 센서입니다.

주요 기능은 다음과 같습니다.

- Load Cell을 이용한 무게 측정
- 측정된 무게를 힘(N)으로 변환
- ADXL345를 이용한 X/Y/Z축 가속도 측정
- STM32에서 센서 데이터 처리
- EMA 필터를 이용한 데이터 안정화
- 영점 조정 지원
- 기준 무게를 이용한 힘 센서 보정
- I2C를 이용한 외부 Master 장치와의 통신

**2. Product**

<img width="840" height="582" alt="Image" src="https://github.com/user-attachments/assets/33daeb7f-3711-4d9c-9872-adcb64a3cae5" />




