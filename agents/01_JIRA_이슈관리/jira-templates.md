# JIRA 양식 및 기준표

## 1. 버그 리포트 양식
```
[제목] [모듈/화면] 현상 요약

■ 환경
- 빌드/버전:
- OS/디바이스:
- 브라우저/앱 버전:
- 서버(Dev/QA/Stage/Prod):
- 테스트 계정: (마스킹)

■ 사전 조건
-

■ 재현 단계
1.
2.
3.

■ 기대 결과
-

■ 실제 결과
-

■ 재현율: 항상 / 간헐(n/10) / 1회
■ 심각도(제안): Blocker / Critical / Major / Minor / Trivial — 근거:
■ 우선순위(제안): Highest / High / Medium / Low — 근거:
■ 첨부: 스크린샷 / 영상 / 로그 (없으면 "첨부 필요")
■ 관련 이슈/TC ID:
```

## 2. 개선·기능 요청 양식 (Story/Task)
```
[제목] [모듈] 요청 요약

■ 배경/문제
■ 요청 내용
■ 기대 효과
■ 인수 조건(Acceptance Criteria)
- [ ]
- [ ]
■ 참고 자료
```

## 3. 수정 확인(Verify) 코멘트 양식
```
■ 확인 결과: Pass / Fail(Reopen)
■ 확인 빌드:
■ 확인 환경:
■ 확인 내용:
1.
■ (Fail 시) 재현 내용 / 기대와 다른 점:
■ 회귀 영향 확인 범위:
```

## 4. 심각도 기준
| 등급 | 기준 |
|---|---|
| Blocker | 테스트/서비스 진행 불가, 앱 실행 불가, 데이터 손실 |
| Critical | 핵심 기능 동작 불가, 우회 방법 없음, 결제·보안 문제 |
| Major | 주요 기능 오동작, 우회 방법 있음 |
| Minor | 부가 기능 오류, 사용성 저하 |
| Trivial | 오탈자, 정렬·색상 등 경미한 UI |

## 5. 우선순위 기준
| 등급 | 기준 |
|---|---|
| Highest | 즉시 처리 (배포 차단) |
| High | 이번 스프린트/릴리즈 내 처리 |
| Medium | 다음 릴리즈 검토 |
| Low | 여유 시 처리 |

## 6. 상태 흐름 (팀 기준에 맞게 수정)
Open → In Progress → Resolved → (QA 확인) → Closed
                                   └ Fail → Reopened → In Progress

## 7. 현황 보고 양식
```
■ 기간: YYYY-MM-DD ~ YYYY-MM-DD
■ 요약: 신규 n건 / 해결 n건 / 잔여 n건 (Blocker n, Critical n)
■ 상태별: Open n / In Progress n / Resolved n / Reopened n / Closed n
■ 주요 위험 이슈
- KEY-123 [제목] — 사유(장기 미처리 n일 / 배포 차단 등)
■ 특이사항 / 요청사항
```

## 8. 자주 쓰는 JQL
- 내 미해결 이슈: `assignee = currentUser() AND resolution = Unresolved ORDER BY priority DESC`
- 이번 주 등록 버그: `issuetype = Bug AND created >= startOfWeek()`
- QA 확인 대기: `status = Resolved AND project = KEY`
- 7일 이상 미처리: `resolution = Unresolved AND updated <= -7d`
- 재오픈 이슈: `status = Reopened`
