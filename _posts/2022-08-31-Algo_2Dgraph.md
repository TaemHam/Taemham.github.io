---
title: "[알고리즘|파이썬] 2차원 그래프 탐색을 빠르게 돌리는 법"
author: TaemHam
date: 2022-08-31 16:32:00 +0900
categories: [Algorithm, Source Code]
tags: [Algorithm, Python, Pypy, Graph Search, DFS, BFS, Matrix]
render_with_liquid: false
---

## 개요
***
백준 사이트에서 알고리즘 문제를 풀면서 생겼던 의문이 있었다.

```python
graph2D[x][y] = 0   # 인덱스 x와 y에서 각각 한 번씩, 총 두 번 호출
graph1D[xy] = 0     # 인덱스 xy에서 한 번 호출
```

위와 같이 2차원 그래프 `graph2D`에서 x행 y열의 원소를 접근 할 때, x 인덱스와 y 인덱스 두 번을 호출 하는 것보다,

2차원 그래프를 일렬로 나열해 놓은 1차원 그래프 `graph1D`에서 xy 인덱스로 한 번에 호출 하는 것이 더 빠르지 않을까?

결론부터 말하자면, **'그렇다'** 이다. 아래의 예시코드를 보자.

```python
import time

# 세로 크기 R, 가로 크기 C의 2차원 그래프에서 
# 왼쪽 위칸에서 시작해 모든 칸을 탐색하며
# 방문 순서에 따라 숫자를 붙여주는 BFS 그래프를 만든다.
R, C = 1000, 1000

# 2차원 그래프에서 BFS를 돌리기 위한 준비물
graph2D = [[0 for _ in range(R)] for _ in range(C)]
visit2D = [[0 for _ in range(R)] for _ in range(C)]
dx = (1, -1, 0, 0)
dy = (0, 0, 1, -1)
que2D = [(0, 0)]

# 1차원 그래프에서 BFS를 돌리기 위한 준비물
L = C+1
graph1D = [0] * R*L
visit1D = [0] * R*L + [1] * L
visit1D[C:R*L:L] = [1] * R
dxy = (1, -1, L, -L)
que1D = [0]

# 2차원 그래프 BFS
ini2D = time.time()

visit2D[0][0] = 1
graph2D[0][0] = 1
for x, y in que2D:
    for i in range(4):
        nx = x + dx[i]
        ny = y + dy[i]
        if 0 <= nx < R and 0 <= ny < C and not visit2D[nx][ny]:
            visit2D[nx][ny] = 1
            que2D.append((nx, ny))
            graph2D[nx][ny] = len(que2D)

fin2D = time.time()

# 1차원 그래프 BFS
ini1D = time.time()

visit1D[0] = 1
graph1D[0] = 1
for xy in que1D:
    for d in dxy:
        if not visit1D[xy+d]:
            visit1D[xy+d] = 1
            que1D.append(xy+d)
            graph1D[xy+d] = len(que1D)

fin1D = time.time()

print(f'2차원 그래프 BFS 실행 시간: {fin2D - ini2D} sec')
print(f'1차원 그래프 BFS 실행 시간: {fin1D - ini1D} sec')

# 출력:
# 2차원 그래프 BFS 실행 시간: 1.1490559577941895 sec
# 1차원 그래프 BFS 실행 시간: 0.3833479881286621 sec

# # 주석 해제 후 실행하면 결과물이 동일함을 확인할 수 있다.
# # 주의: R과 C를 10으로 낮추고 실행하자.
#  
# print('2차원 그래프 BFS 결과')
# print(*graph2D, sep='\n')
# 
# print('1차원 그래프 BFS 결과')
# for r in range(0, R*L, L):
#     print(graph1D[r:r+C])
```

출력 결과를 보면 두 배 이상의 시간 차이가 나는 것을 볼 수 있다. 물론 위의 차이는 2차원 그래프 직렬화만으로 나온 것은 아니다. if 문을 간소화 시키기 위해, 직렬화를 하는 동시에 방문 배열에 간단하게 전처리를 한 것도 한 몫 했을 것이다.

이 글에서는 어떻게 하면 2차원 그래프를 1차원으로 바꾸어 처리할 수 있는지 그 방법에 대해 다루려고 한다.

## 입력
***

입력은 백준 문제를 푸는 것 처럼, 표준 입력을 받아 처리한다고 가정한다. 

입력을 받기 전에 정해야 할 것은 2차원 그래프의 마진 칸 수를 정하고, 마진을 포함한 칸수의 그래프를 미리 선언해 두는 것이다.

<img src="https://user-images.githubusercontent.com/95671168/187602383-ec4d8368-779f-4c9c-b9d9-56c93ac336f7.jpg" width="50%" height="50%"/>

