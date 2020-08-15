# 강의 자료
https://classroom.udacity.com/courses/cs373/lessons/48696618/concepts/487372180923

# 4D Kalman Filter?
여기서 이야기하는 4D는 (x, y, dot x, dot y)를 가리킵니다. (dot x=velocity x, dot y=velocity y를 가리키는 기호입니다.)

# 복습 퀴즈
## 퀴즈 1
![퀴즈1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbLRNeZ%2FbtqGH0Zx2gs%2F39phGwRnHgfg6IKtvQYXp1%2Fimg.png)

위 그림과 같이 분산이 같고 평균이 다른 가우시안 분포가 2개 있을 때, 두 가우시안 분포를 업데이트하여 새로운 가우시안 분포를 생성한다고 가정해봅니다. 평균은 두 가우시안 분포의 평균의 평균일 것입니다. 분산은 어떻게 될까요? 커질까요, 작아질까요, 같을까요?

![해답1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FAxw1w%2FbtqGDP53tME%2FF5ydIWFuhuFLgzWwZnU2hK%2Fimg.png)

정답은 "작아진다" 입니다. 분산이 더 작고 그래프의 높이가 높은 (=불확실성이 작은) 형태가 됩니다.

## 퀴즈 2
![퀴즈2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdaDjcZ%2FbtqGHrC4O75%2FXTKdOOjWfyTPKRPoxKQOG1%2Fimg.png)

위 그림은 사전확률과 측정값이 일치하는 상황입니다. 새롭게 만들어지는 확률 분포를 mu, nu^2으로 표현한다고 할 때, nu^2은 sigma^2의 몇 배일까요?

정답은 0.5배입니다! 두 가우시안 분포를 더하면 다음과 같은 식이 만들어지기 때문입니다.

