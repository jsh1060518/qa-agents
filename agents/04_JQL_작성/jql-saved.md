# 자주 쓰는 함수 목록 (팀 저장 JQL)

팀에서 실제로 쓰는 JQL을 저장해 둔 목록이다. `/목록` 명령이나 비슷한 요청에는 이 쿼리를 먼저 보여준다.
저장된 쿼리는 사용자가 준 그대로다. 사용자가 고쳐 달라고 하지 않으면 다른 형태로 바꾸지 않는다.

## 팀 이슈 현황
우리 팀이 등록하거나 맡은 이슈를 볼 때

### 팀이 등록한 미해결 이슈 (여러 프로젝트)
- 쓰는 경우: QA_TEAM 그룹이 여러 프로젝트에 등록한 이슈 중 아직 해결되지 않은 것을 볼 때
- 저장일: 2026-11-13

```
project IN (PROJ_A, PROJ_B, PROJ_C) AND reporter IN membersOf("QA_TEAM") AND resolution = Unresolved ORDER BY priority DESC, created DESC
```

| 조건 | 뜻 |
|---|---|
| `project IN (…)` | 프로젝트 3개 (PROJ_A, PROJ_B, PROJ_C) |
| `reporter IN membersOf("QA_TEAM")` | 보고자가 QA_TEAM 그룹 구성원 |
| `resolution = Unresolved` | 아직 해결되지 않은 이슈 |
| `ORDER BY priority DESC, created DESC` | 우선순위 높은 순, 같으면 최근에 등록한 순 |

## 기간별 등록 이슈
이번 주, 이번 달처럼 기간을 정해 팀이 등록한 이슈를 볼 때

### 팀이 등록한 이슈: 2026-11-09 ~ 11-13 (날짜 고정)
- 쓰는 경우: QA_TEAM 그룹이 2026-11-09(월)부터 2026-11-13(금) 사이에 등록한 이슈를 볼 때
- 저장일: 2026-11-13

```
reporter IN membersOf("QA_TEAM") AND created >= "2026-11-09" AND created <= "2026-11-13" ORDER BY created DESC
```

| 조건 | 뜻 |
|---|---|
| `reporter IN membersOf("QA_TEAM")` | 보고자가 QA_TEAM 그룹 구성원 |
| `created >= "2026-11-09"` | 2026-11-09(월) 0시 이후에 등록 |
| `created <= "2026-11-13"` | 2026-11-13(금) 0시까지 등록 |
| `ORDER BY created DESC` | 최근에 등록한 순 |

- 알아둘 점: 날짜만 쓰면 그날 0시로 계산된다. 그래서 13일 당일에 등록한 이슈는 빠진다. 13일까지 넣으려면 created < "2026-11-14"로 쓴다.

### 팀이 이번 주에 등록한 이슈 (주마다 자동)
- 쓰는 경우: QA_TEAM 그룹이 이번 주에 등록한 이슈를 볼 때. 주가 바뀌면 자동으로 그 주로 바뀐다
- 저장일: 2026-11-13

```
reporter IN membersOf("QA_TEAM") AND created >= startOfWeek() AND created <= endOfWeek() ORDER BY created DESC
```

| 조건 | 뜻 |
|---|---|
| `reporter IN membersOf("QA_TEAM")` | 보고자가 QA_TEAM 그룹 구성원 |
| `created >= startOfWeek()` | 이번 주 첫날 0시 이후에 등록 |
| `created <= endOfWeek()` | 이번 주 마지막 날 끝까지 등록 (마지막 날 당일 포함) |
| `ORDER BY created DESC` | 최근에 등록한 순 |

- 알아둘 점: 한 주가 일요일에 시작하는지 월요일에 시작하는지는 JIRA 설정에 따른다. 지난주는 startOfWeek(-1)과 endOfWeek(-1)로 바꾼다.

### 팀이 이번 달에 등록한 이슈 (달마다 자동)
- 쓰는 경우: QA_TEAM 그룹이 이번 달에 등록한 이슈를 볼 때. 달이 바뀌면 자동으로 그 달로 바뀐다
- 저장일: 2026-11-13

```
reporter IN membersOf("QA_TEAM") AND created >= startOfMonth() AND created <= endOfMonth() ORDER BY created DESC
```

| 조건 | 뜻 |
|---|---|
| `reporter IN membersOf("QA_TEAM")` | 보고자가 QA_TEAM 그룹 구성원 |
| `created >= startOfMonth()` | 이번 달 1일 0시 이후에 등록 |
| `created <= endOfMonth()` | 이번 달 마지막 날 끝까지 등록 |
| `ORDER BY created DESC` | 최근에 등록한 순 |

- 알아둘 점: 지난달은 startOfMonth(-1)과 endOfMonth(-1)로 바꾼다.

## 저장된 쿼리에서 확인된 우리 팀 값
새 JQL을 만들 때 사용자가 따로 말하지 않으면 예시 값(QA, qa-team) 대신 아래 값을 쓴다.

| 항목 | 값 |
|---|---|
| 프로젝트 키 | PROJ_A, PROJ_B, PROJ_C |
| 팀 그룹 | QA_TEAM |
