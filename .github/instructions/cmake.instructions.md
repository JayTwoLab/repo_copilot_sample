# Modern CMake Instructions

- 레거시 CMake 명령어(`include_directories`, `link_libraries`, `add_compile_options`) 사용을 금지합니다.
- 항상 Target 기반 명령어만 사용합니다:
  - `target_include_directories()`
  - `target_link_libraries()`
  - `target_compile_features(target PRIVATE cxx_std_20)`
- 하위 프로젝트 간 의존성은 반드시 네임스페이스 타깃(`Project::Submodule`) 형식을 사용합니다.
- 모든 서브프로젝트는 단위 테스트 타깃(`ctest` 연동)을 포함해야 합니다.

