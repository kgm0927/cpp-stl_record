

==배열과 벡터는 적재된 객체를 스스로 정렬하지 않는다.== 그러나 정렬이 필요할 때마다 데이터 구조체로 전환해야 할 필요는 없다. 이는 자동으로 정렬될 수 있다. 개발 환경에 std::vector가 잘 부합한다면 여기에 원하는 정렬 조건으로 요소를 추가하는 것 또한 매우 간단하고 유용할 것이다.


---

### 예제 구현

이번 예제에서는 std::vector에 무작위 단어를 넣고 정리한 다음, 또 다른 언어를 추가했을 때도 설정해 놓은 벡터의 단어 정렬을 유지하게 만든다.

1. 가장 먼저 필요한 모든 헤더 파일을 포함시킨다.

``` c++
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
#include <iterator>
#include <cassert>
```

2. 그리고 std 네임스페이스 사용을 정의해 std:: 접두사를 사용할 필요가 없게 한다.
``` c++
using namespace std;
```


3. 그런 다음 간단한 메인 함수를 작성해 일련의 무작위 문자열로 벡터를 채운다.

``` c++
int main(){

    vector<string> v{"some","random","words","without","order","aaa","yyy"};
```


4. 다음은 해당 벡터를 정렬하는 과정이다. 앞서 STL로부터 is_sorted 함수와 assert를 이용한다. 이를 통해 처리 전에는 정렬되지 않았으나 처리 후 정렬된 벡터를 보여준다.

``` c++
  
    assert(false==is_sorted(begin(v),end(v)));
    sort(begin(v),end(v));
    assert(true==is_sorted(begin(v),end(v)));
```


5. 이제 새로운 isert_sorted 함수를 이용해 드디어 무작위 단어를 정렬된 벡터에 추가해보자. 이 함수는 나중에 따로 구현할 것이다. 이 무작위 단어들은 오른쪽에 배치해 벡터가 나중에도 정렬될 수 있게 한다.

``` c++
    insert_sorted(v,"foobar");
    insert_sorted(v,"zzz");
```

6. 그리고 이제 소스 파일의 앞쪽에 insert_sorted를 구현해보자.

``` c++
void insert_sorted(vector<string> &v,const string &word){
    const auto insert_pos(lower_bound(begin(v),end(v),word));
    v.insert(insert_pos,word);
}
```

7. 이제 아까 다뤘던 메인 함수로 돌아가 이어서 벡터를 출력해 삽입한 처리 과정이 잘 작동하는지 확인한다.

``` c++
  

    for (const auto &w : v)
    {
        cout<<w<<" ";
    }
    cout<<endl;
```


8. 프로그램을 컴파일하고 실행하면 다음과 같이 깔끔하게 정렬된 결과가 출력된다.

``` c++
PS C:\C-language\cpp_STL\Chapter02> ./Chapter02_4
aaa foobar order random some without words yyy zzz 
```

---
### 예제 분석

이번 예제를 통해 전체 프로그램이 **insert_sorted** 함수를 중심으로 구성돼 있는 점을 알게 됐다. ==새로운 문자열을 추가할 때마다 정렬된 벡터에 배치해 벡터 내의 문자열 정렬을 보존한다.== 그러나 여기서 전제 조건은 해당 벡터가 이미 정렬돼 있다는 점이다.


삽입 위치를 정하는 과정은 STL 함수인 **lower_bound**로 처리한다. 이때 세 개의 인자를 받는다. 그중 ==두 개의 인자는 범위의 시작과 끝을 나타낸다.== 이번 예제에서 범위는 벡터다. 세 번째 인자는 삽입하는 단어다. 그러면 함수가 범위 내의 세 번째 파라미터보다 크거나 같은 첫 번째 요소를 찾아내 이를 가리키는 반복자를 반환한다.

올바른 위치를 얻기 위해서는 해당 값을 두 개의 인자만 받는 std::vector의 멤버함수 insert에 넣었다. 첫 번째 인자는 반복자인데, 두 번째 파라미터가 삽입될 때 벡터 내 위치를 가리킨다. 같은 반복자를 사용하는 이 방법은 매우 간편했다. 해당 반복자를 lower_bound 함수로 끝내기만 하면 된다. 그리고 예상했듯이 두 번째 인자는 삽입될 요소다.

---
### 부연 설명

insert_sorted 함수는 매우 일반화돼 있다. ==파라미터의 타입을 좀 더 일반화시키려면 다른 컨테이너의 적재 타입은 물론 std::set, std::deque, std::list 등과 같은 다양한 컨테이너에까지 적용할 수 있다.== 이러한 세트(set)는 lower_bound 멤버 함수를 갖고 있다는 것을 염두해두자. 이 함수는 std::lower_bound와 같은 역할을 하지만 해당 컨테이너에 특화돼 있어 좀 더 효율적이다.

``` c++
template<typename C,typename T>

void insert_sorted(C &v,const T &item){

    const auto insert_pos(lower_bound(begin(v),end(v),item));

    v.insert(insert_pos,item);

}
```

---
- [[std__vector에서 삭제-제거 관용구 사용]]
- [[필요한 조건을 걸어 효율적으로 std__map에 요소 삽입]]
