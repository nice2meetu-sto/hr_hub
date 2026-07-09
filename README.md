# HR Hub — 서울관광재단 인사팀 HR 포털 소스

인사팀 웹 도구(직원용 `hr.sto.or.kr` / 담당자용 `hr2.sto.or.kr`)의 페이지 소스를 관리하는 저장소.

## 폴더 구조 (서버 웹루트와 1:1 대응)

```
hr/     → 서버 /usr/share/nginx/html/       (hr.sto.or.kr, 직원용)
hr2/    → 서버 /usr/share/nginx/html/hr2/   (hr2.sto.or.kr, 인사 담당자용)
docs/   → 운영 가이드 문서
```

## 페이지 목록

### hr/ — 직원용 (hr.sto.or.kr)
| 파일 | 내용 |
|---|---|
| `index.html` | 허브(대문) — 카드형 메뉴 |
| `mileage.html` | 국외출장 마일리지 관리 |
| `budgetcal.html` | 출장 여비 계산기 |
| `edu.html` | 교육 현황 (예정) |

### hr2/ — 인사 담당자용 (hr2.sto.or.kr)
| 파일 | 내용 |
|---|---|
| `index.html` | 담당자용 허브(대문) |
| `hrtoday.html` | 인원현황 대시보드 (휴가·휴직·입퇴사) |
| `hrhistory.html` | 인사발령 이력 기반 인력현황표 |

## 규칙

- **이 레포에서는 서버 간판 이름(고정 이름)으로 관리한다.** 버전 관리는 git이 하므로 파일명에 날짜/버전을 붙이지 않는다.
- **index.html 자리에는 허브만 간다.** (hr은 hub, hr2는 담당자 허브 — 다른 페이지를 index로 덮어쓰는 사고 주의)
- hub(hr)와 hub_hr2(hr2)는 쌍둥이 구조 — **한쪽 버그 수정 시 다른 쪽도 확인**한다.
- 배포 절차·서버 설정은 [docs/SERVER_GUIDE.md](docs/SERVER_GUIDE.md) 참고. 서버 IP·계정 등 민감정보는 레포에 올리지 않는다 (별도 보관).
