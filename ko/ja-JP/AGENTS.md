# AGENTS.md - ja-JP 문서 번역 작업 공간

## 다음 경우에 읽기

- `docs/ja-JP/**` 유지 관리
- 일본어 번역 파이프라인(용어집/TM/프롬프트) 업데이트
- 일본어 번역 피드백 또는 회귀 처리

## 파이프라인 (docs-i18n)

- 원본 문서: `docs/**/*.md`
- 대상 문서: `docs/ja-JP/**/*.md`
- 용어집: `docs/.i18n/glossary.ja-JP.json`
- 번역 메모리: `docs/.i18n/ja-JP.tm.jsonl`
- 프롬프트 규칙: `scripts/docs-i18n/prompt.go`

일반 실행 예:

```bash
# 대량 처리 (doc 모드; 병렬 실행 가능)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# 단일 파일
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# 소규모 수정 (segment 모드; TM 사용; 병렬 없음)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

참고:

- 전체 페이지 번역에는 `doc` 모드를, 작은 수정에는 `segment` 모드를 사용하세요.
- 매우 큰 파일이 타임아웃되는 경우, 대상 부분만 수정하거나 페이지를 분할한 뒤 다시 실행하세요.
- 번역 후 확인 사항: 코드 인라인/블록이 변경되지 않았는지, 링크/앵커가 변경되지 않았는지, 플레이스홀더가 유지되었는지 점검하세요.
