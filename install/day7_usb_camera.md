# ROS2에는 uvc_camera가 없습니다
[ROS 강의 Chapter6. ROS 도구](https://www.youtube.com/watch?v=fB2YINZOIng&list=PLRG6WP3c31_VIFtFAxSke2NG_DumVZPgw&index=6)
강의를 듣던 중, `uvc_camera`와 `rqt_image_view`를 이용하여 영상을 띄우는 걸 볼 수 있습니다.

이를 ROS2에 맞게 실행해보려 했지만, `uvc_camera`는 Melodic 버전까지만 지원되는 패키지였습니다. 관련 정보는 아래 링크에서 확인할 수 있습니다.
[https://index.ros.org/p/uvc_camera/](https://index.ros.org/p/uvc_camera/)

ROS2 foxy에서 실행 가능하며, macOS에서도 실행할 수 있는 라이브러리를 찾아보았습니다. 대부분의 라이브러리가 V4L(Video4Linux)를 사용하여 영상을 제공하기에, macOS를 지원하는 라이브러리는 찾기 힘들어보였습니다.

한참을 고생하던 중, ROS2 웹캠 데모를 찾을 수 있었습니다. `image_tools` 패키지의 `cam2image`, `showimage` 노드가 있습니다.

아래 명령어를 사용하여 웹캠으로부터 정보를 받아 msg를 뿌려주는 publisher를 실행합니다.
```
ros2 run image_tools cam2image
```
다른 터미널 창에서 msg를 받아 이미지를 그려주는 subscriber 노드를 실행합니다.
```
ros2 run image_tools showimage
```
두 노드의 연결 상태와 실행 화면은 다음과 같습니다.
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbCHVGU%2FbtqFuimg3su%2FJRwUI4RDvKmDFt5SNXliBK%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FbCHVGU%2FbtqFuimg3su%2FJRwUI4RDvKmDFt5SNXliBK%2Fimg.png)
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FDhVFy%2FbtqFtwFxj2c%2FRIweNvcnKfdnNWHvplDB70%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FDhVFy%2FbtqFtwFxj2c%2FRIweNvcnKfdnNWHvplDB70%2Fimg.png)
![https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FcBO4GC%2FbtqFtigp3cr%2FBK463vPzpJKCKZotg8ecpK%2Fimg.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fk.kakaocdn.net%2Fdn%2FcBO4GC%2FbtqFtigp3cr%2FBK463vPzpJKCKZotg8ecpK%2Fimg.png)