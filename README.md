STM32F103C8T6 기반의 힘 및 3축 가속도 측정 센서입니다.

5 kg Load Cell과 HX711을 이용하여 물체에 작용하는 힘을 측정하고,
ADXL345 3축 가속도 센서를 이용하여 X/Y/Z축 가속도와 합성 가속도를 동시에 측정합니다.

**1. Overview**

본 프로젝트는 교육용 물리 실험 및 센서 데이터 수집을 목적으로 개발한 힘 센서입니다.

주요 기능은 다음과 같습니다.

- Load Cell을 이용한 힘 측정
- 힘을 N 및 kgf 단위로 제공
- ADXL345를 이용한 X/Y/Z축 가속도 측정
- X/Y/Z축을 이용한 합성 가속도 측정
- STM32에서 센서 데이터 처리
- EMA 필터를 이용한 데이터 안정화
- 영점 조정 지원
- 기준 무게를 이용한 Load Cell 보정
- I2C를 이용한 외부 Master 장치와의 통신
- 16 Byte 센서 데이터 패킷 제공


**2. Product**

완성품




PCB
<img width="840" height="582" alt="Image" src="https://github.com/user-attachments/assets/33daeb7f-3711-4d9c-9872-adcb64a3cae5" />

**3. System Architecture**
<img width="1448" height="1086" alt="Image" src="https://github.com/user-attachments/assets/533b14b6-fb33-458f-aa8c-dee86f077a78" />

**4. Hardware**
<img width="1536" height="1024" alt="Image" src="https://github.com/user-attachments/assets/c29a683f-a527-4047-803c-9da5d06a56d2" />


**5. Specifications**
<img width="1122" height="1402" alt="Image" src="https://github.com/user-attachments/assets/b47dfb1b-0898-485e-af73-1430b2353c2c" />


**6. Measurement Data**

힘 센서는 다음 데이터를 제공합니다.

<img width="1536" height="1024" alt="Image" src="https://github.com/user-attachments/assets/cb5195d7-e389-49db-9012-fd9ff6181384" />
Total Acceleration

합성 가속도는 X/Y/Z축 데이터를 이용하여 계산합니다.

Total Acceleration = √(X² + Y² + Z²)


**7. I2C Communication**

STM32는 I2C Slave로 동작하며 외부 Master 장치가 센서 데이터를 요청합니다.

Slave Address
0x08

I2C 주소는 고정으로 사용합니다.

Data Packet

센서 데이터는 총 16 Bytes로 구성됩니다.

typedef union {
    struct __attribute__((packed)) {
        int32_t force_N1000;
        int32_t force_kgf1000;
        int16_t accX_g1000;
        int16_t accY_g1000;
        int16_t accZ_g1000;
        int16_t accTotal_g1000;
    } val;

    uint8_t buffer[16];
} Packet_t;
<img width="1448" height="1086" alt="Image" src="https://github.com/user-attachments/assets/834c726d-6b12-4ea5-a7db-e865fe2c577e" />
Force Example
force_N1000 = 9810

실제 힘은 다음과 같습니다.

Force = 9810 / 1000
      = 9.810 N

kgf 데이터가 다음과 같다면,

force_kgf1000 = 1000

실제 힘은 다음과 같습니다.

Force = 1000 / 1000
      = 1.000 kgf
Acceleration Example
accY_g1000 = 1250

실제 Y축 가속도는 다음과 같습니다.

Acceleration Y = 1250 / 1000
               = 1.250 g

자세한 통신 규격은 아래 문서를 참고합니다.

Communication Documentation


**8. Data Processing**

STM32에서는 각 센서의 데이터를 읽은 후 다음 과정을 수행합니다.

Load Cell Raw Data
       │
       ▼
Zero Adjustment
       │
       ▼
Calibration
       │
       ▼
EMA Filtering
       │
       ▼
Deadband Processing
       │
       ▼
Force Calculation
       │
       ├──▶ N
       │
       └──▶ kgf


ADXL345 Raw Data
       │
       ▼
Offset Processing
       │
       ▼
EMA Filtering
       │
       ├──▶ Acc X
       ├──▶ Acc Y
       ├──▶ Acc Z
       │
       ▼
Total Acceleration
       │
       ▼
16-byte I2C Packet
EMA Filter

측정 데이터의 순간적인 노이즈를 줄이기 위해 EMA(Exponential Moving Average) 필터를 사용합니다.

EMA_ALPHA = 0.3
Weight/Force Deadband

무부하 상태에서 발생하는 미세한 Load Cell 측정값 변동을 줄이기 위해 데드밴드를 적용합니다.

DEADBAND_WEIGHT = [최종 설정값] g


**9. Calibration**

힘 센서는 기준 무게를 이용한 Load Cell 보정을 지원합니다.

Calibration Procedure
Load Cell에 아무것도 걸지 않은 무부하 상태에서 영점을 조정합니다.
알고 있는 기준 무게를 Load Cell에 장착합니다.
보정 버튼을 누릅니다.
Master에서 현재 측정값을 읽습니다.
기준 무게와 측정값을 이용하여 보정 비율을 계산합니다.
계산된 보정값을 STM32로 전송합니다.
STM32 내부 EEPROM에 보정값을 저장합니다.
Calibration Command
0xC1

자세한 보정 방법은 아래 문서를 참고합니다.

Calibration Documentation


**10. Firmware**

펌웨어는 크게 두 부분으로 구성됩니다.

firmware/
│
├── stm32/
│   └── force_sensor_slave.ino
│
└── ezmaker/
    └── force_sensor_master.ino
STM32 Firmware

STM32는 다음 작업을 담당합니다.

HX711 데이터 읽기
ADXL345 데이터 읽기
Load Cell 영점 처리
Load Cell 보정
EMA Filtering
Force(N) 계산
Force(kgf) 계산
X/Y/Z축 가속도 계산
합성 가속도 계산
16 Byte 데이터 패킷 생성
I2C Slave 통신
EZMAKER Firmware

외부 Master는 다음 작업을 담당합니다.

STM32 센서 데이터 요청
16 Byte 패킷 수신
Force(N/kgf) 데이터 처리
X/Y/Z축 가속도 데이터 처리
합성 가속도 데이터 처리
센서 데이터 출력
Load Cell 보정 명령 전송
