

std::map 대신 std::unordered_map을 사용할 경우 선택할 수 있는 키 타입에 다소 차이가 있다. std::map에서는 모든 키 요소가 순서대로 배치되도록 요구된다. 그래서 순서에 따라 요소들이 정렬된다. 그러나 예를 들어 벡터를 키 타입으로 적용하려면 어떻게 해야 할까? 벡터 (0,1)은 (1,0)보다 크지도 작지도 않기 때문에 여기에는 < 관계라는 의미가 성립하지 않는다. 그저 다른 방향을 가리킬 뿐이다. std::unordered_map을 사용할 경우에는 전혀 문제가 되지 않는다. 작거나 큰 순서로 요소를 구분하는 것이 아니라 해시 값으로 구분하기 때문이다. 여기서는 단지 해당 키 타입을 위한 해시 함수와 `equal to==` 연산자를 구현해 두 객체가 같은지 아닌지를 나타내게 된다. 이번 예제에서는 이러한 경우를 시현해본다.


---
### 예제 구현

이번 예제에서는 기본 설정된 해시 함수가 없는 간단한 coord 구조체를 직접 정의해본다. 그리고 coord 값을 숫자로 매핑해 사용하게 된다.

1. 가장 먼저 std::unordered_map을 출력하고 사용할 수 있게 필요한 헤더 파일을 포함시킨다.

``` c++
#include <iostream>

#include <unordered_map>
```

2. 그런 다음 기존의 평범한 해시 함수로는 처리할 수 없는 자신만의 사용자 정의 구조체를 정의한다.

``` c++
struct coord
{
    int x;
    int y;
};
```


3. 해당 구조체를 해시 맵에서 키로 사용하기 위해서는 해시 함수만 필요한 게 아니라 비교 연산자도 구현해줘야 한다.

``` c++

bool operator ==(const coord &l,const coord &r){

    return l.x==r.x && l.y==r.y;

}
```


4. STL 고유의 해시 능력을 최대한 활용하기 위해 ==std 네임스페이스를 열어 자신만의 특수화된 std::hash 템플릿 구조체를 생성한다.== 여타 해시 특수화와 같이 해당 구조체도 같은 using 타입 에일리어스(alias) 절을 포함한다.


``` c++
namespace std
{
    template <>
    struct hash<coord>
    {
      using argument_type=coord;
      using result_type=size_t;  

    result_type operator() (const argument_type &c)const{
        return static_cast<result_type>(c.x)+static_cast<result_type>(c.y);
    }
    };
} // namespace std
```


5. 이때 해다 struct에서의 핵심은 operator() 정의에 있다. 아주 기초적인 해시 기술인 struct coord의 숫자 멤버 변수를 추가하는 단순한 과정이지만, 어떻게 처리하는지 보여주기만 하면 되므로 여기서는 이것으로 충분하다. 훌륭한 해시 함수는 전체 범위에 걸쳐 값을 최대한 균등하게 배분함으로써 해시 충돌을 방지한다.

``` c++
    result_type operator() (const argument_type &c)const{
        return static_cast<result_type>(c.x)+static_cast<result_type>(c.y);
    }
    };
} // namespace std
```


6. 이제 새로운 std::unordered_map 인스턴스를 생성할 수 있게 됐다. 이는 struct coord 인스턴스를 키로 받아 임의의 값으로 매핑한다. 이번 예제의 주 목적은 사용자 고유의 `std::unordered_map` 타입을 사용해보는 것이므로, 이로써 이미 달성했다고 볼 수 있다. 자신만의 타입으로 **해시 기반 맵**을 인스턴스로 만들어 임의의 요소를 채워 넣고 다음을 출력해보자.

``` c++

int main(){

    std::unordered_map<coord,int> m
    {{{0,0},1},{{0,1},2},{{2,1},3}};

     for (const auto & [key, value] : m) {
        std::cout << "{(" << key.x << ", " << key.y << "): " << value << "} ";
    }
    std::cout << "\n";
}
```


7. 프로그램을 컴파일하고 실행하면 다음과 같은 출력물을 얻게 된다.

``` shell
PS C:\C-language\cpp_STL\Chapter02> ./Chapter02_8
{(2, 1): 3} {(0, 1): 2} {(0, 0): 1} 
```


---
### 예제 분석


일반적으로 std::unordered_map과 같이 해시 기반 맵 구현을 인스턴스로 만들 때에는 다음과 같이 작성한다.

``` c++
std::unordered_map<key_type,value_type> my_unordered_map;
```

==컴파일러가 특수화된 std::unodered_map을 생성할 때 마법과 같은 다양한 내용이 뒤에서 처리되고 있다는 사실이 다소 불명확할 수 있다.== 완전한 템플릿 타입 정의를 살펴보자.

``` c++
  template<typename _Key, typename _Tp,
     typename _Hash = hash<_Key>,
     typename _Pred = equal_to<_Key>,
     typename _Alloc = allocator<std::pair<const _Key, _Tp>>>
    class unordered_map
```


처음 두 템플릿 타입은 coord와 int 를 채워 넣은 부분으로, 매우 간단하고 명확하다. ==또 다른 세 개의 선택적 템플릿 타입은 기존의 표준 템플릿 클래스가 자동 입력돼 해당 템플릿 타입을 받는다.== 이 템플릿 타입들은 앞서 선택한 처음 두 파라미터를 기본값으로 넘겨준다.


이번 에제에서 class Hash 템플릿 파라미터를 눈여겨볼 만하다. 명시적으로 정의하지 않으면 class Hash 파라미터는 `std::Hash<key_type>`로 특수화된다. STL는 `std::hash<std::string>`, `std::hash<int>`, `std::hash<unique_ptr>`등과 같이 다양한 타입의 `std::hash` 특수화를 이미 포함하고 있다. 이 클래스들은 각각의 특정 유형에 가장 적합한 해시 값을 계산하기 위해 이들 유형을 어떻게 다뤄야 하는지 잘 알고 있다.

그러나 struct coord의 해시 값을 계산하는 데는 아직 STL이 적합하지 않다. 따라서 이번 예제에서는 이미 다루는 방법을 알고 있는 별도의 특수화를 정의했다. 이제 컴파일러는 모든 std::hash 특수화 목록에서 사용자가 키 타입으로 제시한 해당 타입에 적합한 구현을 찾아낼 수 있게 됐다.

새로운 `std::hash<coord>` 특수화를 추가하지 않고 대신 my_hash_type으로 명명해도 코드를 사용할 수는 있다.

``` c++
std::unordered_map<coord,value_type,my_hash_type> my_unordered_map;
```


그러나 보다시피 타이핑할 내용이 더 많고, 컴파일러가 올바른 해시 구현을 스스로 찾는 것보다 깔끔하지 않다.