마진이 필요한 이유는 각 행의 우측 끝 칸과 그 다음 행의 왼쪽 끝 칸이 인접한 것으로 판단되는 것을 방지하기 위함이다. 위 그림에서의 1번 칸과 3번 칸이 서로 인접하지 않도록 2번칸을 넣어주는 것과 같다. 마진은 보통 1칸으로 설정해두면 되지만, [1726번: 로봇](https://www.acmicpc.net/problem/1726)문제 처럼 한 번에 여러 칸을 이동하는 경우, 그 칸 수만큼 설정해야 할 수 있다.

마진에 어떤 값을 넣느냐에 따라 탐색시의 if 문을 간소화 시킬 수 있다. 마진을 탐색하는 경우는 **2차원 그래프 바깥으로 나가는 경우**이므로, 문제에 주어지는 벽, 장애물 등의 탐색 제외 값을 **입력 배열의 마진**에 넣어주거나, 해당 칸을 미리 방문 한 것처럼 방문 값을 **방문 배열의 마진**에 넣어주면, `0 <= nx < R and 0 <= ny < C` 조건을 넣지 않아도 걸러낼 수 있다. 두 방법 중 어느 걸 택할지는 문제 조건을 보고 가능한 방법을 선택하면 된다.

이렇게 하면 각 칸의 좌표 값은 열값(`y`)에 좌표의 행값 과 마진을 포함한 열의 칸수의 곱(`x*L`)을 더한 값이 된다. 이 상태에서 for문을 사용해 마진칸을 제외하고 입력을 받으려면 `for x in range(0, R*L, L):` 에 `graph[x:x+C] = map(int, input().split())` 을 사용하면 된다. 만약 문제 조건에 의해 모든 칸을 한 번씩 탐색해야 할 때는, 앞의 for문에 이중 포문으로 `for xy in range(x, x+C):` 를 더해 나오는 좌표값 xy를 탐색하면 된다. 

아래는 [7576: 토마토](https://www.acmicpc.net/problem/7576)문제로 작성한 예시이다. 해당 문제의 경우 입력 배열 자체를 방문 배열로 사용 가능하기 때문에, 방문 배열을 선언하지 않아도 되지만, 설명을 위해 선언한 예시를 보여주겠다.

```python
import sys
input = sys.stdin.readline
C, R = map(int, input().split())
L = C+1
que = []
notRipe = 0

# 입력 배열 마진에 탐색 제외값을 넣는 방법
visit = [0] * R * L
graph = [-1] * (R+1) * L 
for x in range(0, R*L, L):
    graph[x:x+C] = map(int, input().split())
    for xy in range(x, x+C):
        if graph[xy] == 1:
            que.append(xy)
        elif graph[xy] == 0:
            notRipe += 1

# 방문 배열 마진에 방문 값을 넣는 방법
visit = [0] * R * L + [1] * L
visit[C:R*L:L] = [1] * R 
graph = [0] * R * L
for x in range(0, R*L, L):
    graph[x:x+C] = map(int, input().split())
    for xy in range(x, x+C):
        if graph[xy] == 1:
            que.append(xy)
        elif graph[xy] == 0:
            notRipe += 1
```

## 탐색
***

탐색은 오히려 더 간단해진다.

일반적인 2차원 그래프 탐색은 델타 값으로 `dx = (0, 0, 1, -1); dy = (1, -1, 0, 0)` 을 선언하고, 탐색시 if 문에 `0 <= nx < R and 0 <= ny < C` 같은 조건을 넣어야 하지만, 직렬화 시켰다면 `dxy = (1, -1, L, -L)` 으로 델타값만 선언하면 된다. 격자 바깥으로 나가는 경우는 입력받을 때 걸러지기 때문이다.

```python
dxy = (1, -1, L, -L)
for time in range(0, R*C):
    if notRipe == 0:
        break

    nextQue = []
    for xy in que:
        for d in dxy:
            if graph[xy+d] == 0 and visit[xy+d] == 0:
            # 입력 배열의 마진을 수정했으면 앞에서 걸러지고,
            # 방문 배열의 마진을 수정했으면 뒤에서 걸러진다.
                visit[xy+d] = 1
                nextQue.append(xy+d)

    if not nextQue:
        time = -1
        break
    
    notRipe -= len(nextQue)
    que = nextQue

print(time)
```

<details>
<summary>예시 전체 코드 및 결과</summary>

```python
def main():
    C, R = map(int, input().split())
    L = C+1
    que = []
    notRipe = 0

    visit = [0] * R * L
    graph = [-1] * (R+1) * L 
    for x in range(0, R*L, L):
        graph[x:x+C] = map(int, input().split())
        for xy in range(x, x+C):
            if graph[xy] == 1:
                que.append(xy)
            elif graph[xy] == 0:
                notRipe += 1
                
    dxy = (1, -1, L, -L)
    for time in range(0, R*C):
        if notRipe == 0:
            break

        nextQue = []
        for xy in que:
            for d in dxy:
                if graph[xy+d] == 0 and visit[xy+d] == 0:
                    visit[xy+d] = 1
                    nextQue.append(xy+d)

        if not nextQue:
            time = -1
            break
        
        notRipe -= len(nextQue)
        que = nextQue

    return time

if __name__ == "__main__":
    print(main())
```

![7576_result](https://user-images.githubusercontent.com/95671168/187618081-78304101-ed70-40d2-827c-3f67820a9d5f.PNG)

</details>