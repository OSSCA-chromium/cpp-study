# Chromium에 C++ 타입을 추가할 때, `struct`와 `class` 중 무엇을 쓸까?

『윤성우의 열혈 C++ 프로그래밍』 3주차 · Chapter 03 클래스의 기본

Chromium 패치에서 함수에 함께 넘기던 값 두 개를 하나의 타입으로 묶기로 했다고 하자. 첫 줄을 `struct`로 시작할까, `class`로 시작할까? 기존 타입의 필드를 수정하는 패치에서도 비슷한 판단이 필요하다. 어떤 필드에는 바로 대입할 수 있고, 어떤 필드는 변경 함수를 거쳐야 한다. 그 차이를 이해하려면 C++의 접근 규칙에서 출발해, Chromium 코드가 실제로 값을 읽고 쓰는 곳까지 따라가 보면 좋다.

## 1. C++에서 `struct`란?

C++ 표준 초안은 클래스를 “A class is a type.”이라고 설명한다. 여기서 클래스는 타입의 한 종류이고, `class`와 `struct`는 그 타입을 정의할 때 쓸 수 있는 키워드다. 따라서 `struct Point`로 정의한 `Point`도 클래스 타입이다. 생성자를 정의할 수도 있고, 아래처럼 멤버 함수를 둘 수도 있다. [C++ 표준 초안의 클래스 정의](https://eel.is/c++draft/class.pre#1)

```cpp
struct Point {
  int x;
  int y;

  bool IsOrigin() const { return x == 0 && y == 0; }
};
```

`Point p{0, 0};`으로 만든 객체에서 `p.x`를 읽고 `p.IsOrigin()`을 호출할 수 있는 이유는 두 멤버가 기본적으로 `public`이기 때문이다. 첫 줄을 `class Point`로 바꾸고 본문 맨 앞에 `public:`을 적어도 같은 코드를 쓸 수 있다. 두 키워드의 문법적 차이는 접근 지정을 생략했을 때 드러난다. 멤버 접근과 상속 접근 모두 `struct`의 기본값은 `public`, `class`의 기본값은 `private`이다. [멤버 접근 규칙](https://eel.is/c++draft/class.access.general#3) · [상속 접근 규칙](https://eel.is/c++draft/class.access.base#2)

## 2. C++ Core Guidelines가 권하는 `struct`와 `class`의 선택 기준

C++ 표준은 두 키워드의 문법과 동작을 정의한다. 타입을 설계할 때 어느 쪽을 고를지에 대해서는 C++ Core Guidelines의 C.2가 기준을 제시한다. **데이터 멤버들이 독립적으로 바뀔 수 있으면 `struct`를, 객체가 불변식을 지켜야 하면 `class`를 권한다.** [C++ Core Guidelines C.2](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#rc-struct)

시작점 `start`와 끝점 `end`를 담는 `Range`를 만든다고 해보자. 사용자가 입력한 범위를 검증하는 화면이라면 `{10, 3}`도 담을 수 있어야 한다. 그래야 화면에서 “끝점이 시작점보다 작습니다”라고 알려줄 수 있다.

같은 두 정수로 구간의 길이를 계산한다면 어떨까? `{3, 10}`의 길이는 `10 - 3`, 즉 7이다. 그런데 호출자가 `range.start = 12;`를 대입하면 같은 계산의 결과가 −2가 된다. 길이를 구하는 쪽에서 `start <= end`를 전제로 삼으려면, 범위를 처음 만들 때도 끝점을 바꿀 때도 이 조건이 성립해야 한다. 이처럼 객체가 계속 만족해야 하는 조건을 **불변식**이라고 한다.

입력 화면에서는 두 값을 독립적으로 고칠 수 있어야 하므로 `struct`가 어울린다. 길이 계산에 쓰는 범위는 한쪽 끝점의 변경이 다른 쪽 값에 제약을 받으므로, `class`에서 두 값의 순서를 보장할 이유가 생긴다.

## 3. Chromium의 C++ 스타일 가이드는 무엇을 권할까?

Chromium의 C++ 스타일 가이드는 별도 예외가 있으면 이를 명시하고, 그 밖에는 Google C++ 스타일 가이드를 따르도록 한다. `struct`와 `class`를 고르는 기준은 Google 가이드의 **Structs vs. Classes** 항목에서 확인할 수 있다. [Chromium C++ 스타일 가이드](https://chromium.googlesource.com/chromium/src/+/main/styleguide/c++/c++.md) · [Google C++ 가이드의 Structs vs. Classes](https://google.github.io/styleguide/cppguide.html#Structs_vs._Classes)

- **`struct`**는 데이터를 담아 전달하는 타입에 쓴다. 모든 필드는 공개하며, 생성자와 보조 함수를 둘 수 있다. 호출자가 각 필드를 직접 바꿀 수 있으므로 타입 자체가 여러 필드 사이의 관계를 항상 보장할 필요가 없어야 한다.
- **`class`**는 타입이 불변식을 지키거나 더 많은 동작을 제공해야 할 때 적합하다. 가이드는 선택이 모호할 때도 `class`를 권한다.

## 4. Chromium 코드에서 쓰이는 `struct`와 `class`

### 공개 필드로 요청 정보를 전달하는 `HttpRequestInfo`

Chromium의 네트워크 테스트에서는 HTTP 요청을 이렇게 만든다. [테스트의 요청 생성 코드](https://source.chromium.org/chromium/chromium/src/+/main:net/http/http_network_transaction_unittest.cc;l=549-551;drc=b54ea5764725c628b8ce7041d594cd509f2f569f)

```cpp
HttpRequestInfo request;
request.method = "GET";
request.url = GURL("http://www.example.org/");
```

`HttpRequestInfo`는 요청 정보를 담는 `struct`다. 호출자가 필요한 필드를 직접 채우고, 멤버 함수로 그 값을 살펴볼 수도 있다. 예를 들어 `IsConsistent()`는 요청에 담긴 두 네트워크 키가 서로 일치하는지 반환한다. HTTP 스트림 생성 코드에서는 `DCHECK(request_info.IsConsistent());`로 그 시점의 일관성을 확인한다. 필드를 바꿀 때마다 구조체가 두 값의 관계를 유지해 주는 방식과는 차이가 있다. [구조체 선언](https://source.chromium.org/chromium/chromium/src/+/main:net/http/http_request_info.h;l=34-50;drc=b54ea5764725c628b8ce7041d594cd509f2f569f) · [검사 함수](https://source.chromium.org/chromium/chromium/src/+/main:net/http/http_request_info.cc;l=22-26;drc=b54ea5764725c628b8ce7041d594cd509f2f569f) · [호출부](https://source.chromium.org/chromium/chromium/src/+/main:net/http/http_stream_factory.cc;l=221;drc=b54ea5764725c628b8ce7041d594cd509f2f569f)

### 너비와 높이의 범위를 지키는 `gfx::Size`

너비와 높이를 담는 `gfx::Size`는 `class`다. `set_width()`의 한 줄을 보면 너비를 비공개로 둔 효과가 드러난다. [Size::set_width() 구현](https://source.chromium.org/chromium/chromium/src/+/main:ui/gfx/geometry/size.h;l=47;drc=b54ea5764725c628b8ce7041d594cd509f2f569f)

```cpp
void set_width(int width) { width_ = std::max(0, width); }
```

`size.set_width(-10)`을 호출하면 저장되는 너비는 0이다. 정수 두 개를 받는 생성자도 같은 규칙을 적용하므로 `gfx::Size(-10, 20)`의 너비 역시 0이다. 호출자는 `width_`에 직접 대입할 수 없고, 음수를 어떻게 처리할지는 `Size`가 결정한다. [생성자](https://source.chromium.org/chromium/chromium/src/+/main:ui/gfx/geometry/size.h;l=25-29;drc=b54ea5764725c628b8ce7041d594cd509f2f569f) · [비공개 멤버](https://source.chromium.org/chromium/chromium/src/+/main:ui/gfx/geometry/size.h;l=81-83;drc=b54ea5764725c628b8ce7041d594cd509f2f569f)

## 5. Chromium에서 `struct`와 `class`를 선택하는 기준

새 타입을 추가하는 패치라면 호출부에서 가장 자연스러운 사용법을 한 줄 써 보자. `request.method = "GET";`처럼 호출자가 각 필드를 채워 넘기는 것이 목적이고 모든 필드를 공개해도 된다면 `struct`가 어울린다. 현재 값을 살펴보는 보조 함수도 여기에 둘 수 있다. 한편 호출자가 필드를 직접 바꾸면 다른 코드가 기대하는 조건이 깨진다면, `class`에서 생성과 변경을 관리해 그 조건을 보장하는 편이 맞다.

기존 Chromium 타입을 수정할 때도 같은 질문을 적용할 수 있다. 예를 들어 `gfx::Size`에 너비를 바꾸는 경로를 추가하는 패치를 리뷰한다면, 그 경로에 `-10`을 넣어 보자. 기존 `set_width(-10)`처럼 너비가 0이 되는가? 새 코드가 `width_`에 바로 대입한다면 기존 호출자가 기대하던 조건이 깨진다. 이렇게 값 하나가 어디에서 들어와 어떤 상태로 남는지 짚어 보면, 리뷰에서 왜 그 필드를 공개하거나 변경 함수를 거쳐야 하는지 설명할 수 있다.
