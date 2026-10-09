## 실습과제 1. 일반 함수와 람다식의 차이를 설명하시오. 일반 함수로 정의하는 대신에 람다식을 사용할 때 장점은 무엇인가?

### 일반 함수 vs 람다식

| 구분 | 일반 함수 | 람다식 (C++11) |
|---|---|---|
| 이름 | 이름이 있음 | **익명 함수** (이름 없음). 필요하면 `auto f = [](...){...};`로 변수에 저장 |
| 정의 위치 | 전역·네임스페이스·클래스 범위에서 함수 밖에 정의 | **식(expression)** 이므로 함수 안, 인자 자리 등 **사용하는 곳에서 바로** 정의 |
| 구문 | `반환형 이름(매개변수) { 본문 }` | `[캡처리스트](매개변수) -> 반환형 { 본  문 }` |
| 반환형 | 반드시 명시 | `->` 반환형 **생략 가능**(본문에서 자동 추론) |
| 외부 지역 변수 접근 | **불가** (전역변수, 매개변수, 포인터 등을 거쳐야 함) | **캡처 리스트**로 접근. `[x]` 값 캡처(읽기 전용), `[&x]` 참조 캡처(수정 가능) |
| 타입 | 함수 타입 `R(Args...)` | 컴파일러가 만드는 **고유한 익명 클로저 타입** → 변수 타입을 직접 쓸 수 없어 `auto` 또는 `std::function` 사용 |
| 호출 | `f(2, 3);` | 정의 직후 `[](int x, int y){...}(2, 3);` 또는 변수로 `fn(2, 3);` |
| 재사용 | 이름으로 여러 곳에서 재사용 | 보통 한 곳에서 일회성으로 사용 |

캡처 리스트는 람다 **밖**에서 선언된 변수를 가져다 쓰기 위한 것이고, 매개변수는 호출할 때 **인자로 전달**받는 람다 내부의 변수다.

### 일반 함수 대신 람다식을 쓸 때의 장점

1. **간결함 / 코드 지역성**: `sort`, `count_if`, 콜백처럼 한 번만 쓸 작은 함수를 별도로 정의하지 않고 쓰는 자리에 바로 작성하므로 코드 흐름이 끊기지 않는다.
2. **이름 공간 오염 방지**: 일회용 함수 이름을 전역에 만들 필요가 없다.
3. **주변 상태를 쉽게 사용**: 캡처 리스트로 지역 변수를 바로 사용할 수 있다. 일반 함수는 전역변수, 추가 매개변수, 함수 객체 클래스가 필요하다.
4. **STL 알고리즘·콜백과 궁합이 좋음**: `std::sort`, `std::for_each`, `std::function` 인자, ROS 2의 콜백에서 널리 쓰인다.
5. **성능**: 컴파일러가 본문을 인라인화하기 쉬워 함수 포인터 호출보다 유리한 경우가 많다.

---

## 실습과제 2.  23페이지 예제처럼 함수에 인자로 람다식을 사용하는 예제를 인터넷에서 찾아서 실행해보고 소스코드를 자세히 설명하시오.

### 소스코드 `lambda_arg.cpp`

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;

