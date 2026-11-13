# 1주차 — chapter1, 2

서적 스터디 이후 관심 부분에 대해서 좀 더 스터디 진행해서 블로그 글을 작성했습니다.

## 블로그 글

### 1. [C++ 포인터에서 JavaScript의 값 전달까지: 메모리와 참조의 기초](https://yelihi.pages.dev/articles/cpp/cpp-to-javascript-values/)

- C의 값 전달, 포인터 값의 복사, C++ 참조자의 차이를 비교.
- 복사 비용과 원본 변경 여부, 객체 수명 및 소유권.
- JavaScript에서 객체 속성 변경과 매개변수 재할당의 결과가 다른 이유를 설명.
- `const`, 얕은 복사, 객체 동일성을 통해 두 언어의 참조 개념을 구분.

### 2. [V8 GC에서 Chromium Oilpan까지: JavaScript 객체는 언제 회수될까?](https://yelihi.pages.dev/articles/cpp/v8-gc-memory-leaks/)

- GC Roots에서의 도달 가능성과 순환 참조의 관계 파악.
- 마킹, 스위핑, 압축과 세대별 관리 등 GC의 기본 원리를 정리.
- Blink의 C++ 객체를 관리하는 Oilpan과 JavaScript 영역 사이의 참조 추적.
- 타이머와 이벤트 리스너에 남은 참조가 불필요한 객체를 유지하는 사례 다루기.
- Heap Snapshot과 Retainers로 객체가 유지되는 경로를 확인하는 방법을 정리.