![-\frac12\frac{(x-\mu)^2}{\sigma^2}-\frac12\frac{(x-\mu)^2}{\sigma^2}=-\frac12\frac{(x-\mu)^2}{1/2\sigma^2}](https://latex.codecogs.com/gif.latex?\bg_white&space;-\frac12\frac{(x-\mu)^2}{\sigma^2}-\frac12\frac{(x-\mu)^2}{\sigma^2}=-\frac12\frac{(x-\mu)^2}{1/2\sigma^2})

## 퀴즈 3
![퀴즈3](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbIcreO%2FbtqGIKa5H3E%2FKEje1uZhIR6CNuJ2ZG3i4k%2Fimg.png)

위 식으로 다음과 같은 가우시안 분포가 성립할 수 있을까요?

![해답3](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbDk0Ut%2FbtqGH0Zx2kf%2FMlUdAIKRt1im3UhkiJuKmK%2Fimg.png)

정답은 아니랍니다. 가우시안 분포는 확률이기 때문에 x축과 그래프가 만드는 도형의 넓이는 1이 되어야 합니다. x축이 무한대로 간다고 가정했을 때, 확률은 1을 넘어 무한대가 나옵니다.

## 퀴즈 4
![퀴즈4](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fcpkb7h%2FbtqGJvdGMsz%2FwrTWkuK5nL2bkTTB2SDml1%2Fimg.png)

2D Kalman Filter의 state vector는 ([[x], [dot x]]) 2차원으로 표현할 수 있습니다. 4D Kalman Filter의 state vector는 몇 차원일까요?

![해답4](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FubdDC%2FbtqGHrwhebo%2FmKQmCMPdvee1fpz0FmIVu0%2Fimg.png)

정답은 4입니다!

## 퀴즈 5
![퀴즈5](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbuu0Du%2FbtqGJwKpSKw%2FnrjAV5wDwVhWWdkCqPkVj1%2Fimg.png)

1차원 좌표 상에서 움직이는 물체의 F(state function)는 다음과 같습니다. 2차원 좌표 상에서 움직이는 물체의 state function은 무엇일까요?

![해답5](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F00O4U%2FbtqGGGnaOd4%2FqvvKoqRlF3KHrSQ4XIy9nK%2Fimg.png)

matrix([[x prime], [z]]) = state function * state vector이며, x prime = x + dot x, z = x의 공식을 갖고 있습니다. 이를 x, y에 대해 확장한 답은 다음과 같습니다.

## 퀴즈 6
아래 코드에서 4D Kalman Filter에 대한 P, F, H, R, I를 올바르게 작성하세요. P는 초기 설정한 불확실성, F는 state function, H는 measurement function, R은 measurement noise, I는 단위행렬입니다.

```py
# Fill in the matrices P, F, H, R and I at the bottom
#
# This question requires NO CODING, just fill in the 
# matrices where indicated. Please do not delete or modify
# any provided code OR comments. Good luck!

from math import *

class matrix:
    
    # implements basic operations of a matrix class
    
    def __init__(self, value):
        self.value = value
        self.dimx = len(value)
        self.dimy = len(value[0])
        if value == [[]]:
            self.dimx = 0
    
    def zero(self, dimx, dimy):
        # check if valid dimensions
        if dimx < 1 or dimy < 1:
            raise ValueError, "Invalid size of matrix"
        else:
            self.dimx = dimx
            self.dimy = dimy
            self.value = [[0 for row in range(dimy)] for col in range(dimx)]
    
    def identity(self, dim):
        # check if valid dimension
        if dim < 1:
            raise ValueError, "Invalid size of matrix"
        else:
            self.dimx = dim
            self.dimy = dim
            self.value = [[0 for row in range(dim)] for col in range(dim)]
            for i in range(dim):
                self.value[i][i] = 1
    
    def show(self):
        for i in range(self.dimx):
            print self.value[i]
        print ' '
    
    def __add__(self, other):
        # check if correct dimensions
        if self.dimx != other.dimx or self.dimy != other.dimy:
            raise ValueError, "Matrices must be of equal dimensions to add"
        else:
            # add if correct dimensions
            res = matrix([[]])
            res.zero(self.dimx, self.dimy)
            for i in range(self.dimx):
                for j in range(self.dimy):
                    res.value[i][j] = self.value[i][j] + other.value[i][j]
            return res
    
    def __sub__(self, other):
        # check if correct dimensions
        if self.dimx != other.dimx or self.dimy != other.dimy:
            raise ValueError, "Matrices must be of equal dimensions to subtract"
        else:
            # subtract if correct dimensions
            res = matrix([[]])
            res.zero(self.dimx, self.dimy)
            for i in range(self.dimx):
                for j in range(self.dimy):
                    res.value[i][j] = self.value[i][j] - other.value[i][j]
            return res
    
    def __mul__(self, other):
        # check if correct dimensions
        if self.dimy != other.dimx:
            raise ValueError, "Matrices must be m*n and n*p to multiply"
        else:
            # subtract if correct dimensions
            res = matrix([[]])
            res.zero(self.dimx, other.dimy)
            for i in range(self.dimx):
                for j in range(other.dimy):
                    for k in range(self.dimy):
                        res.value[i][j] += self.value[i][k] * other.value[k][j]
            return res
    
    def transpose(self):
        # compute transpose
        res = matrix([[]])
        res.zero(self.dimy, self.dimx)
        for i in range(self.dimx):
            for j in range(self.dimy):
                res.value[j][i] = self.value[i][j]
        return res
    
    # Thanks to Ernesto P. Adorio for use of Cholesky and CholeskyInverse functions
    
    def Cholesky(self, ztol=1.0e-5):
        # Computes the upper triangular Cholesky factorization of
        # a positive definite matrix.
        res = matrix([[]])
        res.zero(self.dimx, self.dimx)
        
        for i in range(self.dimx):
            S = sum([(res.value[k][i])**2 for k in range(i)])
            d = self.value[i][i] - S
            if abs(d) < ztol:
                res.value[i][i] = 0.0
            else:
                if d < 0.0:
                    raise ValueError, "Matrix not positive-definite"
                res.value[i][i] = sqrt(d)
            for j in range(i+1, self.dimx):
                S = sum([res.value[k][i] * res.value[k][j] for k in range(self.dimx)])
                if abs(S) < ztol:
                    S = 0.0
                try:
                   res.value[i][j] = (self.value[i][j] - S)/res.value[i][i]
                except:
                   raise ValueError, "Zero diagonal"
        return res
    
    def CholeskyInverse(self):
        # Computes inverse of matrix given its Cholesky upper Triangular
        # decomposition of matrix.
        res = matrix([[]])
        res.zero(self.dimx, self.dimx)
        
        # Backward step for inverse.
        for j in reversed(range(self.dimx)):
            tjj = self.value[j][j]
            S = sum([self.value[j][k]*res.value[j][k] for k in range(j+1, self.dimx)])
            res.value[j][j] = 1.0/tjj**2 - S/tjj
            for i in reversed(range(j)):
                res.value[j][i] = res.value[i][j] = -sum([self.value[i][k]*res.value[k][j] for k in range(i+1, self.dimx)])/self.value[i][i]
        return res
    
    def inverse(self):
        aux = self.Cholesky()
        res = aux.CholeskyInverse()
        return res
    
    def __repr__(self):
        return repr(self.value)


########################################

def filter(x, P):
    for n in range(len(measurements)):
        
        # prediction
        x = (F * x) + u
        P = F * P * F.transpose()
        
        # measurement update
        Z = matrix([measurements[n]])
        y = Z.transpose() - (H * x)
        S = H * P * H.transpose() + R
        K = P * H.transpose() * S.inverse()
        x = x + (K * y)
        P = (I - (K * H)) * P
    
    print 'x= '
    x.show()
    print 'P= '
    P.show()

########################################

print "### 4-dimensional example ###"

measurements = [[5., 10.], [6., 8.], [7., 6.], [8., 4.], [9., 2.], [10., 0.]]
initial_xy = [4., 12.]

# measurements = [[1., 4.], [6., 0.], [11., -4.], [16., -8.]]
# initial_xy = [-4., 8.]

# measurements = [[1., 17.], [1., 15.], [1., 13.], [1., 11.]]
# initial_xy = [1., 19.]

dt = 0.1

x = matrix([[initial_xy[0]], [initial_xy[1]], [0.], [0.]]) # initial state (location and velocity)
u = matrix([[0.], [0.], [0.], [0.]]) # external motion

#### DO NOT MODIFY ANYTHING ABOVE HERE ####
#### fill this in, remember to use the matrix() function!: ####

P =  # initial uncertainty: 0 for positions x and y, 1000 for the two velocities
F =  # next state function: generalize the 2d version to 4d
H =  # measurement function: reflect the fact that we observe x and y but not the two velocities
R =  # measurement uncertainty: use 2x2 matrix with 0.1 as main diagonal
I =  # 4d identity matrix

###### DO NOT MODIFY ANYTHING HERE #######

filter(x, P)
```

이전에 해결하였던 2D Kalman Filter에서 각각의 형태는 다음과 같았습니다. 이때는 dt가 1이었으나, 지금은 dt가 주어져있습니다.

```py
P = matrix([[1000., 0.], [0., 1000.]]) # initial uncertainty
F = matrix([[1., 1.], [0, 1.]])
H = matrix([[1., 0.]])
R = matrix([[1.]])
I = matrix([[1., 0.], [0., 1.]])
```

아래처럼 각 변수가 어떤 형태를 가져야 하는지 기술되어 있습니다. 

```py
P =  # 초기 불확실성: x와 y는 0, dot x, dot y는 1000
F =  # state function: 2d 버전을 4d로 일반화
H =  # measurement function: x와 y는 측정했지만 속력을 측정하지 못한 걸 반영
R =  # measurement uncertainty: 값이 0.1인 2x2 대각 행렬
I =  # 4d 단위 행렬
```

정답은 다음과 같습니다.

```py
P = matrix([[0., 0., 0., 0.], [0., 0., 0., 0.], [0., 0., 1000., 0.], [0., 0., 0., 1000.]])
F = matrix([[1., 0., dt, 0.], [0., 1., 0., dt], [0., 0., 1., 0.], [0., 0., 0., 1.]])
H = matrix([[1., 0., 0., 0.], [0., 1., 0., 0.]])
R = matrix([[0.1, 0.], [0., 0.1]])
I = matrix([[1., 0., 0., 0.], [0., 1., 0., 0.], [0., 0., 1., 0.], [0., 0., 0., 1.]])
```