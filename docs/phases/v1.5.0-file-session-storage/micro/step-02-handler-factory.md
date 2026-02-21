# Step 02: handlers.ts 팩토리 함수 교체

## 메타데이터
- **난이도**: 🟢 쉬움
- **롤백 가능**: ✅ (3줄 변경 복원)
- **선행 조건**: Step 01 완료

---

## 1. 구현 내용 (design.md 기반)
- `src/tools/handlers.ts` 수정
- `FileSessionStorage` import 추가
- `createSessionStorage()` 팩토리 함수 생성
- `new InMemorySessionStorage()` → `createSessionStorage()` 교체
- 환경변수 처리: `CODEX_MCP_MEMORY_ONLY`, `CODEX_MCP_SESSION_FILE`

## 2. 예상 범위 (Step 4에서 확정)
- [ ] Scope 탐색 필요

## 3. 완료 조건
- [ ] `handlers.ts`에서 `FileSessionStorage` import 존재
- [ ] `createSessionStorage()` 함수 존재
- [ ] `CODEX_MCP_MEMORY_ONLY=true` 시 `InMemorySessionStorage` 반환
- [ ] 환경변수 미설정 시 `FileSessionStorage` 반환
- [ ] `CODEX_MCP_SESSION_FILE` 설정 시 해당 경로로 FileSessionStorage 생성
- [ ] `npm run build` 성공

---

## Scope (Step 4에서 작성)
<!-- Explore Agent 결과 -->

## FP/FN 검증 (Step 5에서 작성)
<!-- 검증 결과 -->

---

→ 다음: [Step 03: 테스트 작성](step-03-tests.md)
