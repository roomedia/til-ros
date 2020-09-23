# 데이터 저장 위치 변경
`/home/pi/Dashboard/DriveImg`에 데이터를 저장할 경우, 이미지의 내용을 확인하기 힘들었습니다. 주피터 노트북으로 이미지를 확인하기 위해 root 경로인 `/home/pi/Dashboard/user/{사용자 이름}/DriveImg`로 데이터 저장 위치를 옮겼습니다.

# 지도
지도는 임시로 로보링크에서 제공해준 지도를 수정하여 사용하였습니다. 횡단보도? 같은 하얀 선을 삭제한 형태입니다. A4 용지 4x3이면 얼추 사이즈가 맞더라구요 ㅎ.ㅎ

# 데이터 검수
수집한 데이터가 완벽했으면 좋겠지만, 중간중간 조종을 잘못한 부분도 있고, 세밀한 조종이 불가능하여 생기는 오류도 있습니다. 이러한 부분을 제거하여야 주미가 학습할 때 도움이 됩니다. 주피터 노트북을 이용하여 우리의 데이터를 확인해봅시다.

```python
import os
import cv2
import IPython.display
from PIL import Image

save_path = '../../DriveImg'
jpg_path = os.path.join(save_path, '%s.jpg')
number_txt = os.path.join(save_path, 'number.txt')
direction_txt = os.path.join(save_path, 'direction.txt')

with open(direction_txt, 'r') as f:
    datum = f.readlines()
    
for data in datum:
    n, direction = data.split(' ')
    frame = cv2.imread(jpg_path % n)
    IPython.display.display(Image.fromarray(frame))
    print(n, direction)
```

![예시](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlhP9n%2FbtqJs5Q6csv%2FDq0dkNr8Y4HqVnJ0AHIuIK%2Fimg.png)

다음과 같이 이미지와 방향이 표시됩니다. 눈으로 확인하고 잘못 체크된 부분들을 찾아 [http://192.168.0.34:5555/tree/DriveImg](http://192.168.0.34:5555/tree/DriveImg)에서 일일히 삭제하였습니다. 

![급발진](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Ftdzfy%2FbtqJptk7cDi%2FuRnW8Nsrk611b4zl7zvtZ0%2Fimg.png)

# 라벨 삭제
삭제된 이미지의 라벨도 `direction.txt`에서 삭제하도록 하겠습니다. 새로운 셀을 만들어 다음 코드를 사용하였습니다.

```python
with open(direction_txt, 'r') as f:
    datum = f.readlines()

new_datum = []
imgs = sorted([f.split('.')[0] for f in os.listdir(save_path) if 'jpg' in f])

for idx, data in enumerate(datum):
    d = data.split(' ')[0]
    if d in imgs:
        new_datum.append(data)

with open(direction_txt, 'w') as f:
    f.writelines(new_datum)
```

![ex](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fbad5EA%2FbtqJluSo7oB%2FwDN7bvQGClCVj4YE1VQ03k%2Fimg.png)

![ex](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F7JDaR%2FbtqJs5jfBBd%2FkyMAyZDCkc3ZmtupDxXtxK%2Fimg.png)