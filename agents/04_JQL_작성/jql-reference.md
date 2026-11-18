# JQL 레퍼런스

Jira Cloud와 Data Center(Server)의 기본 기능 기준이다. 차이가 있는 부분은 따로 표시했다.

---

## 1. 기본 구조
```
필드 연산자 값 [AND|OR 필드 연산자 값 ...] ORDER BY 필드 [ASC|DESC]
```
- 예: `project = QA AND issuetype = Bug AND resolution = Unresolved ORDER BY priority DESC, created ASC`
- 공백이나 특수문자가 있는 값은 큰따옴표로 감싼다: `status = "In Progress"`
- 커스텀 필드는 이름 또는 ID로 쓴다: `"Test Version" = 1.2.0` 또는 `cf[10100] = 1.2.0`

## 2. 자주 쓰는 필드
| 필드 | 의미 | 예시 |
|---|---|---|
| project | 프로젝트 | `project = QA` / `project IN (QA, DEV)` |
| issuetype (type) | 이슈 유형 | `issuetype = Bug` |
| status | 상태 | `status = "In Progress"` |
| statusCategory | 상태 분류 (To Do / In Progress / Done) | `statusCategory != Done` |
| resolution | 해결 여부 | `resolution = Unresolved` |
| priority | 우선순위 | `priority IN (Highest, High)` |
| assignee | 담당자 | `assignee = currentUser()` / `assignee IS EMPTY` |
| reporter | 보고자 | `reporter = currentUser()` |
| creator | 생성자 | `creator = currentUser()` |
| watcher | 지켜보는 사람 | `watcher = currentUser()` |
| labels | 라벨 | `labels IN (회귀, 스모크)` |
| component | 컴포넌트 | `component = 상점` |
| fixVersion | 수정 버전 | `fixVersion = "1.2.0"` |
| affectedVersion | 발생 버전 | `affectedVersion = "1.1.0"` |
| sprint | 스프린트 | `sprint IN openSprints()` |
| parent | 상위 이슈 (Cloud) | `parent = QA-100` |
| "Epic Link" | 에픽 (Data Center) | `"Epic Link" = QA-100` |
| issuekey (key) | 이슈 키 | `key IN (QA-1, QA-2)` |
| created / updated / resolved | 생성일 / 수정일 / 해결일 | `created >= -7d` |
| due (duedate) | 마감일 | `due < now()` |
| summary / description / comment | 제목 / 설명 / 코멘트 | `summary ~ "상점"` |
| text | 제목+설명+코멘트 전체 | `text ~ "보상"` |
| issueLinkType | 링크 유형 | `issueLinkType = "is blocked by"` |
| attachments | 첨부 여부 | `attachments IS EMPTY` |

## 3. 연산자
| 연산자 | 의미 | 예시 |
|---|---|---|
| = / != | 같음 / 다름 | `status = Open` |
| > >= < <= | 크기·날짜 비교 | `created >= -30d` |
| IN / NOT IN | 여러 값 중 하나 | `priority IN (Highest, High)` |
| ~ / !~ | 텍스트 포함 / 미포함 | `summary ~ "로그인"` |
| IS EMPTY / IS NOT EMPTY | 값 없음 / 있음 | `fixVersion IS EMPTY` |
| WAS / WAS NOT | 과거에 그 값이었던 적 있음 | `status WAS Reopened` |
| WAS IN / WAS NOT IN | 과거에 여러 값 중 하나였음 | `status WAS IN (Resolved, Closed)` |
| CHANGED | 값이 바뀐 적 있음 | `status CHANGED` |

**WAS / CHANGED 보조 조건** (status, assignee, priority, resolution, fixVersion, reporter 필드에서 사용)
| 조건 | 예시 |
|---|---|
| FROM / TO | `status CHANGED FROM "In Progress" TO Resolved` |
| BY | `status CHANGED BY currentUser()` |
| AFTER / BEFORE / ON | `status CHANGED AFTER -7d` |
| DURING | `status WAS Reopened DURING (startOfWeek(), now())` |

