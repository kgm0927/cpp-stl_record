

때로는 한 쌍으로 된 키(key)-값(value)으로 맵을 채울 때가 있다. 이때 다음과 같은 두 가지 경우가 있다.

1. 해당 키가 아직 존재하지 않는 경우 새로운 키-값 한 쌍을 생성한다.
2. 해당 키가 이미 존재하지 않는 경우 기존 요소를 수정해 사용한다.

일단 map의 insert나 emplace 함수를 적용해 제대로 실행되는지 살펴볼 수 있다. 제대로 처리되지 않는다면 2번째 경우이므로 기존 요소를 수정한다. 두 경우 모두 insert와 emplace가 삽입하려는 요소를 생성하지만, 2번째 경우에는 새롭게 생성한 요소를 제거한다. 두 경우 모두 쓸모없는 생성사 호출이 발생한다.

C++17에는 try_emplace 함수가 있어 삽입에 따른 조건부 요소를 생성할 수 있다. 전 세계 억만장자들의 목록을 받아 국가별로 부호가 몇 명이 있는지 알려주는 프로그램을 만들어보자. 이에 대해 각 국가에서 가장 부자인 사람을 나타내는 기능을 더해보자. 여기에서 다룰 예제는 요소를 생성하는데 큰 비용이 들지는 않지만, 실제 개발 프로젝트 환경에서는 try_emplace가 얼마나 유용한지 알게 될 것이다.


---
### 예제 구현

이번 예제에서는 억만장자의 목록으로 맵 생성하는 애플리케이션을 구현한다. 그리고 이 맵에서는 각 국가에서 가장 부자인 사람의 참조 값과 각 국가별 억만장자의 수를 알려주는 카운터(counter)를 매핑한다.


1. 늘 그렇듯 가장 먼저 헤더 파일을 포함하고, 기본적으로 std 네임스페이스 사용을 선언한다.

``` c++
#include <iostream>
#include <functional>
#include <list>
#include <map>

using namespace std;
```

2. 이제 목록에 들어갈 억만장자를 나타내는 구조체를 정의한다.

``` c++
  

struct billionaire {
    string name;
    double dollars;
    string country;
};
```


3. 메인 함수에서 먼저 억만장자의 목록을 정의한다. 실제 전 세계 억만장자의 숫자는 매우 많으므로 일부 국가에서 가장 부자인 사람들로만 제한된 목록을 생성하자. 이 목록은 이미 정렬돼 있다. 
``` c++
int main()
{
    list<billionaire> billionaires {
        {"Bill Gates", 86.0, "USA"},
        {"Warren Buffet", 75.6, "USA"},
        {"Jeff Bezos", 72.8, "USA"},
        {"Amancio Ortega", 71.3, "Spain"},
        {"Mark Zuckerberg", 56.0, "USA"},
        {"Carlos Slim", 54.5, "Mexico"},
        // ...
        {"Bernard Arnault", 41.5, "France"},
        // ...
        {"Liliane Bettencourt", 39.5, "France"},
        // ...
        {"Wang Jianlin", 31.3, "China"},
        {"Li Ka-shing", 31.2, "Hong Kong"}
        // ...
    };
```

4. 이제 맵을 정의해보자. 국가 문자열을 한 쌍으로 매핑한다. 여기에는 목록의 각 국가에서 첫 번째 부자의 사본(const)이 포함돼 있다. 이는 자동으로 해당 국가의 가장 부자인 사람을 나타낸다. ==이 문자열 한 쌍에 들어있는 또 다른 변수는 카운터==로, 그 국가 내에 다음으로 부자인 사람마다 값을 하나씩 증가시킨다.

``` c++
    map<string, pair<const billionaire, size_t>> m;
```


5. 이제 목록으로 돌아가 각 국가에 새롭게 적재될 한 쌍을 포함하자. 여기에는 현재 나와 있는 억만장자의 참조 값과 카운터 값 1이 들어있다.
``` c++
    for (const auto &b : billionaires) {

        auto [iterator, success] =m.try_emplace(b.country, b, 1); // try_emplace함수가 오류가 뜨지만, 컴파일 하는 데에는 문제가 없다.
```

