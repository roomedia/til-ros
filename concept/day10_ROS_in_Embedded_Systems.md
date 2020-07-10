[ROS 강의 Chapter9. 임베디드 시스템](https://www.youtube.com/watch?v=VKNVj9IDMeo&list=PLRG6WP3c31_VIFtFAxSke2NG_DumVZPgw&index=9)
# 컴퓨터 자원의 종류와 ROS 지원
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FK1m7r%2FbtqFAghYkwE%2FjQFXIRY2pojViV9ft6ybik%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FK1m7r%2FbtqFAghYkwE%2FjQFXIRY2pojViV9ft6ybik%2Fimg.png)
x86/64, ARM A-class는 ROS 설치가 가능하나, 펌웨어 레벨의 MCU는 ROS를 설치할 수 없습니다.

## ROS에서 임베디드 시스템
- PC와는 달리 임베디드 시스템에서 ROS 설치 불가능
- 그러나 실시간성 확보 및 하드웨어 제어를 위해서는 ROS가 설치된 PC와 임베디드 시스템 간의 연결이 요구됨
- ROS에서는 이를 위해 `rosserial` 패키지를 제공합니다.

![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbqUv89%2FbtqFyD6VpZB%2FVTkaZaAck98KzhEg2AC181%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbqUv89%2FbtqFyD6VpZB%2FVTkaZaAck98KzhEg2AC181%2Fimg.png)
다이나믹셀, 센서 등에 연결된 OpenCR(ARM Cortex-M7)과 Raspberry Pi 3(ARM Cortex-A53)은 USB, serial을 통해 연결되며, Raspberry Pi에서는 ROS message을 사용할 수 있으나 OpenCR에서는 바로 사용할 수 없으며, 변환 작업을 수행하는 게 `rosserial` 패키지의 역할입니다.

제어기 -> 시리얼(rosserial 프로토콜) -> PC(ROS message 형태로 수신)
제어기 <- 시리얼(rosserial 프로토콜) <- PC(ROS message를 시리얼로 변경)

## rosserial server/client
PC 단에서 rosserial server를 실행하여야 하며, MCU 단에서는 rosserial client가 실행되어야 합니다.

다음 명령어를 이용하여 필요 패키지를 설치합니다. `rosserial`, `rosserial_server`, `rosserial_arduino` 이외에도 `rosserial_embeddedlinux`, `rosserial_xbee` 등 다양한 패키지가 함께 설치됩니다.
```
cd ~/catkin_ws/src
git clone -b noetic-devel --single-branch https://github.com/ros-drivers/rosserial
cd ..
catkin_make
```
설치되는 패키지는 다음과 같이 분류될 수 있습니다.
### rosserial server
- rosserial_python: Python 기반 server, 자주 사용됩니다.
- rosserial_server: C++ 기반 server, 일부 기능 제약이 있습니다.
- rosserial_java: Java 기반 server, 안드로이드 SDK와 함께 사용합니다.

### rosserial client
- rosserial_arduino: Arduino와 Leonardo를 지원하며, OpenCR의 경우 패키지 수정이 필요합니다.
- rosserial_embeddedlinux: 임베디드 리눅스용 클라이언트입니다.
- rosserial_windows: 윈도우 운영체제를 지원하며 윈도우 응용프로그램과 통신을 지원합니다.
- rosserial_mbed: ARM 사의 mbed를 지원합니다.
- rosserial_tivac: TI사의 Launchpad를 지원합니다.

## rosserial 프로토콜
serial은 8 bit(=1 Byte) 단위로 데이터가 전송됩니다. rosserial은 원활한 정보 전송을 위해 다음과 같은 전송 규칙이 정해져 있는데요. 각각의 Byte가 나타내는 바는 다음과 같습니다.
- 1st Byte: Sync Flag (Value: 0xff), 패킷의 시작 위치를 알기 위한 헤더로 항상 0xff의 값을 가집니다.
- 2nd Byte: Sync Flag / Protocol version, rosserial 프로토콜 버전으로 ROS Groovy가 0xff, 이후 Hydro, Indigo, Jade, Kinetic는 0xfe 값을 가집니다.
- 3rd Byte: Message Length (N), 2 Byte 중 Low Byte입니다. High, Low 순으로 이어붙인 값이 메시지의 길이가 됩니다. (Low = 0x00, High = 0x01인 경우 0x0100이 되어 N = 256 Byte)
- 4th Byte: Message Length (N), 2 Byte 중 High Byte입니다.
- 5th Byte: 메시지 길이로 계산한 Checksum, 계산 공식은 255 - ((Low Byte + High Byte) % 256)입니다.
- 6th Byte: 메시지의 형태를 구분하기 위한 Topic ID로, 2 Byte 중 Low Byte입니다.
0=PUBLISHER, 1=SUBSCRIBER, 2=SERVICE_SERVER, 4=SERVICE_CLIENT, 6=PARAMETER_REQUEST, 7=LOG, 10=TIME, 11=TX_STOP으로 구분합니다.
- 7th Byte: 메시지의 형태를 구분하기 위한 Topic ID로, 2 Byte 중 High Byte입니다.
- N Bytes : Serialized Message Data로, 송수신 메시지를 시리얼 형태로 전송하기 위한 데이터입니다. IMU, TF, GPIO 데이터가 여기 해당됩니다.
- Byte N+8: Topic ID와 Message Data로 계산한 Checksum, 계산 공식은 255 - ((Low Byte + High Byte + data byte value) % 256)입니다.

