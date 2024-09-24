# wsl2_linux_kernel_usbcam_enable_conf
Configuration file to build the kernel to access the USB camera connected to the host PC using USBIP from inside the WSL2.
[Windows 11 + WSL2 + USB Camera + Serial](https://zenn.dev/pinto0309/articles/e1432253d29e30)

## 1. Environment
- Windows 11 (OS build: 22000.194)
- Windows 11 (OS build: 22000.469)
- Windows 11 (OS build: 22000.918)
- Windows 10 (OS build: 19043.1237+)
- Windows 10 (OS build: 19045.2846)
- WSL2
- **`[Mandatory] WSL2 Ubuntu 20.04 or 22.04`**
- https://github.com/microsoft/WSL2-Linux-Kernel

## 2. Usage
![yth06](https://user-images.githubusercontent.com/33194443/156264003-009b7508-c61e-4cda-8c6f-bcd3840b991f.png)  
On WSL2.
```bash
$ uname -r -v
5.15.90.1-microsoft-standard-WSL2 #1 SMP Fri Jan 27 02:56:13 UTC 2023
```
On Windows Terminal.
```shell
C:\> wsl --update
Checking for updates...
Downloading update...
Installing update...
This change will take effect the next time you reboot WSL. To force a reboot, run 'wsl --shutdown'.
Kernel version: 5.10.102.1

C:\> wsl --shutdown
```

```shell
# Ubuntu 20.04
C:\> wsl --install -d Ubuntu-20.04

# Ubuntu 22.04
C:\> wsl --install -d Ubuntu-22.04
```


On WSL2. The **`<windows username>`** part should be replaced with the user name you are logging on to the Windows host PC. There are two corrections.
```bash
$ uname -r -v
5.15.90.1-microsoft-standard-WSL2 #1 SMP Fri Jan 27 02:56:13 UTC 2023

$ sudo apt update && sudo apt upgrade -y
$ sudo apt install -y \
    build-essential flex bison \
    libgtk-3-dev libelf-dev libncurses-dev autoconf \
    libudev-dev libtool zip unzip v4l-utils libssl-dev \
    python3-pip cmake git iputils-ping net-tools dwarves \
    guvcview python-is-python3 bc

$ cd /usr/src
$ TAGVERNUM=$(uname -r | awk -F'-' '{print $1}') \
  && TAGVER=linux-msft-wsl-${TAGVERNUM} \
  && WINUSERNAME=<windows username>
$ sudo git clone --depth 1 -b ${TAGVER} \
    https://github.com/microsoft/WSL2-Linux-Kernel.git \
    ${TAGVERNUM}-microsoft-standard \
  && cd ${TAGVERNUM}-microsoft-standard

$ sudo wget -O .config https://raw.githubusercontent.com/mateicojo/wsl2_linux_kernel_usbcam_enable_conf/refs/heads/main/config \
  && sudo chmod 777 .config \
  && sudo make clean

$ sudo make -j$(nproc) KCONFIG_CONFIG=.config \
  && sudo make modules_install -j$(nproc) \
  && sudo make install -j$(nproc)

$ sudo rm /mnt/c/Users/${WINUSERNAME}/vmlinux
$ sudo cp /usr/src/${TAGVERNUM}-microsoft-standard/vmlinux /mnt/c/Users/${WINUSERNAME}/
```
![w3vld](https://user-images.githubusercontent.com/33194443/155730584-5bbf6a8a-3bf5-44dc-b009-8f8b703d4f5a.png)
![49p0d](https://user-images.githubusercontent.com/33194443/155734242-a03fceaa-687a-42ec-b130-d6cd82dbf935.png)
```
$ cat << 'EOT' > /mnt/c/Users/${WINUSERNAME}/.wslconfig
[wsl2]
kernel=C:\\Users\\<windows username>\\vmlinux
EOT
```
![fyh68](https://user-images.githubusercontent.com/33194443/155730997-cca7d47b-df10-45d3-a70e-b2267a480504.png)  
On Windows Terminal.
```shell
C:\> wsl --shutdown
```
On WSL2. If the built kernel has been loaded successfully, you will see **`+`** at the end of the kernel name. **`#n`** is the number of times the kernel was built.
```bash
$ uname -r -v
5.15.90.1-microsoft-standard-WSL2+ #1 SMP Sun Apr 16 18:14:34 JST 2023
```
## 3. Note
### 3-1. New WSL Linux kernel configuration changes
From here, select the `TAG` for the target kernel version.

https://github.com/microsoft/WSL2-Linux-Kernel/tags

On WSL2.
```bash
$ cd /usr/src
$ TAGVERNUM=x.x.x.x
$ TAGVER=linux-msft-wsl-${TAGVERNUM}
$ sudo git clone \
    -b ${TAGVER} \
    https://github.com/microsoft/WSL2-Linux-Kernel.git \
    ${TAGVERNUM}-microsoft-standard \
  && cd ${TAGVERNUM}-microsoft-standard
$ sudo cp /proc/config.gz config.gz \
  && sudo gunzip config.gz \
  && sudo mv config .config
```
**Note: If the line 
```bash
$ sudo git clone \
    -b ${TAGVER} \
    https://github.com/microsoft/WSL2-Linux-Kernel.git \
    ${TAGVERNUM}-microsoft-standard \
  && cd ${TAGVERNUM}-microsoft-standard
```
does not work for you, try manually pasting the path to the directory corresponding to your distribution from this repo:
https://github.com/microsoft/WSL2-Linux-Kernel.git

```bash

$ sudo make menuconfig

# After completing all necessary configurations,
# "save" will save the modified `.config` file.
```
### 3-2. Troubleshooting
If you receive the following error, please follow the additional steps: USB bandwidth issues may cause the information exchange with the camera to time out.
- Error Message
```
[ WARN:0] global /tmp/pip-req-build-xw6jtoah/opencv/modules/videoio/src/cap_v4l.cpp (1001) tryIoctl VIDEOIO(V4L2:/dev/video0): select() timeout.
```

Try one or both of the following.

- Try1. Additional command
```bash
$ sudo apt install v4l-utils \
  && sudo chmod 777 /dev/video0 \
  && v4l2-ctl -d /dev/video0 --all

Driver Info:
	Driver name      : uvcvideo
	Card type        : papalook FHD Camera: papalook F
	Bus info         : usb-0000:00:14.0-11.1
	Driver version   : 5.11.22
	Capabilities     : 0x84a00001
		Video Capture
		Metadata Capture
		Streaming
		Extended Pix Format
		Device Capabilities
	Device Caps      : 0x04200001
		Video Capture
		Streaming
		Extended Pix Format
Media Driver Info:
	Driver name      : uvcvideo
	Model            : papalook FHD Camera: papalook F
	Serial           : 
	Bus info         : usb-0000:00:14.0-11.1
	Media version    : 5.11.22
	Hardware revision: 0x00000100 (256)
	Driver version   : 5.11.22
Interface Info:
	ID               : 0x03000002
	Type             : V4L Video
Entity Info:
	ID               : 0x00000001 (1)
	Name             : papalook FHD Camera: papalook F
	Function         : V4L2 I/O
	Flags         : default
	Pad 0x01000007   : 0: Sink
	  Link 0x02000010: from remote pad 0x100000a of entity 'Extension 3': Data, Enabled, Immutable
Priority: 2
Video input : 0 (Camera 1: ok)
Format Video Capture:
	Width/Height      : 1280/720
	Pixel Format      : 'MJPG' (Motion-JPEG)
	Field             : None
	Bytes per Line    : 0
	Size Image        : 1843200
	Colorspace        : sRGB
	Transfer Function : Rec. 709
	YCbCr/HSV Encoding: ITU-R 601
	Quantization      : Default (maps to Full Range)
	Flags             : 
Crop Capability Video Capture:
	Bounds      : Left 0, Top 0, Width 1280, Height 720
	Default     : Left 0, Top 0, Width 1280, Height 720
	Pixel Aspect: 1/1
Selection Video Capture: crop_default, Left 0, Top 0, Width 1280, Height 720, Flags: 
Selection Video Capture: crop_bounds, Left 0, Top 0, Width 1280, Height 720, Flags: 


$ sudo apt install v4l-utils \
  && sudo chmod 777 /dev/video2 \
  && v4l2-ctl -d /dev/video2 --list-formats-ext

ioctl: VIDIOC_ENUM_FMT
        Type: Video Capture

        [0]: 'GREY' (8-bit Greyscale)
                Size: Discrete 256x144
                        Interval: Discrete 0.003s (300.000 fps)
                        Interval: Discrete 0.011s (90.000 fps)
                Size: Discrete 424x240
                        Interval: Discrete 0.011s (90.000 fps)
                        Interval: Discrete 0.017s (60.000 fps)
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.167s (6.000 fps)
    :
                Size: Discrete 1280x720
                        Interval: Discrete 0.033s (30.000 fps)
                        Interval: Discrete 0.067s (15.000 fps)
                        Interval: Discrete 0.167s (6.000 fps)
        [2]: 'GREY' (8-bit Greyscale)
                Size: Discrete 256x144
                        Interval: Discrete 0.003s (300.000 fps)
                        Interval: Discrete 0.011s (90.000 fps)
                Size: Discrete 424x240
                        Interval: Discrete 0.011s (90.000 fps)
```
Check the WIDTH and HEIGHT of the standard resolution displayed, and correct the input resolution listed in **`usbcam_test.py`**.

e.g.

From:
```python
frame = cv2.resize(frame, (640, 480))
```
To:
```python
frame = cv2.resize(frame, (1280, 720))
```
Run it again.
```bash
$ sudo chmod 777 /dev/video* && python ${HOME}/usbcam_test.py
```

- Try2. It may not work well with UVC-compatible USB cameras depending on their compatibility. Try replacing several USB cameras. The following cameras were recognized successfully. I have tried four different USB cameras and only one was successful.

  - [ELP 8 Megapixel USB Camera Module High Definition 2448P Webcam Wide Angle 180 Degree Fisheye Camera Module Full HD High Speed 2448P 15FPS Camera HD](https://www.amazon.co.jp/ELP-USB%E3%82%AB%E3%83%A1%E3%83%A9%E3%83%A2%E3%82%B8%E3%83%A5%E3%83%BC%E3%83%AB-%E3%83%95%E3%83%AA%E3%83%BC%E3%83%89%E3%83%A9%E3%82%A4%E3%83%90%E3%83%BC%E3%82%A6%E3%82%A7%E3%83%96-VR%E3%82%AB%E3%83%A1%E3%83%A9%EF%BC%88%E3%83%A2%E3%83%87%E3%83%AB%EF%BC%9AELP-USB8MP02G-L180-JP%EF%BC%89/dp/B08FDJW3HS?language=en_US&th=1)
  - [OAK-D-Lite](https://www.kickstarter.com/projects/opencv/opencv-ai-kit-oak-depth-camera-4k-cv-edge-object-detection?lang=en)
    - https://github.com/luxonis/depthai
      ```bash
      # Disable the Windows Defender Firewall
      $ python3 install_requirements.py
      $ python3 -m pip install blobconverter --upgrade
      $ sudo echo 'SUBSYSTEM=="usb", ATTRS{idVendor}=="03e7", MODE="0666"' | \
      sudo tee /etc/udev/rules.d/80-movidius.rules
      $ service udev restart
      $ sudo udevadm control --reload-rules && sudo udevadm trigger
      $ python3 depthai_demo.py
      ```
  - Built-in camera for Thinkpad laptops
  - RealSense D435
    ![image](https://user-images.githubusercontent.com/33194443/152624363-ee4e7b47-fa3d-415f-b860-eecd5bc30275.png)
  - RealSense L515
	![image](https://user-images.githubusercontent.com/17954673/232305361-adcd7545-7ac6-490b-ba9c-d7a414804c5c.png)
  - Elecom WEBCAM-102BK Webcam Conference Camera
    https://www.amazon.co.jp/gp/product/B08BHWR8ZR/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1&language=ja_JP
- Confirmed to work with Windows 11 Home (21H2) OS build: 22000.194
- Confirmed to work with Windows 11 Pro (21H2) OS build: 22000.469
- Confirmed to work with Windows 10 Pro (21H1) OS build: 19043.1237

