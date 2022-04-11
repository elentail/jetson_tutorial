# jetson_tutorial

1. [jetson-stats](#1-jetson-stats)
2. [lightdm 설치](#2-lightdm-설치)
3. [VNC 설치](#3-vnc-설치)
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
