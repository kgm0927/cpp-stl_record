

std::stack은 어댑터 클래스로, 실제 물건을 쌓아 올리듯이 객체를 위로 밀어 올리거나 다시 끌어내릴 수 있게 한다. 이번 예제에서는 데이터 구조체에 **역폴란드 표기법**(RPN, reverse polish notation) 계산기를 생성해 이를 어떻게 사용하는지 살펴보자.


RPN은 수학적 표현식을 나타낼 때 사용하는 개념으로, 분석하기가 매우 간단하다. RPN에서 $1+2$는 $1 2 +$이다. 피연산자가 먼저 오고 연산이 그 뒤에 온다. 또 다른 예시가 있다. $(1+2)*3$은 RPN에서 $12+3*$인데, 이를 보면 RPN이 매우 간단히 분석할 수 있는 이유를 눈치 챘을 것이다. 부분식을 정의하기 위해 괄호가 별도로 필요하지 않다.



---
### 예제 구현

이번 예제에서는 RPN을 이용해 표준 입력으로 수학식을 읽어내 이를 함수로 념겨줘 평가한다. 그리고 마지막에는 숫자로 된 결과물을 사용자에게 출력한다.


1. STL로부터 많은 헬터 함수를 사용할 것이므로, 그중 몇 개를 다음과 같이 포함시킨다.

``` c++

#include <iostream>
#include <stack>
#include <iterator>
#include <map>
#include <sstream>
#include <cassert>
#include <vector>
#include <stdexcept>
#include <cmath>
```

2.  그리고 코드 작성을 줄여줄 std 네임스페이스의 사용을 선언한다.


``` c++
using namespace std;
```

3. 그런 다음 곧바로 RPN 파서 구현을 시작한다. RPN 파서는 문제는 내에서 수학식의 시작과 끝을 나타내는 반복자 쌍을 받는다. 이는 토큰 하나하나씩 처리될 것이다.

``` c++
template <typename IT>
double evaluate_rpn(IT it, IT end)
{
```


4. 토큰을 순환하는 동안 연산이 보일 때까지 나오는 모든 피연산자를 기억해야 한다. 이 피연산자들이 바로 쌓아 올릴 대상이다. 모든 숫자를 분석해 double 정밀도의 부동소수점에 저장하면 double 값으로 쌓이게 된다.

``` c++
    stack<double> val_stack;
```


5. ==스택에 있는 내용물에 편리하게 접근하기 위해 헬퍼 함수를 구현한다.== 가장 위에 있는 요소를 끌어내 반환함으로써 해당 스택을 수정한다. 이렇게 하면 나중에 한 단계만으로 원하는 작업을 처리할 수 있다.
``` c++
    auto pop_stack ([&](){ auto r (val_stack.top()); val_stack.pop(); return r; });
```


6. 또 다른 준비 단계로는 지원하는 수학 연산 전부를 정의한다. 이를 맵에 저장해서 모든 연산 토큰을 실제 연산과 연결한다. 이때 연산은 호출 가능한 람다(lambda)로 표현하는데, 피연산자 두 개를 받아 더하거나 곱한 후 결과물을 반환한다.

``` c++
 map<string, double (*)(double, double)> ops {
        {"+", [](double a, double b) { return a + b; }},
        {"-", [](double a, double b) { return a - b; }},
        {"*", [](double a, double b) { return a * b; }},
        {"/", [](double a, double b) { return a / b; }},
        {"^", [](double a, double b) { return pow(a, b); }},
        {"%", [](double a, double b) { return fmod(a, b); }},
    };
```


7. 이제 드디어 입력 내용을 순환할 수 있다. 입력 반복자가 문자열을 제공하는 가정하에 각 토큰당 새로운 `std::stringstream`을 넘겨주는데, 이는 숫자를 피싱할 수 있기 때문이다.

``` c++
    for (; it != end; ++it) {
        stringstream ss {*it};
```


8. 이제 각각 모든 토큰으로부터 double 값을 얻어내 보자. 성공적으로 처리되면 스택을 쌓아놓은 피연산자를 갖는다.
``` c++
if (double val; ss >> val) {

            val_stack.push(val);

        }
```

9. 성공하지 못하더라도 연산자의 문제는 아니다. 사용자가 다루는 모든 연산은 2항으로 돼 있으므로, 스택으로부터 가장 마지막 두 피연산자를 꺼내야 한다.

``` c++
else{
  const auto r {pop_stack()};

            const auto l {pop_stack()};
```

