## 🔌 API 명세 (API Specification)

> **Base URL:** `http://localhost:8080`  
> **Auth Header:** 모든 API 요청 헤더에는 `Authorization: Bearer {accessToken}`이 필요합니다.

### 1. 👤 인증 및 사용자 (User & Auth)

| Method | URI | 설명 | 비고 |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/users/login` | 로그인 | |
| `POST` | `/api/users/register` | 회원가입 | |
| `GET` | `/api/users/info` | 사용자 정보 조회 | |
| `GET` | `/api/users/check` | 군번 중복 확인 | |
| `POST` | `/api/users/logout` | 로그아웃 | Refresh Token 만료 처리 |
| `POST` | `/api/users/refresh` | 토큰 재발급 | Access Token 갱신 |

### 2. 🏠 방 관리 (Room & Lobby)

| Method | URI | 설명 | 비고 |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/rooms` | 방 목록 조회 | 검색, 필터링 지원 |
| `POST` | `/api/rooms` | 방 생성 | OTP 자동 생성 |
| `DELETE` | `/api/rooms/{roomId}` | 방 삭제 | |
| `PATCH` | `/api/rooms/{roomId}` | 방 정보 수정 | |
| `POST` | `/api/rooms/{roomId}` | **방 참여 (OTP 인증)** ||
| `GET` | `/api/rooms/category` | 전술 대분류 목록 호출 | |
| `GET` | `/api/tactics` | 작전 전술 목록 호출 | |
| `GET` | `/api/rooms/{roomId}/lobby` | 대기실 조회 | 참여자, 역할 현황 등 |
| `PATCH` | `/api/rooms/{roomId}/otp` | OTP 코드 재발급 | 방장 전용 |
| `PATCH` | `/api/rooms/{roomId}/extend` | 방 만료 시간 연장 | 2시간 연장 |
| `PATCH` | `/api/rooms/{roomId}/host` | 방장 권한 위임 | |

### 3. 🎖️ 역할 및 시나리오 (Role & Scenario)

| Method | URI | 설명 | 비고 |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/rooms/{roomId}/roles` | 역할 목록 조회 | |
| `POST` | `/api/rooms/{roomId}/roles` | 역할 선택 | 중복 선택 불가 |
| `PATCH` | `/api/rooms/{roomId}/roles/cancel` | 역할 선택 취소 | |
| `GET` | `/api/rooms/{roomId}/scenario` | 맵 및 시나리오 호출 | |

### 4. 🎥 훈련 및 화상 (Training & OpenVidu)

| Method | URI | 설명 | 비고 |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/rooms/{roomId}/start` | 훈련 시작 | |
| `POST` | `/api/rooms/{roomId}/connection` | **OpenVidu 토큰 발급** | 화상 연결용 |
| `POST` | `/api/rooms/{roomId}/event` | 돌발상황 트리거 | |
| `POST` | `/api/rooms/{roomId}/log` | 단계별 타임스탬프 | 로그 기록 |
| `GET` | `/api/rooms/events` | 돌발상황 리스트 조회 | |
| `POST` | `/api/rooms/{roomId}/end` | 훈련 종료 호출 | |
| `GET` | `/api/rooms/{roomId}/actions` | 현재 방의 모든 액션 조회 | 재접속 시 동기화용 |

### 5. 📊 결과 및 AI 분석 (Result)

| Method | URI | 설명 | 비고 |
| :--- | :--- | :--- | :--- |
| `GET` | `/api/results` | 훈련 내역 목록 조회 | |
| `GET` | `/api/results/{resultId}` | 훈련 핵심 요약 정보 | |
| `GET` | `/api/results/{resultId}/logs` | 전체 채팅/STT 로그 조회 | |
| `DELETE` | `/api/results/{resultId}` | 훈련 내역 삭제 | |
| `PATCH` | `/api/results/{resultId}/ai-analysis` | **AI(RAG) 데이터 반환** | 분석 결과 업데이트 |

---

## 📡 실시간 통신 명세 (WebSocket)

* **Socket Endpoint:** `ws://{domain}/ws-stomp`
* **Protocol:** STOMP

### 1. 로비 (Lobby)
* **Subscribe:** `/sub/rooms/{roomId}/lobby`
    * **설명:** 입장/퇴장, 역할 변경 시 최신 로비 상태(유저 목록, 역할 현황) 수신
* **Publish:** `/pub/rooms/enter`
    * **설명:** 소켓 연결 후 입장 감지용 메시지 전송

### 2. 지도 조작 (Map Action)
* **Subscribe:** `/sub/rooms/{roomId}/action`
* **Publish:** `/pub/rooms/action`
* **Payload 구조:**
```json
{
  "type": "MARKER", // MARKER, MOVE, DELETE_MARKER, CLEAR_MARKERS, PING
  "data": {
    "markerId": "m-uuid-001",
    "x": 500.0,
    "y": 120.5,
    "markerType": "SOLDIER"
  }
}