# Zumi를 이용한 보드게임? Zumi Heroes 소개
![zumi heroes](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FboLVa3%2FbtqHelbUJuF%2FMZ4UKU4H8sWgW05BgEubw0%2Fimg.jpg)

주미 파워 유저에 선정되어 주미를 제공받고, 콘텐츠를 제작하는 임무!!를 받았습니다. 저는 주미와 보드게임을 더한 주미 히어로즈를 개발하려고 하는데요. 주미 히어로즈는 보드게임에 주미를 더해 실시간성, 몰입성을 높인 작품입니다.

![기획의도](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FHPVT1%2FbtqHq1itbaR%2Fuse0FYKStkf3NIfkE9lTXK%2Fimg.jpg)

플레이어들은 히어로가 되어 주미와 함께 악당과 맞서 싸웁니다. 플레이어 간 협력, 플레이어와 주미의 상호작용을 통해 협동심을 기르고, 자율주행과 인공지능 개념을 자연스럽게 배우도록 하는 것이 게임의 목표입니다.

![게임준비](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdhOWyZ%2FbtqHbY2oND5%2FxDj7ptFKKPq9YBVXkSfLtK%2Fimg.jpg)

현재까지 기획한 게임의 규칙은 다음과 같습니다. 먼저 카드를 다음과 같이 배치해주세요. 각 플레이어는 히어로 카드 뭉치에서 카드를 두 장씩 뽑아 가집니다.

![게임시작](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbaargD%2FbtqHcekgwwu%2FW6o3PVDNG8qOLswhmW1Uw1%2Fimg.jpg)
  
플레이어는 자신의 차례가 되면 히어로 카드를 한 장, 장소 카드를 두 장 뽑습니다. 장소 카드는 뽑은 순서대로 히어로 존과 빌런 존에 놓습니다. 빌런 존 장소 카드에 적힌 위치에 빌런 토큰을 놓습니다. 히어로 존 장소 카드에 적힌 위치에 자신이 지닌 히어로 카드 중 한 장을 놓습니다. 히어로는 저마다 능력을 갖고 있으며, 히어로를 적재적소에 히어로를 배치해야만 빌런을 물리칠 수 있습니다. 그런데... 주미가 빠졌네요?

![기획의도](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fem0PUP%2FbtqHiMfYUOD%2F4G4KuhlRtCkKPzlkQeJXg0%2Fimg.jpg)
  
추가된 룰의 핵심은 Zumi에게 카드를 보여준다는 점입니다. Zumi에게 출발점, 도착점이 될 장소 카드를 보여주면 이동합니다. 이외에도 히어로 효과로 주미의 속도를 높인다던지 늦게 도착하면 패널티로 잠시 간 수동 조종으로 변경한다던지 하면 재미있을 것 같다는 생각이 듭니다.

![기획의도](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb4kFwx%2FbtqHgpSNpDf%2FajVBOXC1X7uNRjvxKq2JY0%2Fimg.jpg)
  
승리 조건은 다음과 같습니다. 지도에 놓인 빌런 토큰이 모두 없어질 때까지 플레이해야 하며, 더이상 뽑을 장소 카드가 없어도 승리합니다. 반대로 더이상 놓을 빌런 토큰이 없으면 패배합니다.

![기획의도](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlafGG%2FbtqHq2uUDOi%2FTflsKw0f3fkCiELiyKY9Z0%2Fimg.jpg)
  
작품을 구현하기 위해 필요한 작업 목록입니다. 주미를 이용한 자율주행, 물체인식 등의 작업이 필요하며, 리소스는 오픈소스 도트 이미지를 사용하여 작업할 생각입니다.