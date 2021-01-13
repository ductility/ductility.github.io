---
layout: post
title: Spot Micro
subtitle: 저예산 사족보행로봇 Spot Micro 만들기
background: '/img/posts/210112/01대표사진.jpg'
comments: true

date:   2021-01-12 15:00:00 
lastmod : 2021-01-12 15:00:00
sitemap :
  changefreq : weekly
  priority : 1.0
---

# Spot Micro 


[![_config.yml]({{ site.baseurl }}/img/posts/210112/썸네일(수정).jpg){: width="100%"}](https://www.youtube.com/watch?v=KfkMx_psZdc=0s){: target="_blank"}
*이미지를 누르면 해당 동영상 링크가 새 창에서 열립니다.*
<!-- ![_config.yml]({{ site.baseurl }}/img/posts/210112/01대표사진.jpg){: width="100%"} -->

작년 2020년 3월부터 12월까지, 1년 가까이 성균관대 [Robotics Innovatory](https://mecha.skku.ac.kr/roboticsinnovatory/index.do) 에서 스핀오프한 기업 [Aidin Robotics](https://www.aidinrobotics.com/)의 사족보행로봇 팀에서 학부연구생 겸 인턴으로 일했다. 사족보행로봇 팀에 참여하다보니 자연스레 전반적인 사족보행로봇의 로봇 시스템에 대해 궁금해져 선배님들께 이것저것 물어보았고 선배님들은 최근 기술 동향이나 제작 노하우 등을 알려주셨다.

그러면서 전체 로봇 시스템을 내 손으로 만들어 보고 싶다는 생각을 하던 차에 Boston Dynamics의 Spot Mini를 축소하여 저예산으로 만든 오픈소스 프로젝트 "Spot Micro"를 알게 되었다. 저예산 프로젝트임에도 5~60만원에 달하는 12개의 서보모터 비용이 부담되어 제작을 고민하고 있던 중에 그 모습을 지켜보던 Aidin Robotics 대표님께서 재료비를 지원해 주셔 프로젝트를 진행할 수 있었다.
<br>
<br>

## 개요
![_config.yml]({{ site.baseurl }}/img/posts/210112/02개요.jpg){: width="100%"}

로봇에 장착된 Jetson Nano의 전원을 켜고, 동일한 네트워크에 연결된 PC에서 SSH를 이용하여 Jetson Nano에 접속한다. ROS를 이용하여 로봇을 제어하는데 필요한 프로세스들을 실행하면 Jetson Nano와 PCA9685 서보모터 제어 보드가 I2C 통신을 시작한다. PWM 제어 보드는 7.4V 18650 리튬이온전지에게 전원을 공급받고 있는 12개의 서보모터에게 제어 명령을 내리고 로봇의 모션 제어가 이루어진다.

<br>
<br>


## 기반 프로젝트 선택

Spot Micro 오픈소스 프로젝트는 싱기버스의 KDY0523(Deok-yeon Kim) 님의 3D 모델링(https://www.thingiverse.com/thing:3445283)에서 시작되어 많은 파생 버전을 만들었다. 파생 버전에 따라 사용하는 하드웨어나 소프트웨어 스택이 정말 다양했는데 나는 나에게 익숙하고 구매가 쉬운 부품과 기반 소스코드를 선택했다.

### 하드웨어 선택

<!-- ![_config.yml]({{ site.baseurl }}/img/posts/210112/03모터.jpg){: width="60%"} -->

조이스틱/키보드 등을 이용해 단순 보행만을 구현할 계획이었기에 LCD 모니터, 초음파센서, 자이로센서 등은 구매하지 않았다. 

- Jetson Nano(주 제어기)   
저예산 SBC 중 Jetson Nano와 호환되는 부품들을 가지고 있었기 때문에 Raspberry Pi 대신에 Jetson Nano를 주 제어기로 선정했다.

- DGS-1199(서보 모터)   
[MG996R](https://www.youtube.com/watch?v=LqLittumdvQ), [cls6636hv](https://www.youtube.com/watch?v=bnEtJSkUFDk&t=11s) 등 선택지가 있었는데 전자는 모터 스펙이 부족해서 원활한 구동이 안되어 보였고 후자는 국내에서는 판매하지 않아 비슷한 스펙의 대체품인 DGS-1199를 선택했다.

- PCA9685(모터 제어기)   
[Spot Mini Mini](https://github.com/OpenQuadruped/spot_mini_mini) 처럼 Teensy 4.0을 이용한 제어도 고려했으나 PCB 제작이 부담되어 i2c로 pwm 서보를 제어하는 PCA965를 선택했다.

<br>


### 기반 소스코드 선택

많은 오픈소스 프로젝트 소스코드 중 ROS에 익숙했기 때문에 [mike4192님의 프로젝트](https://github.com/mike4192/spotMicro)를 기반 소스코드로 삼았다. 내가 선택한 하드웨어와 몇 가지 다른점이 있어서 소스코드를 일부 수정해야 했다.

mike4192님의 프로젝트는 Raspberry Pi 3B에 Ubuntu 16.04, ROS Kinetic을 사용했는데 나는 Jetson Nano에 Ubuntu 18.04, ROS Melodic을 사용했다. 이때문에 apt 저장소에 올라오는 라이브러리의 버전이 일부 달랐고 이를 해결해 주었다.

**spotMicro/ros-i2cpwmboard/src/i2cpwm_controller.cpp**
```cpp
#include <stdio.h>
...
#include <linux/i2c-dev.h>

// 추가한 헤더파일
#include <i2c/smbus.h>
```

**spotMicro/ros-i2cpwmboard/CMakeLists.txt**
```cmake
add_executable(i2cpwm_board src/i2cpwm_controller.cpp)
target_link_libraries(i2cpwm_board ${catkin_LIBRARIES} i2c) #i2c 라이브러리 추가
add_dependencies(i2cpwm_board i2cpwm_board_generate_messages_cpp)
```

위와 같이 `<i2c/smbus.h>`를 추가하고 `i2c`라이브러리를 링크하여 해결했다.

<br>
<br>


## 제작

### 3D 프린팅

선택한 서보모터 DGS-1199가 CLS6336hv와 사이즈가 비슷해서 [KDY0523 님의 3D 모델링](https://www.thingiverse.com/thing:3445283) 중 CLS6336HV 버전을 사용했다. 여기서 초음파센서와 LCD모니터를 사용하지 않기 때문에 이것들이 장착되는 전면과 후면 커버의 모델링을 일부 수정했다. Autodesk Inventor을 이용하여 전면커버에는 Jetson Nano 전원 버튼 위한 구멍을 내고, 후면 커버에는 모터 전원 공급 스위치를 달기 위한 구멍을 냈다.

![_config.yml]({{ site.baseurl }}/img/posts/210112/04모델수정.jpg){: width="100%"}

Aidin Robotics의 지원을 받아 Markforged Onyx-One 3D 프린터를 사용하여 부품들을 3D 프린팅했다. 이 프린터는 PLA나 ABS 등 일반적으로 쓰이는 필라멘트와 달리 Onyx라는 독특한 재료를 사용했는데, 나일론과 유사한 연성재료여서 FDM 방식의 프린터임에도 출력물이 잘 깨져서 부러지지 않는다. 따라서 사족보행 로봇의 다리나 몸체의 제작에 효과적이다.

![_config.yml]({{ site.baseurl }}/img/posts/210112/05프린터.jpg){: width="100%"}

![_config.yml]({{ site.baseurl }}/img/posts/210112/06프린팅.jpg){: width="100%"}

<br>


### 배선

18650 리튬이온전지, PCA9685, Jetson Nano, 12 x PWM 서보모터 등을 배선한다.

안정적인 전력 공급을 위해 Jetson Nano에는 휴대폰 보조배터리를 이용해 전력을 공급했다.

PCA9685보드 1개에 12개의 서보모터가 모두 연결되어있는 구조이기 때문에, 보드로 전력을 공급하면 최대 전압, 전류 제한 때문에 구동에 필요한 전력이 부족해진다.

이에 따로 전원을 공급할 수 있게 하는 간이 모터 쉴드를 만능기판에 납땜하고 저항과 커패시터를 달아 안정적인 전력 공급을 유도했다.

<br>
<br>

## 모터 정렬

구매한 모터는 구동 범위가 제한된 PWM서보이기 때문에 모터 정렬을 해야 한다. mike4194님의 Servo Calibration Guide에 따라 특정 자세일 때 모터의 PWM 값을
config.yaml 파일에 기록하여 정렬하는 것이다. 하지만 예상과는 달리 정렬을 마친 뒤에도 로봇의 다리가 움직이기는 하지만 제대로 걷지 못하는 현상을 겪었다.   

세밀한 모터 정렬이 문제인가 싶어 여러 번 재시도 하던 중, 시계방향으로 돌아야 할 PWM 신호에서 반대방향으로 모터가 동작하는 것을 발견했다. 구매한 모터의 회전방향이 가이드에 올라온 것과 반대였던 것이다.

이를 발견하고 config의 방향 설정을 반대로 바꿔 주었고, 보행 테스트를 할 수 있었다.

<br>
<br>

## 8-Phase 보행

mike4192님 코드의 master branch에는 기본적으로 8-Phase로 왼발 앞다리 - 오른발 뒷다리 - 오른발 앞다리 - 왼발 뒷다리 순으로 한다리씩 움직여 걷는 알고리즘이 적용되어있다. SSH로 Jetson Nano에 접속한 뒤 다음 코드들을 실행하여 구동할 수 있다.

**bash**
```bash
$ roscore

$ rosrun i2cpwm_board i2cpwm_board

$ roslaunch spot_micro_motion_cmd spot_micro_motion_cmd.launch

$ rosrun spot_micro_keyboard_command spotMicroKeyboardMove.py
```

ROS 노드간의 통신을 위한 roscore, i2c 보드와의 통신을 위한 i2cpwm_board, 8-phase 모션 제어를 위한 spot_micro_motion_cmd, 키보드 입력으로 명렁어 전달을 위한 spotMicroKeyboardMove.py를 실행시킨다. 이 코드들은 ROS Topic을 이용한 프로세스 간 통신 - i2c 통신을 거쳐 서보모터에게 제어명령을 전달하고 최종적으로 로봇이 동작하게 된다.

![_config.yml]({{ site.baseurl }}/img/posts/210112/07한발씩gait.gif){: width="100%"}


<!-- 동작하는 움짤 및 유튜브 영상 -->

<br>
<br>

## Trot Gait 보행 (4-Phase)

mike4192님 코드의 alternate_gate branch에는 대각선에 있는 두 다리를 동시에 움직이는 4-Phase Trot Gait 보행이 적용된 코드가 있다. Trot Gait은 두 다리를 동시에 움직이는 만큼, 모터 정렬이 어긋나있거나 무게 중심이 올바르지 않다면 로봇이 넘어질 가능성이 더 커진다. 작동법은 8-Phase 보행과 비슷하게 위 코드를 실행하면 된다.

![_config.yml]({{ site.baseurl }}/img/posts/210112/08두발씩.gif){: width="100%"}


<!-- 동작하는 움짤 -->

<br>
<br>

## 조이스틱으로 동작

기존 소스코드는 키보드로 로봇을 동작시키는 노드 밖에 없었다. 로봇을 움직이려는데 컴퓨터 앞에 앉아 키보드를 두드리다보면 로봇의 돌발 행동에 대비하기도 힘들고, 폼도 나지 않는다. 따라서 Logitech F710 조이스틱을 이용해 로봇을 조종하는 코드를 작성했다.  

기존 코드는 python으로 작성되었지만, 나는 python 보다 C++에 익숙해서 C++로 **spotMicro/spot_micro_joy/src/joy.cpp**를 작성했다. 다음 명령어로 코드를 실행한다.

**bash**
```bash
$ sudo chmod a+rw /dev/input/js0

$ roscore

$ rosrun i2cpwm_board i2cpwm_board

$ roslaunch spot_micro_motion_cmd spot_micro_motion_cmd.launch

$ rosparam set joy_node/dev "/dev/input/js0"
$ rosrun joy joy_node

$ rosrun spot_micro_joy spot_micro_joy

```

`spot_micro_joy`라는 ROS 패키지를 만들었다. 조이스틱 신호를 받는 ROS 노드를 켜고 새로 만든 노드를 실행시키면 조이스틱 입력을 받아 로봇이 작동한다.

![_config.yml]({{ site.baseurl }}/img/posts/210112/joystick_1.gif){: width="100%"}

![_config.yml]({{ site.baseurl }}/img/posts/210112/joystick_2.gif){: width="100%"}


<br>
<br>

## 도움 링크

Aidin Robotics: https://www.aidinrobotics.com/

Deok-yeon Kim 님의 모델링: https://www.thingiverse.com/kdy0523/designs

Mike4192 님의 GitHub: https://github.com/mike4192/spotMicro

Spot Mini Mini: https://github.com/OpenQuadruped/spot_mini_mini