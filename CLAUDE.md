# Claude Code Plugin Development Guide

## 버전 트래킹 규칙

### Semantic Versioning

| 타입 | 버전 변경 | 예시 |
|------|----------|------|
| MAJOR | X.0.0 | Breaking changes |
| MINOR | X.Y.0 | 새 기능 추가 (하위 호환) |
| PATCH | X.Y.Z | 버그 수정, 문서 개선 |

### 버전 동기화 파일

플러그인 변경 시 다음 파일들 동시 수정:

| 파일 | 필드 |
|------|------|
| `package.json` | version |
| `.claude-plugin/marketplace.json` | metadata.version |
| `.claude-plugin/marketplace.json` | plugins[해당].version |
| `plugins/{name}/.claude-plugin/plugin.json` | version |

### 커밋 규칙

- 버전 변경 커밋: `:bookmark: release: v{version}` 형식
- 플러그인 단독 업데이트: 해당 플러그인 버전만 올림
- 전체 릴리즈: 모든 버전 동기화

### 예시

```bash
# notion 플러그인만 업데이트 (1.0.0 → 1.1.0)
# 수정할 파일:
# - package.json (전체 버전)
# - .claude-plugin/marketplace.json (metadata.version + plugins[notion].version)
# - plugins/notion/.claude-plugin/plugin.json
```
