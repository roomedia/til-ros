# Motion Planning
![Motion Planning](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbOLxi4%2FbtqHXCKVTbQ%2FQwSi72eL3iPcq3NKQnQwdK%2Fimg.png)

Motion Planning은 로봇이 목적지로 향하는 데에 필요한 알고리즘입니다. 자율주행 자동차에 필요한 모션 플래닝의 형태는 다음과 같습니다.

![Motion Planning of Car](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FCv9QX%2FbtqHXDwkWsL%2FkAzcwKRyz2EUqygcTQuwvK%2Fimg.png)

출발점에서 도착점으로 가려면 우회전, 차선 변경, 좌회전을 해야겠지요. 너무 짧은 거리에서 차선을 변경하는 건 위험할 수 있으니 대안을 찾아봅시다.

![Alternative](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcjjTn6%2FbtqHWuNpP4q%2FD14QB9Y9UlFkKf987PTOtK%2Fimg.png)

오늘 얘기할 것은 바로 이러한 모션 플래닝 문제를 해결하는 방법입니다. 모션 플래닝 문제에서는 다음 세 가지가 주어집니다.

1. 지도
1. 시작점
1. 도착점
1. cost 함수

구해야 할 것은 하나, `Minimum Cost Path`입니다.

# Compute Cost
![지도](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F3Jroh%2FbtqHVnm0Dky%2FLZeaOAmvY4JVsKoZs1tqDK%2Fimg.png)

주어진 지도는 그리드 지도, 시작점과 도착점은 다음과 같다고 생각해봅시다. 각각의 시점에 우리는 직진하거나 회전할 수 있습니다. 직진하거나 회전할 때 1 단위 시간이 필요하다고 할 때, 출발점에서 도착점에 도달하려면 얼마의 시간이 필요할까요?

![문제 1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdCkZsr%2FbtqH2d4J1Mj%2F4M7FPBDFmQX2lgQnRhUx6k%2Fimg.png)

세 번 직진 후 좌회전, 이후 다시 세 번 직진하므로 7 단위 시간만큼의 시간이 소요됩니다.

![답안 1](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FmSnwO%2FbtqH4teZvF1%2FzI9hGCVkqLLk1SL13sDG10%2Fimg.png)

이번엔 다른 액션 모델을 사용해볼까요? 이번 액션 모델은 세 가지 액션을 가지고 있습니다.

1. 직진
1. 좌회전 후 직진
1. 우회전 후 직진

액션을 수행하는 데에 1 단위 시간이 걸린다면, 출발점에서 도착점에 도달하려면 얼마의 시간이 필요할까요?

![문제 2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FYKbwp%2FbtqHXDQGM41%2FARxpxlnJV8qmCu8wyctxI0%2Fimg.png)

이전에는 좌회전 후 직진이 2 단위시간 동안 이루어졌다면, 이번에는 해당 작업이 1 단위시간만에 이루어집니다. 따라서 정답은 6!

![답안 2](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FsktvL%2FbtqH4teZvIL%2FnotuJMOFEmmukYIcGKGW71%2Fimg.png)

# Optimal Path
액션 모델을 다시 변경해봅시다. 이번에는 좌회전 후 직진에 10의 단위시간이 소요된다고 가정하겠습니다. 실제 교통 상황에서, 좌회전은 우회전보다 하기 힘듭니다. 좌회전을 하기 위해선 종종 신호를 기다려야 하며, 따라서 좌회전이 우회전보다 더 많은 비용이 필요하다고 합시다. 이는 실제로 페덱스 등의 택배 회사에서 적용하고 있는 규칙입니다.

![문제 3](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdbKAF6%2FbtqH49AGysS%2FNDor1XLR28nWFSNkZ0yRkK%2Fimg.png)

이번에는 비교를 해봐야 합니다. 좌회전을 통해 도착점에 갈 때 소요되는 시간은 15입니다. 오른쪽을 크게 도는 경로의 비용은 16입니다. 아직까지는 좌회전이 효율이 더 좋네요.

![답안 3](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FdDDTai%2FbtqHTZ0TLIi%2F166KTFB9DHB4RUhDn2NuQK%2Fimg.png)

다음과 같이 좌회전 비용이 20이 되면 어떨까요?

![문제 4](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FRXJIQ%2FbtqHZkwGjQQ%2FFz8AbxVPHk184xCBPZxjxk%2Fimg.png)

가능한 다른 경로가 16의 비용으로 도착점에 도달할 수 있으니, 이를 선택하면 됩니다.

![답안 4](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbClZNO%2FbtqHZlhTovx%2FPO5kGhcNPDrLCHrTMhX5k0%2Fimg.png)

