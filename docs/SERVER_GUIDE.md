# 서울관광재단 인사팀 HR 포털 — 서버 운영 가이드

> 이 문서는 hr.sto.or.kr / hr2.sto.or.kr 웹 포털의 서버 구조, 배포 절차, 운영 규칙을 정리한 것.
> Claude Code 등 다른 작업 환경에서 이 프로젝트를 이어받을 때 이 문서를 먼저 읽을 것.
> 최초 작성: 2026-07-08 (서버 최초 구축일)
>
> ⚠️ **이 레포는 public이므로 서버 IP·계정명·비밀번호·방화벽 상세는 기록하지 않는다.**
> 아래에서 `<서버IP>`, `<서버계정>`으로 표기된 값은 별도 보관 중인 원본 문서를 참고할 것.

---

## 1. 시스템 개요

| 항목 | 내용 |
|---|---|
| 용도 | 인사팀 웹 도구 호스팅 (직원용 + 담당자용) |
| 서버 | 네이버클라우드(NCP) VPC 서버, Rocky Linux |
| 도메인 | `hr.sto.or.kr` (직원용), `hr2.sto.or.kr` (인사 담당자용) — 둘 다 같은 서버, DNS 등록 완료 |
| 웹서버 | nginx (직접 설치, `systemctl enable --now nginx` 완료) |
| 접근 제한 | **내부 전용.** ACG(방화벽)로 재단본사 IP·VPN IP에서만 접속 가능. 그 외에서는 timeout |
| HTTPS | 미적용. 반드시 `http://` 명시 접속. 인증서는 추후 회사 와일드카드(*.sto.or.kr) 확인 필요 |
| 접속 정보 | 서버 IP·계정·비밀번호는 별도 보관 (이 레포에 기록하지 말 것) |

## 2. 사이트 구조

### hr.sto.or.kr — 직원용
| 주소 | 서버 파일 | 내용 |
|---|---|---|
| `/` | `/usr/share/nginx/html/index.html` | 허브(대문) — 카드형 메뉴 |
| `/mileage.html` | `.../html/mileage.html` | 국외출장 마일리지 관리 |
| `/budgetcal.html` | `.../html/budgetcal.html` | 출장 여비 계산기 |
| `/edu.html` | `.../html/edu.html` | 교육 현황 (예정) |

### hr2.sto.or.kr — 인사 담당자용
| 주소 | 서버 파일 | 내용 |
|---|---|---|
| `/` | `/usr/share/nginx/html/hr2/index.html` | 담당자용 허브(대문) |
| `/hrtoday.html` | `.../hr2/hrtoday.html` | 인원현황 대시보드 (휴가·휴직·입퇴사) |
| `/hrhistory.html` | `.../hr2/hrhistory.html` | 인사발령 이력 기반 인력현황표 |

- try_files 설정으로 확장자 없는 주소도 동작 (`/mileage` = `/mileage.html`)
- 허브 페이지는 JS의 `TOOLS` 배열에 항목을 추가하면 카드가 자동 생성되는 구조

## 3. 소스 관리 (이 레포 기준)

이 레포의 `hr/`, `hr2/` 폴더가 서버 웹루트와 1:1 대응한다:

```
hr/    → 서버 /usr/share/nginx/html/       (hr.sto.or.kr)
hr2/   → 서버 /usr/share/nginx/html/hr2/   (hr2.sto.or.kr)
```

**파일명 규칙:** 레포와 서버 모두 **고정 간판 이름**(`index.html`, `mileage.html`, …)을 쓴다.
주소·허브 링크·북마크가 절대 안 깨지게 하기 위함. 버전 이력은 git이 관리하므로
파일명에 날짜/버전(`stoml0708.html` 등)을 붙이지 않는다.

> (참고) 레포 도입 전에는 PC 폴더 `...\호스팅\hr\`, `...\호스팅\hr2\`에서
> 버전 붙은 파일명으로 관리하고 서버에 간판 이름으로 복사하는 방식이었다.

**절대 규칙: index.html 자리에는 허브만 간다.** (hr은 hub, hr2는 담당자 허브.
다른 페이지를 index로 복사하는 사고 2회 발생했음)

## 4. 표준 배포 절차

배포는 2단계: ① PC→서버 홈 업로드(scp), ② 홈→웹루트 복사(sudo cp). **①만 하면 웹에 반영 안 됨.**

### ① PC 명령창(PowerShell/cmd)에서 — 업로드
```
scp "C:\...\hr\파일명.html" <서버계정>@<서버IP>:~/
```
- 비밀번호 입력 시 화면에 아무것도 안 보이는 게 정상 (입력하고 엔터)
- 한/영 상태 확인, 붙여넣기는 마우스 오른쪽 클릭
- 성공 시 `파일명 100%` 표시

### ② ssh 창에서 — 웹루트 반영
```
ssh <서버계정>@<서버IP>
sudo cp ~/파일명.html /usr/share/nginx/html/간판이름.html        ← hr 것
sudo cp ~/파일명.html /usr/share/nginx/html/hr2/간판이름.html    ← hr2 것
```

### ③ 확인
- 브라우저에서 `http://` 명시하여 접속, **Ctrl+F5** (강력 새로고침)
- 그래도 이상하면 시크릿 창, 또는 F12 → 새로고침 버튼 우클릭 → "캐시 비우기 및 강력 새로고침"

