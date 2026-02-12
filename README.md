# ♻️ Environment Guardian: AIoT Resource Circulation Robot
> **"Detection to Action: AI의 판단을 물리적 제어로 연결하다"**
> Raspberry Pi와 YOLOv5 객체 인식을 활용한 **캠퍼스 자원 순환 자동화 로봇** (Convergence Capstone Design)

[![Hardware](https://img.shields.io/badge/Hardware-Raspberry_Pi_4-C51A4A?style=for-the-badge&logo=raspberrypi&logoColor=white)]()
[![AI](https://img.shields.io/badge/AI-YOLOv5_Object_Detection-00FFFF?style=for-the-badge&logo=pytorch&logoColor=white)]()
[![Control](https://img.shields.io/badge/Control-Node.js_Communication-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)]()
[![Mobile](https://img.shields.io/badge/App-Flutter-02569B?style=for-the-badge&logo=flutter&logoColor=white)]()

<br>

## 📢 Project Notice
**본 프로젝트는 하드웨어 반납 및 개발 환경 종료로 인해 소스 코드가 포함되어 있지 않습니다.**
대신, 시스템의 **시연 영상, 최종 결과 보고서**를 통해 엔지니어링 역량을 확인하실 수 있습니다.

* **📄 [👉 최종 결과 보고서 (PDF) 보기](docs/Final_Report.pdf)**
* **🎥 [👉 시연 영상 (YouTube) 보기](https://youtu.be/MpQHj2ONUuY)**

<br>

---

## 📖 1. Project Overview
강의동이나 도서관에 시험기간처럼 학생들이 몰릴 때, 분리수거장이 꽉 차거나, 일반쓰레기에 재활용 쓰레기를 버리는 등의 문제를 해결하고자 시작한 프로젝트입니다.
딥러닝(YOLOv5)을 통해 **투명 플라스틱, 캔, 종이컵**을 정확히 식별하고, **분류-압축-분쇄**까지 원스톱으로 수행하는 **AIoT 자원 순환 시스템**입니다. 또한 모바일 앱과 연동하여 사용자에게 **포인트 리워드**를 제공함으로써 자발적인 분리수거를 유도합니다. 

* **프로젝트 유형:** 기계/전자/컴퓨터공학부 융합 캡스톤 디자인 
* **주요 역할:**
    * **AI Model Engineering:** YOLOv5 데이터 수집(1,500장) 및 학습, 투명 물체 인식률 개선 
    * **System Control:** 라즈베리파이 GPIO를 활용한 분류기/압축기/분쇄기 모터 제어 
    * **Communication:** Node.js 기반의 라즈베리파이 간(User Display ↔ Control System) 실시간 통신 구현 

<br>

---

## 🏗️ 2. System Architecture & Workflow

### 2-1. 전체 시스템 구조
본 시스템은 **사용자 인터페이스**(App/Kiosk)와 **제어 시스템**(Robot)이 분리되어 있으며, Node.js 서버를 통해 유기적으로 연결됩니다.

1.  **Input (Vision AI):** 웹캠이 투입된 쓰레기를 촬영하고 YOLOv5 모델이 객체를 탐지합니다.
2.  **Process (Raspberry Pi B):** 탐지된 클래스(Can, Plastic, Paper)에 따라 모터 드라이버(L298N, Relay)에 제어 신호를 보냅니다.
3.  **Communication (Node.js):** 분류 결과 데이터를 라즈베리파이 A(키오스크)로 전송하여 사용자에게 포인트를 적립합니다. 

<p align="left">
  <img src="assets/system_architecture.png" alt="System Architecture" width="600px">
</p>

<br>

---

## 👨‍💻 3. Key Engineering Challenges & Solutions

### 🚀 Challenge 1: 투명 플라스틱 인식률 개선 (Advanced Data Augmentation)
* **문제 상황 (Problem):**
    * 형태가 뚜렷한 '캔'과 달리, '투명 아이스컵'이나 '생수병'은 배경색이 투과되거나 조명 반사(Specularity)가 심해, 초기 모델에서는 인식률이 40% 미만이었습니다.
    * 특히 조도 변화에 따라 객체가 아예 사라지거나 배경으로 오인되는 현상이 발생했습니다.

* **해결 과정 (Technical Approach):**
    * **Roboflow**와 **YOLOv5 Hyperparameter** 튜닝을 통해 투명 물체의 광학적 특성을 고려한 3가지 핵심 증강 기법을 적용했습니다.
    1.  **Mosaic Augmentation:** 4장의 이미지를 무작위로 합쳐 학습시킴으로써, 모델이 특정 배경에 의존하지 않고 객체의 고유한 엣지(Edge) 특징만 학습하도록 유도했습니다.
    2.  **HSV (Hue, Saturation, Value) Shift:** 투명 플라스틱의 반사광이 조명에 따라 달라지는 점을 고려하여, 채도(Saturation)와 명도(Value) 값을 임의로 변조해 다양한 조명 환경을 시뮬레이션했습니다.
    3.  **Gaussian Noise & Blur:** 쓰레기가 투입될 때의 흔들림(Motion Blur)과 카메라 노이즈에 대응하기 위해, 이미지에 가우시안 노이즈를 주입하여 모델의 강건성(Robustness)을 확보했습니다.

* **결과 (Result):**
    * 위 기법들을 적용하여 데이터셋을 1,500장까지 증강한 결과, mAP@0.5 기준 92%의 인식률을 달성했습니다.
    * 어두운 실내나 강한 조명 아래에서도 투명 컵의 윤곽선을 정확히 탐지하는 모델을 완성했습니다.

### ⚡ Challenge 2: 이기종 OS 간 통신 문제 (Node.js)
* **문제 상황:**
    * Raspberry Pi A (Display/App)와 Raspberry Pi B (Control/AI)가 서로 다른 역할을 수행하므로 데이터 통신이 필수적이었습니다.
    * 초기에는 블루투스 통신을 시도했으나, OS 호환성 문제로 연결이 불안정한 현상이 발생했습니다. 
* **해결 과정:**
    * **Node.js**를 활용하여 로컬 네트워크 상에서 **TCP/IP 소켓 통신 서버**를 구축했습니다.
    * Pi A가 "시작 명령"을 보내면 Pi B가 객체 인식을 수행하고, 결과를 다시 Pi A로 반환하는 안정적인 양방향 통신을 구현했습니다. 

<br>

---

## 🛠️ 4. Hardware Control Logic
기계/전자공학부 팀원들과 협업하여 **3단계 처리 프로세스**를 정밀하게 제어했습니다. 

| 구분 | 제어 방식 및 부품 | 동작 원리 |
| :--- | :--- | :--- |
| **분류기 (Sorter)** | **L298N + 소형 모터 2개** | 인식된 종류에 따라 모터의 회전 방향과 속도를 PWM으로 제어하여 3방향(파쇄/캔/종이)으로 분류  |
| **압축기 (Compressor)** | **4채널 릴레이 + DC 모터(24V)** | 고출력 DC 모터를 릴레이로 제어하여 캔과 종이컵을 납작하게 압축 (120:1 감속비 사용)  |
| **파쇄기 (Shredder)** | **전자개폐기(MC) + 단상 모터(220V)** | 재사용 가능한 플라스틱 컵을 잘게 파쇄하여 부피를 줄임. 안전을 위해 MC와 릴레이를 조합하여 제어  |

<br>

---

## 💻 5. Tech Stack

* **Hardware:** Raspberry Pi 4 (x2), Pi Camera, SMPS (24V/12V/5V), Relay Module, L298N Driver 
* **AI & Vision:** Python, PyTorch, YOLOv5, OpenCV, Roboflow 
* **Software:** Raspberry Pi OS, Node.js (Communication), Flutter (App), Spring Boot & MySQL (Server) 

<br>