이번에는 문제를 조금 바꿔서, 상하좌우로 움직일 수 있는 로봇이 다음과 같은 미로를 통과하는 데에 얼마의 시간이 걸릴지 계산해봅시다.

![문제 5](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FoDeHe%2FbtqH3CXCauj%2FrkUkZMhSyTxkFf3jDIgc1k%2Fimg.png)

칸이 7칸이므로 답은 7이 됩니다.

![답안 5](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FsYN62%2FbtqHZIK67MQ%2FSYRaHnE6kGajNXHuhveR0k%2Fimg.png)

이번에는 미로를 조금 확장해봅시다. 상하좌우로 움직이는 로봇이 6x5 미로의 S에서 출발하여 G로 가려 할 때, 얼마나 많은 액션을 취해야 할까요?

![문제 6](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fca2DG3%2FbtqHXEaYk3r%2FBIQmEhuttpJSg8njtk6l1K%2Fimg.png)

2 Down + 3 Right + 1 Up + 2 Right + 3 Down = 11 Actions이 됩니다.

![답안 6](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbPQVQT%2FbtqH88nI1dj%2F5zcpgvevuQakvvaceDyHI0%2Fimg.png)

여기서 드는 의문은, 시작점에서 도착점까지 최단 경로를 찾아낼 수 있는가? 입니다. 강의에서는 최단 경로를 찾기 위해 다음과 같은 지시사항을 줍니다.

1. 현재 점에서 이동 가능한 점을 체크합니다.
1. 해당 점들의 g-value는 현재 점의 g-value + 1입니다. (이는 나중에 경로의 길이로 사용됩니다.)
1. 이미 체크된 점으로는 이동할 수 없습니다.
1. 단계 별로 이를 반복합니다.
1. 도착점에 도달하면 종료합니다.

즉, BFS를 사용하라는 얘기입니다.

![문제 7](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FxX7rC%2FbtqHXC5bMXM%2FFKfSxyseVK7hyHuezRlKfK%2Fimg.png)

주어진 템플릿 코드는 다음과 같습니다.

```py
# ----------
# User Instructions:
# 
# Define a function, search() that returns a list
# in the form of [optimal path length, row, col]. For
# the grid shown below, your function should output
# [11, 4, 5].
#
# If there is no valid path from the start point
# to the goal, your function should return the string
# 'fail'
# ----------

# Grid format:
#   0 = Navigable space
#   1 = Occupied space


grid = [[0, 0, 1, 0, 0, 0],
        [0, 0, 1, 0, 0, 0],
        [0, 0, 0, 0, 1, 0],
        [0, 0, 1, 1, 1, 0],
        [0, 0, 0, 0, 1, 0]]
init = [0, 0]
goal = [len(grid)-1, len(grid[0])-1]
cost = 1

delta = [[-1, 0], # go up
         [ 0,-1], # go left
         [ 1, 0], # go down
         [ 0, 1]] # go right

delta_name = ['^', '<', 'v', '>']

def search(grid,init,goal,cost):
    # ----------------------------------------
    # insert code here
    # ----------------------------------------
    
    return path

##### Do Not Modify ######

import grader

try:
    response = grader.run_grader(search)
    print(response)    
    
except Exception as err:
    print(str(err))

##### SOLUTION: Run this cell to watch the solution video ######
from IPython.display import HTML
HTML('<iframe width="560" height="315" src="https://www.youtube.com/embed/cl8Kdkr4Gbg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>')
```

작성한 코드는 다음과 같습니다.

```py

def search(grid, init, goal, cost):
    y, x = init
    g = 0
    
    closed = [[0 for _ in range(len(grid[0]))] for _ in range(len(grid))]
    closed[y][x] = 1
    open = [[g, y, x]]

    found = False
    resign = False
    while found is False and resign is False:
        if len(open) == 0:
            resign = True
            raise Exception("fail")

        open.sort(reverse=True)
        g, y, x = open.pop()
        
        if [y, x] == goal:
            found = True
            return [g, y, x]

        for i in range(len(delta)):
            y2 = y + delta[i][0]
            x2 = x + delta[i][1]
            
            if y2 < 0 or y2 >= len(grid) or x2 < 0 or x2 >= len(grid[0]):
                continue
                
            if closed[y2][x2] == 0 and grid[y2][x2] == 0:
                g2 = g + cost
                open.append([g2, y2, x2])
                closed[y2][x2] = 1
            
    raise Exception("fail")
```

해설은 너무 길기에 링크로 남겨놓겠습니다.

https://www.youtube.com/embed/cl8Kdkr4Gbg