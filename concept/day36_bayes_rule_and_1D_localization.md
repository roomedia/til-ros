# 조건부확률

조건부확률은 주어진 사건(A)이 일어났을 때 다른 사건(B)이 일어날 확률을 말하며, 원래 확률 함수를 p라고 할 때 조건부확률은 p(B | A)라고 표기합니다. p(B | A)는 다시 그 정의에 의해, p(B | A) = p(B ∩ A) / p(A)와 같습니다.

# 용어 설명

prior probability(사전확률): p(A), p(B) 등 어떤 시스템이나 모델에 대해 알고있는 선험적 확률, 관측 하기 전에 가지고 있는 확률. 현재 알고 있는 정보를 나타냅니다.  
likelihood (우도) : p(B | A) 사전 확률에 근거하여 데이터가 발생할 확률. 조건부 확률을 나타냅니다.  
posterior probability(사후 확률): p(A | B), 데이터가 발생했을 때 해당 데이터가 관측 모델에 속할 확률입니다.

# Udacity 강의입니다.

Sebastian Thrun 교수님의 Probabilistic Robotics가 Artificial Intelligence for Robotics로 진화했습니다..

[https://classroom.udacity.com/courses/cs373](https://classroom.udacity.com/courses/cs373)

# Localization

Localization이란 지도 위에서 로봇이 어디쯤 있는지 알기 위한 알고리즘입니다. 기존 사용하던 GPS(Global Positioning System)의 정확도는 실제 위치와 2 ~ 10m 차이나며, 자율주행을 위해서는 이를 2 ~ 10cm로 줄이는 게 관건이었습니다.

직선 통로와 로봇을 기준으로 이야기 해보겠습니다.

<table style="border-collapse: collapse; width: 0%; height: 63px;" border="1"><tbody><tr style="height: 19px;"><td style="height: 19px; width: 19.8837%;"><span style="color: #333333;">🤖?</span></td><td style="width: 20%;"><span style="color: #333333;">&nbsp;</span></td><td style="width: 20%;"><span style="color: #333333;">&nbsp;</span></td><td style="width: 20%;"><span style="color: #333333;">&nbsp;</span></td><td style="width: 20%;"><span style="color: #333333;">&nbsp;</span></td></tr><tr style="height: 25px;"><td style="width: 19.8837%; height: 25px;"><span style="color: #333333;">❤️</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">❤️</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">❤️</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">❤️</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">❤️</span></td></tr><tr style="height: 19px;"><td style="width: 19.8837%; height: 19px;">0.2</td><td style="width: 20%; height: 19px;">0.2</td><td style="width: 20%; height: 19px;">0.2</td><td style="width: 20%; height: 19px;">0.2</td><td style="width: 20%; height: 19px;">0.2</td></tr></tbody></table>

아무런 특징이 없는 직선 통로에선, 로봇의 위치를 특정할 수 없습니다. 로봇은 어디에나 있을 수 있기 때문에 로봇의 위치를 확률로 나타내면 다섯 칸 모두 0.2의 확률을 가집니다. 일정하죠? 이렇게 일정한 확률을 uniform하다고 합니다.

<table style="border-collapse: collapse; width: 0%; height: 63px;" border="1"><tbody><tr style="height: 19px;"><td style="height: 19px; width: 19.8837%;"><span style="color: #333333;">🤖=<span style="color: #333333;">❤️</span></span></td><td style="width: 20%; height: 19px;"><span style="color: #333333;">Hit=0.6</span></td><td style="width: 20%; height: 19px;"><span style="color: #333333;">Miss=0.2</span></td><td style="width: 20%; height: 19px;"><span style="color: #333333;">&nbsp;</span></td><td style="width: 20%; height: 19px;"><span style="color: #333333;">&nbsp;</span></td></tr><tr style="height: 25px;"><td style="width: 19.8837%; height: 25px;"><span style="color: #333333;">💚</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">❤️</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">❤️</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">💚</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">💚</span></td></tr><tr style="height: 19px;"><td style="width: 19.8837%; height: 19px;">0.2 * 0.2 = 0.04</td><td style="width: 20%; height: 19px;">0.2 * 0.6 = 0.12</td><td style="width: 20%; height: 19px;">0.2 * 0.6 = 0.12</td><td style="width: 20%; height: 19px;">0.2 * 0.2 = 0.04</td><td style="width: 20%; height: 19px;">0.2 * 0.2 = 0.04</td></tr></tbody></table>

이번에는 위와 같이 색상을 측정할 수 있는 상황입니다. 로봇이 바닥에서 빨간색 하트 검출했습니다. 이때는 빨간 하트에 로봇이 있을 확률이 증가하고, 초록 하트에 존재할 확률도 미약하지만 존재합니다. 이는 로봇의 센서를 완전히 신뢰할 수 없기 때문입니다. 이렇게 측정 이후에 얻을 수 있는 확률을 posterior probability라고 합니다. 센서 측정 값과 일치하는 바닥은 0.6의 가중치를 곱하고, 다른 바닥은 0.2의 가중치를 곱했을 때 위와 같은 확률이 나옵니다.

그런데 위 표의 모든 확률을 합쳐도 1이 되지 않습니다. 이를 nonnormalized라고 하며, 합쳐서 1이 될 수 있도록 각각의 확률을 sum으로 나눠주겠습니다. 이를 renormalize 과정이라 합니다.

<table style="border-collapse: collapse; width: 100%; height: 69px;" border="1"><tbody><tr style="height: 19px;"><td style="height: 25px;"><span style="color: #333333;">🤖=<span style="color: #333333;">❤️</span></span></td><td style="height: 25px;"><span style="color: #333333;">Hit=0.6</span></td><td style="height: 25px;"><span style="color: #333333;">Miss=0.2</span></td><td style="height: 25px;"><span style="color: #333333;">&nbsp;</span></td><td style="height: 25px;"><span style="color: #333333;">&nbsp;</span></td></tr><tr style="height: 25px;"><td style="height: 25px;"><span style="color: #333333;">💚</span></td><td style="height: 25px;"><span style="color: #333333;">❤️</span></td><td style="height: 25px;"><span style="color: #333333;">❤️</span></td><td style="height: 25px;"><span style="color: #333333;">💚</span></td><td style="height: 25px;"><span style="color: #333333;">💚</span></td></tr><tr style="height: 19px;"><td style="height: 19px;">0.2 * 0.2 = 0.04</td><td style="height: 19px;">0.2 * 0.6 = 0.12</td><td style="height: 19px;">0.2 * 0.6 = 0.12</td><td style="height: 19px;">0.2 * 0.2 = 0.04</td><td style="height: 19px;">0.2 * 0.2 = 0.04</td></tr><tr><td>0.04 / 0.36 = 0.11</td><td>0.12 / 0.36 = 0.33</td><td><span style="color: #333333;">0.12 / 0.36 = 0.33</span></td><td><span style="color: #333333;">0.04 / 0.36 = 0.11</span></td><td><span style="color: #333333;">0.04 / 0.36 = 0.11</span></td></tr></tbody></table>

자 이제 로봇이 오른쪽으로 한 칸 이동한다고 생각해봅시다. 가장 오른쪽 하트가 맨 왼쪽 하트와 이어져있다고 가정합니다. 로봇의 이동 여부를 완벽하게 신뢰할 수 있으면, 각 셀의 확률을 한 칸씩 오른쪽으로 밀면 됩니다. perfect motion입니다.

<table style="border-collapse: collapse; width: 100%;" border="1"><tbody><tr><td style="width: 19.8837%;"><span style="color: #333333;">🤖</span></td><td style="width: 20%;"><span style="color: #333333;">Hit=0.6</span></td><td style="width: 20%;"><span style="color: #333333;">Miss=0.2</span></td><td style="width: 20%;"><span style="color: #333333;">Move Probability=1.0</span></td><td style="width: 20%;"><span style="color: #333333;">&nbsp;</span></td></tr><tr><td style="width: 19.8837%;"><span style="color: #333333;">💚</span></td><td style="width: 20%;"><span style="color: #333333;">❤️</span></td><td style="width: 20%;"><span style="color: #333333;">❤️</span></td><td style="width: 20%;"><span style="color: #333333;">💚</span></td><td style="width: 20%;"><span style="color: #333333;">💚</span></td></tr><tr><td style="width: 19.8837%;">0.11</td><td style="width: 20%;">0.11</td><td style="width: 20%;">0.33</td><td style="width: 20%;">0.33</td><td style="width: 20%;">0.11</td></tr></tbody></table>

만약 로봇이 한 칸보다 덜 이동하거나 (undershoot) 더 이동하는 상황(overshoot)이 발생할 확률이 각각 0.1이라면 어떻게 될까요?

<table style="border-collapse: collapse; width: 100%; height: 69px;" border="1"><tbody><tr style="height: 25px;"><td style="width: 19.8837%; height: 25px;"><span style="color: #333333;">🤖</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">Hit=0.6 / Miss=0.2</span></td><td style="width: 20%; height: 25px;">Move Probability=0.8</td><td style="width: 20%; height: 25px;"><span style="color: #333333;">Overshot=0.1</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">Undershoot=0.1</span></td></tr><tr style="height: 25px;"><td style="width: 19.8837%; height: 25px;"><span style="color: #333333;">💚</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">❤️</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">❤️</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">💚</span></td><td style="width: 20%; height: 25px;"><span style="color: #333333;">💚</span></td></tr><tr style="height: 19px;"><td style="width: 19.8837%; height: 19px;">0.11 * 0.8 + 0.11 * 0.1 + 0.11 * 0.1</td><td style="width: 20%; height: 19px;">0.11 * 0.8 + 0.11 * 0.1 + 0.33 * 0.1</td><td style="width: 20%; height: 19px;">0.33 * 0.8 + 0.11 * 0.1 + 0.33 * 0.1</td><td style="width: 20%; height: 19px;">0.33 * 0.8 + 0.33 * 0.1 + 0.11 * 0.1</td><td style="width: 20%; height: 19px;">0.11 * 0.8 + 0.33 * 0.1 + 0.11 * 0.1</td></tr><tr><td style="width: 19.8837%;">= 0.11</td><td style="width: 20%;">= 0.132</td><td style="width: 20%;">= 0.308</td><td style="width: 20%;">= 0.308</td><td style="width: 20%;">= 0.132</td></tr></tbody></table>

대략 이렇습니다. 제대로 이동할 확률 + overshoot 확률 + undershoot 확률로 계산합니다. 이 과정에서 확률이 분산되며 분포가 완만해지는 것을 볼 수 있습니다. (convolution) Localization은 sense와 move 과정을 지속적으로 반복합니다. sense를 통해 가중치를 곱해 신뢰도를 높이고, move 시 분포가 완만해지며 신뢰도를 잃습니다.

# Bayes Rule

Xi가 점유 격자 지도의 i번째 셀을 나타내고, Z가 측정 값을 나타낼 때, 측정 후 각 셀에 로봇이 존재할 확률을 베이즈 정리를 사용하여 나타내면 다음과 같습니다.

p(Xi | Z) = p(Z | Xi) \* p(Xi) / p(z)

여기서 p(Z | Xi)는 측정된 확률이며, p(Xi)는 사전 확률(prior)입니다. p(Z)는 모든 셀에 대해 같습니다. 더 자세히는 p(Z) = 𝚺p(Z | Xi) \* p(Xi)입니다. 왜냐하면 𝚺p(Xi | Z) = 1이기 때문이죠...

nonnormalized인 p\_(Xi | Z) = p(Z | Xi) \* p(Xi)로 표현하고, α = 𝚺p\_(Xi | Z)일 때, p(Xi | Z)는 다시 다음과 같이 나타낼 수 있습니다.

p(Xi | Z) = p\_(Xi | Z) / α

수업에서는 이해를 돕기 위해 다양한 예시 문제를 주는데요.

# Cancer Test

암 발병률이 0.001이고, 암 검사를 했을 때 실제 암이면 0.8의 확률로, 암이 아닐 때에도 0.1 확률로 양성 판정이 난다고 합니다. 양성 판정이 났을 때 암일 확률을 구해봅시다. 주어진 지문에서 구할 수 있는 확률은 다음과 같습니다.

p(cancer) = 0.001  
p(~cancer) = 0.999  
p(pos | cancer) = 0.8  
p(pos | ~cancer) = 0.1

p(cancer | pos) = ?

p(cancer | pos) = p(pos | cancer) \* p(cancer) / p(pos),  
p(pos) = p(pos | cancer) \* p(cancer) + p(pos | ~cancer) \* p(~cancer),  
즉, p(cancer | pos) = 0.8 \* 0.001 / (0.8 \* 0.001 + 0.1 \* 0.999) = 8 / 1007 = 0.0079...

# Coin Toss

동전을 던져서 앞면, 뒷면이 나올 확률이 같으며, 뒷면일 경우 결과를 그대로 반영, 앞면일 경우 동전을 한 번 더 던져 결과를 반영합니다. 동전을 던졌을 때 앞면이 나올 확률을 구해봅시다.

p(tail) = p(head) = 0.5  
p(head2) = p(head2 | head) \* p(head) + p(head2 | tail) \* p(tail)  
독립 시행이므로 p(head2 | head) = 0.5, 뒷면이 나올 경우 결과를 그대로 반영하니 p(head2 | tail) = 0이므로..

p(head2) = 0.5 \* 0.5 + 0 \* 0.5 = 0.25

# Coin Toss 2

앞뒷면이 나올 확률이 동일한 동전 fair와 앞면이 나올 확률 0.1, 뒷면 0.9인 동전 loaded가 있다. 동전을 무작위로 뽑아서 앞면이 나왔을 때 그 동전이 fair일 확률을 구하세요.

p(head | fair) = p(tail | fair) = 0.5  
p(head | loaded) = 0.1, p(tail | loaded) = 0.9  
p(fair) = p(loaded) = 0.5

p(fair | head) = ?

p(fair | head) = p(head | fair) \* p(fair) / p(head) = 0.5 \* 0.5 / p(head)  
p(head) = p(head | fair) \* p(fair) + p(head | loaded) \* p(loaded) = 0.5 \* 0.5 + 0.1 \* 0.5 = 0.3  
고로고로 p(fair | head) = 0.25 / 0.3 = 5 / 6