---
title: "[자료구조]동적배열 구현하기"
date: "2022-03-07 18:27:00 +0900"
last_modified_at: "2022-03-07 18:27:00 +0900"
categories:
    - 자료구조

tags:
    - [자료구조, C++]

toc: true
excerpt: "stl에서 제공하는 동적배열 자료구조인 vector의 기능 중 조회, 삽입/삭제, 맨뒤에 삽입, 초기화를 구현해 보았다."
---

&nbsp;동적배열에 대한 자세한 설명은 해당 포스트를 참고해주시기 바랍니다. <br>
&nbsp;[[자료구조]배열, 연결리스트](https://culrry.github.io/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/Array-and-List/)



stl에서 제공하는 동적배열 자료구조인 vector의 기능 중 조회, 삽입/삭제, 맨뒤에 삽입, 초기화를 구현해 보았다. 

## 전체코드
```c++
template <typename T>
class Vector
{
public:
	Vector()
	{
		
	}
	~Vector()
	{
		if (_data)
			delete[] _data;
	}
	void insert(int pos, const T& data)
	{
		if (_size == _capacity)
		{
			// 증설 작업
			int newCapacity = _capacity * 1.5;
			if (newCapacity == _capacity)
				newCapacity++;

			reserve(newCapacity);
		}

		// 뒤로 한칸씩 밀기
		for (int i = _size - 1; i >= pos; i--)
		{
			_data[i + 1] = _data[i];
		}

		// 데이터 삽입
		_data[pos] = data;
		_size++;
	}

	void erase(int pos)
	{
		// 앞으로 한칸씩 땡기기
		for(int i = pos; i < _size - 1; i++)
		{
			_data[i] = _data[i + 1];
		}
		_size--;
	}

	void push_back(const T& data)
	{
		if (_size == _capacity)
		{
			int newCapacity = _capacity * 1.5;
			if (newCapacity == _capacity)
				newCapacity++;
			
			reserve(newCapacity);
		}

		_data[_size] = data;
		
		_size++;
	}

	void reserve(int capacity)
	{
		if (_capacity >= capacity)
			return;

		_capacity = capacity;

		T* newData = new T[_capacity];

		// 데이터 복사
		for (int i = 0; i < _size; i++)
			newData[i] = _data[i];

		if (_data)
			delete[] _data;

		//교체
		_data = newData;
	}

	T& operator[](const int pos) { return _data[pos]; }

	int size() { return _size; }
	int capacity() { return _capacity; }

	void clear()
	{
		if (_data)
		{
			delete[] _data;
			_data = new T[_capacity];
		}
		_size = 0;
	}

private:
	T* _data = nullptr;
	int _size = 0;
	int _capacity = 0;
};
```



## 세부설명

#### 멤버변수 설명

| 변수명    | 설명                              |
| --------- | --------------------------------- |
| _data     | 자료들을 저장하고 있는 배열.      |
| _size     | 현재 저장되어 있는 자료의 개수.   |
| _capacity | 현재 메모리에 할당되어 있는 공간. |



#### 함수 설명

- **용량 재할당**

  - capacity만큼 메모리에 공간을 재할당 한다.

  	```c++
    void reserve(int capacity)
    {
        if (_capacity >= capacity)
            return;
    
        _capacity = capacity;
    
        T* newData = new T[_capacity];
    
        // 데이터 복사
        for (int i = 0; i < _size; i++)
            newData[i] = _data[i];
    
        if (_data)
            delete[] _data;
    
        //교체
        _data = newData;
    }
    ```

  capacity만큼의 공간을 가진 배열을 새로 생성하고, 새로운 배열에 원래있던 데이터들을 옮긴다. 이때 새로 요청들어온 수용량이 원래의 수용량보다 적다면 할당을 실행하지 않는다. 이렇게 새로운 크기의 배열을 생성하고 이곳으로 자료들을 이주하는 것이 동적배열의 핵심 원리이다.
  
  
  
- **맨뒤에 데이터 추가**

  - 배열의 맨 뒤에 새로운 원소를 추가하는 함수로써, 배열이 꽉찼다면 capacity를 1.5배만큼 늘리고 원소를 추가한다.
  
    ```c++
    void push_back(const T& data)
    {
        if (_size == _capacity)
        {
            //증설 작업
            int newCapacity = _capacity * 1.5;
            if (newCapacity == _capacity)
                newCapacity++;
    
            reserve(newCapacity);
        }
    
        _data[_size] = data;
    
        _size++;
    }
    ```
  
    



  &nbsp;만약 현재 배열이 꽉찬 상태라면, 현재 capacity에 1.5배를 늘린 값으로  `reserve`를 통해 새로운 배열을 생성하고 그 뒤에 추가할 데이터를 붙힌다.

   

- **데이터 삽입/삭제**

  - 배열이라는 자료구조는 데이터들의 연속성을 유지하여야 하므로, 데이터와 데이터 사이에 빈공간이 있어서는 안된다. 따라서 데이터가 삭제되면 그 자리를 매워야하고, 데이터가 중간에 추가되면 그자리 이후에 있는 데이터를 뒤로 미루어야한다. 따라서 데이터가 많아질수록 효율적인 방법은 아니다.

    ```c++
    void insert(int pos, const T& data)
    {
        if (_size == _capacity)
    	{
    		// 증설 작업
    		int newCapacity = _capacity * 1.5;
    
            if (newCapacity == _capacity)
    			newCapacity++;
    		
            reserve(newCapacity);
    	}
    
    	// 뒤로 한칸씩 밀기
    	for (int i = _size - 1; i >= pos; i--)
    	{
    		_data[i + 1] = _data[i];
    	}
    
    	// 데이터 삽입
    	_data[pos] = data;
    	_size++;
    }
    
    void erase(int pos)
    {
    	// 앞으로 한칸씩 땡기기
    	for(int i = pos; i < _size - 1; i++)
    	{
    		_data[i] = _data[i + 1];
    	}
    	_size--;
    }		
    ```
    
    데이터의 삽입은 `insert`라는 함수로 구현하였고, 삭제는 `erase`라는 함수로 구현하였다.
    
    insert같은 경우는 데이터를 추가하는 것이므로 배열이 꽉찼다면 당연히 증설을 해야하는 것이고, 데이터가 들어갈 자리를 마련해야하므로 뒤쪽 데이터들을 뒤로 한칸씩 미는 것을 볼 수 있다. 따라서 O(n)의 시간복잡도가 발생한다.
    
    erase도 마찬가지로 삭제된 공간을 매우기 위해 앞으로 한칸씩 땡겨하 하므로 O(n)의 시간복잡도가 발생한다.



- **데이터 조회**

  - 배열의 강점이라고 할 수 있는 데이터 조회인데, 배열에는 각 자료마다 인덱스가 있으므로 그냥 그것을 불러오면 된다.

    ```c++
    T& operator[](const int pos) { return _data[pos]; }
    ```

    데이터 조회부분을 []연산자 오버라이딩을 통해서 구현하였으며, 해당 위치에 맞는 데이터를 불러오는데는 O(1)의 시간복잡도를 가진다.

    

- **데이터 초기화**

  - size를 0으로 만들지만, capacity는 그대로 유지된다.

    ```c++
    void clear()
    {
    	if (_data)
    	{
    		delete[] _data;
    		_data = new T[_capacity];
    	}
    	_size = 0;
    }
    ```

    