## 4. 상대 날짜
| 표현 | 의미 |
|---|---|
| `-1d`, `-7d`, `-2w`, `-1h`, `-30m` | 지금으로부터 1일 / 7일 / 2주 / 1시간 / 30분 전 |
| `"YYYY-MM-DD"` | 특정 날짜 (그날 0시) |
| `"YYYY-MM-DD HH:mm"` | 특정 날짜와 시간 |

- `created >= -7d`는 지금부터 정확히 168시간 전이다. "이번 주"처럼 달력 기준이 필요하면 `startOfWeek()`를 쓴다.

## 5. 기본 제공 함수

### 사람·그룹
| 함수 | 의미 | 예시 |
|---|---|---|
| currentUser() | 지금 로그인한 사람 | `assignee = currentUser()` |
| membersOf("그룹") | 그룹에 속한 사람 | `assignee IN membersOf("qa-team")` |

### 날짜
| 함수 | 의미 | 예시 |
|---|---|---|
| now() | 지금 | `due < now()` |
| startOfDay() / endOfDay() | 오늘 0시 / 오늘 끝 | `created >= startOfDay()` |
| startOfWeek() / endOfWeek() | 이번 주 시작 / 끝 | `created >= startOfWeek()` |
| startOfMonth() / endOfMonth() | 이번 달 시작 / 끝 | `resolved >= startOfMonth()` |
| startOfYear() / endOfYear() | 올해 시작 / 끝 | `created >= startOfYear()` |
| currentLogin() / lastLogin() | 이번 로그인 시각 / 직전 로그인 시각 | `updated >= lastLogin()` |

- 괄호 안에 숫자를 넣으면 앞뒤로 옮긴다: `startOfDay(-1)` 어제 0시, `startOfWeek(-1)` 지난주 시작, `endOfWeek(-1)` 지난주 끝, `startOfMonth(-1)` 지난달 시작
- 단위를 붙일 수도 있다: `startOfDay("-3d")`, `startOfWeek("+1w")`
- 한 주의 시작 요일(일요일/월요일)은 JIRA 설정에 따라 다르다.

### 스프린트
| 함수 | 의미 | 예시 |
|---|---|---|
| openSprints() | 진행 중 스프린트 | `sprint IN openSprints()` |
| closedSprints() | 종료된 스프린트 | `sprint IN closedSprints()` |
| futureSprints() | 예정된 스프린트 | `sprint IN futureSprints()` |

### 버전
| 함수 | 의미 | 예시 |
|---|---|---|
| releasedVersions("프로젝트") | 릴리즈된 버전 | `fixVersion IN releasedVersions(QA)` |
| unreleasedVersions("프로젝트") | 릴리즈 전 버전 | `fixVersion IN unreleasedVersions(QA)` |
| latestReleasedVersion("프로젝트") | 가장 최근 릴리즈 버전 | `affectedVersion = latestReleasedVersion(QA)` |
| earliestUnreleasedVersion("프로젝트") | 다음 릴리즈 예정 버전 | `fixVersion = earliestUnreleasedVersion(QA)` |

### 이슈 관계·내 활동
| 함수 | 의미 | 예시 |
|---|---|---|
| linkedIssues("키", "링크 유형") | 특정 이슈와 링크된 이슈 | `issue IN linkedIssues(QA-100, "blocks")` |
| issueHistory() | 내가 최근 본 이슈 | `issue IN issueHistory()` |
| watchedIssues() | 내가 지켜보는 이슈 | `issue IN watchedIssues()` |
| votedIssues() | 내가 투표한 이슈 | `issue IN votedIssues()` |
| updatedBy("사용자", "시작", "끝") | 특정 사람이 수정한 이슈 (Cloud) | `issue IN updatedBy(currentUser(), -7d)` |

