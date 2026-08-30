# C++ Multi-Module Project Copilot Instructions

## 1. Environment & Standards
- Standard: C++20 (GCC 12+, Clang 15+, MSVC 2022+ 호환)
- Build System: Modern CMake (Target-based, >= 3.22)
- Architecture: Root CMakeLists.txt with `add_subdirectory()` for subprojects.

## 2. C++ Coding Guidelines
- **Memory Safety:** 원시 포인터(Raw pointer)를 통한 소유권 관리는 금지합니다. `std::unique_ptr`, `std::shared_ptr`을 사용하세요.
- **Resource Management:** 모든 리소스는 RAII 원칙을 따릅니다.
- **Modern Features:**
  - 반복문 및 람다에서 `auto`를 적절히 활용하되, 가독성을 해치지 않도록 합니다.
  - 불변 객체 및 함수에는 `const`, `constexpr`, `noexcept`를 적극적으로 지정합니다.
  - C 스타일 캐스팅 대신 `static_cast`, `dynamic_cast`, `reinterpret_cast`를 명시합니다.
- **Naming Conventions:**
  - 클래스/구조체: `PascalCase`
  - 함수/메서드: `camelCase`
  - 멤버 변수: `m_camelCase` 또는 `camelCase_`
  - 상수/매크로: `UPPER_SNAKE_CASE`

## 3. Subproject Structure
- 모든 하위 프로젝트는 독립적으로 빌드 및 테스트가 가능한 인터페이스 타깃을 정의해야 합니다.
- 헤더 노출은 `PUBLIC`, 내부 구현은 `PRIVATE` 가시성을 엄격히 분리합니다.

