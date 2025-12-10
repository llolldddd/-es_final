# 🧊 Smart Refrigerator – 스마트 냉장고 관리 시스템

Arduino + Processing Server + App Inventor 를 이용한  
**문 열림 감지 / 온습도 모니터링 / 부저 경고 / 앱 제어** 시스템.

냉장고 문이 일정 시간 열려 있으면 부저로 경고하고,  
스마트폰 앱에서 LED와 부저를 원격 제어할 수 있습니다.

---

## ✨ 주요 기능 (Features)

- **문 열림 감지**
  - 초음파 센서(HC-SR04)로 냉장고 문과의 거리를 측정
  - 기준 거리(예: 10cm)를 넘으면 `문 열림`, 이하면 `문 닫힘`으로 판별

- **온도·습도 모니터링**
  - DHT11 센서로 냉장고 내부 온도(℃)와 습도(%) 측정
  - 1초마다 Arduino → Processing 서버로 상태 전송
  - App Inventor 앱에서 실시간으로 확인

- **문 장시간 열림 경고 (부저 알림)**
  - 문이 **10초 이상** 계속 열려 있는 경우 부저 자동 ON
  - 앱에서 “알람 끄기” 버튼을 누르면 서버 → Arduino로 `BUZZER_OFF` 명령 전송

- **앱을 통한 원격 제어**
  - App Inventor 앱에서
    - 💡 LED 켜기 (`/cmd?led=1`)
    - 🌑 LED 끄기 (`/cmd?led=0`)
    - 🔕 부저 끄기 (`/cmd?buzzer=off`)
  - 버튼 클릭 → HTTP 요청 → Processing 서버 → Arduino 제어

- **Arduino – 서버 – 앱 전체 연동**
  - Arduino: 센싱(초음파, DHT11) + 제어(LED, 부저) + 상태 문자열 전송
  - Processing: 시리얼 수신 + HTTP 서버 역할
  - App Inventor: Web 컴포넌트로 서버와 통신, UI 표시 및 제어

---

## 🧱 시스템 구조 (Architecture)

[ Arduino UNO ]
    ├─ DHT11 (온습도)
    ├─ HC-SR04 (초음파, 문 거리)
    ├─ LED (상태/조명)
    └─ Buzzer (알림)

        │ (USB, Serial 9600bps)
        ▼

[ Processing Server (PC) ]
    ├─ Arduino와 시리얼 통신
    ├─ /state  : 상태 CSV 제공 (STATE,door,temp,hum)
    └─ /cmd    : 제어 명령을 Arduino로 전달

        │ (HTTP, 같은 Wi-Fi)
        ▼

[ App Inventor App (Android) ]
    ├─ 센서값 실시간 표시 (문 상태, 온도, 습도)
    └─ LED / Buzzer 제어 버튼

---

## 🔌 하드웨어 구성 (Hardware)
사용 부품
Arduino Uno R3

DHT11 온습도 센서

HC-SR04 초음파 센서

LED 1개 + 저항

피에조 부저 1개

브레드보드, 점퍼 케이블

핀 연결
text
코드 복사
DHT11
  DATA → D2
  VCC  → 5V
  GND  → GND

HC-SR04
  TRIG → D9
  ECHO → D10
  VCC  → 5V
  GND  → GND

LED
  (+, 긴 다리) → 저항 → D3  (LED_PIN)
  (−, 짧은 다리) → GND

Buzzer
  + → D5 (BUZZER_PIN)
  − → GND
⚠ GND는 반드시 공통으로 연결해야 합니다.

---

## 💻 소프트웨어 구성 (Software Components)
1) Arduino
언어: Arduino C/C++

역할:

DHT11 / HC-SR04 센서값 읽기

문 상태 판단 (isDoorOpen)

문이 10초 이상 열려 있으면 BUZZER_PIN HIGH → 부저 ON

1초마다 상태 전송:

text
코드 복사
STATE,<door>,<temp>,<hum>
예: STATE,1,4.20,60.5
시리얼로 들어오는 명령 처리:

"LED_ON" → LED 켜기

"LED_OFF" → LED 끄기

"BUZZER_OFF" → 부저 끄기

2) Processing Server
언어: Processing (Java 기반)

라이브러리:

java
코드 복사
import processing.serial.*;
import processing.net.*;
역할:

Arduino와 시리얼 통신

serialEvent()에서 STATE,door,temp,hum 문자열 수신

split()으로 파싱 후 전역 변수 door, temp, hum에 저장

HTTP 서버 (포트 8080)

/state
→ 현재 상태를 CSV 문자열로 응답
예: STATE,1,4.20,60.5

/cmd
→ 쿼리 파라미터에 따라 Arduino에 명령 전송:

java
코드 복사
// 예시
if (path.indexOf("buzzer=off") != -1) arduino.write("BUZZER_OFF\n");
if (path.indexOf("led=1")      != -1) arduino.write("LED_ON\n");
if (path.indexOf("led=0")      != -1) arduino.write("LED_OFF\n");
PC 화면 출력

draw()에서 문 상태, 온도, 습도 텍스트로 표시

3) App Inventor App
플랫폼: MIT App Inventor (웹)

주요 컴포넌트:

Web1 (서버와 HTTP 통신)

Clock1 (2초마다 상태 요청)

LabelDoor, LabelTemp, LabelHum 등

ButtonLedOn, ButtonLedOff, ButtonBuzzerOff

동작 요약:

Clock1.Timer → /state 호출

Web1.GotText → 응답 문자열을 split해서 문 상태/온도/습도 UI 표시

버튼 클릭 시 /cmd?... 요청으로 LED/부저 제어

---

## 📁 저장소 구조 예시 (Repository Structure)
실제 폴더 구조에 맞게 파일 이름만 수정해서 쓰면 됩니다.

text
코드 복사
.
├─ arduino/
│  └─ smart_fridge.ino
├─ processing/
│  └─ processing_server.pde
├─ appinventor/
│  └─ smart_fridge_app.aia
├─ docs/
│  └─ report.pdf (선택)
└─ README.md
⚙ 설치 및 실행 방법 (Setup & Run)
1) Arduino 설정
Arduino IDE 설치

DHT 라이브러리 설치

스케치 → 라이브러리 포함하기 → 라이브러리 관리

DHT sensor library 검색 후 설치

arduino/smart_fridge.ino 열기

보드: Arduino Uno, 포트: 실제 연결 포트 선택

업로드 버튼 클릭

2) Processing 서버 실행
Processing 설치 (https://processing.org)

processing/processing_server.pde 파일 열기

코드 상단에서 포트 목록 확인:

java
코드 복사
println(Serial.list());
arduino = new Serial(this, Serial.list()[2], 9600);
프로그램을 한 번 실행해보고 콘솔에 나오는 목록 중
Arduino가 연결된 포트 인덱스로 [2] 부분 수정

스케치 실행

콘솔에 STATE,... 로그가 뜨면 Arduino와 정상 통신 중

Processing 서버 시작됨 → http://localhost:8080 같은 메시지 확인

3) App Inventor 앱 설정
MIT App Inventor 접속

새 프로젝트 생성 후, UI 구성:

문 상태 / 온도 / 습도 표시용 Label

LED 켜기/끄기, 알람 끄기 Button

Non-visible 컴포넌트: Web1, Clock1

Clock1.Timer 블록:

Clock1.Timer 이벤트에서:

text
코드 복사
set Web1.Url to "http://<PC_IP>:8080/state"
call Web1.Get
Web1.GotText 블록:

responseContent 안에 "STATE"가 포함된 경우에만
split text at ","로 나누어

2번째 값: 문 상태 (0/1)

3번째 값: 온도

4번째 값: 습도
를 Label에 표시

"OK" 응답(/cmd)의 경우는 파싱하지 않고 무시

버튼 블록:

text
코드 복사
LED 켜기 버튼:  Web1.Url = "http://<PC_IP>:8080/cmd?led=1"
LED 끄기 버튼:  Web1.Url = "http://<PC_IP>:8080/cmd?led=0"
알람 끄기 버튼: Web1.Url = "http://<PC_IP>:8080/cmd?buzzer=off"
→ call Web1.Get
앱 빌드(.apk) 후 안드로이드 폰에 설치
👉 PC와 스마트폰은 같은 Wi-Fi 네트워크에 있어야 함

---

## 🚀 사용 방법 (How to Use)
Arduino 전원을 켜고, 센서/부저/LED가 제대로 연결되어 있는지 확인합니다.

Processing 서버를 실행하여 시리얼과 HTTP 서버가 정상 동작하는지 확인합니다.

스마트폰에서 앱을 실행합니다.

앱 메인 화면에서:

문 상태(열림/닫힘), 현재 온도, 습도를 확인할 수 있습니다.

냉장고 문을 실제로 열어두면 일정 시간이 지난 후 부저가 울립니다.

“LED 켜기 / 끄기” 버튼으로 냉장고 LED를 제어합니다.

“알람 끄기” 버튼으로 부저를 끌 수 있습니다.

---

## 🎥 시연 영상 (YouTube Demo)
아래 링크에 실제 시연 영상 업로드 후 주소만 바꿔 넣으면 됩니다.

https://youtu.be/4QHldONTP5U?si=ZSPAZ87JWnnMseJX

---

## 📌 기타
과제: 임베디드 소프트웨어 개인 종합과제
주제: 스마트 냉장고 관리 시스템 (Smart Refrigerator Inventory & Alert System)
사용 기술: Arduino, Processing, App Inventor, 시리얼 통신, HTTP 통신, 센서 제어


