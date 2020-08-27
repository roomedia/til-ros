# Mathmatics for Circular Motion
## Radius
과제 3, 4는 로봇의 원형 움직임을 시뮬레이션 하는 과제였습니다. 강의 자료에서는 시뮬레이션을 돕기 위해 몇몇 식을 제공했습니다. 오늘은 해당 식이 어떤 과정을 거쳐 유도되는지 알아봅시다. 먼저 a, r, L 기호를 다음과 같이 설정합니다.

```
a = 앞 바퀴의 회전 각도
r = 회전의 반지름
L = 자동차의 길이
```

그러면 다음과 같은 식이 성립합니다.

```
r = L / tan(a)
```

이를 유도하기 위해선, 먼저 앞 바퀴와 뒷 바퀴가 같은 원을 따라 움직이지 않는다는 사실을 알아야 합니다. 앞 바퀴와 뒷 바퀴가 그리는 궤적은 다음과 같습니다.

![앞 바퀴와 뒷 바퀴 궤적](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbbjnne%2FbtqHujKKYCA%2Fp8Zgy9wc5Fxbkn0zrKcN9k%2Fimg.png)

그리고 뒷 바퀴의 진행 방향과 앞 바퀴의 진행 방향(= 앞 바퀴 궤적의 접선)이 이루는 각도가 a인 것입니다. 반지름과 접선이 이루는 각도는 반드시 90도이기 때문에, 나머지 각은 90-a, 고로 cx, cy와 앞 바퀴, 뒷 바퀴가 이루는 각도는 a가 되게 됩니다.

![반지름끼리 이루는 각도는 a가 된다](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbI9suy%2FbtqHxbk68li%2Fsmniw37K6RB4Y18MWb2apK%2Fimg.png)

그러므로 tan(a) = L / r, r = L / tan(a)가 됩니다.

## Position
이번에는 로봇의 circular motion 중심 위치(cx, cy) 공식이 어떻게 도출되었는지 알아봅시다. 먼저 다음과 같이 x, y, theta를 갖는 로봇이 circular motion을 하고 있다고 합니다. cx, cy의 위치는 어떻게 구할까요?

![moving robot](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F2Q7ek%2FbtqHhQjhB5H%2FOidXnii5kQhPg01xsUM8x1%2Fimg.png)

이번에도 먼저 cx, cy에서 x축을 향해 수직인 선을 내려 삼각형을 만듭니다. cx, cy, x, y가 이루는 선과 로봇의 오리엔테이션이 이루는 각도는 90도입니다. cx, cy, x, y가 이루는 선이 r이면 오리엔테이션이 이루는 각도가 원 운동의 접선이기 때문입니다. 그렇다면 우리가 만든 삼각형의 모든 각을 알아내게 됩니다.

수직으로 내린 선과 cx, cy, x, y가 이루는 각도가 theta였네요! 그렇다면 가로선과 세로선의 길이는 각각 R * sin(theta), R * cos(theta)가 되게 됩니다.

![r * sin(theta), r * cos(theta)](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F2dSG9%2FbtqHsdjUxgn%2FpM19yu7qaosndni8f9Zqlk%2Fimg.png)

이를 다시 정리하면 x, y, theta를 이용하여 cx, cy의 좌표를 구할 수 있습니다.

![cx, cy](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbtiq8d%2FbtqHtFf8myF%2FQkD2Rjvq7IwrI5EU5qAZa1%2Fimg.png)

```
cx = x - R * sin(theta)
cy = y + R * cos(theta)
```

이제 이동한 각도를 beta라고 하고, 이동한 위치를 x', y'라고 해봅시다. x', y'은 어떻게 구할 수 있을까요?

![베타, x', y'](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbtlTPX%2FbtqHjpZ2to5%2F0wAwEt7HCuCXk6SX8q1p51%2Fimg.png)

(cx, cy), (x', y')과 (cx, cy)에서 x축으로 내린 수선을 연결하여 새로운 삼각형을 만들 수 있습니다. 이때 각도는 beta + theta가 되며, 빗변은 여전히 R입니다. 따라서 x', y'는 다음과 같아집니다.

![아래와 같은 수식](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbK3tOO%2FbtqHxcj2ytJ%2FGpW1zkGrwkGvSvS26oaIw0%2Fimg.png)

```
x' = cx + r * sin(beta + theta)
y' = cy - r * cos(beta + theta)
theta' = (theta + beta) % 2 * pi    # 정규화
```