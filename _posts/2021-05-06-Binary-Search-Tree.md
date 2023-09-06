---
title: "이진 탐색 트리(Binary Search Tree)"
date: "2021-05-06 19:31:00 +0900"
categories:
    - 자료구조

tags:
    - [JAVA, 자료구조]

toc: true
---

## 이진탐색트리(Binary Search Tree) 란?


이진탐색트리의 정의는 다음과 같다.

-   이진트리(Binary Tree)이다.
-   각 노드는 (key, value)쌍을 가지고 있다.
-   임의의 노드 `x`에 대해서 `x`의 왼쪽 서브트리에 있는 노드들의 key는 모두 `x`보다 작고 오른쪽 서브트리에 있는 노드들의 key는 모두 `x`보다 크다.

## 이진탐색트리의 연산


이진탐색트리는 탐색, 삽입, 수정, 삭제의 연산을 가진다.

-   IsEmpty()
-   Get(key) -> 탐색
-   Insert(key, value) -> 삽입, 수정
-   Delete(key) -> 삭제

---

#### Get : 탐색

시간복잡도: **O(n)** (n은 Tree의 원소수)

```java
    T Get(T item)  {
        TreeNode<T> ptr;
        ptr = root;
        while(ptr != null)
        {
            if(ptr.data.GetKey() > item.GetKey())
                ptr = ptr.leftChild;

            else if(ptr.data.GetKey() < item.GetKey())
                ptr = ptr.rightChild;

            else
                return ptr.data;
        }
        return ptr.data;
    }
```

코드를 짠다면 이런식으로 짤 수 있을 것이다.

1.  기준점(ptr)을 설정한다. (초기값 root)
2.  찾고자하는 item의 key값과 기준점의 key값을 비교한다.
3.  item의 key값이 기준점의 key값보다 크다면 기준점을 원래 기준점의 rightChild노드로 재설정하고, 작다면 leftChild노드로 재설정한다.
4.  기준점의 key값이 item의 key과 같아진다면 그 key값을 가진 노드의 data를 반환한다.
5.  반복을 수행했을 떄 기준점이 null이 되는 순간 같은 값이 없다는 의미이므로 반복문을 종료하고 null데이터를 반환한다.

---

#### Insert : 삽입, 수정

```java
boolean  Insert(T item)  {
        if(root == null) {

            root = new TreeNode<T>(item);
            return true;
        }
        TreeNode<T> ptr, parent;
        parent = root;
        ptr = root;
        while(true){
            if(ptr.data.GetKey() > item.GetKey()){
                if(ptr.leftChild != null){
                    parent = ptr;
                    ptr = ptr.leftChild;
                    continue;
                }
                ptr.leftChild = new TreeNode<T>(item);
                return true;
            }
            else if (ptr.data.GetKey() < item.GetKey()){
                if(ptr.rightChild != null){
                    parent = ptr;
                    ptr = ptr.rightChild;
                    continue;
                }
                ptr.rightChild = new TreeNode<T>(item);
                return true;
            }
            else{
                if(parent.data.GetKey() > item.GetKey()){
                    parent.leftChild = new TreeNode<T>(item);
                    parent.leftChild.leftChild = ptr.leftChild;
                    parent.leftChild.rightChild = ptr.rightChild;
                }
                else if(parent.data.GetKey() < item.GetKey()){
                    parent.rightChild = new TreeNode<T>(item);
                    parent.rightChild.leftChild = ptr.leftChild;
                    parent.rightChild.rightChild = ptr.rightChild;
                }
                else{
                    ptr = new TreeNode<T>(item);
                    ptr.rightChild = parent.rightChild;
                    ptr.leftChild = parent.leftChild;
                    root = ptr;
                }
                return false;
            }
        }
    }
```

코드가 많이 난잡해서 더 깔끔하고 명료하게 짜는 방법을 고민해봐야 할 것 같다.

해당 함수는 원소의 삽입과 수정을 담당하는 함수이다.

기본적인 기반은 위에서 살펴본 Get과 비슷한 논리를 가지고있으며 세부적인 논리는 다음과 같다.

1.  트리가 비어있을 경우, root에 삽입하고자 하는 원소를 넣는다.
2.  비어있지 않을 경우,
    1.  부모노드(parent)를 저장할 수 있는 변수, 기준점(ptr) 변수를 설정한다. (초기값 둘다 root)
    2.  부모노드를 원래 기준점으로 설정하고 item의 key값이 기준점의 key값보다 크다면 기준점을 원래 기준점의 rightChild노드로 재설정하고, 작다면 leftChild노드로 재설정한다.
    3.  item의 key값이 기준점의 key값과 같다면, 기준점의 key값에 해당하는 value를 item의 value로 치환한다.(수정)
    4.  반복을 수행했을 떄 기준점이 null이 되는 순간 같은 값이 없다는 의미이므로 그 자리에 item의 데이터를 집어넣는다.

