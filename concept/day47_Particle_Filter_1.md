# Particle Filter
앞서 살펴본 Histogram Filter, Kalman Filter와 Particle Filter를 비교해보겠습니다.

\ |State Space|Belief|Efficiency|in Robotics
---|---|---|---|---
Histogram Filter|discrete|multimodal|exponential|approximate
Kalman Filter|continuous|unimodal|quadratic (covariance matrix)|approximate (exact only in linear system)
Particle Filter|continuous|multimodal|? (4D 이상의 도메인에선 추천되지 않음)|approximate

이외에도 파티클 필터는 프로그래밍이 가장 쉽고, 가장 유연합니다.

![초기 상태](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fci4GwQ%2FbtqG4ahpoLq%2Fsabqg9gwqfYyQdZFZ9q2yK%2Fimg.png)

다음은 로봇이 처음 로컬라이제이션을 실행할 때 파티클의 상태입니다. 초기 위치가 특정되지 않아 전지역에 고르게 분포한 것이 보입니다.

![중반](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FyCI1I%2FbtqG7ROrOja%2FwS4qVuPeNUGW1eyt9PpW6k%2Fimg.png)

로봇이 이동할 수록 위치가 특정되어 파티클이 모이기 시작합니다. 이 과정에서도 다음과 같이 여러 군집이 형성될 수 있습니다. (=multimodal)

![후반](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FwSsYL%2FbtqG774Owwv%2FhkfKm8xUMIMOf4FJEDIJH1%2Fimg.png)

이동이 계속되면서 가능한 위치가 한 곳으로 좁혀지는 모습입니다.

# Robot Class

파티클 필터를 사용하는 로봇을 파이썬 코드로 나타내봅시다. robot 클래스는 로봇의 x, y, orientation을 가지고 있으며, 다양한 메소드를 가지고 있습니다. 템플릿 코드는 다음과 같습니다.

```py
from math import *
import random

landmarks  = [[20.0, 20.0], [80.0, 80.0], [20.0, 80.0], [80.0, 20.0]]
world_size = 100.0

class robot:
    def __init__(self):
        self.x = random.random() * world_size
        self.y = random.random() * world_size
        self.orientation = random.random() * 2.0 * pi
        self.forward_noise = 0.0
        self.turn_noise    = 0.0
        self.sense_noise   = 0.0
    
    def set(self, new_x, new_y, new_orientation):
        if new_x < 0 or new_x >= world_size:
            raise ValueError, 'X coordinate out of bound'
        if new_y < 0 or new_y >= world_size:
            raise ValueError, 'Y coordinate out of bound'
        if new_orientation < 0 or new_orientation >= 2 * pi:
            raise ValueError, 'Orientation must be in [0..2pi]'
        self.x = float(new_x)
        self.y = float(new_y)
        self.orientation = float(new_orientation)
    
    def set_noise(self, new_f_noise, new_t_noise, new_s_noise):
        # makes it possible to change the noise parameters
        # this is often useful in particle filters
        self.forward_noise = float(new_f_noise)
        self.turn_noise    = float(new_t_noise)
        self.sense_noise   = float(new_s_noise)
    
    def sense(self):
        Z = []
        for i in range(len(landmarks)):
            dist = sqrt((self.x - landmarks[i][0]) ** 2 + (self.y - landmarks[i][1]) ** 2)
            dist += random.gauss(0.0, self.sense_noise)
            Z.append(dist)
        return Z
    
    def move(self, turn, forward):
        if forward < 0:
            raise ValueError, 'Robot cant move backwards'         
        
        # turn, and add randomness to the turning command
        orientation = self.orientation + float(turn) + random.gauss(0.0, self.turn_noise)
        orientation %= 2 * pi
        
        # move, and add randomness to the motion command
        dist = float(forward) + random.gauss(0.0, self.forward_noise)
        x = self.x + (cos(orientation) * dist)
        y = self.y + (sin(orientation) * dist)
        x %= world_size    # cyclic truncate
        y %= world_size
        
        # set particle
        res = robot()
        res.set(x, y, orientation)
        res.set_noise(self.forward_noise, self.turn_noise, self.sense_noise)
        return res
    
    def Gaussian(self, mu, sigma, x):
        
        # calculates the probability of x for 1-dim Gaussian with mean mu and var. sigma
        return exp(- ((mu - x) ** 2) / (sigma ** 2) / 2.0) / sqrt(2.0 * pi * (sigma ** 2))
    
    def measurement_prob(self, measurement):
        
        # calculates how likely a measurement should be
        
        prob = 1.0
        for i in range(len(landmarks)):
            dist = sqrt((self.x - landmarks[i][0]) ** 2 + (self.y - landmarks[i][1]) ** 2)
            prob *= self.Gaussian(dist, self.sense_noise, measurement[i])
        return prob    
    
    def __repr__(self):
        return '[x=%.6s y=%.6s orient=%.6s]' % (str(self.x), str(self.y), str(self.orientation))


def eval(r, p):
    sum = 0.0;
    for i in range(len(p)): # calculate mean error
        dx = (p[i].x - r.x + (world_size/2.0)) % world_size - (world_size/2.0)
        dy = (p[i].y - r.y + (world_size/2.0)) % world_size - (world_size/2.0)
        err = sqrt(dx * dx + dy * dy)
        sum += err
    return sum / float(len(p))

myrobot = robot()
```

