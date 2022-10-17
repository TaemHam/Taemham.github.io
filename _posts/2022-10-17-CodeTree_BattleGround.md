---
title: "[알고리즘|파이썬] 코드트리: 싸움땅"
author: TaemHam
date: 2022-10-17 17:40:00 +0900
categories: [Algorithm, CodeTree]
tags: [Algorithm, Code_Tree, Python, Pypy, 삼성, 역테, 싸움땅]
---

## **문제**
***
[[원문 링크](https://www.codetree.ai/frequent-problems/battle-ground/description)]

## **해설**
***

과정들을 그대로 구현만 해주면 되는 간단한 문제였다.

### 초기화

```python
from heapq import heapreplace, heappush, heappop

N, M, K = map(int, input().split())
L = N+1
guns = [[] for _ in range(L*L)]
board = [0] * (L*N) + [1] * L

for x in range(0, L*N, L):
    board[x+N] = 1
    line = map(int, input().split())
    for xy, gun in enumerate(line, x):
        if gun:
            guns[xy].append(-gun)
```

N, M, K 는 문제에서 주어지는 [맵 크기, 플레이어 숫자, 게임이 진행되는 턴 수] 이다.

L은 [2차원 배열을 1차원 배열로 만들기](https://taemham.github.io/posts/Algo_2Dgraph/)위해 선언했다.

guns 배열은 땅에 떨어져 있는 총의 배열을 위해 선언했는데, 가장 큰 총을 빠르게 찾기 위해 최대힙 으로 구현해주었다.

board 배열은 플레이어가 어디에 존재하는지 확인하기 위한 용도인데, 0이면 빈칸, 1이면 격자의 바깥, 그리고 플레이어의 위치마다 다음에 후술할 Player 클래스의 인스턴스들을 넣어주었다.

### Player 클래스

```python

class Player:
    
    order = []
    delta = (-L, 1, L, -1)
    guns_heap = guns

    def __init__(self, x, y, d, s):
        self.coord = (x-1) * L + (y-1)
        self.direc = d
        self.stat = -s
        self.gun = 0
        self.score = 0

        Player.order.append(self)

    def move(self):
        next_coord = self.coord + Player.delta[self.direc]
        self.direc = (self.direc + 2) % 4 if board[next_coord] == 1 else self.direc
        self.coord = self.coord + Player.delta[self.direc]

    def compare(self):
        if not Player.guns_heap[self.coord]:
            return
        elif not self.gun:
            self.gun = heappop(Player.guns_heap[self.coord])
        elif self.gun > Player.guns_heap[self.coord][0]:
            self.gun = heapreplace(Player.guns_heap[self.coord], self.gun)
    
    def fight(self, another_player):
        winner, loser = sorted([self, another_player], key = lambda x: (x.stat + x.gun, x.stat))
        score = winner.stat + winner.gun - loser.stat - loser.gun
        loser.lose()
        winner.win(score)
        return (loser, winner)
    
    def lose(self):

        if self.gun:
            heappush(Player.guns_heap[self.coord], self.gun)
        self.gun = 0

        for d in range(4):
            next_direc = (self.direc + d) % 4
            next_coord = self.coord + Player.delta[next_direc]
            if board[next_coord]:
                continue

            self.direc = next_direc
            self.coord = next_coord
            self.compare()
            return

    def win(self, score):

        self.score += score
        self.compare()

for _ in range(M):
    x, y, d, s = map(int, input().split())
    Player(x, y, d, s)

for player in Player.order:
    board[player.coord] = player

```

가독성을 위해 플레이어를 클래스로 만들고, 플레이어가 하는 행위들은 모두 메서드로 추가해주었다. 플레이어가 생성되면 정적변수 `Player.order`배열에 추가해서, for 문을 돌리며 순서대로 행동하도록 했다. 

#### 생성자 (`def __init__(self, x, y, d, s):`)

생성자 매개변수로 입력에 주어지는 x, y, d, s를 받으면 x와 y는 1차원 배열의 좌표로 바꿔 `self.coord`에 저장하고, 방향은 그대로 `self.direc`에, 기본 공격력은 총에 사용되는 최대힙 반영을 위해 음수로 바꿔 `self.stat`에 저장해 주었다.

`self.gun`에는 총이 없을 시 0, 있으면 해당 공격력으로 저장해주었고, `self.score`에는 점수를 저장해 주었다.

#### 이동 (`def move(self):`)

메서드가 호출되면 가지고 있던 좌표와 방향에 따라 이동시켰다.

만약 이동할 좌표의 board 배열이 1 이라면, 해당 칸은 격자 바깥이라는 의미, 따라서 방향을 거꾸로 돌리고 해당 방향으로 이동시켰다.

#### 비교 (`def compare(self):`)

해당 플레이어가 위치한 곳의 좌표에서 총 배열을 확인해주었다.

1. 만약 해당 위치에 총이 없다면 넘어가고,
2. 플레이어가 지닌 총이 없다면 최대힙의 0번째 총을 장착(`heappop`)시키고,
3. 플레이어가 지닌 총이 땅에 있는 총보다 약하다면, 바꿔(`heapreplace`)주었다.

#### 싸움 (`def fight(self, another_player):`)

다른 플레이어와 위치가 겹쳤는지에 대한 확인은 클래스 밖에서 할 예정이니 넘어간다.

일단 다른 플레이어와 위치가 겹쳤으면 해당 플레이어의 인스턴스를 board에서 가져와 확인해준다.

승자와 패자에 대한 판정은 `sorted` 함수로 key를 (기본 공격력 + 총, 기본 공격력) 순서로 확인하도록 했다. 현재 공격력은 모두 음수이므로, 합이 더 작은 쪽이 승자다.

진 사람은 후술할 `lose` 메서드를 실행시키고, 이긴 사람은 점수를 계산해 `win` 메서드를 실행시켰다.

#### 짐 (`def lose(self):`)

1. 진 사람은 해당 칸에서 총을 내려놓아야 하므로, gun 배열에서 해당하는 좌표의 최대힙에 총을 추가해주고 자신의 총은 0으로 초기화 시킨다.

2. for문으로 사방을 확인하면서 벽이나 다른 플레이어가 있는지 (board의 해당 좌표가 0이 아니면 된다) 확인하고 이동, 있다면 continue해서 다음 방향을 확인해준다.

    * 이동할 장소를 발견하면 방향과 좌표를 업데이트 시키고, 총을 줍는 `compare` 메서드를 실행시킨다.

#### 이김 (`def win(self, score):`)

1. 매개변수로 받은 공격력 차이를 `self.score`에 더해주고,

2. 총을 줍는 `compare` 메서드를 실행시킨다.

#### 초기화

이후는 입력을 받아 Player 인스턴스를 생성해주고, board 배열에 플레이어 위치마다 인스턴스를 넣어주었다.

### 시뮬레이션

```python

for turn in range(K):
    for player in Player.order:
        board[player.coord] = 0
        player.move()

        if board[player.coord]:
            loser, winner = player.fight(board[player.coord])
            board[loser.coord] = loser
            board[winner.coord] = winner
        else:
            player.compare()
            board[player.coord] = player

```

K 만큼 턴을 돌면서 `Player.order`에서 플레이어를 하나씩 꺼내가며 시뮬레이션을 돌렸다.

`board[player.coord]` 를 0으로 초기화 하지 않으면, loser가 다음 위치를 찾을 때 해당 플레이어 때문에 위치를 못찾을 수 있다.

### 정답 출력

```python
answer = []
for player in Player.order:
    answer.append(str(-player.score))

print(' '.join(answer))
```

answer 배열을 만들어 음수로 되어있는 점수를 양수로 바꿔 배열에 저장해주었다.

마지막으로 출력하면 된다.