int main() {
    vector<int> v = {5, 2, 8, 1, 9, 3};

    // 1) sort: 비교 기준을 람다식으로 전달 (내림차순)
    sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
    for (int x : v) cout << x << " ";
    cout << endl;

    // 2) count_if: 조건을 람다식으로 전달 (짝수 개수)
    int cnt = count_if(v.begin(), v.end(), [](int x) { return x % 2 == 0; });
    cout << "짝수 개수=" << cnt << endl;

    // 3) for_each + 참조 캡처: 외부 변수 sum에 누적
    int sum = 0;
    for_each(v.begin(), v.end(), [&sum](int x) { sum += x; });
    cout << "sum=" << sum << endl;

    // 4) find_if + 값 캡처: 외부 변수 threshold 값을 복사해 사용
    int threshold = 4;
    auto it = find_if(v.begin(), v.end(), [threshold](int x) { return x < threshold; });
    cout << "threshold보다 작은 첫 원소=" << *it << endl;
    return 0;
}
```

### 라인별 설명

| 라인 | 설명 |
|---|---|
| 1~3 | `sort`/`count_if`/`for_each`/`find_if`가 있는 `<algorithm>`, 출력용 `<iostream>`, 컨테이너 `<vector>` 포함 |
| 4 | `std::` 접두사 생략 |
| 7 | 정수 6개를 가진 `vector<int>` 생성: `{5, 2, 8, 1, 9, 3}` |
| 10 | `sort`의 세 번째 인자로 **비교 함수 역할의 람다식** 전달. `a > b`이면 `a`가 앞에 오므로 **내림차순**. 캡처 없음(`[]`), 반환형(`bool`)은 자동 추론 → `9 8 5 3 2 1` |
| 11~12 | 범위 기반 for문으로 정렬된 원소 출력 후 줄바꿈 |
| 15 | `count_if`는 람다식이 `true`를 반환하는 원소의 개수를 센다. `x % 2 == 0`(짝수)인 원소는 8, 2 → 2개 |
| 16 | 개수 출력 |
| 19 | 합계를 저장할 지역 변수 `sum` 선언 |
| 20 | `for_each`가 각 원소를 람다식에 전달. `[&sum]`은 **참조 캡처**이므로 람다 안에서 바깥 변수 `sum`을 직접 수정할 수 있다 (9+8+5+3+2+1 = 28) |
| 21 | 누적 결과 출력 |
| 24 | 비교 기준값 `threshold` 선언 |
| 25 | `find_if`는 조건을 만족하는 **첫 원소의 반복자**를 반환. `[threshold]`는 **값 캡처**(읽기만 가능). 정렬된 순서 `9 8 5 3 2 1`에서 4보다 작은 첫 원소는 3 |
| 26 | 반복자를 역참조(`*it`)해 출력 |

### 실행 결과

컴파일: `g++ -std=c++14 -Wall -Wextra -o lambda_arg lambda_arg.cpp`

```
9 8 5 3 2 1 
짝수 개수=2
sum=28
threshold보다 작은 첫 원소=3
```

---

## 실습과제 3. 함수 포인터와 std::function 객체의 차이를 자세히 설명하시오.

**함수 포인터**는 함수의 시작 주소를 저장하는 C 스타일 포인터이고, **`std::function`**(`<functional>`)은 호출 가능한 모든 것을 저장하는 **템플릿 클래스(객체)** 로 함수 포인터의 역할을 대신한다.

| 구분 | 함수 포인터 | `std::function` |
|---|---|---|
| 정체 | 주소를 담은 **포인터** (기본 타입) | `<functional>`의 **템플릿 클래스 객체** |
| 선언 | `void (*f)(int, const string&) = func;` | `function<void(int, const string&)> f = func;` |
| 저장 가능 대상 | 일반 함수, static 함수, **캡처 없는** 람다 | 일반 함수, 함수 포인터, **캡처 있는 람다**, 함수 객체(functor), `std::bind` 결과, 멤버 함수(`&클래스::함수`) 등 **호출 가능한 모든 것** |
| 상태(캡처) 보유 | 불가 (주소만 저장) | 가능 (캡처한 변수나 functor의 멤버를 객체 내부에 보관) |
| 멤버 함수 | 일반 함수 포인터에 못 담음 (별도의 멤버 함수 포인터 `void (A::*)()` 타입 필요) | `function<void(A&)> f = &A::show;`처럼 **`this` 대응 매개변수**(`A*` 또는 `A&`)를 추가해 저장 |
| 타입 추론 | `auto f = &A::show;` → 멤버 함수 포인터가 되어 `f(&a)`처럼 호출 불가 | `std::mem_fn(&A::show)`와 함께 사용 가능 |
| 비어 있는 상태 | `nullptr` 가능 | 비어 있음 → `if (f)`로 확인, 비어 있는 채로 호출하면 `std::bad_function_call` 예외 |
| 오버헤드 | 거의 없음 (포인터 1개, 직접 호출) | 타입 소거로 약간의 간접 호출 비용과 객체 크기, 큰 호출체는 동적 할당 가능 |
| 안정성 | 포인터라서 잘못된 주소·널 사용에 주의 | 객체이지만 함수 포인터처럼 사용 가능, **포인터 사용으로 인한 오류 방지** |
| 용도 | C 라이브러리 콜백, 단순 함수 주소 전달 | 유연한 콜백 인자, ROS 라이브러리처럼 람다·멤버 함수·functor를 모두 받아야 하는 API |

---

## 실습과제 4. 23페이지 예제처럼 인터넷에서 std::function 클래스를 함수에 매개변수에 사용한 예제를 찾아서 실행해보고 소스코드를 자세히 설명하시오.

### 소스코드 `function_arg.cpp`

```cpp
#include <functional>
#include <iostream>
using namespace std;

