# Sentry Guide (AgentTeams)

> ⚠️ 이 파일은 서버에서 자동 배포됩니다. 직접 수정하지 마세요.

프로젝트에 연결된 Sentry 이슈를 읽거나 플랜 원본 이슈로 연결할 때 이 가이드를 사용합니다.

## 이슈 조회

사람 자격증명은 선택된 Sentry 프로젝트의 이슈 목록과 상세를 조회할 수 있습니다.

```bash
agentteams sentry issue list [--query "is:unresolved"] [--cursor <cursor>] [--limit <1-100>]
agentteams sentry issue get --issue-id <numericSentryIssueId>
```

- `--project-id`를 지정하면 해당 AgentTeams 프로젝트 경계를 사용합니다.
- `--format json`, `--output-file`, `--verbose`는 다른 읽기 명령과 같은 출력 규칙을 따릅니다.
- 목록의 `pagination.nextCursor`를 다음 `--cursor` 값으로 전달합니다.
- Sentry token, 원본 event payload, 사용자·요청 PII는 CLI 입력이나 출력에 포함하지 않습니다.

## Agent API key 제한

- Agent API key로 이슈 목록과 통합 검색을 조회할 수 없습니다.
- 단건 `get`은 같은 AgentTeams 프로젝트의 `PlanOriginIssue(provider=SENTRY)`에 이미 연결된 canonical ID만 허용됩니다.
- 다른 프로젝트, 미연결 ID, 선택된 Sentry 프로젝트 밖의 ID는 fail-closed로 거부됩니다.

## 플랜에 연결

Sentry의 숫자 issue ID를 canonical identity로 사용합니다.

```bash
agentteams plan link-issue --id <planId> --provider SENTRY --external-id <numericSentryIssueId>
```

Markdown 참조 형식은 `[label](SENTRY_ISSUE:<numeric-id>)`입니다. permalink를 marker에 넣거나 숫자 ID에서 URL을 직접 조합하지 마세요. 서버가 프로젝트 연결을 확인하고 Sentry detail을 다시 읽은 뒤 permalink, short ID, 제목, 상태 metadata를 저장합니다.

## 실행 안전성

Sentry 이슈는 읽기 컨텍스트와 원본 이슈 추적 정보입니다. webhook 또는 이슈 조회만으로 플랜, Task, Runner를 만들거나 시작하지 마세요. 실행은 사용자의 명시적 요청과 AgentTeams 플랜 lifecycle을 따라 별도로 시작해야 합니다.

연결이 해제되어도 기존 원본 이슈와 permalink는 보존됩니다. 다시 동기화하려면 프로젝트 관리자가 Sentry 연결과 프로젝트 선택을 복구해야 합니다.