---

#### Delete: 삭제

삭제연산은 4가지의 경우의 수를 가진다.

-   삭제하고자하는 key값을 가진 원소가 없을 경우
-   삭제하고자하는 원소가 leaf 노드(childNode가 0개)인 경우
-   삭제하고자하는 원소가 1개의 자식을 가진 원소인 경우
-   삭제하고자하는 원소가 2개의 자식을 가진 원소인 경우

**#Case1.** 삭제하고자하는 key값을 가진 원소가 없을가 없을 경우

 이 경우에는 탐색을 수행했을 때, 삭제하고자하는 key값을 가진원소가 없을 경우 false를 리턴하며 함수를 종료한다.

**#Case2.** 삭제하고자하는 원소가 leaf 노드(childNode가 0개)인 경우

 삭제하고자 하는 원소를 탐색했을때 그 원소가 leaf node일 경우에는 그 노드의 부모의 left또는 rightChild의 값을 null로 바꿔주면 될 것이다.

**#Case3.**삭제하고자하는 원소가 1개의 자식을 가진 원소인 경우

 이 경우에는 삭제하고자하는 원소의 자식과 부모를 서로 연결해주면 된다.

**#Case4.**삭제하고자하는 원소가2개의 자식을 가진 원소인 경우

 자식이 두개인 노드를 삭제하고자 하는 경우에는 그 노드를 기준으로 왼쪽 subTree에서 가장 큰값을 찾아 그값을 삭제하고자하는 노드의 위치에 넣는다. (오른쪽 subTree에서 가장 작은 값도 가능) 가장 큰값을 가졌던 노드는 무조건 1개의 자식만을 가진 노드이거나 leaf노드 이므로 그 노드의 부모와 자식을 서로 연결해주면 된다.

```java
boolean Delete(T item)  {
        if(root == null)
            return false;	// non existing key
        TreeNode<T> target = root;
        TreeNode<T> parent = root;

        while(target != null){
            if(target.data.GetKey() > item.GetKey()){
                parent = target;
                target = target.leftChild;
            }
            else if(target.data.GetKey() < item.GetKey()){
                parent = target;
                target = target.rightChild;
            }
            else
                break;
        }

        //#Case1. 삭제하고자하는 key값을 가진 원소가 없을 경우
        if (target == null)
            return false;
        else{
            //#Case2. 삭제하고자하는 원소가 leaf 노드(childNode가 0개)인 경우
            if(target.leftChild == null && target.rightChild == null ){
                if(parent.data.GetKey() > target.data.GetKey()){
                    parent.leftChild = null;
                }

                else if(parent.data.GetKey() < target.data.GetKey())
                    parent.rightChild = null;
                else {
                    if(root.rightChild == null)
                        root = root.leftChild;
                    else
                        root = root.rightChild;
                }

            }
            //#Case3. 삭제하고자하는 원소가 1개의 자식을 가진 원소인 경우
            else if(target.leftChild == null || target.rightChild == null){
                if(parent.data.GetKey() > target.data.GetKey()){
                    if(target.leftChild == null){
                        parent.leftChild = target.rightChild;
                    }
                    else{
                       parent.leftChild = target.leftChild;
                    }
                }
                else if(parent.data.GetKey() < target.data.GetKey()){
                    if(target.leftChild == null){
                        parent.rightChild = target.rightChild;
                    }else{
                        parent.rightChild = target.leftChild;
                    }
                }
                else{
                    if(target.leftChild == null){
                        root = target.rightChild;
                    }else{
                        root = target.leftChild;
                    }
                }
            }
            //#Case4. 삭제하고자하는 원소가2개의 자식을 가진 원소인 경우
            else{
                TreeNode<T> minParent = target;
                TreeNode<T> min = target.rightChild;
                while (min.leftChild != null){
                    minParent = min;
                    min = min.leftChild;
                }
                if(min == target.rightChild){
                    target.rightChild = null;
                }
                //#Case4-1. 최소 노드가 leaf일 경우
                else if(min.rightChild == null){
                    minParent.leftChild = null;
                }
                //#Case4-2. 최소 노드의 자식이 1개일 경우
                else{
                        minParent.leftChild = min.rightChild;
                }
                min.leftChild = target.leftChild;
                min.rightChild = target.rightChild ;

                if(parent.data.GetKey() > target.data.GetKey())
                    parent.leftChild = min;
                else if (parent.data.GetKey() < target.data.GetKey())
                    parent.rightChild = min;
                else
                    root = min;
            }
        }
        return true;
    }
```

코드로 풀어보면 다음과 같다. 내가 생각해도 정말 못 만들었다....
