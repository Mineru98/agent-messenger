## 작업 요약

sync-state로 로그인하면 LOGINLIST가 변경분(delta)만 돌려주고, 같이 오는 `lastTokenId/lastChatId` 커서는 모든 방을 지난 위치를 가리킨다. `getChats`가 이 delta를 전체 목록으로 보고 그 커서에서 LCHATLIST를 이어 요청해서 `[]`이 나왔다. 이제 sync-state로 로그인한 세션에서는 LCHATLIST를 커서 0부터 EOF까지 받아 delta와 합친다.

## 측정 비교

| 지표 (sync-state 있음, 실계정) | 전 | 후 | 변화 |
| --- | ---: | ---: | ---: |
| `getChats({ all: true })` 반환 개수 | 0 | 6 | **+6** |
| `chat list` (옵션 없음) 반환 개수 | 0 | 6 | **+6** |
| LCHATLIST 요청 커서 | 로그인 커서 | 0 (기본값) | — |
| LCHATLIST 응답 `chatDatas` | 0 | 6 | **+6** |
| sync-state 파일 저장 | 유지 | 유지 | 동일 |

측정: sync-state 포함 설정 복사본(`AGENT_MESSENGER_CONFIG_DIR`)으로 probe 스크립트 실행, 실제 설정은 불변. 결과가 개수라 결정적이어서 서버 부하를 피하려고 각 1회 측정. 원본은 `.issue/1/evidence/{before,after}/probe.txt` (chat_id·이름은 가림).

## 변경 파일

- `src/platforms/kakaotalk/client.ts` — `SessionState.loginUsedSyncState` 추가, `getChats`가 sync-state 로그인이면 LCHATLIST를 커서 0부터 페이지네이션
- `src/platforms/kakaotalk/client.test.ts` — sync-state가 있을 때 delta가 비었거나 일부만 있는 경우의 재현 테스트 2건

## 검증

- 새 테스트 2건이 수정 전 실패하고 수정 후 통과
- `bun test` 4155 pass / 0 fail
- `bun typecheck` 통과, `bun lint` 에러 0 (경고 4건은 기존 teams 파일)

## 남은 이슈

- `message upload` 결과의 `sent_at`이 `0`으로 나옴 (별도 이슈 후보)
- CLI `upload`가 파일 2개 이상이면 MIME 검사 없이 `sendMultiPhoto` 호출 (별도 이슈 후보)
