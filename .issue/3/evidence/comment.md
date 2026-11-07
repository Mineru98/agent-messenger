## 작업 요약

`memberIdKey` 가 `Number.MAX_SAFE_INTEGER`(2^53−1)보다 큰 user_id 를 `null` 로 처리하고 있었습니다. 그래서 실제 카카오 사용자가 한 명이라도 있는 방은 멤버 스냅샷이 `CHATINFO member snapshot invalid` 로 실패했습니다. Long 분기의 상한을 int64 최댓값(2^63−1)으로 바꿔 양수 64비트 id 를 정확한 문자열로 받게 했습니다. 0·음수·형식 오류 id 거부와 이중 읽기 정합성 검사는 그대로입니다.

## 테스트 비교

| 항목 | 전 | 후 | 변화 |
| --- | ---: | ---: | ---: |
| kakaotalk client 테스트 통과 | 203 / 203 | 205 / 205 | **+2 (회귀 테스트)** |
| 64비트 id 스냅샷 회귀 테스트 | 실패 (`CHATINFO member snapshot invalid`) | 통과 | **해결** |
| 음수 64비트 id 거부 테스트 | 통과 | 통과 | 유지 |
| `bun run typecheck` | 미측정 (수정 전 기준선 불필요) | 통과 | - |
| 전체 `bun test src/` | 4154 pass / 1 fail | 4156 pass / 1 fail | 실패 1건은 기존 실패 |

전체 스위트의 실패 1건(`list: uses live team display names for a device-code account`)은 수정 전 main(`0e44c7b1`)에서도 똑같이 실패합니다. 이번 변경과 관계없습니다.

원본 출력:
- [before/kakaotalk-client-test.txt](https://github.com/Mineru98/agent-messenger/blob/main/.issue/3/evidence/before/kakaotalk-client-test.txt)
- [before/regression-test.txt](https://github.com/Mineru98/agent-messenger/blob/main/.issue/3/evidence/before/regression-test.txt): 수정 전 코드에서 운영 로그와 같은 에러로 재현됩니다
- [after/tests.txt](https://github.com/Mineru98/agent-messenger/blob/main/.issue/3/evidence/after/tests.txt)
- [after/full-suite.txt](https://github.com/Mineru98/agent-messenger/blob/main/.issue/3/evidence/after/full-suite.txt)

## 변경 파일

- `src/platforms/kakaotalk/client.ts`: `memberIdKey` 상한을 `Number.MAX_SAFE_INTEGER` 에서 `0x7fffffffffffffffn` 으로 변경
- `src/platforms/kakaotalk/client.test.ts`: 큰 id 스냅샷 성공 테스트와 음수 id 거부 테스트 추가

## 검증

- `bun test src/platforms/kakaotalk/client.test.ts`: 205 pass
- `bun run typecheck`: 통과

## 남은 이슈

- kakao-openclaw-harness 쪽 `agent-messenger` 핀을 이 수정으로 올리고 테스트방에서 다시 확인해야 합니다(하네스 저장소 작업).
