# Step 03: 테스트 작성

## 메타데이터
- **난이도**: 🟡 보통
- **롤백 가능**: ✅ (새 파일 삭제)
- **선행 조건**: Step 01, 02 완료

---

## 1. 구현 내용 (design.md 기반)
- `src/__tests__/file-storage.test.ts` 신규 생성
- 실제 파일 I/O 테스트 (tmpDir 기반, fs mock 없음)
- 기존 `session.test.ts` 패턴 준수 (Jest)

## 2. 예상 범위 (Step 4에서 확정)
- [ ] Scope 탐색 필요

## 3. 완료 조건
- [ ] `src/__tests__/file-storage.test.ts` 파일 존재
- [ ] 기본 CRUD 테스트 6개 통과 (생성/조회, 턴 추가, 리셋, 목록, 삭제, 존재하지 않는 세션 삭제)
- [ ] 영속성 테스트 3개 통과 (인스턴스 재생성 후 복원, Date 복원, codexConversationId 복원)
- [ ] 에러 복구 테스트 4개 통과 (ENOENT, JSON 손상 백업, 미래 스키마 버전, tmp 파일 정리)
- [ ] TTL/LRU 테스트 2개 통과 (만료 세션 정리, 100개 초과 정리)
- [ ] 원자적 저장 테스트 2개 통과 (유효 JSON 확인, 퍼미션 0o600)
- [ ] 저장 직렬화 테스트 2개 통과 (연속 변경 데이터 손실 없음, flush 후 영속화)
- [ ] 스키마 테스트 1개 통과 (v1 정상 로드)
- [ ] `npm test` 전체 통과 (기존 테스트 영향 없음)

---

## Scope (Step 4에서 작성)
<!-- Explore Agent 결과 -->

## FP/FN 검증 (Step 5에서 작성)
<!-- 검증 결과 -->
