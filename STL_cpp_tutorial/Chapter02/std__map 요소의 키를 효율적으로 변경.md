

키가 항상 고유한 값으로 정렬될 수 있게 std::map 데이터 구조체가 키에서 값으로 매핑하기 때문에 사용자가 변경할 수 없는 맵 노드 키로 이미 삽입돼 있다. 완벽하게 구별된 맵 노드의 키 요소를 사용자가 임의로 변경할 수 없게끔 하기 위해 const 수식어가 키 타입에 추가돼 있다.


이러한 제약이 있는 이유는 std::map을 잘못된 방향으로 사용하는 것을 방지하기 위해서다. 그러나 특정 맵 요소의 키를 정말 반드시 변경해야 할 때는 어떻게 해야 할까?

==C++17 이전 버전에는 트리에서 변경할 키 값 요소를 제거한 다음 다시 재삽입을 해줘야 했다.== 이 방법의 좋지 않은 점은 쓸데없이 일정 메모리를 재할당하는 과정이 따르기 때문에 결코 좋은 영향을 끼친다고 할 수 없었다는 점이다.

C++17 이후부터는 메모리 재할당 없이 맵 노드를 삭제하고 재삽입할 수 있게 됐다. 이번 예제에서는 이 방법을 알아보자.


---
### 예제 구현


간단한 애플리케이션을 하나 구현할 것이다. 이 애플리케이션은 std::map 구조체 안에 자동차 경주를 임의로 설정해놓고 참가 운전자의 배치 순서를 결정한다. 레이스 도중 한 운전자가 다른 운전자를 제치고 앞서갈 때 이 운전자들의 배치 키를 변경해줘야 한다. 여기서는 C++17의 새로운 방법으로 이를 구현해 볼 것이다.


1. 가장 먼저 헤더 파일을 포함시키고 std 네임스페이스의 사용을 선언한다. 

``` c++
#include <iostream>
#include <map>

  

using namespace std;
```


2. 맵 구조체를 처리하기 이전과 이후의 자동차 경주 배치를 출력할 것이다. 이를 위해 간단한 헬퍼 함수를 구현한다.

``` c++
template <typename M>

void print(const M &m)

{
    cout << "Race placement:\n";
    for (const auto &[placement, driver] : m) {
        cout << placement << ": " << driver << '\n';
    }
}
```


3. 메인 함수에서는 운전자의 위치를 이름과 함께 문자열로 나타내는 정수 값으로 된 맵을 인스턴스로 만들고 초기화한다. 해당 맵을 출력한 후 다음 단계에서 이를 수정한다.

``` c++
int main()
{
    map<int, string> race_placement {
        {1, "Mario"}, {2, "Luigi"}, {3, "Bowser"},
        {4, "Peach"}, {5, "Yoshi"}, {6, "Koopa"},
        {7, "Toad"}, {8, "Donkey Kong Jr."}
    };

  

    print(race_placement);
```

4. 경주 한 바퀴를 도는 동안 쿠파(Bowser)에게 작은 사고가 일어나서 맨 꼴찌가 되고, 이를 기회 삼아 원래 꼴찌였던 동키콩(Donkey kong) 주니어가 3등으로 들어왔다고 가정해보자. 이 경우 맵에서 쿠파와 동기콩 주니어의 맵 노드를 추출해야 한다. 이렇게 해자지만 이 둘의 키를 조작할 수 있다. 여기서 사용하는 extract 함수는 C++17의 새로운 기능 중 하나다. 이 함수는 할당 처리로 발생하는 어떤 부작용도 없이 맵으로부터 특정 요소를 효과적으로 제거한다. 이번 작업을 위해 새로운 영역을 열어보자.

``` c++
  

    {

        auto a (race_placement.extract(3)); // 현재 extract 함수를 찾을 수 없다고 하지만, 실행 자체는 가능하다.

        auto b (race_placement.extract(8));
```