10. it 반복자를 참조해 이미 문자열로 나타내는 피연산자를 얻는다. ops 맵을 질의해서 파라미터로 l과 r 두 개의 피연산자를 허용하는 람다 객체를 얻는다.

``` c++
try{
                val_stack.push(ops.at(*it)(l, r));
                const double result {op(l,r)};
                val_stack.push(result);
}
```

11. 수학적 응용 부분을 try 절로 감싸서 혹시 있을지 모를 예외의 경우를 잡아낼 수 있게 한다. 알 수 없는 수학 연산을 사용자가 제시하면 맵의 at 호출이 out_of_range 예외를 던진다. 이러한 경우 invalid argument로 명확하지 않은 해당 문자열을 나르는 예외를 다시 던지게 된다.

``` c++
} catch (const out_of_range &) {

                throw invalid_argument(*it);

            }

```

12. 이게 끝이다. 반복문이 종료하면 스택에 최종 결과물이 나온다. 그러면 이것을 그냥 구현하면 된다(여기서 스택 크기가 1이면 어설트(assert)가 발생한다. 그 외에도 연산이 없어서 어설트가 발생할 수 있다).

``` c++
        }
    }

    return val_stack.top();
}
```


13. 앞서 언급했던 간단한 RPN 파서를 사용해보자. 이 처리를 위해 표준 입력을 std::istream_iterator 쌍으로 감싸 넣은 다음, 이를 RPN 파서 함수로 넘겨준다. 마지막으로 결과물을 출력한다.

``` c++

int main()
{
    try {
        cout << evaluate_rpn(istream_iterator<string>{cin}, {}) << '\n';
    }

```


14. 코드를 try 절로 다시 감싸 넣은 것을 알 수 있는데, 이는 사용자 입력에 우리가 구현하지 않은 연산자가 있을 수 있기 때문이다. 이러한 경우 던져놓은 예외를 잡아내 다음과 같은 오류 메시지를 출력한다.

``` c++
 catch (const invalid_argument &e) {

        cout << "Invalid operator: " << e.what() << '\n';

    }
```


15. 프로그램을 컴파일한다. "3 1 2 + * 2 /"는 (3*(1+2))/2로 나타내며, 다음과 같이 올바른 결과물을 산출한다.


``` c++
PS C:\C-language\cpp_STL\Chapter02> echo "3 1 4 + * 2 /" | ./Chapter02_10
7.5
```


---

### 예제 분석


이번 예제에서는 피연산자를 스택에 추가해 입력 내용에서 연산이 보일 때까지의 과정을 전반적으로 보여준다. 스택에서 가장 마지막 두 피연산자를 꺼내 원하는 연산을 적용한 후 결과물을 스택에 다시 추가한다. 이번 예제에서 다룬 모든 코드를 제대로 이해하기 위해서는 피연산자와 연산을 입력된 내용과 어떻게 구분하는지, 스택을 어떻게 다루는지, 그리고 올바른 수학 연산을 어떻게 선택하고 적용하는지를 이해해야 한다.


- **스택 다루기**

스택에 요소를 넣을 때에는 다음과 같이 간단하게 `std::stack`의 `push`함수를 사용한다.

``` c++
val_stack.push(val);
```

앞에서 val_stack 객체에 참조 값을 담기 위한 람다를 구현해서 원하는 겂을 꺼내는 처리 과정이 조금 까다로워 보일 수 있다. 주석을 추가해 보강한 코드를 살펴보자.

``` c++
auto pop_stack([&]{
auto r (val_stack.top()); // 가장 위의 값을 복사
val_stack.pop(); // 가장 위의 값을 버림
return r;        // 복사본을 반환
})
```

이 람다는 스택의 가장 위에 있는 값을 구해 한 번에 제거해야 한다. std::stack의 인터페이스로는 이러한 처리 과정을 한 번의 호출로 구현할 수 없다. 그러나 람다로 쉽고 빠르게 정의함으로서 다음과 같이 값을 얻을 수 있다.

``` c++
double top_value{pop_stack()};
```


- **사용자 입력에서 연산자 피연산자 구분**

evaluate_rpn의 주요 반복문에서 반복자로부터 현재의 문자열 토큰을 받아 이것이 피연산자인지 아닌지 확인한다. 문자열이 double 변수로 파싱될 수 있다면 이는 곧 숫자라는 의미이고, 따라서 피연산자가 된다. 숫자로 쉽게 분석될 수 없는 것은 모두 연산자로 간주한다. 예를 들어 '+'와 같은 것이 여기에 해당한다.