// std::function을 매개변수로 받는 함수: f를 두 번 적용
int applyTwice(function<int(int)> f, int x) {
    return f(f(x));
}

// 콜백(callback)을 n번 호출하는 함수
void repeat(int n, function<void(int)> cb) {
    for (int i = 0; i < n; i++) cb(i);
}

int square(int x) { return x * x; }  // 일반 함수

struct Mul {                          // 함수 객체(functor)
    int k;
    int operator()(int x) const { return x * k; }
};

int main() {
    cout << "일반 함수  : " << applyTwice(square, 3) << endl;
    cout << "람다식     : " << applyTwice([](int x) { return x + 10; }, 3) << endl;
    cout << "함수 객체  : " << applyTwice(Mul{2}, 3) << endl;

    int total = 0;
    repeat(3, [&total](int i) {
        total += i;
        cout << "callback " << i << " (total=" << total << ")" << endl;
    });
    return 0;
}
```

### 라인별 설명

| 라인 | 설명 |
|---|---|
| 1~3 | `std::function`이 정의된 `<functional>`, 출력용 `<iostream>` 포함, `std::` 생략 |
| 6~8 | `applyTwice`: 첫 번째 매개변수가 `function<int(int)>` 타입. 즉 **int를 받아 int를 반환하는 호출 가능한 것**이면 무엇이든 받는다. `f(f(x))`로 함수를 두 번 적용 |
| 11~13 | `repeat`: `function<void(int)>` 타입의 콜백 `cb`를 받아 `0 ~ n-1`을 인자로 `n`번 호출. ROS의 콜백 등록 방식과 같은 패턴 |
| 15 | 일반 함수 `square` (`int(int)`) |
| 17~20 | 함수 객체(functor) `Mul`. `operator()`를 정의해 함수처럼 호출 가능. 멤버 `k`가 곱할 값(상태)을 보관 |
| 23 | **일반 함수** 전달: `square(square(3))` = 9 → 81 |
| 24 | **람다식** 전달 (캡처 없음): `(3+10)+10` = 23 |
| 25 | **함수 객체** 전달: `Mul{2}`는 `k=2`로 초기화. `(3*2)*2` = 12 |
| 27 | 콜백 안에서 누적할 지역 변수 `total` 선언 |
| 28~31 | **캡처 있는 람다**를 `function`으로 전달. `[&total]` 참조 캡처로 `total`을 수정하고 진행 상황 출력. (캡처 있는 람다는 함수 포인터로는 못 받지만 `std::function`은 받을 수 있다) |

### 실행 결과

컴파일: `g++ -std=c++14 -Wall -Wextra -o function_arg function_arg.cpp`

```
일반 함수  : 81
람다식     : 23
함수 객체  : 12
callback 0 (total=0)
callback 1 (total=1)
callback 2 (total=3)
```
