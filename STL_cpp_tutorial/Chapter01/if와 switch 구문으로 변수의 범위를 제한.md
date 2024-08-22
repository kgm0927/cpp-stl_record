

변수의 범위를 최대한 제한하는 것은 좋은 코딩 습관이다. ==그러나 처음에는 몇 가지 값만 갖고 있다가 확실한 조건을 만족하면 그때 더 넓은 범위로 처리해야 하는 경우도 있을 것이다.==

이러한 목적으로 C++17은 초기화 가능한 if와 switch구문을 갖고 있다.


### 예제 구현

이번 예제에서는 두 가지 방법의 초기화 구문을 사용해 코드를 깔끔하게 정리하는 방법을 살펴보자.

- if 구문: std::map의 find 함수를 사용해 문자열 맵에서 특정 문자를 찾는 경우를 생각해보자.

```c++

if(auto itr(character_map.find(c));itr!=character_map.end()){
//*itr이 유효하며, 작업을 수행한다.
}else{
// itr은 마지막 반복자다. 참조하지 말 것
}
// itr은 더 이상 반복할 수 없다.
```

- switch 구문: 다음은 키 입력 문자를 구하는 동시에 switch문으로 해당 값을 확인해 컴퓨터 게임을 제어하는 방법을 보여준다.

``` c++
switch(char c(getchar());c){
case 'a':move_left(); break;
case 'b':move_back(); break;
case 'c':move_fwd(); break;
case 'd':move_right(); break;

case 'q':quit_game();break;
case '0'...'9':select_tool('0'-c);break;

default:
std::cout<<"invalid input:"<<c<<'\n';
}
```


### 예제 분석

if와 switch문의 초기화는 기본적으로 신택스 슈거(syntax sugar)다. 다음 두 예제는 동일하다.


- C++17 이전
``` c++


{
auto var(init_value);
if (condition){
	// A 조건, var에 접근 가능하다.
}else{
	// B 조건, var에 접근 가능하다.
}
// 여전히 var에 접근 가능하다.
}
```

- C++17

``` c++
if(auto var(init_value); condition){
// A 조건, var에 접근 가능하다.
}else{
// B 조건, var에 접근 가능하다.
}
// 더 이상 var에 접근할 수 없다.
```


switch 구문에도 동일하게 적용된다.

- C++17 이전

``` c++
{
 auto var(init_value);
 switch(var){
 case 1: ...
 case 2: ...
 ...
 }
 // 여전히 var에 접근이 가능하다.
}
```

- C++17
``` c++
switch(auto var(init_value);var){
case 1: ...
case 2: ...
}

// 더 이상 var에 접근할 수 없다.
```

이 기능은 변수의 사용 범위를 가능한 한 작게 만드는 데 유용하다. C++17 이전 버전의 예제에서 봤듯이 변수의 사용 범위는 코드에 추가적인 중괄호를 사용해서만 가능했다. ==변수의 짧은 수명은 코드를 간결하게 유지하고, 리팩토링이 쉽게 해당 밤위 내에서 변수의 개수를 줄여준다.==

### 부연 설명

또 다른 흥미로운 점은 제한된 크리티컬 섹션의 범위에 있다. 다음 예제를 살펴보자.

``` c++
if(std::lock_guard<std::mutex> lg {my_mutex}; some_condition){
// 작업을 수행한다.
}

```

가장 먼저 std::lock_guard를 생성했다. 이는 생성자의 인자로 뮤텍스(mutex)를 받는 클래스다. 생성자에서 뮤텍스를 잠그며, 범위를 벗어나면 소멸자에서 다시 잠금을 푼다. 이러한 방법으로 뮤텍스 잠금을 잊지 않게한다.

그 외에도 weak 포인터 범위와 관련된 흥미로운 점이 있다. 다음의 경우를 고려해보자.

``` c++
if(auto shared_pointer(weak_pointer.lock());shared_pointer!=nullptr){
// 예상대로 shared 객체가 계속해서 존재한다.
}else{
// shared_pointer 변수에 접근은 가능하나 null포인터이다.
}
//더 이상 shared_pointer 변수에 접근할 수 있다.
```

if 조건문이나 추가 중괄호 밖에서 잠재적으로 불필요한 상태면서 현재 범위에서도 필요 없는 shared_pointer 변수를 갖는 또 다른 예제라고 할 수 있다.

``` c++
if(DWORD exit_code; GetExitCodeProcess(process_handle,&exit_code)){
	std::cout<<"Exit code of process was:"<<exit_code<<'\n';
}
// if 조건문 밖에서는 불필요한 exit_code 변수가 사용되지 않는다.
```

**GetExitCodeProcess는** 윈도우 커널 API 함수이다. 이 함수는 주어진 핸들이 유효할 때에만 해당 프로세스 핸들의 exit 코드를 반환한다. 이 조건문을 벗어나면 해당 변수는 쓸모없어지며, 더 이상 다른 곳에서 필요하지 않다.