이번 과제에 사용하게 될 코드의 기본 뼈대는 다음과 같다.

``` c++
stringstream ss{*it};
if(double val ; ss>>val){
// 이는 숫자다!
}else{
// 이는 숫자 이외의 것, 즉 연산자가 된다!
}
```

스트림 연산자 >>는 해당 요소가 숫자인지 아닌지 알려준다. 가장 먼저 문자열을 std::stringstream으로 감싸 넣는다.  그런 다음 std::string으로부터 double 변수로 파싱하기 때문에 stringstream 객체의 허용치를 이용한다. 숫자로 파싱하라고 명령했기 때문에 이 분석이 실패하면 곧 숫자가 아니라는 것을 알게 된다.


- **올바른 수학 연산을 선택과 적용**

현재 사용자 입력 토큰이 숫자가 아니라는 것을 알고 나면 이것이 +나 `*`와 같은 연산이라고 간주한다. 그리고 앞서 ops라고 이름 지은 맵을 질의해 연산을 찾은 후 함수로 반환한다. ==이 함수는 두 개의 피연산자를 받아 합계와 같은 적절한 결과물을 반환한다.==

맵 타입 자체만 보면 다소 복잡해 볼 수 있을 것이다.


``` c++
map<string,double (*)(double,double)>ops{...};
```

이 맵은 string으로부터 `double (*)(double,double)`을 매핑한다. 여기서 뒷부분은 무엇을 뜻하는 걸까? 해당 타입 서술 부분은 "두 개의 double을 받는 함수의 포인터며, double을 반환한다"는 의미다. `(*)` 부분을 double sum(double,double)과 같이 함수 이름을 나타낸다고 생각해보자. 그러면 훨씬 수월하게 알아볼 수 있을 것이다. 여기서 트릭은 람다 표현식 `[](double, double){return /*some double*/}`를 실제 해당 포인터의 서술과 일치하는 함수 포인터로 전환될 수 있다는 것이다. 아무것도 담지 못하는 람다는 보통 함수 포인터로 전환할 수 있다.

이러한 방식을 이용하면 맵에서 올바른 연산을 손쉽게 얻어낼 수 있다.


``` c++
const auto & op{ops.at(*it)};
const auto & result {op(l,r)};
```

맵은 또 다른 기능을 내재하고 있다. `ops.at("foo")`를 호출하면 "foo"는 유효한 키 값이 되지만, 이와 같은 이름의 연산을 저장해놓은 적이 없다. 이럴 때 해당 맵은 예외를 던져 이번 예제에서 다룬 바와 같이 잡아낼 수 있다. 이런 예외를 잡아낼 때마다 다른 예외를 던짐으로써 해당 오류의 서술 가능한 의미를 제공할 수 있다. 그러면 사용자는 범위 밖의 예외와 비교해 `invalid argument` 예외가 무엇을 의미하는지 더 잘 알 수 있을 것이다. `evaluate_rpn` 함수를 쓰는 사용자는 해당 구현 내용을 확인하지 못할 수 있으므로, 내부에서 맵을 사용한다는 사실을 전혀 모를 수 있다.

---
### 부연 설명

`evaluate_rpn` 함수는 반복자를 받음으로써 표준 입력 스트림에 비해 좀 더 다양한 입력을 넘겨줄 수 있다. 시험하거나 다른 다양한 사용자 입력 소스에 적용하기도 쉽다.

문자열 스트림이나 문자열 벡터로부터 반복자를 넘겨주는 코드는 다음과 같다. 이때 evaluate_rpn은 변경할 필요가 전혀 없다.

```c++
int main()
{
    try {
        cout << evaluate_rpn(istream_iterator<string>{cin}, {}) << '\n';
    } catch (const invalid_argument &e) {
        cout << "Invalid operator: " << e.what() << '\n';
    }

  

#if 0

    stringstream s {"3 2 1 + * 2 /"};

    cout << evaluate_rpn(istream_iterator<string>{s}, {}) << '\n';

  

    vector<string> v {"3", "2", "1", "+", "*", "2", "/"};

    cout << evaluate_rpn(begin(v), end(v)) << '\n';

#endif

}
```

>[!tip]
>반복자는 어디든 큰 상관없이 필요한 곳에 사용하면 된다. 자신의 코드를 자유롭게 구성하고 재사용하는 데 큰 도움이 될 것이다.