## rosserial 제약사항
- 메모리: 퍼블리셔, 섭스크라이버 개수 및 송수신 버퍼 크기를 미리 정의해야 합니다.
- Float64: MCU는 64비트 실수 연산을 지원하지 않아 32비트 형태로 변환해야 합니다. 정확성이 떨어질 수 있습니다.
- Strings: 문자열 데이터를 String 메시지 안에 저장하지 않고 문자열 데이터의 포인터(시작점) 값만 메시지에 저장합니다.
- Arrays: 메모리 제약사항으로 배열의 크기를 미리 지정하여 사용해야 합니다. (int32[] 등의 형태는 vector<int>로 구성된 동적 배열이기 때문에 사용 불가합니다.)
- 통신 속도: UART의 경우 115200bps의 속도로 데이터 통신이 이루어지기 때문에 메시지 개수가 늘어날수록 응답 및 처리 속도가 느려집니다.

## build rosserial_arduino
`rosserial_arduino` 패키지가 설치되었다면 이를 아두이노 라이브러리 폴더에 포함하여야 합니다. 아래 명령어를 이용합니다.
```
cd ~/Documents/Arduino/libraries # 아두이노 라이브러리 폴더 경로는 다를 수 있습니다.
rm -rf ros_lib # 기존 라이브러리를 갱신해야 하는 경우
rosrun rosserial_arduino make_libraries.py .
```
혹은 아두이노 라이브러리 관리를 통해 설치할 수도 있습니다.
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FqoEt0%2FbtqFAeRZnC4%2FzJV6zr0dApjPRHsNG0d3hk%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FqoEt0%2FbtqFAeRZnC4%2FzJV6zr0dApjPRHsNG0d3hk%2Fimg.png)
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FqffHI%2FbtqFyDsjWJ1%2Fsquv9alzrSfeP6KZtqKtgk%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FqffHI%2FbtqFyDsjWJ1%2Fsquv9alzrSfeP6KZtqKtgk%2Fimg.png)
두 라이브러리 모두 ROS 기본 패키지의 메시지 파일을 포함하고 있지만, `make_libraries.py`를 통해 생성된 라이브러리는 개인이 작성한 메시지 파일까지 포함하고 있습니다.

## OpenCR
OpenCR은 ROS를 지원하는 임베디드 보드로 터틀봇3에서 메인 제어기로 사용됩니다. 오픈소스 H/W, S/W로 회로, BOM, 거버 데이터 등 H/W 정보 및 OpenCR S/W를 오픈소스로 공개하고 있습니다.

OpenCR은 rosserial의 제약 사항을 극복하기 위해 32-bit ARM Cortex-M7와 Float64 계산용 FPU를 탑재하고 있습니다. 또한 1MB 플래시 메모리와 320KB SRAM을 통해 메시지 개수나 송수신 버퍼 크기를 늘려 사용할 수 있습니다. UART가 아닌 USB 패킷 전송을 사용하여 메시지 전송 속도도 빠릅니다.

SBC 계열 컴퓨터와 다양한 센서를 함께 사용하기 위해 12V@1A, 5V@4A, 3.3V@800mA를 모두 탑재하고 있으며, 32개의 pin과 OLLO 센서나 확장 포트를 충분히 확보하고 있습니다.

기본 장착 센서로는 Gyroscope, Accelerometer, Magnetometer, 전압 측정 회로를 탑재하고 있으며, USB, SPI, I2C, TTL, RS485, CAN 통신을 지원합니다.

OpenCR에서 `rosserial_arduino` 패키지를 사용하기 위해선 아래처럼 `ros_lib`의 코드를 수정해야 합니다. 해당 코드는 통신 방식을 USBSerial 방식으로 변경합니다.
```c
// ros_lib/ArduinoHardware.h
// from
// #define SERIAL_CLASS HardwareSerial
// to
#define SERIAL_CLASS USBSerial
```

## example code
아두이노에서 [파일] > [예제] > [ros_lib] > [pubsub]을 선택하고 업로드합니다. 해당 예제는 hello world! 문자열을 publish하고 led를 토글합니다. `loop()`로 인해 실행되기 때문에 `nh.spin()` 대신 `nh.spinOnce()`를 이용합니다. 마찬가지로 `ros::Rate` 대신 `delay(DELAY_MS)` 함수를 이용합니다.
```c
/*
 * rosserial PubSub Example
 * Prints "hello world!" and toggles led
 */

#include <ros.h>
#include <std_msgs/String.h>
#include <std_msgs/Empty.h>

ros::NodeHandle  nh;


void messageCb( const std_msgs::Empty& toggle_msg){
  digitalWrite(13, HIGH-digitalRead(13));   // blink the led
}

ros::Subscriber<std_msgs::Empty> sub("toggle_led", messageCb );



std_msgs::String str_msg;
ros::Publisher chatter("chatter", &str_msg);

char hello[13] = "hello world!";

void setup()
{
  pinMode(13, OUTPUT);
  nh.initNode();
  nh.advertise(chatter);
  nh.subscribe(sub);
}

void loop()
{
  str_msg.data = hello;
  chatter.publish( &str_msg );
  nh.spinOnce();
  delay(500);
}
```
스케치를 업로드한 후에는 PC에서 `roscore`를 실행한 후 `serial_node.py`를 실행합니다.
```
roscore & # ctrl + c로 탈출
rosrun rosserial_python serial_node.py _port:=/dev/ttyACM0
```
hello world!라는 메시지가 출력되며, 아두이노 기본 LED가 500ms 주기로 깜박거립니다.