if 블록 내에서 변수를 초기화할 수 있다는 점은 많은 경우 아주 유용하게 쓰인다. 출력 파라미터를 사용하는 오래된 API를 다룰 때 더욱 그렇다.

>[!TIP]
>if와 switch 초기화 구문을 이용해 범위를 작게 유지하자. 코드를 더욱 간결하게 읽기 편하게 만들며, 리팩토링할 때 옮기기 쉬워지기 때문이다.


---
# 새로운 중괄호 초기화 규칙의 강점


C++11에서 새로운 중괄호 초기화 구문인 {}가 추가됐다. ==이는 집합체 초기화뿐만 아니라 일반적으로 생성자 호출을 가능하게 하는 목적을 뒀다==. 그러나 아쉽게도 auto 변수와 결합하면 잘못된 값을 사용하기 일쑤였다. C++17에서는 향상된 초기화 규칙들이 추가됐다. 이번 예제에 C+17의 구문으로 변수를 초기화하는 올바른 방법을 명확히 알아보자.



### 예제 구현

변수는 한 번에 초기화된다. 다음은 초기화 구문을 사용한 두 가지 다른 상황이다.

- auto 자료형의 추론 없이 중괄호 초기화 구문의 사용

``` c++

// int를 초기화하는 세 가지 동일한 방법

int x1=1;
int x2{3};
int x3{1};

std::vector v1 {1,2,3}; // 1, 2, 3 세 개의 int 값을 갖는 Vector
std::vector<int> v2={1,2,3}; // 이전과 동일함
std::vecotr<int> v3{10,20}; // 각 값이 20인 int 값 10개를 갖는 Vector
```

- auto 자료형의 추론으로 중괄호 초기화 구문의 사용

``` c++
auto v {1}; // v는 int 이다.
auto w {1,2}; // 오류: 단일 요소만 auto 초기화가 허용된다(새롭게 추가된 내용이다).
auto x {1}; // x는 std::initializer_list<int>
auto y={1,2}; // y는 std::initializer_list<int>
auto z={1,2,3.0}; // 오류: 해당 자료형을 추론할 수 없다.
```


### 예제 분석

최소한 정규 자료형의 초기화에 있어서 auto 자료형 추론이 없는 중괄호 {} 연산은 딱히 놀랄 많한 것이 별로 없다. ==std::vector나 std::list 같은 컨테이너를 초기화 할 때의 중괄호 초기화는 컨테이너 클래스의 std::initializer_list 생성자와 일치하게 된다.== 이는 매우 탐욕스러운 방식으로 처리되는데, 즉 비집합체 생성자는 일치시키기가 불가능하다(비집합체 생성자는 일반적으로 초기화 목록이 허용되지 않는 생성자를 말한다).

예를 들어 std::vector는 명확한 비집합체 생성자를 제공한다. `std::vector<int> str::vector<int> v {N,value}`를 사용하면 initializer_list 생성자가 선택되는데, 이는 해당 백터를 개수(N)와 값(value) 두 가지 요소로 초기화한다. 방심하면 실수하기 쉬운 부분이니 조심하자.

일반적인 () 괄호로 호출하는 생성자와 비교했을 때 {} 연산의 매력적인 사실 중 하나는 암시적으로 자료형이 변환할 수 없다는 점이다. int x (1.2);과 int x=1.2는 x의 실수 값을 내림해 int로 변환해서 1의 값으로 초기화한다. ==반면 int x{1,2}는 명시적으로 생성자 자료형과 일치해야 하므로 컴파일 될 수 없다.==

>[!info]
> 가장 좋은 초기화 방법이 무엇이냐는 질문에 대해서는 논란의 소지가 있다.
> 중괄호 초기화 스타일을 선호하는 개발자들은 중괄호를 사용하면 매우 명쾌하게 만들 수 있다고 말한다. 즉, 변수는 생성자가 호출될 때에만 초기화되며, 해당 코드는 또다시 초기화 되지 않는 것이다. 게다가 () 괄호를 사용한 초기화는 가장 적절한 생성자를 찾아낼 뿐만 아니라 형 변환까지 처리하는 반면 {} 중괄호를 사용하면 올바른 생성자를 선택하는데 그치기 때문이다.


C++ 11에서는 변수 auto x {123};의 자료형을 정확하게 맞추는 만면 C++17에서 소개된 추가적인 규칙은 auto 자료형의 추론을 통한 초기화에 영향을 미친다. 하나의 요소를 갖는 `std::initializer_list<int>`는 원하는 대로 처리가 불가능하기 때문이다. C++17은 동일한 변수인 int로 처리할 것이다.

경험을 통해 알게 된 것들은 다음과 같다.


