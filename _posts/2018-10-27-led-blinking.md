---
title: DIY, code nháy LED với Python trên Raspberry Pi 2
tags: [news]
last_updated: October 27, 2018
sidebar: mydoc_sidebar
permalink: led-blinking.html
folder: mydoc
---

Nhân một buổi chiều rảnh rỗi, mình ngồi viết bài này hướng dẫn các bạn code một chương trình nho nhỏ bằng Python để điều khiển đèn LED nháy trên [Rasberry Pi 2](https://www.raspberrypi.org/products/raspberry-pi-2-model-b/).  

Chiếc Rasberry Pi của mình là Rasberry Pi 2, mình tậu em nó từ hồi năm cuối Đại học (2015), hàng UK nên dùng rất bền, mua về nhà cũng vọc vạch được khá nhiều trò hay ho với cái board này: Cài [OSMC](https://osmc.tv/download/) lên rồi cắm vô chiếc "stupid" TV nhà mình để biến nó thành smart hơn và cũng hỗ trợ đầy đủ app Youtube, multimedia server các kiểu...

Pi cũng rất thích hợp để nhóc em trai mình học lập trình, với cấu hình RAM: 1GB, SoC: BCM2836 kèm thẻ nhớ 8GB của nó là quá đủ để code Python, C... rồi.  

Chiếc Pi của mình trông nó như thế này:

![My workingspace](/static/img/raspberrypi/mypi.jpg)

Để thực hiện việc điều kiển nháy LED bằng code Python, thì việc đầu tiên cần làm là chuẩn bị những phần cứng và phần mềm cấn thiết.

## Phần cứng
* Board mạch Raspberry Pi
* Board trắng để cắm đèn LED, dây nối và jump
* Điện trở 220 Ohm 

![Hardware](/static/img/raspberrypi/hardware.jpg)

## Phần mềm
Mình sẽ cài [Raspbian](https://www.raspberrypi.org/downloads/raspbian/) lên Pi, **Raspbian** là hệ điều hành nhân Linux được tuỳ biến riêng cho Rasberry Pi. [Hướng dẫn này](https://www.raspberrypi.org/documentation/installation/installing-images/README.md) sẽ nêu chi tiết các bước cài đặt. Remote đến Pi thông qua giao thức ssh để bắt tay vào lập trình, chi tiết [tại đây](https://truongnh1992.github.io/linux/networking/raspberry/2018/10/30/remote-to-raspberrypi.html).

## Lập trình
Giờ đến công đoạn cuối cùng là code và cài cắm đèn LED vào Pi.  

![pins-gpio](/static/img/raspberrypi/gpio-pins-pi2.jpg)

{:.image-caption}
*Raspberry Pi 2 có 40 chân GPIO (general-purpose input/output). Ảnh: rasberrypi.org*

Sơ đồ chân GPIO phục vụ cho việc lập trình như hình dưới đây:

![pins-gpio](/static/img/raspberrypi/gpio-numbers-pi2.jpg)

{:.image-caption}
*Ảnh: rasberrypi.org*

Mình sẽ cắm dây nối đèn LED vào board trắng thông qua jumb để kết nối với chân GPIO trên Pi theo sơ đồ sau:

![noiday](/static/img/raspberrypi/noiday.jpg)

Đèn LED được mắc nối tiếp với điện trở, cực dương của LED nối với **chân số 19** tức GPIO10 của Raspberry Pi còn cực âm được nối với **chân số 9** tức Ground nối đất của Raspberry Pi

## Mã nguồn chương trình

*blink.py*
```python
import RPi.GPIO as GPIO
import time

numTimes = int(input("Enter total number of times to blink: "))
speed = float(input("Enter length of each blink (seconds): "))

GPIO.setwarnings(False)
GPIO.setmode(GPIO.BOARD)
GPIO.setup(19, GPIO.OUT)

def Blink(numTimes, speed):
    for i in range (0, numTimes):
        GPIO.output(19, True)
        print "Iteration ", (i+1)
        time.sleep(speed)
        GPIO.output(19, False)
        time.sleep(speed)

Blink(numTimes, speed)
print("Done")
```

Chạy chương trình trên, nhập vào số lần nháy mong muốn và *"tần số nháy"*
```
$ python blink.py
Enter total number of times to blink: 50
Enter length of each blink (seconds): 0.5
```
Và đây là kết quả:

![Demo](/static/img/raspberrypi/demo_led.gif)

Hi vọng bài viết sẽ giúp các bạn DIY được một ứng dụng nho nhỏ và thú vị này.

Hải Phòng, 27-10-2018.
