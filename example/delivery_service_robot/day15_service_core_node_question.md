# 서비스 코어 노드
서비스 코어 노드는 로봇 한 대 당 2개의 subscriber, 2개의 publisher를 가집니다. 각각의 pub/sub 항목은 다음과 같습니다.

항목 | 설명
---|---
pubServiceStatusPad | 패드에 배달 서비스 상황 publisher
pubPlaySound | 녹음 음성 파일의 위치 publisher
subPadOrder | 패드로부터 고객의 주문 subscriber
subArrivalStatus | 로봇의 도착 여부 subscriber

으음.. 사실 납득이 잘 가진 않습니다.
요구사항 | 메시지 타입
---|---
1. 태블릿으로부터 주문 받아 | service server
2. 로봇이 이동할 좌표를 지속적으로 전달하고 | publisher
3. 로봇의 이동 상황을 지속적으로 받아 | subscriber
4. 태블릿에 물건 받았을 때/배달 완료했을 때 전달한다 | service client
정도로 이해하고 있었거든요. 그래서 직접 짜보기로 했습니다.