---
title: "선택트리(Selection Tree)"
date: "2021-05-08 19:37:00 +0900"
categories:
    - 자료구조

tags:
    - [자료구조]

toc: true
---

선택트리는 **승자트리(Winner Tree)**와 **패자트리(Loser Tree)**로 구분된다.

### 승자트리 (Winner Tree)

---

-   n개의 외부 노드(external node)와 n-1개의 내부 노드(internal node)를 가지는 완전 이진 트리(Complete Binary Tree) 이다.
-   외부 노드는 토너먼트의 플레이어라고 생각하면되고 내부 노드는 플레어이 간의 경기라고 생각하면 된다.
-   가장 작은 수가 우승한다.
-   이 트리의 루트는 최종 승자이다.

![image](https://user-images.githubusercontent.com/39365034/128294433-80a215ef-fa71-447e-ab71-38140fdae562.png){: .align-center}

#### 승자트리를 이용한 정렬

 n개의 수를 정렬하려고 할때, Winner Tree를 가장 작은 수가 우승하는 트리라고 정하면, n개의 수를 트리의 외부 노드레 넣고 승자를 찾는다. 정렬된 숫자들을 넣은 배열을 sortedArr이라 한다면, 승자는 가장 작은수 이므로 승자가 된 수를 sortedArr에 넣고 그 수를 트리에서 제거한다. 이렇게 n번을 반복하게 되면 정렬된 숫자배열을 얻을 수 있다.

이떄 걸리는 시간 복잡도는 트리 초기화 O(n), 승자를 제거하고 다시 승자를 찾는 과정 O(log(n)) (=트리의 height) 이 과정을 n번 반복하므로 O(n\*log(n) + n) = **O(n\*log(n))**이다.

### 패자트리(Loser Tree)

---

-   승자트리와 같은 특성을 가지고 있지만, 루트노드위에 최상위 0번 노드를 가진다는 점이 다르다.
-   패자를 부모노드에 저장하고 승자는 부모의 부모노드로 올라가서 다시 경기를 한다.
-   루트노드에는 마지막 패자를 정하고, 최종 승자는 0번노드에 넣는다.

![image](https://user-images.githubusercontent.com/39365034/128294524-d3d70d66-8127-4221-812b-d47118b9739b.png){: .align-center}
