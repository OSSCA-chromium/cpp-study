# 2주차 — Chapter 03, 04

## Chapter 03 클래스의 기본

[Chromium: C++에서 struct와 class를 고르는 기준](https://bako94.tistory.com/362)

- C++의 기본 접근 권한과 C++ Core Guidelines의 선택 기준을 정리했습니다.
- Chromium의 스타일 가이드를 읽고, `HttpRequestInfo`와 `gfx::Size`에서 필드를 공개하거나 변경 함수로 관리하는 이유를 비교했습니다.

## Chapter 04 클래스의 완성

[Chrome에서 체크박스 하나를 클릭하면 무슨 일이 일어날까?](https://bako94.tistory.com/363)

- 첫 실행 대화상자에서 체크박스를 생성하고, 클릭과 확인 시점에 선택값을 어떻게 바꾸고 읽는지 따라갔습니다.
- 대화상자가 포인터로 체크박스에 접근하는 과정과 부모 뷰가 소멸할 때 자식을 정리하는 과정을 살펴봤습니다.
