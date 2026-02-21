# FileSessionStorage - v1.5.0

## 문제 정의

### 현상
- MCP 서버 재시작 시 모든 세션 데이터가 소실됨
- sessionId ↔ codexConversationId 매핑이 날아가서 Codex 대화를 이어갈 수 없음
- 대화 턴 히스토리도 함께 소실됨

### 원인
- 현재 `InMemorySessionStorage`만 존재하며, 세션 데이터를 `Map<string, SessionData>`에만 보관
- MCP 서버는 stdio 기반으로 Claude Code 세션마다 별도 프로세스로 실행되며, 프로세스 종료 시 메모리가 해제됨
- 디스크 영속화 구현체가 없음

### 영향
- `/mcp` 재연결 시 기존 Codex 대화 컨텍스트를 잃어버림
- 긴 작업 중간에 MCP가 재시작되면 Codex resume이 불가능해짐
- 사용자가 매번 sessionId를 새로 시작해야 하는 불편함

### 목표
- JSON 파일 기반 `FileSessionStorage` 구현
- MCP 서버 재시작 후에도 세션 데이터가 유지되어 Codex 대화를 이어갈 수 있음
- 기존 `SessionStorage` 인터페이스를 그대로 구현하여 드롭인 교체 가능

## 성공 기준
- [ ] 프로세스 재시작 후 동일 sessionId로 조회 시 동일 codexConversationId와 turn 목록이 복원됨
- [ ] 재시작 후 동일 sessionId로 Codex 대화가 resume됨
- [ ] I/O 오류나 JSON 손상 시 서버가 크래시하지 않고 안전하게 실패함 (백업 후 빈 상태로 시작)
- [ ] 기존 `npm test` 전체 통과 + FileSessionStorage 전용 테스트 통과 (20+ 케이스)
- [ ] 환경변수로 InMemory ↔ File 전환 시 동일한 SessionStorage 인터페이스 동작 보장 (`CODEX_MCP_MEMORY_ONLY=true`)

## 제약사항
- 기존 `SessionStorage` 인터페이스 변경 불가 (하위 호환)
- Node.js 내장 모듈만 사용 (외부 의존성 추가 없음)
- 동기 인터페이스 유지 (메서드 시그니처 변경 불가)

## 설계 결정사항 (이미 확인됨)
- **동시성**: 여러 MCP 프로세스가 동일 파일 접근 가능 → README에 경고 추가, 단일 프로세스 내에서는 save queue로 직렬화
- **저장 경로**: 기본 `~/.codex-mcp/sessions.json`, 환경변수 `CODEX_MCP_SESSION_FILE`로 변경 가능
- **보안**: codexConversationId 포함으로 파일 퍼미션 0o600 적용, 암호화는 불필요
- **데이터 수명**: TTL 24시간, maxSessions 100, LRU 정리
- **무결성**: 원자적 쓰기 (임시파일 + rename)
