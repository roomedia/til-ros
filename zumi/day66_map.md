# 레퍼런스
![레퍼런스](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fx0orU%2FbtqH8dbu6mK%2FYjmoni3w4qusm0OdutBa20%2Fimg.jpg)

# 러프
![스케치](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FXgIF8%2FbtqHWwElhqi%2Fl7numVumVPMCZP94K3PQg0%2Fimg.jpg)

# 주미 회전 코드
```py
from zumi.zumi import Zumi
import time

zumi = Zumi()

for i in range(6):
    zumi.turn_right(30)
    
for i in range(6):
    zumi.turn_left(30)
```

![![회전](https://t1.daumcdn.net/thumb/C640x360.q50.fjpg/?fname=http%3A%2F%2Fthumb.kakaocdn.net%2Fdna%2Fkamp%2Fsource%2Frv0d3dv7l4ofz7mbetoqeydr3%2Fthumbs%2Fthumb.jpg%3Fcredential%3DTuMuFGKUIcirOSjFzOpncbomGFEIdZWK%26expires%3D33156216320%26signature%3DmP6kEMEJseUY0HAMLuWmw5Ccn1M%253D)](https://play-tv.kakao.com/embed/player/cliplink/412158498?service=player_share)

회전 후 딜레이가 있는 걸 보니 저런 곡선형 도로를 주행하지 못할 것 같습니다. 지도를 좀더 직선형으로 만들어봅시다.

# 스켈레톤
![skeleton](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FejJEpM%2FbtqHT1j8VOw%2FDywJP2tPPJWzCHXf4gX9R1%2Fimg.jpg)

커다란 지도를 만들기 위해 위 이미지를 16장으로 분할하여 사용합니다. 이렇게 보니 번호를 집어넣어야 할 것 같네요.

![1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdk3KBp%2FbtqH8bEKJMf%2FhwCalK47CBFS86eSHXa6zK%2Fimg.jpg)

![2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FBiG3h%2FbtqH2cxPFTd%2FdPrODAZfxzhtJOqkupPm3K%2Fimg.jpg)

![3](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F6c4AO%2FbtqH8bY3rva%2FTGisUIoCK0v8lFonUHHau0%2Fimg.jpg)

![4](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FQtkAi%2FbtqH4tlDUPR%2F7JyuG7TNXe0O1XvVul6IOk%2Fimg.jpg)

![5](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FQtkAi%2FbtqH4tlDUPR%2F7JyuG7TNXe0O1XvVul6IOk%2Fimg.jpg)

![6](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FKmm9K%2FbtqHZJiPKZR%2FkLFj4H7Vl7ewCkt7xJktw1%2Fimg.jpg)

![7](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FmSbZM%2FbtqHYIkaIZ8%2FCF4kS9onfFLTDGQkS5SkbK%2Fimg.jpg)

![8](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fv3kAK%2FbtqH0Wbaw5Y%2FDuUUyOJCpQwzjVlWdZYZx0%2Fimg.jpg)

![9](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdUhTn4%2FbtqHTZNmK6E%2F27ifvRDxbDvMJufPubZEtK%2Fimg.jpg)

![10](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FLmaOs%2FbtqHXDQCGTm%2FQtt4UKPYq3QeanTp0dQqAK%2Fimg.jpg)

![11](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbniarz%2FbtqH2c5NuQ2%2Fwi56AK9jE4DKEAy2OVh0U0%2Fimg.jpg)

![12](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcDOGMs%2FbtqHZIK1KLb%2FCFSk7bR45Lyjptf3ek1Sr0%2Fimg.jpg)

![13](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbNCj0f%2FbtqHT0ZTxvI%2F7awND1W7zymZZL2IAPkhK1%2Fimg.jpg)

![14](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fd2429F%2FbtqH8bxYXRq%2FP8BIyGnUPXtUSIcFqhfsK1%2Fimg.jpg)

![15](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbMcuaZ%2FbtqH4koPuod%2FGvdULO752MQo0RxNMC5FGk%2Fimg.jpg)

![16](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FWRL9X%2FbtqHT0S25Uz%2FkRoKKnI5fTrmkdnxxYaBkK%2Fimg.jpg)

# 시야 측정
![원거리](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FsyfoK%2FbtqHT1xKodC%2Fpe39IsP1O4wpBNXLaNrlf1%2Fimg.png)

![근거리](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdzDhJj%2FbtqHYIRZvUt%2FreFLO6KdMPw0F1QGPfqA1k%2Fimg.png)

주미의 카메라가 전방을 향하고 있기 때문에, 대략 14cm보다 가까이 있는 도로는 보이지 않았습니다.

# 회전 시 프레임 잔상

![![동영상](https://tv.kakao.com/v/412157732)](https://t1.daumcdn.net/thumb/C640x360.q50.fjpg/?fname=http%3A%2F%2Fthumb.kakaocdn.net%2Fdna%2Fkamp%2Fsource%2Frv0uahq5d7c0gtfd4hsm25f6b%2Fthumbs%2Fthumb.jpg%3Fcredential%3DTuMuFGKUIcirOSjFzOpncbomGFEIdZWK%26expires%3D33156214489%26signature%3DY1OHKXBz03%252FYG1zvlxZ0wb6oyB4%253D%26ts%3D1599305689)

주행 모드에서 카메라를 열고 한 바퀴 회전했을 때, FPS가 너무 낮아 잔상이 너무 심했습니다.