5. 이제 쿠파와 동키콩 주니어의 키가 서로 바뀌었다. 맵 노드 키는 const로 정의돼 보통 변경되지 않지만, extract 함수를 이용해 추출한 해당 요소의 키를 변경할 수 있다.

``` c++
        swap(a.key(), b.key());
```

6. std::map의 insert 함수는 C++17의 새로운 오버로드 기능을 갖고 있다. 즉, 할당자의 힘을 빌리지 않도고 삽입할 수 있게 추출된 노드를 다룰 수 있다.

``` c++
     race_placement.insert(move(a));

        race_placement.insert(move(b));

    }
```

7. 해당 영역을 벗어나면 처리 과정이 끝난다. 새로운 경주 순위 배치를 출력한 후 애플리케이션을 종료한다.

``` c++
  

    print(race_placement);
}
```

8. ==프로그램을 컴파일하고 실행하면 다음과 같은 출력물을 얻는다.== 갱신된 맵 인스턴스에서 경주 배치가 먼저 나타난 후 뒤이어 쿠파와 동키콩 주니어의 위치가 바뀐 배치가 나타난다.

``` shell
PS C:\C-language\cpp_STL\Chapter02> ./Chapter02_7
Race placement:
1: Mario
2: Luigi
3: Bowser
4: Peach
5: Yoshi
6: Koopa
7: Toad
8: Donkey Kong Jr.
Race placement:
1: Mario
2: Luigi
3: Bowser
4: Peach
5: Yoshi
6: Koopa
7: Toad
8: Donkey Kong Jr.
Race placement:
1: Mario
2: Luigi
3: Donkey Kong Jr.
4: Peach
5: Yoshi
6: Koopa
7: Toad
8: Bowser
```


---
### 예제 분석

C++17에서는 std::map에 새롭게 추가된 추출 함수가 있다. 이 함수에는 다음과 같은 두 가지 종류가 있다.

``` c++
node_type extract(const_iterator position);
node_type extract(const key_type& x);
```

이번 예제에서는 두 번째 함수를 사용해서 키를 받고 해당 키 파라미터와 일치하는 맵 노드를 찾아 추출하는 과정을 살펴봤다. 첫 번째 함수는 반복자를 받는데, 해당 요소를 검색할 필요가 없기 때문에 좀 더 빠르다.

두 번째 함수로는 존재하지 않는 요소(키를 이용해 검색하는 요소)를 추출할 경우 빈 node_type 인스턴스를 반환하게 된다. empty() 함수는 불리언 값을 반환해 node_type 인스턴스가 비어있는지 아닌지를 알려준다. ==비어 있는 인스턴스에 대해 어떤 함수의 접근도 정의되지 않은 행동이 발생한다.==

노드를 추출한 후 key() 함수를 이용해 해당 키를 변경할 수 있다. 이렇게 하면 일반적으로 키는 const이지만 nonconst 접근이 가능하다.


노드를 맵에 다시 삽입하기 위해 insert 함수로 해당 노드를 옮겼던 것을 기억하자. 불필요한 복사와 할당을 피하는 것이 extract 함수를 사용하는 근본적인 이유이기 때문이다. 여기서 node_type 인스턴스를 옮기지만, 이것이 실제로 컨테이너 값을 옮기는 것이 아니라는 점에 주목하자.

---
### 부연 설명

추출 함수로 이용해 추출된 맵 노드는 매우 다양한 방면으로 활용할 수 있다. 하나의 맵 인스턴스로부터 노드를 추출해 다른 맵이나 멀티맵(multimap) 인스턴스에 삽입할 수도 있다. 또한 unordered_map과 unordered_multimap 인스턴스에서 동작할 수도 있고, set/multiset처럼 unordered_multiset에서도 사용할 수도 있다.

서로 다른 맵이나 세트 구조체 간 요소를 이동시킬 때에는 키와 값, 할당자가 모두 같은 타입이어야 한다. 그러나 이러한 경우에도 map과 unordered_map이나 set과 unordered_set에서 노드를 이동시키는 것은 불가능하다.