robot 인스턴스에서 `myrobot.set(x, y, orientation)`을 통해 설정 가능합니다.

`myrobot.move(turn, forward)`를 호출하면, turn 값만큼 orientation이 변화하며, 이후 해당 방향을 향해 forward만큼 이동한 뒤 객체를 반환합니다. turn_noise, forward_noise는 아직 0인 상황입니다.

`myrobot.sense()`를 통해 4개의 오브젝트와 로봇 간의 거리를 측정할 수 있습니다. sense_noise는 아직 0이며, 4개의 측정 값이 반환됩니다.

(x, y) = (30, 50) 위치, 북쪽(= pi/2)을 향한 상태에서 로봇을 시작해봅시다. 시계 방향으로 pi/2 회전 후 15m 이동한 뒤 측정값을 출력합니다. 다시 시계 방향으로 pi/2 회전한 후 10m 이동하여 측정한 값을 출력해보세요.

변경된 코드는 다음과 같습니다. 시계 방향으로 pi/2 회전하라는 의미는, -pi/2만큼 회전하라는 말과 같습니다.

```py
# ...
####   DON'T MODIFY ANYTHING ABOVE HERE! ENTER CODE BELOW ####

myrobot = robot()
myrobot.set(30, 50, pi/2)
myrobot = myrobot.move(-pi/2, 15)
print myrobot.sense()
myrobot = myrobot.move(-pi/2, 10)
print myrobot.sense()
```

노이즈는 다음과 같이 `myrobot.set_noise(...)`로 설정해주면 됩니다.

```py
myrobot = robot()
# enter code here
myrobot.set_noise(5.0, 0.1, 5.0)
# ...
```

# Particle Filter
이제 우리는 회전할 수 있고, 회전 후 직진할 수 있는 로봇을 만들었습니다. 코드를 시각화하면 다음과 같습니다.

![시각화](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FMreJW%2FbtqG4Bl102a%2FiADQPGY1Xr8uOkVrm0CXq0%2Fimg.png)

로봇이 존재하는 곳을 다음과 같이 cyclic 하다고 가정한 뒤 파티클을 생성해봅시다.

![시각화2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FBxBRY%2FbtqHcfH2wAC%2FBYshjGiDp36cEaJgkkrMw1%2Fimg.png)

파티클은 로봇과 마찬가지로 x, y, orientation을 가집니다. 파티클의 초기 x, y, orientatino은 랜덤하게 정해집니다. 로봇 클래스를 사용하여 1000개의 파티클을 생성해봅시다.

```py
N = 1000
p = [robot() for _ in range(N)]
print len(p)
print p
```

![print](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcrglu2%2FbtqG6Jci8za%2FVBGgX6JE2pK1d3InIWigkk%2Fimg.png)

천 개의 파티클의 x, y, orientation이 보입니다. 이제 천 개의 파티클을 시뮬레이션 해봅시다.

![simulation](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FSDx4I%2FbtqG6cTyXQE%2FkacFjwTSpzRWksWCJShWWk%2Fimg.png)

```py
# generate
N = 1000
p = [robot() for _ in range(N)]
print p

# simulate
p = [particle.move(0.1, 5.0) for particle in p]
print p
```

# Importance Weight
![초기 로봇 상태](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FU3e1b%2FbtqG62XeSxS%2Fdan4lmYYvyEHBdyUhzAke1%2Fimg.png)
초기 로봇 상태가 다음과 같다고 합니다. 4개의 장애물을 향하는 4개의 측정 벡터가 생성됩니다. 

![파티클](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbXIJ6u%2FbtqG3lwGno5%2FTKOBDaXx3waqrYlvNRhNok%2Fimg.png)
이제 파티클을 생각해봅시다. 해당 파티클은 로봇이 붉은 위치, 방향에 있다고 가정하는 상황입니다. 측정값은 당연히 맞지 않겠죠.  이러한 실제 측정값과 예측값의 차이가 importance weight이 됩니다. 차이가 작을수록 weight는 커집니다. weight가 클수록 중요한 파티클입니다.

![파티클들](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F2LmV0%2FbtqG4bN8Swo%2FP4c2G628KmK3W0dGjWNauK%2Fimg.png)

Importance Weights가 높을수록 리샘플링 이후에도 존재할 가능성이 높다는 의미입니다. 리샘플링은 Importance Weights에 따라 N개의 파티클을 생성하는 과정이고, Importance Weights가 작을수록 파티클이 사라질 확률이 높습니다.

우리의 robot 클래스에서 Weights는 다음과 같이 계산합니다.

```py
# generate robot
myrobot = robot()
myrobot = myrobot.move(0.1, 5.0)
Z = myrobot.sense()

# generate particles
N = 1000
p = []
for i in range(N):
    x = robot()
    x.set_noise(0.05, 0.05, 5.0)
    p.append(x)

# move particles
p = [x.move(0.1, 5.0) for x in p]

# get particle weights
w = [particle.measurement_prob(Z) for particle in p]
print w
```

다음 시간에는 파티클 필터의 리샘플링부터 이어서 진행하도록 하겠습니다.