### 이슈 유형·프로젝트
| 함수 | 의미 | 예시 |
|---|---|---|
| standardIssueTypes() | 하위 작업이 아닌 유형 | `issuetype IN standardIssueTypes()` |
| subtaskIssueTypes() | 하위 작업 유형 | `issuetype IN subtaskIssueTypes()` |
| projectsLeadByUser() | 내가 리드인 프로젝트 | `project IN projectsLeadByUser()` |
| componentsLeadByUser() | 내가 리드인 컴포넌트 | `component IN componentsLeadByUser()` |
| projectsWhereUserHasPermission("권한") | 권한이 있는 프로젝트 | `project IN projectsWhereUserHasPermission("Resolve Issues")` |
| projectsWhereUserHasRole("역할") | 역할이 있는 프로젝트 | `project IN projectsWhereUserHasRole("Developers")` |

### 저장된 필터 재사용
- `filter = "필터 이름"` 또는 `filter = 10200`
- 자주 쓰는 범위(예: "QA 프로젝트의 미해결 버그")를 필터로 저장해 두고 다른 쿼리에서 불러 쓰면 짧고 관리하기 쉽다.

### 플러그인 함수 (설치된 경우만)
ScriptRunner 같은 플러그인이 있으면 `issueFunction in ...` 형태의 함수를 쓸 수 있다. 예: 하위 작업 조회, 코멘트 유무, 마지막 코멘트 작성자. **설치되지 않은 JIRA에서는 오류가 나므로, 사용자가 쓴다고 했을 때만 쓴다.**

---

## 6. 효율적인 JQL 작성 원칙

| 원칙 | 나쁜 예 | 좋은 예 | 이유 |
|---|---|---|---|
| 범위를 먼저 좁힌다 | `summary ~ "상점"` | `project = QA AND summary ~ "상점"` | 전체 JIRA 검색은 느리고 결과가 섞인다 |
| 상태 나열 대신 분류 사용 | `status IN (Open, "In Progress", Reopened, "To Do")` | `statusCategory != Done` 또는 `resolution = Unresolved` | 상태가 추가돼도 그대로 동작한다 |
| OR 대신 IN | `priority = High OR priority = Highest` | `priority IN (Highest, High)` | 짧고 실수가 적다 |
| AND·OR 섞이면 괄호 | `project = QA AND priority = High OR priority = Highest` | `project = QA AND priority IN (High, Highest)` | 괄호가 없으면 다른 프로젝트 이슈도 섞인다 |
| 고정 날짜 대신 상대 날짜 | `created >= "2026-10-20"` | `created >= startOfWeek()` | 필터를 저장해도 매주 다시 고칠 필요가 없다 |
| 사람 이름 대신 함수 | `assignee = "홍길동"` | `assignee = currentUser()` | 필터를 팀원과 공유해도 각자 기준으로 동작한다 |
| 텍스트 검색 최소화 | `text ~ "보상"` | `component = 보상` 또는 `labels = 보상` | 텍스트 검색은 느리고 관계없는 이슈도 잡힌다 |
| 반복 범위는 필터로 저장 | 매번 같은 조건 복사 | `filter = "QA 미해결 버그" AND assignee = currentUser()` | 조건을 한곳에서 관리할 수 있다 |
| 빈 값은 IS EMPTY | `assignee = ""` | `assignee IS EMPTY` | `= ""`는 동작하지 않는다 |
| 정렬은 마지막에 | `ORDER BY created AND project = QA` | `project = QA ORDER BY created DESC` | ORDER BY 뒤에는 조건을 쓸 수 없다 |

**주의할 점**
- `!=`, `NOT IN`은 값이 비어 있는 이슈를 포함하지 않는다. 빈 값도 포함하려면 `(assignee != currentUser() OR assignee IS EMPTY)`처럼 쓴다.
- `~` 검색은 단어 단위라서 한국어는 원하는 대로 검색되지 않을 수 있다. 정확히 일치하는 문구는 `summary ~ "\"출석 보상\""`처럼 따옴표를 한 번 더 감싼다.
- 상태, 이슈 유형, 커스텀 필드 이름은 회사마다 다르다. 처음에는 JIRA 고급 검색 화면의 자동 완성으로 이름을 확인한다.

