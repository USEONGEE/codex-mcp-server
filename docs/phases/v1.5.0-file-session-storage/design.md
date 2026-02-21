# 설계 - v1.5.0 FileSessionStorage

## 해결 방식

### 접근법
- `SessionStorage` 인터페이스를 구현하는 `FileSessionStorage` 클래스를 별도 파일로 생성
- 메모리 Map을 source of truth로 유지하면서, 변경 시 비동기로 JSON 파일에 영속화
- 환경변수 기반 팩토리 함수로 InMemory/File 전환

### 대안 검토

| 방식 | 장점 | 단점 | 선택 |
|------|------|------|------|
| A: InMemorySessionStorage에 파일 저장 기능 추가 | 파일 하나로 해결 | SRP 위반, 기존 코드 변경 많음 | ❌ |
| B: 별도 FileSessionStorage 클래스 생성 | 기존 코드 변경 최소, 테스트 격리 | 코드 일부 중복 (10개 메서드) | ✅ |
| C: SQLite/LevelDB 기반 저장소 | 동시성/성능 우수 | 외부 의존성 추가 필요 (제약 위반) | ❌ |

**선택 이유**: B를 선택. 기존 `InMemorySessionStorage`를 건드리지 않아 하위 호환을 보장하고, 외부 의존성 없이 Node.js 내장 `fs` 모듈만으로 구현 가능.

### 기술 결정

1. **파일 구조**: `src/session/file-storage.ts` (별도 파일)
2. **저장 포맷**: JSON, `version` 필드 포함 (스키마 버전 관리)
3. **Date 직렬화**: `Date` → ISO 8601 문자열, 로드 시 역변환
4. **비동기 저장 직렬화**: `queueSave()` + `dirty` 플래그 + Promise 체이닝
   - 싱글 스레드 내 비동기 레이스 방지
   - 최대 2개 save만 동시 존재 (실행 중 1 + 대기 1)
5. **원자적 쓰기**: 임시파일(`{path}.{pid}.tmp`) → `rename()`
6. **에러 복구**: ENOENT=정상, EACCES=치명적, JSON 손상=백업 후 빈 시작
7. **파일 퍼미션**: 디렉토리 `0o700`, 파일 `0o600`
8. **동기 로드**: constructor에서 `readFileSync` (인터페이스가 동기이므로)
9. **팩토리 함수**: `handlers.ts`에서 환경변수 기반 저장소 선택

## 핵심 설계 상세

### 저장 트리거

| 메서드 | queueSave? |
|--------|------------|
| `createSession()` | O |
| `ensureSession()` | O |
| `getSession()` | O (lastAccessedAt 갱신) |
| `updateSession()` | O |
| `deleteSession()` | O (삭제 시만) |
| `listSessions()` | dirty 자동 |
| `addTurn()` | O |
| `resetSession()` | O |
| `setCodexConversationId()` | O |
| `getCodexConversationId()` | X |

### 파일 포맷 (v1)

```json
{
  "version": 1,
  "sessions": [{
    "id": "session-name",
    "createdAt": "2026-02-20T12:00:00.000Z",
    "lastAccessedAt": "2026-02-20T12:13:00.000Z",
    "turns": [{ "prompt": "...", "response": "...", "timestamp": "..." }],
    "codexConversationId": "019befec-..."
  }]
}
```

### 저장소 선택

```typescript
function createSessionStorage(): SessionStorage {
  if (CODEX_MCP_MEMORY_ONLY) return new InMemorySessionStorage();
  return new FileSessionStorage(CODEX_MCP_SESSION_FILE || undefined);
}
```