- auto var_name {one_element}; one_element와 같은 자료형이 되도록 var_name을 추론한다.
- auto var_name {element1,element2, ...}; 유효하지 않으며 컴파일되지 않는다.
- auto var_name={element1,element2, ...}; 목록의 모든 요소가 같은 자료형이 되도록 T로 `std::initializer_list<T>`를 추론한다.


C++17을 실수로 초기화 목록을 정의하는 위험을 크게 줄였다.

>[!info]
> C++11/C++14와 같이 다른 컴파일러로 똑같은 작업을 해보면 실제로 일부는 auto x {123};을 int로 추론하지만, 일부는 `std::initializer_list<int>`로 추론하는 걸 볼 수 있다. 이처럼 코드를 작성하면 관련한 문제가 생길 수 있다.


---
# 생성자에서 자동으로 템플릿 클래스 타입 추론



==일반적으로 많은 종류의 C++클래스가 자료형에 대해 특화돼 있으며,== ==사용자가 생성자를 호출할 때 배치하는 변수들의 자료형으로부터 손쉽게 추론할 수 있다. ==그런데도 C++17 이전에는 표준화된 기능이 아니었다. **C++17에서는 생성자 호출 시 컴파일러가 자동으로 템플릿 타입을 추론한다.**


### 예제 구현

이에 대한 아주 간단한 사용 예제로 std::pair와 std::tuple 인스턴스 생성이 있다. 이 둘은 한 번에 특수화 및 인스턴스로 만들 수 있다.

``` c++
std::pair my_pair(123,"abc"); // std::pair <int,const char*>
std::tuple my_tuple (123,12.3,"abc"); // std::tuple<int, double,const char*>
```

### 예제 분석

값이 되는 자동 템플릿 타입 추론의 예제 클래스를 정의해보자.

``` c++
#include <tuple>
#include <string>
#include <type_traits>
#include <iostream>
  

template <typename T1,typename T2,typename T3>

class my_wrapper
{
private:
    T1 t1;
    T2 t2;
    T3 t3;

public:

    my_wrapper(T1 t1_,T2 t2_,T3 t3_):t1{t1_},t2{t2_},t3{t3_}

{}

};
```

이는 단지 또 다른 템플릿 클래스일 뿐이다. 이전에는 인스턴스로 만들기 위해 다음과 같이 작성해야 했다.

```c++
my_wrapper<int,double,const char*> wrapper{123,1.23,"abc"};
```

지금은 템플릿 특수화 부분을 생략할 수 있다.

``` c++
my_wrapper wrapper{123,1.23,"abc"};
```

C++17 이전에는 **헬퍼(helper) 함수**를 구현해야만 생략이 가능했다.

``` c++
my_wrapper<T1,T2,T3> make_wrapper(T1 t1,T2 t2,T3 t3){
return {t1,t2,t3};
}
```

이와 같은 헬퍼 함수를 사용해야만 비슷한 효과를 낼 수 있었다.

``` c++
auto wrapper(make_wrapper(123,1.23,"abc"));
```

>[!info]
>STL은 std::make_shared, std::make_unique, std::make_tuple 등과 같이 여전히 많은 헬퍼 함수가 존재한다. C++17에서는 이와 같은 함수들 대부분이 거의 필요없게 됐다. 물론 호환성의 이유로 계속해서 제공이 될 것이다.


### 부연 설명

이번 예제는 암시적 템플릿 타입 추론에 대해 배웠다. 일부 경우에는 암시적 타입 추론에 의해 의존할 수가 없다. 다음 예제 클래스를 살펴보자.

``` c++

template <typename T>

struct sum{
    T value;

    template<typename ... Ts>
    sum(Ts&& ... values): value{(values+...)}{}
};
```

sum 구조체는 임의의 개수의 파라미터를 허용하며, 표현식 접기를 사용해 이 파라미터를 모두 한꺼번에 추가한다(표현식 접기에 대해서는 1장 뒷부분에서 더 자세히 살펴본다). 

sum 구조체는 멤버 변수 값을 저장한다. 여기서 질문이 생길 것이다. 과연 T는 어떤 타입일까? 이를 명시적으로 나타내지 않을 경우 의심의 여지 없이 생성자에서 제공되는 값에 따라 달라질 것이다. 즉, 문자열 인스턴스가 사용되면 std::string이 된다. 또는 정수가 사용되면 int 형이 돼야 한다. 정수와 실수, double을 함께 사용하면 컴파일러는 자료의 손시 없이 모든 값에 맞는 자료형을 나타낼 것이다. 이를 위한 명시적 추론 가이드가 있다.


``` c++
template <typename ... Ts>
sum(Ts&& ... ts) -> sum<std::common_type_t<Ts...>>;
```

이 내용은 컴파일러에게 std::common_type_t 특성값을 사용하라고 알려준다. 그래서 모든 값에 맞는 값에 기본 자료형을 찾을 수 있게 해 준다. 어떻게 사용하는지 살펴보자.