

std::map은 특정 대상을 정렬해 해당 데이터의 통계를 낼 때 매우 유용하다. 객체 범주를 나타내는 각각의 키에 변경할 수 있는 페이로드(payload) 객체를 첨부하면 단어 빈도수 막대그래프와 같은 것들을 모우 간단하게 구현할 수 있다. 이번 예제에서는 이러한 내용을 다룰 것이다.

---
### 예제 구현


이번 예제에서는 짤막한 글의 문서 파일과 같이 표준 입력으로부터 모든 사용자 입력을 읽어와 처리한다. 입력 내용을 단어로 토큰화해서 어느 단어가 얼마나 자주 반복해 등장하는지도 계산한다.


1. 언제나처럼 사용될 데이터 구조체에서 필요한 모든 헤더 파일을 포함시킨다.

``` c++

#include <iostream>
#include <map>
#include <vector>
#include <algorithm>
#include <iomanip>
```


2. 코드 작성을 줄이기 위해 std 네임스페이스의 사용을 선언한다.

``` c++
using namespace std;
```

3. 나중에 덧붙여진 쉼표와 마침표, 콜론을 단어로부터 분리해내기 위해 헬퍼 함수 하나를 사용한다.

``` c++
string filter_punctuation(const string &s){

    const char *forbidden{".,:;"};
    const auto idx_start (s.find_first_not_of(forbidden));
    const auto idx_end(s.find_last_not_of(forbidden));
    return s.substr(idx_start,idx_end-idx_start+1);

}
```


4. 이제 실제 프로그램 작업을 시작해보자. ==특정 단어 사용 빈도수 카운터에 보이는 모든 단어와 연관된 맵을 수집한다.== 추가로 지금까지 등장한 단어 중 가잔 긴 단어의 길이를 기록하는 변수를 갖는다. 이렇게 하면 나중에 작업한 내용을 출력할 때 단어 빈도수 도표를 깔끔하게 들여 쓰는데 도움이 된다.

``` c++
  

int main(){
    map<string,size_t> words;
    int max_word_len{0};
```

5. std::cin에서 std::string 변수로 나르는 도중 해당 입력 스트림이 공백을 제거할 것이다. 이렇게 하면 입력 내용을 단어별로 얻을 수 있다.

``` c++
string s;
while(cin>>s){


```

6. 여기서 나온 단어에는 쉼표나 마침표, 콜론 등이 있을 수 있는데, 마지막 문장에서 대개 사용된다. 앞서 정의한 헬퍼 함수를 이용해 이러한 쉼표, 마침표, 콜론을 분리해낸다.

``` c++
        auto filtered (filter_punctuation(s));
```


7. 해당 단어가 지금까지 나온 단어 중 가장 길 경우 max_word_len 변수를 갱신해줘야 한다.

``` c++
        max_word_len=max<int>(max_word_len,filtered.length());
```


8. 이제 word 맵 안에서 단어의 카운터 값을 증가시킨다. 처음에는 증가시키기 전에 암묵적으로 생성된다.

``` c++
++word[filtered];}
```

9. 반복문이 종료되면 words 맵에서 입력 스트림으로부터 모든 고유 단어를 저장했다는 것을 알 수 있다. 이는 각 단어의 빈도수를 나타내는 카운터와 짝을 짓는다. 해당 맵은 단어들을 키로 사용하며, 이는 알파벳 순서로 정렬돼 있다. 여기에서 목표는 빈도수에 따라 모든 단어를 출력해 정렬시키는 것이므로, 가장 높은 빈도수를 가진 단어가 먼저 와야 한다. 이러한 결과를 얻기 위해서는 먼저 이 모든 단어-빈도수 쌍이 들어갈 벡터의 인스턴스를 만들고, 이를 맵에서 벡터로 옮겨놓는다.

``` c++
   vector<pair<string,size_t>> word_counts;
    word_counts.reserve(words.size());
    move(begin(words),end(words),back_inserter(word_counts));
```


10. 현재 벡터의 모든 단어-빈도수 쌍은 아직 원래 words 맵과 같은 순서로 들어있다. 이제 이를 다시 정렬해 가장 빈도수가 높은 단어를 처음에 두고 가장 빈도수가 낮은 단어를 마지막에 놓는다.

