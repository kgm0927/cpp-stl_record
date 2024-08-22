
템플릿 코드는 특수화된 각각의 템플릿 타입에 따라 특정한 작업을 해줘야 한다. C++17에서는 constexpr-if 표현식이 추가됐다. constexpr-if 표현식은 이런 상황에서 쓰이는 코드를 아주 효과적으로 단순화시킨다.


### 예제 구현


이번 예제에서는 간단한 **헬퍼 템플릿 클래스** 하나를 구현할 것이다. ==이 헬퍼 템플릿 클래스는 다양한 템플릿 타입 특수화를 다룰 수 있는데, 어떤 타입을 특수화하는지에 따라 완전히 다른 코드를 선택할 수 있기 때문이다.==
1. 제너릭 코드 일부를 작성한다. 이 예제는 add 함수를 사용해 멤버 변수 T에 U 값을 추가할 수 있는 단순한 클래스이다.

``` c++
#include <iostream>

template <typename T>

class addable
{

private:
    T val;

public:

    addable(T v):val{v} {}
    template<typename U>
    T add(U x)const{
        return val+x;
    }
};
```


2. 타입 T가 `std::vector<something>`이고 타입 U는 int이라고 생각해보자. 전체 벡터에 정수를 더한다는 것이 어떤 의미일까? ==이는 곧 벡터의 모든 요소에 대해 해당 정수를 더한다는 의미이다.== 반복문을 통하여 이를 처리한다.

``` c++
  template <typename U>
    T add(U x) {
        auto copy(val); // 벡터 멤버의 사본을 구한다.
        for (auto &&n : copy)
        {
            n+=x;
        }
        return copy;
    }
```


3. 다음 마지막 단계는 두 개의 벡터를 합치는 것이다. T가 U 요소를 갖는 벡터라면 반복문을 통해 변경하게 된다. 그렇지 않으면 기본 더하기를 통해 구현한다.
``` c++

   template <typename U>
    T add(U x) {
    if constexpr(std::is_same_v<T,std::vector<U>>)
    {    
    auto copy(val); // 벡터 멤버의 사본을 구한다.
        for (auto &&n : copy)
        {
            n+=x;
        }
        return copy;
        }else{
            return val+x;
        }
    }
```


4. 이제 이 클래스를 실제 사용에 적용해보자. int나 float, `std::vector<int>`, `std::vector<string>`과 같이 전혀 다른 타입에서는 어떻게 작동하는지 살펴보자.

``` c++
int main(){
    addable<int> {1}.add(2);                    // 3이 된다.
    addable<float> {1.0}.add(2);                // 3.0이 된다.
    addable<std::string>{"aa"}.add("bb");       // "aabb"가 된다.

  

    std::vector<int> v {1,2,3};
    addable<std::vector<int>> {v}.add(10);
    // std::vector<int>{11,12,13}이 된다.

    std::vector<std::string> sv {"a","b","c"};
    addable<std::vector<std::string>>{sv}.add(std::string{"z"});

    // {"az","bz","cz"}이 된다.

}
```

### 예제 분석

새로운 constexpr-if는 일반적인 if-else 구조와 하는 일이 똑같다. ==차이점이 있다면 검사하는 조건이 컴파일 타임에 평가되어야 한다는 점이다.== 프로그램에서 컴파일러가 생성한 모든 런타임 코드는 constexpr-if 조건 분기 명령을 포함하지 않는다. 전처리기 `#if`와 `#else` 텍스트 치환 매크로와 유사한 방법으로 동작할 수 있지만, 코드 문법에 맞지 않을 수도 있다. 모든 constexpr-if 분기 명령은 문법에 맞아야 하지만, 그렇지 않은 분기 명령은 의미상 유효하지 않아도 된다.

코드에서 하나의 벡터에 x값을 더해야 하는지 판별하기 위해서는 std::is_same 특성 타입을 사용한다. `std::is_same<A,B>::value` 표현식은 A와 B가 같은 타입이면 불리언(boolean) 값을 true로 평가한다. 이 예제에서 사용된 조건식은 `std::is_same<T,std::vector<U>>::value`로, `T=std::vector<X>`로 클래스를 특수화하면 true로 평가하고 U=X 타입의 파라미터로 add 함수를 호출하게 된다.


물론 하나의 constexpr-if-else 블록에 여러 개의 조건이 있을 수 있다(a와 b는 컴파일 타임 상수 값뿐만 아니라 템플릿 파라미터에 따라서도 달라진다는 점을 유의한다).

``` c++
if constexpr(a){
// 작업을 수행한다.
}else if constexpr(b){
// 또 다른 작업을 수행한다.
}else{
// 완전히 다른 작업을 수행한다.
}
```

C++17를 이용해 다양한 메타프로그래밍 상황을 표현하거나 읽기가 더욱 쉬워졌다.


### 부연 설명

C++17 이전에는 동일한 기능을 어떻게 구현했는지 살펴보면 constexpr-if 생성이 C++이 얼마나 큰 발전을 가져왔는지 확실히 체감할 수 있을 것이다.

``` c++

template <typename T>

class addable

{

private:

    T val;

public:

    addable(T v):val{v} {}

    // C++ 11

    #if 0

    template<typename U>

    std::enable_if_t<!std::is_same<T,std::vector<U>>::value,T>

    T add(U x)const{

        return val+x;

    }

  

    template<typename U>

    std::enable_if_t<std::is_same<T,std::vector<U>>::value,std::vector<U>>
    add(U x) const{
        auto copy(val);
        for (auto &n : copy)

        {
           n+=x;
        }
        return copy;
    }
```


이 클래스는 constexpr-if를 사용하지 않고도 다양한 타입에 대해서 동작하기는 하지만 상당히 복잡해보인다. 어떻게 동작하는 것일까?

두 개의 서로 다른 add 함수의 구현만 보면 단순하다. 그러나 반환 타입의 정의 부분이 매우 복잡해 보인다. ==그리고 조건식이 true이면 타입을 평가하는 std::enable_if_t<condition,type>과 같은 표현식을 통하여 약간의 트릭을 사용하고 있다.== ==이런 방법 없이는 std::enable_if_t 표현식이 아무것도 평가하지 않기 때문이다==. 보통은 이런 경우 오류고 보지만, 왜 여기서는 그렇지 않은지 살펴보자.

두 번째 add 함수에서는 같은 조건을 반대로 사용한다. 이 방법으로 한 번에 두 개의 구현 중 하나만 true가 될 수 있는 것이다.

컴파일러가 같은 이름의 서로 다른 템플릿 함수를 바라보며 이들 중 하나를 선택해야 할 때 중요한 원칙은 **SFINAE**가 진행된다는 점이다. 즉, Substitiution Failure is not an Error(치환 실패가 오류는 아니다)라는 얘기다. ==이 경우 해당 함수 중 하나의 반환값이 잘못된 템플릿 표현식으로부터 추론할 수 없더라도 (std:enable_if의 경우 조건이 false로 평가될 때)컴파일러에서 오류가 발생하지 않는 것을 의미한다.== 단순히 다른 함수 구현을 시도해볼 것이다. 이게 바로 이 트릭이 동작하는 원리다.

참으로 번거로운 과정이다. C++17에서는 이 작업이 크게 단순화돼서 정말 다행이다.