6. 이 과정이 성공적으로 실행되면 더는 처리할 것이 없다. 앞서 제공한 생성자 인수, b, 1의 한 쌍이 생성됐고, 이를 맵에 삽입했다. 그러나 국가 키가 이미 존재하는 경우 삽입에 실패하고, 해당 데이터가 생성되지 않는다. 억만장자의 구조체가 매우 컸더라면 이를 복사하는 실행 비용이 크게 절약될 수 있었을 것이다.
삽입이 성공하지 않은 경우에도 해당 국가에 대해 카운터 값을 증가시켜줘야 하는 과정이 남아 있다.

``` c++
  if (!success) {

            iterator->second.second += 1;

        }
```

7. 자, 이제 끝났다. 그리고 국가별로 억만장자가 몇 명이 있는지 출력하고, 그 중 가장 부자는 누군 출력한다.

``` c++
  

    for (const auto & [key, value] : m) {

        const auto &[b, count] = value;

  

        cout << b.country << " : " << count << " billionaires. Richest is "

             << b.name << " with " << b.dollars << " B$\n";

    }
```

8. 프로그램을 컴파일하고 실행하면 다음과 같이 출력된다(물론 앞에서 입력 맵을 제한했기 때문에 출력 결과물 또한 제한적이다).

``` c++
PS C:\C-language\cpp_STL\Chapter02> ./Chapter02_5
China : 1 billionaires. Richest is Wang Jianlin with 31.3 B$
France : 2 billionaires. Richest is Bernard Arnault with 41.5 B$
Hong Kong : 1 billionaires. Richest is Li Ka-shing with 31.2 B$
Mexico : 1 billionaires. Richest is Carlos Slim with 54.5 B$
Spain : 1 billionaires. Richest is Amancio Ortega with 71.3 B$
USA : 4 billionaires. Richest is Bill Gates with 86 B$
PS C:\C-language\cpp_STL\Chapter02> 



```


---
### 예제 분석


이번에 살펴본 전체 예제는 std::map의 try_emplace 함수를 중심으로 이뤄져 있다. C++17에 새롭게 추가된 이 기능은 다음과 같은 서명을 가진다.

``` c++
std::pair<iterator,bool> try_emplace(const key_type& k, Args&& ... args);
```

삽입되는 키는 파라미터 k이고, 이와 연관된 값은 파라미터 묶음인 args로부터 생성된다. **요소를 삽입하는데 성공했다면 해당 함수가 반복자를 반환한다.** ==이 반복자는 true로 설정된 불리언 값과 짝지어진 새로운 노드를 가리킨다.== ==요소를 삽입하는데 실패했다면 반환되는 쌍에서 불리언 값이 false로 되며, 반복자는 충돌된 새로운 요소를 가리키게 될 것이다.==

이러한 특징은 이번 예제의 경우 매우 유용하다. 즉, 가장 먼저 특정 국가에 어떤 억만장자의 이름을 봤을 때 해당 국가는 맵에서 아직 키가 아닌 경우다. 이런 경우 새로운 카운터를 1로 설정해서 키를 삽입해야 한다. ==그러나 반대로 이미 특정 국가에서 억만장자가 나타나 있는 경우 기존 카운터를 참조해 증가시켜줘야 한다.== 이 과정은 앞의 6단계에서 했던 것과 같다.


``` c++
if (!success) {
            iterator->second.second += 1;
        }
```

>[!info]
>std::map의 insert와 emplace 함수 모두 똑같은 방식으로 작동한다는 것을 기억하자. 주요 차이점은 키가 이미 존재하는 경우 try_emplace가 해당 키와 관련된 객체를 생성하지 않는 다는 점이다. 이는 특히 해당 타입의 객체 생성 비용이 매우 높은 경우 성능 향상에 매우 큰 도움이 된다.



---
### 부연 설명

==맵을 std::map에서 std::unordered_map으로 변경해도 전체 프로그램은 그대로 작동한다. 이런 방식으로 다른 구현으로 간단하게 변경해 각기 다른 특징의 성능을 낼 수 있다.== 이번 예제에서 유일하게 식별할 수 있는 차이점은 해당 맵이 알파벳순으로 출력되지 않는다는 것이다. ==이는 해시 맵이 해당 객체를 검색 트리가 하는 것과는 다른 방식으로 정렬하기 때문이다.==