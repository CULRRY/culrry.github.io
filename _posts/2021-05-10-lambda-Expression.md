---
title: "[C++] 람다 표현식(lambda expression)"
date: "2021-05-10 00:00:00 +0900"
categories:
    - C++

tags:
    - [C++]

toc: true
---
## 람다 표현식(lambda expression) 이란?/

 람다 표현식(lambda expression)은 C++11 부터 지원하는 문법으로, 익명의 함수를 호출할 수 있는 기능을 지원한다. 알아둔다면 매우 편리한 기능이다.

## 람다 표현식의 기본형태


![image](https://user-images.githubusercontent.com/39365034/127728136-a5c5c2c7-71cf-4ad0-b975-d86756da928a.png){: .align-center}

 람다식은 위 그림과 같이 캡쳐, 인자, 반환 타입, 람다 본문 이렇게 네가지로 구성된다.

\[ \] ( ) -> typename { } 와 같은 형식이다.

```cpp
#include <algorithm>
#include <cmath>
using namespace std;
void abssort(int* x, unsigned n) {
    sort(x, x + n,
        // 람다 시작
        [](int a, int b) {
            return (abs(a) < abs(b));
        } // 람다 끝
    );
}
```

위의 코드와 같이 사용될 수 있다.

### 캡쳐(Capture)

\[ \]부분에 해당하며, 람다안에서 람다밖의 변수를 사용해야 하는 경우에 사용한다. 캡쳐하고자 하는 변수들이 \[ \]안에 들어오게 되는데 대표적으로 5가지의 경우가있다.

1.  `[ ]` 외부의 모든 변수에 대해 접근을 하지 않는다.
2.  `[&]` 외부의 모든 변수들을 레퍼런스로 가져온다. (Call By Reference)
3.  `[=]` 외부의 모든 변수들을 값으로 가져온다. (Call By Value)
4.  `[=, &x, &y]` or `[&, x, y]` x, y를 제외한 외부의 모든 변수들을 값 / 레퍼런스로 가져온다. x, y는 레퍼런스 / 값으로 가져온다. \[(default), exception\]의 형식이다.
5.  `[x,. &y, &z]` 지정한대로 가져온다.

캡쳐되는 개체들은 모두 람다가 정의된 위치에서 접근이 가능해야한다.

---

### 인자(Parameter)

( )부분에 해당하며, 일반 함수와 같이 매개 변수를 사용 할 수 있다.

```cpp
auto y = [] (int first, int second)
{
    return first + second;
};
```

인자부분에 있어서는 일반 함수와 별 차이가 없는 것 같다.

---

### 반환 타입(Return Type)

람다식의 반환형식은 자동으로 추론된다. `->` 와 같은 후행반환 형식 키워드를 사용하면 반환 형식을 직접 지정할 수 있다.

```cpp
auto f1 = [](int a, int b) { return (abs(a) < abs(b));} // 자동으로 추론(bool)
auto f2 = [](int a, int b) -> bool { return (abs(a) < abs(b));} // 이런식으로 직접 지정가능
```

---

### 람다 본문

람다 본문에서는 일반 함수에 포함될 수 있는 모든 항목이 포함될 수 있다.

다음과 같은 종류의 변수에 액세스할 수 있다.

-   캡처된 변수
-   매개 변수
-   로컬로 선언된 변수
-   클래스 데이터 멤버 (클래스 내부에 선언되거나 this를 캡처하는 경우)
-   정적 변수 (ex: 지역변수)

---

## 사용 예시


```cpp
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;

int main()
{
   // 9개의 원소를 가지고 있는 벡터 생성
   vector<int> v;
   for (int i = 1; i < 10; ++i) {
      v.push_back(i);
   }

   // for_each와 lambda를 이용하여 벡터안에 있는 짝수의 개수 세기
   int evenCount = 0;
   for_each(v.begin(), v.end(), [&evenCount] (int n) {
      cout << n;
      if (n % 2 == 0) {
         cout << " is even " << endl;
         ++evenCount;
      } else {
         cout << " is odd " << endl;
      }
   });
   cout << "There are " << evenCount
        << " even numbers in the vector." << endl;
}
```

<div class="output">
<h4 style="text-align: center; font-size: 15px">[Output]</h4>
<hr style="border-color : #eff" />
<pre style="font-size:14px">1 is odd
2 is even
3 is odd
4 is even
5 is odd
6 is even
7 is odd
8 is even
9 is odd
There are 4 even numbers in the vector.</pre>
</div>

for\_each의 세번째 인자에 홀수, 짝수를 구분하는 람다를 넣어서 짝수의 개수를 구하는 코드이다.

```cpp
class FunctorClass
{
public:
    explicit FunctorClass(int& evenCount)
        : m_evenCount(evenCount) { }

    void operator()(int n) const {
        cout << n;

        if (n % 2 == 0) {
            cout << " is even " << endl;
            ++m_evenCount;
        } else {
            cout << " is odd " << endl;
        }
    }

private:
    FunctorClass& operator=(const FunctorClass&);
    int& m_evenCount;
};
```

람다식을 사용하지 않았더라면 이런 functor 클래스를 만들어서 for\_each에 집어넣어줘야 하는데 람다식 한줄로 표현할 수 있다는 것에서 람다식의 위대함을 다시 한번 느꼇다. 아마 std::algorithm 라이브러리에서 find\_if나 sort와 같은 함수에서 세번째 인자에 조건을 넣을 때 많이 사용할 것 같다. 람다식을 굉장히 유용해서 계속하서 익혀나가야겠다.
