---
title: "[자료구조]연결리스트 구현하기"
date: "2022-03-07 22:07:00 +0900"
last_modified_at: "2022-03-07 22:07:00 +0900"
categories:
    - 자료구조

tags:
    - [자료구조, C++]

toc: true
excerpt: "stl에서 제공하는 연결리스트 자료구조인 list의 기능 중 push_back, pop_back을 구현하면서 연결리스트를 간단히 구현해보도록 하겠다. 여기서 노드를 삭제하거나, 삽입하는 기능을 추가하려면 연결리스트는 임의접근이 불가능하므로 iterator를 구현해야하는데 iterator까지 구현하면 볼륨이 너무 커질 것 같아서 생략하였다."
---

&nbsp;연결리스트에 대한 자세한 설명은 해당 포스트를 참고해주시기 바랍니다. <br>
&nbsp;[[자료구조]배열, 연결리스트](https://culrry.github.io/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/Array-and-List/)



stl에서 제공하는 연결리스트 자료구조인 list의 기능 중 push_back, pop_back을 구현하면서 연결리스트를 간단히 구현해보도록 하겠다. 여기서 노드를 삭제하거나, 삽입하는 기능을 추가하려면 연결리스트는 임의접근이 불가능하므로 iterator를 구현해야하는데 iterator까지 구현하면 볼륨이 너무 커질 것 같아서 생략하였다.

연결리스트는 노드를 다루는 다른 자료구조의 기초가 되는 자료구조이므로, 한번쯤 구현해보는 것이 좋다고 생각한다.

## 전체코드
```c++
template <typename T>
class Node
{
public:
	Node() : _prev(nullptr), _next(nullptr), _data(T())
	{

	}

	Node(const T& val) : _prev(nullptr), _next(nullptr), _data(val)
	{

	}

private:
	Node* _prev;
	Node* _next;
	T* _data;
};



template <typename T>
class List
{
public:
	List()
	{
		_head = new Node<T>();
		_tail = new Node<T>();

		_head->_next = _tail;
		_tail->_prev = _head;
	}

	~List()
	{
		while (_size > 0)
			pop_back();
		delete _head;
		delete _tail;
	}

	void push_back(const T& val)
	{
		AddNode(_tail, val);
	}

	void pop_back()
	{
		RemoveNode(_tail->_prev);
	}

private:
	Node<T>* AddNode(Node<T>* before, const T& val)
	{
		Node<T>* newNode = new Node<T>(val);
		Node<T>* prevNode = before->_prev;

		prevNode->_next = newNode;
		newNode->_prev = prevNode;

		newNode->_next = before;
		before->_prev = newNode;

		_size++;

		return newNode;

	}

	Node<T>* RemoveNode(Node<T>* node)
	{
		Node<T>* prevNode = node->_prev;
		Node<T>* nextNode = node->_next;

		prevNode->_next = nextNode;
		nextNode->_prev = prevNode;

		delete node;
		_size--;

		return nextNode;
	}

private:
	Node<T>* _head;
	Node<T>* _tail;
	int _size;

};
```



## 세부설명

### 구조

![image](https://user-images.githubusercontent.com/39365034/157042707-150f1b2d-7837-4db1-8f59-7ab94a844242.png)

&nbsp; 기본적인 구조는 위와 같다. 각각의 노드는 Data와 이전 노드와 다음 노드의 포인터를 가지고 있다. 그리고 list는 그러한 노드들로 구성된다.



### 구현

#### 기본 구성

```c++
template <typename T>
class Node
{
public:
	Node() : _prev(nullptr), _next(nullptr), _data(T())
	{

	}

	Node(const T& val) : _prev(nullptr), _next(nullptr), _data(val)
	{

	}

private:
	Node* _prev;
	Node* _next;
	T* _data;
};
```

먼저 노드의 구현은 위와 같다.

당연히 저장할 자료를 가지고 있고, 그 외에 추가로 `_prev`와 `_next` 포인터를 가지고 있다. 이 변수들이 해당 노드 전후에 있는 노드들을 가르킨다.



```c++
template <typename T>
class List
{
public:
	List()
	{
		_head = new Node<T>();
		_tail = new Node<T>();

		_head->_next = _tail;
		_tail->_prev = _head;
	}

private:
	Node<T>* _head;
	Node<T>* _tail;
	int _size;

};
```

node를 구현하였으면 다음은 List를 구성해보자. 먼저 List는 기본적으로 안에 아무런 자료가 들어가지 않아도 디폴트로 head노드와 tail노드를 가지고 있다. 이 노드는 아무런 데이터도 들어있지않은 Dummy Node인데, 이러한 노드를 시작부터 배치해놓은 이유는 리스트를 생성하고이 리스트에 데이터를 삽입할 때, 이런 노드가 없다면 리스트에 아무것도 없을 때와 있을 때를 구분해서 구현해야한다. 따라서 편의를 위해서 리스트의 시작과 끝을 나타내는 노드들을 dummy로 배치해 놓는 것이다.

![image](https://user-images.githubusercontent.com/39365034/157044157-fd3d4fbd-32fe-4641-be8c-8b1f2be241b4.png)

리스트를 맨 처음 생성하였다면 이러한 상태일 것이다.



#### 노드 삽입/삭제

🔹**노드삽입**

```c++
Node<T>* AddNode(Node<T>* before, const T& val)
{
	Node<T>* newNode = new Node<T>(val);
	Node<T>* prevNode = before->_prev;

	prevNode->_next = newNode;
	newNode->_prev = prevNode;

	newNode->_next = before;
	before->_prev = newNode;

	_size++;

	return newNode;
}
```

![image](https://user-images.githubusercontent.com/39365034/157045800-474bb779-5355-43c8-adab-cc8dc68150de.png)

만약 넣고자하는 위치가 A와 B사이라면,  B에 있는 노드를 before로 설정한다. 그러면 before->_prev가 prevNode이므로 A는 prevNode가 된다.

![image](https://user-images.githubusercontent.com/39365034/157046554-d0105327-74fa-487d-950d-664b6d326033.png)

&nbsp; 위와 같이, prevNode의 _next와 newNode의 _prev를 이어주고 before의 _prev와 newNode의 _next를 이어주면 자연스레 prevNode와 before의 연결은 끊어지고 그사이에 newNode가 삽입되게 된다. 이 과정이 단 코드 4줄에 의해 실행되므로 시간복잡도는 O(1)을 가진다고 할 수 있다.



🔹**노드삭제**

```c++
Node<T>* RemoveNode(Node<T>* node)
{
	Node<T>* prevNode = node->_prev;
	Node<T>* nextNode = node->_next;

	prevNode->_next = nextNode;
	nextNode->_prev = prevNode;

	delete node;
	_size--;

	return nextNode;
}	
```

![image](https://user-images.githubusercontent.com/39365034/157048096-e631e575-ad92-40be-81e5-0c78c0e1fcc0.png)

삭제는 더욱 간단하다. 지우고자 하는 노드의 앞노드와 뒷노드를 이어주면 되는데, 앞노드의 _next와 뒷노드의 _prev를 이어주면 자연스럽게 연결된다. 이또한 보다시피 상수시간에 해결됨을 볼 수 있다. 따라서 시간복잡도는 O(1)이다.



🔹**push_back, pop_back**

```cpp
void push_back(const T& val)
{
	AddNode(_tail, val);
}

void pop_back()
{
	RemoveNode(_tail->_prev);
}
```

이미 앞에서 노드를 삽입하고 삭제하는 로직을 모두 구현했다. 따라서 맨뒤에 추가하는 push_back연산은 _tail을 before로 설절하여 AddNode함수를 실행하면 되고, 맨뒤에 원소를 제거하는 pop_back은 _tail의 이전노드를 인자로 넣어 RemoveNode함수를 실행하면 된다.



## 마치며

&nbsp;이렇게 연결리스트의 기능을 간단하게 나마 구현해 보았는데, 이러한 노드를 사용하는 자료구조는  연결리스트 이외에도 트리, 그래프등 다양한 자료구조들이 존재한다. 연결리스트는 이러한 자료구조중 가장 기초가 되는 자료구조라고 생각하고, 노드를 다루는 것을 연습해볼겸 한번 직접 구현해보는 것이 나쁘지 않다고 생각한다. 사실 우리가 배열을 써도 vector와 같은 연속적인 배열을 자주사용하지 연결리스트는 vector와 같은 자료구조에 비해 활용도가 떨어진다고 생각한다. 하지만 그럼에도 불구하고 앞으로 나올 자료구조를 익히기 위해서는 굉장히 중요한 자료구조라고 생각이 든다. 

