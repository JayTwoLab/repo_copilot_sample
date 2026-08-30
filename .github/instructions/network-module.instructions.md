# Network Module Subproject Instructions

- 대상 경로: `src/network/`, `include/network/`
- I/O 모델: Asio / `std::jthread` 기반의 비동기 이벤트 루프를 사용합니다.
- 소켓 버퍼 관리: 제로 카피(Zero-copy) 버퍼(`std::span<const uint8_t>`)를 우선 사용합니다.
- 연결 생명주기: 세션 객체는 `std::enable_shared_from_this`를 통해 비동기 콜백 중 안전한 생명주기를 보장합니다.

