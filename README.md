# jetson_tutorial

1. [jetson-stats](#1-jetson-stats)
2. [lightdm 설치](#2-lightdm-설치)
3. [VNC 설치](#3-vnc-설치)
4. [Opencv + CUDA 설치](#4-opencv-설치)
5. [torch 설치](#5-pytorch-설치)
6. [wifi 포트포워딩 설정](#6-포트포워딩-설정)

---

## 1. jetson-stats

##### PU, GPU, RAM, Swap, Fan 등의 H/W 정보와 CUDA,TensorRT, JetPack 버전 확인 및 Fan Speed, Swap 공간 설정

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install python-pip
sudo -H pip install -U jetson-stats
sudo reboot
 
# jetson-stats 실행
jtop
```

## 2. lightdm 설치

##### gdm3 경우 RAM 사용량이 급격하게 치솟아, YOLOv5 등 실행을 위해서 gdm3 제거 및 lightdm 설치 (LXDE)

```
sudo apt-get install lightdm
sudo apt-get purge gdm3
```

## 3. VNC 설치

ubuntu 20.04 기본 내장된 Vino VNC를 사용하여 설정 

[NVIDIA VNC 설정](https://developer.nvidia.com/embedded/learn/tutorials/vnc-setup)


1. Enable the VNC server to start each time you log in

LXDE (jetson nano 2GB 는 기본 LXDE)
    
```bash
mkdir -p ~/.config/autostart
cp /usr/share/applications/vino-server.desktop ~/.config/autostart/.
```

GNOME (jetson nano 4GB는 기본)
    
```bash
cd /usr/lib/systemd/user/graphical-session.target.wants
sudo ln -s ../vino-server.service ./.
```

2. Configure the VNC server

```bash
gsettings set org.gnome.Vino prompt-enabled false
gsettings set org.gnome.Vino require-encryption false
```

3. Set a password to access the VNC server

    thepassword 부분에 원하는 password 설정

```bash
# Replace thepassword with your desired password
gsettings set org.gnome.Vino authentication-methods "['vnc']"
gsettings set org.gnome.Vino vnc-password $(echo -n 'thepassword'|base64)
```

4. Reboot the system so that the settings take effect
```bash
sudo reboot
```

## 4. OpenCV 설치

[Q-Engineering 블로그 방법](https://qengineering.eu/install-opencv-4.5-on-jetson-nano.html)



```bash
## 복붙의 편의성을 위해 $를 생략합니다.
 
# 업데이트를 확인합니다. 이전 포스팅에서 이미 이걸 했다면, 굳이 하지 않아도 좋습니다.
sudo apt-get update
sudo apt-get upgrade
 
# nano 에디터를 설치합니다. 이미 설치했다면, 굳이 하지 않아도 좋습니다.
sudo apt-get install nano
 
# dphys-swapfile을 설치합니다.
sudo apt-get install dphys-swapfile
 
## 두 Swap파일의 값이 다음과 같도록 값을 추가하거나, 파일 내 주석을 해제합니다.
# CONF_SWAPSIZE=4096
# CONF_SWAPFACTOR=2
# CONF_MAXSWAP=4096
 
# /sbin/dphys-swapfile를 엽니다.
sudo nano /sbin/dphys-swapfile
 
# 값을 수정한 후 [Ctrl] + [X], [y], [Enter]를 눌러 저장하고 닫습니다
 
 
# /etc/dphys-swapfile를 편집합니다.
sudo nano /etc/dphys-swapfile
 
# 값을 수정한 후 [Ctrl] + [X], [y], [Enter]를 눌러 저장하고 닫습니다
 
# Jetson Nano 재부팅
sudo reboot
```

    시간과의 싸움

```bash
wget https://github.com/Qengineering/Install-OpenCV-Jetson-Nano/raw/main/OpenCV-4-5-5.sh
sudo chmod 755 ./OpenCV-4-5-5.sh
./OpenCV-4-5-5.sh
```

    SD카드 공간 확보와 스왑 공간 해제를 위해 다음의 명령을 수행할 수 있으나, 그냥 쓰기로 함
    
```bash
rm OpenCV-4-5-5.sh
 
# Swap 제거
sudo /etc/init.d/dphys-swapfile stop
sudo apt-get remove --purge dphys-swapfile
 
# 275MB 추가 SD공간 확보를 원한다면 
sudo rm -rf ~/opencv
sudo rm -rf ~/opencv_contrib
```


## 5. Pytorch 설치

- [동영상 참조](https://www.youtube.com/watch?v=8nHZKTkYACc)

- [Pytorch ARM wheels](https://forums.developer.nvidia.com/t/pytorch-for-jetson-version-1-10-now-available/72048)

---

> pip3 install numpy==1.19.4 # 1.19.5 설치시 torch 오류 발생

> pip3 install --no-deps matplotlib==3.2.2

> Python 3.6
```bash
wget https://nvidia.box.com/shared/static/h1z9sw4bb1ybi0rm3tu8qdj8hs05ljbm.whl -O torch-1.9.0-cp36-cp36m-linux_aarch64.whl
sudo apt-get install python3-pip libopenblas-base libopenmpi-dev libomp-dev
pip3 install Cython
pip3 install numpy torch-1.9.0-cp36-cp36m-linux_aarch64.whl
```
> torchvision
```bash
$ sudo apt-get install libjpeg-dev zlib1g-dev libpython3-dev libavcodec-dev libavformat-dev libswscale-dev
$ git clone --branch v0.10.0 https://github.com/pytorch/vision torchvision   # see below for version of torchvision to download
$ cd torchvision
$ export BUILD_VERSION=0.10.0  # where 0.x.0 is the torchvision version  
$ python3 setup.py install --user
$ cd ../  # attempting to load torchvision from build dir will result in import error
```


[참고1](https://velog.io/@jjun8177/Jetson-Nano%EC%99%80-YOLO-v5%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-detection-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8)

[참고2](https://whiteknight3672.tistory.com/316)



## 6. 포트포워딩 설정

> 무선 공유기 설정을 이용하여 포트포워딩 후 재시작 필요
- 무선 공유기가 기본 게이트 웨이로 하는 IP가 중요함, 랜선으로 연결된 PC 에서 wifi 접속을 위해서는 해당 IP로 접근 후 무선공유릐 포트포워딩을 통해 접속되는 구조

> 안드로이드로 RTSP 설정 (astra streaming, rtsp://id:passwd@IP:port/name 형식으로 스트리밍 됨)
> 
- 아래 코드를 참조

```python
import cv2
import datetime
import os


def write_video():
    #현재시간 가져오기
    current_time = datetime.datetime.now()

    #RTSP를 불러오는 곳
    video_capture = cv2.VideoCapture('rtsp://192.168.35.190:5554/tt')

    # 웹캠 설정
    video_capture.set(3, 800)  # 영상 가로길이 설정
    video_capture.set(4, 600)  # 영상 세로길이 설정
    fps = 20
    # 가로 길이 가져오기
    streaming_window_width = int(video_capture.get(3))
    # 세로 길이 가져오기
    streaming_window_height = int(video_capture.get(4))

    #현재 시간을 '년도 달 일 시간 분 초'로 가져와서 문자열로 생성
    file_name = current_time.strftime('%Y%m%d_%H%M%S')

    #파일 저장하기 위한 변수 선언
    path = f'./{file_name}.avi'

    # DIVX 코덱 적용 # 코덱 종류 # DIVX, XVID, MJPG, X264, WMV1, WMV2
    # 무료 라이선스의 이점이 있는 XVID를 사용
    fourcc = cv2.VideoWriter_fourcc('X', 'V', 'I', 'D')

    # 비디오 저장
    # cv2.VideoWriter(저장 위치, 코덱, 프레임, (가로, 세로))
    out = cv2.VideoWriter(path, fourcc, fps, (streaming_window_width, streaming_window_height))

    while True:
        ret, frame = video_capture.read()

        # 영상을 저장한다.
        out.write(frame)

        # 1ms뒤에 뒤에 코드 실행해준다.
        k = cv2.waitKey(1) & 0xff
        #키보드 q 누르면 종료된다.
        if k == 113:
            break
    video_capture.release()  # cap 객체 해제
    out.release()  # out 객체 해제
    cv2.destroyAllWindows()

if __name__ == "__main__":
    write_video()
```

> 저장된 avi 영상을 다시 불러 변환 (안경 트래킹)
```python
import cv2
import numpy as np

cap = cv2.VideoCapture('/home/roadcom/20220417_001514.avi')
fps = 20

#fourcc = cv2.VideoWriter_fourcc('X', 'V', 'I', 'D')
#out = cv2.VideoWriter('gray_output.avi', fourcc, 20, (1280,720))

if cap.isOpened():  
    
    fourcc = cv2.VideoWriter_fourcc(*'DIVX') # 인코딩 포맷 문자
    width = cap.get(cv2.CAP_PROP_FRAME_WIDTH)
    height = cap.get(cv2.CAP_PROP_FRAME_HEIGHT)
    size = (int(width), int(height))   
    out = cv2.VideoWriter('gray_output.avi', fourcc, fps, size) 
    
    while True:
        ret, frame = cap.read()
        
        if ret:                      # 프레임 읽기 정상
            gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
            cv2.imshow('frame',gray) # 화면에 표시  --- ③
            
            ret, dst = cv2.threshold(gray,0,255,cv2.THRESH_BINARY_INV|cv2.THRESH_OTSU)
            conts, _ = cv2.findContours(dst, cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_NONE)
            
            valid_cons = []
            
            for con in conts:
                ret = cv2.contourArea(con)
                if 800 < ret < 3500:
                    (x,y), radius = cv2.minEnclosingCircle(con)
                    if 20 < radius < 40 and 400 < x < 850 and  150 < y < 400:
                        valid_cons.append(con)
                        #cv2.drawContours(frame, [con],0, (20,255,200), 3)    
            
            if len(valid_cons) > 0:
                convex = cv2.convexHull(np.vstack(valid_cons))
                cv2.drawContours(frame, [convex],0, (255,51,51), -1)    
            
            #out.write(cv2.cvtColor(gray, cv2.COLOR_GRAY2BGR));
            out.write(frame);
            if cv2.waitKey(int(1000/fps)) != -1: 
                break
        else:                        # 다음 프레임 읽을 수 없슴,
            break                    # 재생 완료        
    out.release()  
    
cap.release()
#out.release()
cv2.destroyAllWindows()
```
