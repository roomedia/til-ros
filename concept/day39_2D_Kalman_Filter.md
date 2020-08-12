# 39일차 - 2차원 Kalman Filter, Position과 Velocity의 상관관계
# High Dimension Gaussian

2차원 이상 차원 D의 가우시안 분포는 High Dimension Gaussian 또는 Multivariabte Gaussian이라고 합니다. 이때 가우시안 분포의 평균은 주어진 차원에 맞는 행렬이 됩니다. (Dx1 행렬) 분산은 DxD 행렬로 표현할 수 있습니다. 확장된 확률 밀도는 다음과 같습니다...만 강의에서는 계산식이 별로 중요하지 않다고 말합니다. 어차피 컴퓨터가 계산해서 그런 걸까요??

![다변량 정규분포의 확률 밀도](https://wikimedia.org/api/rest_v1/media/math/render/svg/134909956e76f262655f5a4cddedc792c19b55ea)

High Dimension Gaussian의 그림을 그려보면 다음과 같습니다. 아래와 같이 contour를 그리는 게 가능하며, X 좌표에 따라 Y 좌표가 결정되는 correlate 관계에 있습니다. 각 축의 분산(=불확실성)이 작을수록 점에 가까운 형태를 보이며, 불확실할수록(=분산이 클수록) 해당 축을 기준으로 넓게 퍼지는 것을 볼 수 있습니다.

![다변량 정규분포의 그림](https://kr.mathworks.com/help/stats/computecumulativeprobabilitiesoverregionsexample_01_ko_KR.png)

mu = (x, y)

칼만 필터는 이 가우시안 분포를 이용해 다음 위치를 추정합니다. 아래 그림을 보면 예측 위치와 보정 위치가 측정에 따라 점차 실제 위치에 가까워지며, 일정한 궤도를 갖는 것을 볼 수 있습니다.

![칼만 필터 추정 위치와 실제 위치](https://www.mathworks.com/help/examples/driving/win64/ConstructConstantVelocityLinearKalmanFilterExample_01.png)

관측 결과나 예측 하나만 가지고는 이러한 결과를 내기 힘들지만, 물체의 위치는 물체가 이동해온 속도와 연관이 있습니다.

![](https://t1.daumcdn.net/cfile/tistory/9979044E5C1F356C2B)

위 그림처럼 x는 물체의 위치, x dot은 속도이며, x의 최초 위치를 1로 추정했다고 해봅시다. 이때 속도에 대한 불확실성은 매우 커 길쭉한 ㅏ란색 분포를 따르게 됩니다. 이때 속도가 1이면 dt 이후 위치는 2가 되고, 2면 3이 되고, ... 이를 빨간색 분포(= predict)로 놓고, dt 이후 측정했더니 초록색 분포가 나왔다고 가정합니다. 빨간 분포와 초록 분포를 곱하여 속도에 대한 새로운 가우시안을 얻을 수 있으며, 두 가우시안은 곱하는 과정에서 무조건 분산이 작아집니다.

![##_Image|kage@xjsuE/btqGurYeJBE/GZxXz7pB7FWAQQR7FKP7h1/img.png|alignCenter|data-origin-width="0" data-origin-height="0" data-ke-mobilestyle="widthContent"|||_##](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FxjsuE%2FbtqGurYeJBE%2FGZxXz7pB7FWAQQR7FKP7h1%2Fimg.png)

이렇게 Kalman Filter를 통해 위치만 측정했는데도 속도를 얻을 수 있게 되는 것입니다. 칼만 필터의 이러한 특성은 아주 유용하여 로보틱스, 자율주행 등에서 자주 사용됩니다.

# 2차원 칼만 필터

2차원 칼만 필터의 state는 X, dot X로 이루어진 2x1 행렬이고, 이를 업데이트하는 state transition function은 다음과 같습니다.

![](https://latex.codecogs.com/png.latex?%5Cbg_white&space;%5Cbinom%7BX%7D%7B%5Cdot&space;X%7D&space;%5Cleftarrow&space;%5Cbegin%7Bpmatrix%7D&space;1&space;&&space;dt%5C&space;0&space;&&space;1&space;%5Cend%7Bpmatrix%7D%5Cbinom%7BX%7D%7B%5Cdot&space;X%7D)

measurement는 Z로 표시하며 measurement function은 다음과 같습니다.

![](https://latex.codecogs.com/png.latex?%5Cbg_white&space;Z&space;%5Cleftarrow&space;%5Cbegin%7Bpmatrix%7D&space;1&space;&&space;0&space;%5Cend%7Bpmatrix%7D%5Cbinom%7BX%7D%7B%5Cdot&space;X%7D)

-   dt로 표기하여야 더 일반적인 형태를 표현할 수 있다는 강의 주석을 참고하여 dt를 추가하였습니다.