# Core Engine Subproject Instructions

- 대상 경로: `src/core/`, `include/core/`
- 성능 최적화: 가상 함수(Virtual function) 오버헤드를 최소화하고, CRTP나 템플릿 메타프로그래밍을 고려합니다.
- 예외 처리: 성능 크리티컬 경로에서는 C++ 예외 대신 `std::expected` 또는 Result 타입을 사용합니다.
- 동시성: 락 프리(Lock-free) 자료구조 또는 최소 범위의 `std::scoped_lock`을 적용합니다.