``` c++
    sort(begin(word_counts),end(word_counts),[](const auto &a,const auto &b){
        return a.second> b.second;
    });
```


11. 이제 모든 데이터가 원하는 순서대로 놓였으므로, 이를 사용자 터미널에 출력한다. `std::setw`를 사용해 깔끔한 들여쓰기 형태의 서식으로 스트림 데이터를 정리해 테이블과 같은 모습으로 만든다.

``` c++
 cout<<"# "<<setw(max_word_len)<<"<WORD>"<<" #<COUNT>\n";
    for (const auto &[word,count]: word_counts)
    {
        cout<<setw(max_word_len+2)<<word<<" #"<<count<<'\n';
    }
```


12. 프로그램을 컴파일한 후 어떤 문자 파일이든 여기에 넣으면 다음과 같은 빈도 수 목록을 얻을 수 있다.
``` c++
# <WORD> #<COUNT>
PS C:\C-language\cpp_STL\Chapter02> cat lorem.txt | ./Chapter02_11      
#       <WORD> #<COUNT>
            et #574
         dolor #302
           sed #273
          diam #273
```


---

### 예제 분석


이번 예제는 모든 단어를 std::map 으로 수집하고, 다시 모든 요소를 해당 맵에서 꺼내 std::vector에 넣는데 중점을 뒀다. 그러면 다르게 정렬한 다음 데이터를 출력한다.

다음의 예제를 살펴보자. 'a a b c b b b d c c'라는 문자열에서 단어 빈도수를 세면 다음과 같은 맵의 내용물을 얻게 된다.


``` 
a -> 2
b -> 4
c -> 3
d -> 1
```


그러나 이것은 사용자에게 보여주고자 하는 정렬 순서가 아니다. 원하는 결과물은 가장 높은 빈도수를 가진 b를 먼저 출력해야 한다. 그런 다음 c, a, d 순서가 돼야 할 것이다. ==아쉽게도 '가장 큰 연관 값의 키'나 '두 번째로 큰 연관 값의 키'를 출력하라고 맵에 직접 요구하는 것은 불가능하다.==

그래서 여기에 벡터가 필요하다. 문자열과 카운터 값 쌍을 포함하는 내용을 벡터에 입력한다. 이런 방식을 통해 맵에서 꺼낸 데이터 형태를 그대로 벡터에 담을 수 있다.

``` c++
    vector<pair<string,size_t>> word_counts;
```

그런 다음 `std::move` 알고리즘을 이용해 벡터를 단어-빈도수 쌍으로 채운다. 이 ==방법의 장점은 힙에 보관되는 문자열 부분이 중복 없이 맵에서 벡터로 옮길 수 있다는 것이다.== 이렇게 하면 불필요한 복사를 피할 수 있다.


``` c++
    move(begin(words),end(words),back_inserter(word_counts));
```

>[!info]
>일부 STL 구현 과정은 짧은 문자열 최적화를 거치기도 한다. 문자열이 별로 길지 않을 경우 힙에 할당되지 않고 문자열 객체에 직접 저장된다. 이러한 경우 옮기는 속도가 그다지 빨라지지는 않지만 느려지지도 않는다.


다음에 나오는 흥미로운 단계는 **람다를 사용자 지정 비교 연산자로 사용하는 과정**이다.

``` c++
 sort(begin(word_counts),end(word_counts),[](const auto &a,const auto &b){
        return a.second> b.second;
    });
```


이 정렬 알고리즘은 요소들을 한 쌍으로 받아 비교한다. 이것이 정렬 알고리즘이 하는 역할이다. 이러한 람다 함수를 제공하면 단순히 기본 구현인 a가 b보다 작은지 아닌지만 비교하지 않고 a.second가 b.second보다 큰지 아닌지도 비교한다. ==여기서 주목할 것은 모든 객체가 문자열과 해당 카운터 값으로 이뤄지 한쌍이라는 것과 a.second를 입력해 특정 단어의 카운터 값에 접근한다는 점이다.== 이렇게 하면 높은 빈도수 단어를 전부 벡터의 앞부분으로 옮기고 낮은 빈도수 단어는 끝으로 옮긴다.


