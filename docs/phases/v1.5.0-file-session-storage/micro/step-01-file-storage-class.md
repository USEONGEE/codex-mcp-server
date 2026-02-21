# Step 01: FileSessionStorage 클래스 생성

## 메타데이터
- **난이도**: 🟠 중간
- **롤백 가능**: ✅ (새 파일 생성이므로 삭제만 하면 됨)
- **선행 조건**: 없음

---

## 1. 구현 내용 (design.md 기반)
- `src/session/file-storage.ts` 신규 생성
- `SessionStorage` 인터페이스 10개 메서드 구현
- 직렬화/역직렬화 (Date ↔ ISO 8601)
- `queueSave()` + `dirty` 플래그 비동기 저장 직렬화
- `executeSave()` 원자적 쓰기 (임시파일 + rename)
- `loadSync()` 동기 로드 + 에러 복구 (ENOENT/EACCES/손상)
- `validateAndMigrate()` 스키마 버전 관리
- `cleanupTmpFiles()` 잔여 임시파일 정리
- `cleanupExpiredSessions()` TTL 24시간 정리
- `enforceMaxSessions()` LRU 100개 제한
- `flush()` 테스트/graceful shutdown용

## 2. 예상 범위 (Step 4에서 확정)
- [ ] Scope 탐색 필요

## 3. 완료 조건
- [ ] `src/session/file-storage.ts` 파일 존재
- [ ] `FileSessionStorage` 클래스가 `SessionStorage` 인터페이스를 implements
- [ ] `npm run build` 타입 에러 없이 빌드 성공
- [ ] constructor에서 `~/.codex-mcp/sessions.json` 기본 경로 사용
- [ ] 커스텀 경로를 constructor 인자로 전달 가능
- [ ] `flush()` 메서드가 공개 API로 존재

---

## Scope (Step 4에서 작성)
<!-- Explore Agent 결과 -->

## FP/FN 검증 (Step 5에서 작성)
<!-- 검증 결과 -->

---

→ 다음: [Step 02: handlers.ts 팩토리 함수 교체](step-02-handler-factory.md)