---

## 7. 자주 쓰는 JQL 모음 (QA 기준)

### 내 업무
| 용도 | JQL |
|---|---|
| 내 미해결 이슈 | `assignee = currentUser() AND resolution = Unresolved ORDER BY priority DESC, updated DESC` |
| 내가 등록한 미해결 버그 | `reporter = currentUser() AND issuetype = Bug AND resolution = Unresolved ORDER BY created DESC` |
| 내가 확인해야 할 수정 완료 이슈 | `reporter = currentUser() AND status = Resolved ORDER BY updated ASC` |
| 마지막 로그인 후 바뀐 내 이슈 | `(assignee = currentUser() OR reporter = currentUser()) AND updated >= lastLogin()` |

### 버그 현황
| 용도 | JQL |
|---|---|
| 프로젝트 미해결 버그 | `project = QA AND issuetype = Bug AND resolution = Unresolved ORDER BY priority DESC` |
| 오늘 등록된 버그 | `project = QA AND issuetype = Bug AND created >= startOfDay()` |
| 이번 주 등록된 버그 | `project = QA AND issuetype = Bug AND created >= startOfWeek()` |
| 지난주 등록된 버그 | `project = QA AND issuetype = Bug AND created >= startOfWeek(-1) AND created < startOfWeek()` |
| 이번 주 해결된 버그 | `project = QA AND issuetype = Bug AND resolved >= startOfWeek()` |
| 긴급 미해결 버그 | `project = QA AND issuetype = Bug AND priority IN (Highest, High) AND resolution = Unresolved` |
| 담당자 미지정 버그 | `project = QA AND issuetype = Bug AND assignee IS EMPTY AND resolution = Unresolved` |
| 7일 이상 업데이트 없는 이슈 | `project = QA AND resolution = Unresolved AND updated <= -7d ORDER BY updated ASC` |

### 재오픈·이력
| 용도 | JQL |
|---|---|
| 한 번이라도 재오픈된 이슈 | `project = QA AND status WAS Reopened` |
| 이번 주 재오픈된 이슈 | `project = QA AND status CHANGED TO Reopened DURING (startOfWeek(), now())` |
| 이번 주 내가 상태를 바꾼 이슈 | `status CHANGED BY currentUser() AFTER startOfWeek()` |
| 해결됐다가 다시 열린 이슈 | `project = QA AND status CHANGED FROM Resolved TO Reopened` |

### 버전·스프린트
| 용도 | JQL |
|---|---|
| 다음 릴리즈 대상 미해결 이슈 | `project = QA AND fixVersion = earliestUnreleasedVersion(QA) AND resolution = Unresolved` |
| 최근 릴리즈 버전에서 발견된 버그 | `project = QA AND issuetype = Bug AND affectedVersion = latestReleasedVersion(QA)` |
| 수정 버전 미지정 버그 | `project = QA AND issuetype = Bug AND fixVersion IS EMPTY AND resolution = Unresolved` |
| 현재 스프린트 미완료 이슈 | `project = QA AND sprint IN openSprints() AND statusCategory != Done` |

### 기타
| 용도 | JQL |
|---|---|
| 특정 이슈를 막고 있는 이슈 | `issue IN linkedIssues(QA-100, "is blocked by")` |
| 첨부 없는 버그 | `project = QA AND issuetype = Bug AND attachments IS EMPTY` |
| 특정 라벨 이슈 | `project = QA AND labels IN (회귀, 스모크)` |
| 마감일 지난 이슈 | `project = QA AND due < now() AND resolution = Unresolved` |
| QA팀이 담당한 이슈 | `assignee IN membersOf("qa-team") AND resolution = Unresolved` |
