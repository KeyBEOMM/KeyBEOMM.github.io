---
title: "PlatfromIO 환경 구축"
description: "Linux환경(wsl2)에서의 PlatformIO 환경 구축 및 antigravity 통합"
author: KEYBEOMM
date: 2026-03-20 20:44:00 +0900
categories: [Embedded Systems, PlatfromIO]
tags: [PlatformIO, esp32]
---

# WSL2에서 PlatformIO로 ESP32 개발 환경 구축 및 업로드 

WSL2는 리눅스의 강력한 툴체인을 사용할 수 있게 해주지만, 하드웨어(USB 포트) 접근에는 추가 설정이 필요하다. 본 가이드는 PlatformIO를 활용한 패키지 빌드 및 업로드 과정과 WSL2에서의 **명령어 경로 설정**부터 **USB 패스스루**, **업로드 권한 해결**까지의 전반적인 과정을 단계별로 정리한다.

먼저 WSL2 환경에서 PlatformIO를 설치해야 한다. PlatformIO는 본 저자는 Antigravity를 사용한다. vscode의 경우 내부 extention 인터페이스를 통해 쉽게 설치할 수 있지만, antigravity의 경우는 PlatformIO가 검색으로 나오지 않는다.


## 1. Antigravity에서 PlatformIO 설치
1. OPen VSX registry에서 PlatfromIO의 VSIX 파일을 다운받는다. 
2. wsl2 환경 내의 원하는 경로에 VSIX 파일을 복사한다.
3. antigravity의 extentions 메뉴에서 우측 점 세게를 누르면 "install from VSIX"를 확인할 수 있다
4. 우분투 내 환경에서 VSIX 파일을 선택하여 설치하면 완료.


## 2. PlatformIO CLI 명령어 전역 설정 (PATH)
설치 직후 터미널에서 `pio` 명령어가 작동하지 않는다면, PlatformIO의 실행 파일 경로를 시스템에 등록해야 한다.

터미널은 pio명령어의 경로를 모르기에 소싱해줘야 한다. ROS2 사용 시 source /opt/ros/humble/setup.bash를 하는 것과 비슷하다. 

매번 적용하기 싫으니 환경변수로 설정해준다. 

```bash
# .bashrc 파일 끝에 PlatformIO 실행 경로 추가
echo 'export PATH="$PATH:$HOME/.platformio/penv/bin"' >> ~/.bashrc

# 변경사항 적용
source ~/.bashrc
```
이제 어느 폴더에서든 `pio` 명령어를 사용할 수 있다.

---

작성한 코드들을 빌드하고 업로드하려는데 99-platformio-udev.rules 에러가 발생한다.


작성한 코드들을 빌드하고 esp32에 업로드하기 위해서 선행해주어야하는 과정이 있다.
첫번째, 리눅스를 사용하는 사용자는 USB 접근 권한 설정을 해주어야한다.

자세한 내용 PlatformIO Documentation 확인
: https://docs.platformio.org/en/latest/core/installation/udev-rules.html

## 3. 리눅스 USB 접근 권한 설정 (udev rules)
리눅스 시스템이 USB 장치를 인식하고 읽기/쓰기 권한을 부여하도록 규칙파일을 설치한다.

```bash
# 1. PlatformIO 공식 udev 규칙 다운로드 및 설치
curl -fsSL https://raw.githubusercontent.com/platformio/platformio-core/master/scripts/99-platformio-udev.rules | sudo tee /etc/udev/rules.d/99-platformio-udev.rules

# 2. 서비스 재시작
sudo udevadm control --reload-rules
sudo udevadm trigger

# 3. 현재 유저를 시리얼 통신 그룹(dialout)에 추가
sudo usermod -a -G dialout $USER
```
*주의: 그룹 추가 후에는 WSL 세션을 완전히 껐다 켜거나 재부팅해야 권한이 유효해진다*

---

두번째 본인은 평소 사용하는 노트북으로도 개발을 자주 한다. 따라서 편의를 위해 WSL2를 사용 중이다. 즉, 윈도우에서 리눅스를 사용 중이기에 윈도우에 연결된 USB를 리눅스에서 인식할 수 있도록 설정해야한다.

## 3. WSL2 USB Passthrough 연결 (Windows 설정)
WSL2는 독립된 가상 머신이므로 윈도우에 꽂힌 USB를 직접 볼 수 없다. `usbipd-win`을 사용하여 하위 제어기로 USB를 "넘겨주는" 작업이 필요하다.

자세한 내용 마소 docu나 github docu 활용
: https://learn.microsoft.com/ko-kr/windows/wsl/connect-usb
: https://github.com/dorssel/usbipd-win

### Windows 관리자 터미널(PowerShell) 작업
1. **usbipd-win 설치:** [GitHub 공식 릴리즈](https://github.com/dorssel/usbipd-win/releases)에서 `.msi` 설치.

2. **장치 목록 확인:**
   ```powershell
   usbipd list
   ```
3. **공유 대상 지정 (최초 1회):** ESP32 장치의 BUSID(예: 2-1)를 찾아 바인딩
   ```powershell
   usbipd bind --busid <BUSID>
   ```
4. **WSL로 연결:**
   ```powershell
   usbipd attach --wsl --busid <BUSID>
   ```

이제 WSL 터미널에서 `ls /dev/ttyUSB*`를 입력하여 포트가 나타나는지 확인한다.

---

## 4. PlatformIO 주요 명령어
platformIO에서 자주 사용할 명령어는 다음과 같다.

| 동작 | 명령어 | 비고 |
| :--- | :--- | :--- |
| **코드 빌드** | `pio run` | 컴파일만 수행 |
| **코드 업로드** | `pio run -t upload` | 빌드 후 ESP32 플래싱 |
| **시리얼 모니터** | `pio device monitor` | 로그 확인 |
| **ALL-IN-ONE** | `pio run -t upload -t monitor` | **추천:** 업로드 후 즉시 로그 확인 |

---

## 5. 트러블슈팅: 포트 수동 지정
자동 인식이 되지 않을 경우 `platformio.ini` 파일에 포트를 명시하는 것이 가장 확실하다.

```ini
; platformio.ini 예시
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = espidf
monitor_speed = 115200

; 포트가 /dev/ttyUSB0 로 잡힌 경우
upload_port = /dev/ttyUSB0
monitor_port = /dev/ttyUSB0
```
