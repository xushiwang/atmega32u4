# MacOS开发ATMega32u4

## 一、环境准备
## 1. dfu-programmer 
### 1.1 dfu-programmer是什么？
> [dfu-programmer is a multi-platform command-line programmer for Atmel (8051, AVR, XMEGA & AVR32) chips with a USB bootloader supporting ISP. Most Atmel devices having a USB port come pre-programmed with the bootloader, and this is a lightweight alternative to Atmel's own FLIP/BatchISP program. This is a mostly Device Firmware Update (DFU) 1.0 compliant user-space application.](http://dfu-programmer.github.io/)

### 1.2 如何安装dfu-programmer?
mac命令行执行如下命令来安装dfu-programmer
```
brew install dfu-programmer
```
- 一些基本命令
```
擦除开发板上的程序
sudo dfu-programmer atmega32u4 erase --force
向开发板上写入程序
sudo dfu-programmer atmega32u4 flash KeyboardLogout.ino.leonardo.hex
```

## 2. Arduino
### 2.1 Arduino是什么
> Arduino是一款便捷灵活、方便上手的开源电子原型平台。包含硬件（各种型号的Arduino板）和软件（ArduinoIDE）。由一个欧洲开发团队于2005年冬季开发。其成员包括Massimo Banzi、David Cuartielles、Tom Igoe、Gianluca Martino、David Mellis和Nicholas Zambetti等。
它构建于开放原始码simple I/O介面版，并且具有使用类似Java、C语言的Processing/Wiring开发环境。主要包含两个的部分：硬件部分是可以用来做电路连接的Arduino电路板；另外一个则是Arduino IDE，你的计算机中的程序开发环境。你只要在IDE中编写程序代码，将程序上传到Arduino电路板后，程序便会告诉Arduino电路板要做些什么了。
Arduino能通过各种各样的传感器来感知环境，通过控制灯光、马达和其他的装置来反馈、影响环境。板子上的微控制器可以通过Arduino的编程语言来编写程序，编译成二进制文件，烧录进微控制器。对Arduino的编程是通过 Arduino编程语言 (基于 Wiring)和Arduino开发环境(基于 Processing)来实现的。基于Arduino的项目，可以只包含Arduino，也可以包含Arduino和其他一些在PC上运行的软件，它们之间进行通信 (比如 Flash, Processing, MaxMSP)来实现。 [引用百度百科]()
### 2.2 如何安装
[下载地址](https://downloads.arduino.cc/arduino-nightly-macosx.zip)


## 二、开发流程
### 1.安装完Arduino后打开应用，编写程序。
demo示例如下
```
/*
  ButtonMouseControl

  For Leonardo and Due boards only.

  Controls the mouse from five pushbuttons on an Arduino Leonardo, Micro or Due.

  Hardware:
  - five pushbuttons attached to D2, D3, D4, D5, D6

  The mouse movement is always relative. This sketch reads four pushbuttons,
  and uses them to set the movement of the mouse.

  WARNING: When you use the Mouse.move() command, the Arduino takes over your
  mouse! Make sure you have control before you use the mouse commands.

  created 15 Mar 2012
  modified 27 Mar 2012
  by Tom Igoe

  This example code is in the public domain.

  https://www.arduino.cc/en/Tutorial/BuiltInExamples/ButtonMouseControl
*/

#include "Mouse.h"

// set pin numbers for the five buttons:
const int upButton = 2;
const int downButton = 3;
const int leftButton = 4;
const int rightButton = 5;
const int mouseButton = 6;

int range = 5;              // output range of X or Y movement; affects movement speed
int responseDelay = 10;     // response delay of the mouse, in ms


void setup() {
  // initialize the buttons' inputs:
  pinMode(upButton, INPUT);
  pinMode(downButton, INPUT);
  pinMode(leftButton, INPUT);
  pinMode(rightButton, INPUT);
  pinMode(mouseButton, INPUT);
  // initialize mouse control:
  Mouse.begin();
}

void loop() {
  // read the buttons:
  int upState = digitalRead(upButton);
  int downState = digitalRead(downButton);
  int rightState = digitalRead(rightButton);
  int leftState = digitalRead(leftButton);
  int clickState = digitalRead(mouseButton);

  // calculate the movement distance based on the button states:
  int  xDistance = (leftState - rightState) * range;
  int  yDistance = (upState - downState) * range;

  // if X or Y is non-zero, move:
  if ((xDistance != 0) || (yDistance != 0)) {
    Mouse.move(xDistance, yDistance, 0);
  }

  // if the mouse button is pressed:
  if (clickState == HIGH) {
    // if the mouse is not pressed, press it:
    if (!Mouse.isPressed(MOUSE_LEFT)) {
      Mouse.press(MOUSE_LEFT);
    }
  }
  // else the mouse button is not pressed:
  else {
    // if the mouse is pressed, release it:
    if (Mouse.isPressed(MOUSE_LEFT)) {
      Mouse.release(MOUSE_LEFT);
    }
  }

  // a delay so the mouse doesn't move too fast:
  delay(responseDelay);
}
```
command+R进行编译，编译通过后点击 项目>导出已编译的二进制文件

2. 插入开发板，并按一下开发板上的reset按钮
执行如下命令以擦除原有数据
```
sudo dfu-programmer atmega32u4 erase --force
```
进入到刚才导出的项目目录，执行如下命令烧录程序
```
sudo dfu-programmer atmega32u4 flash 文件名称.ino.leonardo.hex
```
执行后出现如下效果代表烧录成功：
```
Checking memory from 0x0 to 0x167F...  Empty.
0%                            100%  Programming 0x1680 bytes...
[>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]  Success
0%                            100%  Reading 0x7000 bytes...
[>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>]  Success
Validating...  Success
0x1680 bytes written into 0x7000 bytes memory (20.09%).
```

3. 拔掉开发板重新插入即可看到鼠标自己就移动了，因为上面demo中的代码就是控制鼠标移动的。