## 5. nginx 설정 (현재 상태)

설정 파일 2개가 `/etc/nginx/conf.d/`에 있음:

```nginx
# /etc/nginx/conf.d/hr.conf
server {
    listen 80;
    server_name hr.sto.or.kr;
    root /usr/share/nginx/html;
    index index.html;
    location / { try_files $uri $uri.html $uri/ =404; }
}

# /etc/nginx/conf.d/hr2.conf
server {
    listen 80;
    server_name hr2.sto.or.kr;
    root /usr/share/nginx/html/hr2;
    index index.html;
    location / { try_files $uri $uri.html $uri/ =404; }
}
```

**설정을 바꾼 뒤에는 반드시:**
```
sudo nginx -t                    ← syntax is ok / test is successful 확인
sudo systemctl reload nginx      ← 이걸 해야 적용됨
```

⚠️ **hr.conf를 절대 지우지 말 것.** hr.conf가 없으면 conf.d가 기본 서버 블록보다 먼저 로드되는 구조 때문에
hr2가 기본 서버가 되어, hr.sto.or.kr 요청까지 hr2(담당자 허브)로 가는 버그가 재발한다. (실제 겪음)

## 6. 트러블슈팅 (실제 겪은 사례)

| 증상 | 원인 | 해결 |
|---|---|---|
| 접속 timeout | ACG 미허용 위치(외부망)에서 접속 | 사내망 또는 VPN에서 접속 |
| "연결을 거부했습니다" | https로 자동 접속 시도 (인증서 없음) | 주소에 `http://` 명시 |
| scp `No such file or directory` | PC 파일 경로 오타 | 탐색기에서 폴더 열고 주소창에 `cmd` 입력 → `scp 파일명 <서버계정>@...:~/` |
| scp/ssh `Permission denied` (비번 후) | 로그인 인증 실패 | 한/영 확인, 비번 붙여넣기(우클릭). sudo 문제 아님 |
| `Connection closed by ... port 22` | 비번 연속 실패로 일시 차단 | 5~10분 대기 후 재시도 |
| 수정했는데 화면 그대로 | ② sudo cp 누락 or 브라우저 캐시 | 2단계 배포 확인 + Ctrl+F5 |
| 이상한 페이지가 뜸 | index 덮어쓰기 사고 or nginx 설정 | 아래 진단 명령으로 확인 |
| 404 (nginx error) | 그 이름의 파일이 웹루트에 없음 | `ls -al /usr/share/nginx/html/` 로 확인 후 cp |
| 403 Forbidden | SELinux 컨텍스트 | `sudo restorecon -Rv /usr/share/nginx/html/` |

**진단 명령 모음 (ssh 창):**
```
# 서버가 실제로 내보내는 페이지 확인 (브라우저 캐시 배제)
curl -s -H "Host: hr.sto.or.kr" http://localhost/ | grep -o "<title>[^<]*</title>"
curl -s -H "Host: hr2.sto.or.kr" http://localhost/ | grep -o "<title>[^<]*</title>"

# 웹루트 파일 목록·시각 확인
ls -al /usr/share/nginx/html/ /usr/share/nginx/html/hr2/

# 로드된 nginx 설정 전체 확인
sudo nginx -T | grep -n "listen\|server_name\|root\|index"

# 특정 문자열이 어느 파일에 있는지 (예: 남은 GitHub 링크 찾기)
grep -o 'href="[^"]*github[^"]*"' /usr/share/nginx/html/hr2/*.html
```

## 7. 히스토리 / 알아두면 좋은 것

- 2026-07-08: nginx 설치, hr/hr2 분리 설정, 허브 2종 배포 완료
- 2026-07-09: 소스 관리를 이 레포(`nice2meetu-sto/hr_hub`)로 이관 시작
- 기존 index.html이 testpage로 가는 **심볼릭 링크**였음 → 삭제하고 실파일로 교체함 (재발 시 `ls -al`에서 `->` 표시로 확인)
- 페이지들은 기존 GitHub Pages(nice2meetu-sto.github.io)에서 이 서버로 이전 중 — 코드 내 github.io 링크를 서버 상대경로로 바꾸는 작업 병행
- 서버에 웹(80) 외 일부 앱/DB 포트도 ACG에 열려 있음 (상세는 별도 보관) — 추후 정적 HTML을 넘어 앱/DB 확장 여지 있음
- 허브 스타일: Pretendard 서체, 카드 그리드, CSS 변수(--ink, --blue 등). hub와 hub_hr2는 쌍둥이 구조라 **한쪽 버그 수정 시 다른 쪽도 확인**
