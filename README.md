# Telegram News & Price Bot

뉴스 RSS 수집 + 시장가격 알림 텔레그램 봇. EV·배터리·반도체·원자재 뉴스를
Claude Haiku로 한국어 3줄 요약 + 환율/지수/원자재/국채/광물 가격 조회.

## 명령어

| 명령 | 설명 |
|---|---|
| `/update` | 글로벌 주요 지수 7종 (🇰🇷한국·🇺🇸S&P500·나스닥100·다우·🇯🇵닛케이225·🇨🇳CSI300·🇪🇺유로스톡스50) |
| `/status` | 시장 상태(KST/ET, 미국장·Forex) + 최근 사이클 통계 |
| `/force` | 시장 휴장 무시하고 시세 강제 조회 |
| `/rate` | 미 국채 7개 만기 수익률 + 직전 거래일 대비 bp 변동 |
| `/mineral` | 광물 선물 3종 (탄산리튬·구리·니켈) + 귀금속 (금·은) |
| `/oil` | 국제 유가 (WTI·Brent) |
| `/diag` | 21개 RSS 피드 각각의 상태 진단 |
| `/clear` | 전송 기록 초기화 |

## 데이터 소스

- **뉴스**: 21개 RSS (Electrek, CleanTechnica, mining.com, CNBC, Reuters/Bloomberg via Google News 등)
- **요약**: Anthropic Claude Haiku 4.5 (영문→한국어 3줄, 본문 부재 시 제목만 번역)
- **시세 일반**: yfinance (commodities/indices/FX)
- **미국채 수익률**: U.S. Treasury 공식 일별 CSV
- **광물 선물 (`/mineral`)**:
  - 탄산리튬: eastmoney GFEX 주력 계약 자동 선택 (만기 롤오버 자동 대응)
  - 구리 선물: yfinance `HG=F` (COMEX 구리 선물 = investing.com Copper와 동일 시장)
  - 니켈 선물: TradingEconomics (LME 니켈 = investing.com Nickel과 동일 시장)
  - ※ investing.com은 Cloudflare로 서버 크롤링 불가 → 동일 거래소·동일 선물을 다른 경로로 조회
- **국제 유가 (`/oil`)**: 가격은 oilprice.com (WTI/Brent), 증감률은 **12시간 전 대비** (yfinance 시간봉). 등락 이모티콘 없이 (±x.xx%)만 표기

## 운영 주기

- 뉴스: 10분마다 자동 체크
- 시세: 3시간마다 자동 (미국장/Forex 운영 시간만)
- 키워드 미스매치/날짜 초과/이미 전송한 링크는 자동 필터

## 급변 알람 (5% 룰)

전일 종가 대비 **±5% 이상 변동**하면 `⚠️ 가격 급변 알람` 발송:
- **자동**: 3시간마다 지수(/update)·광물(/mineral)·유가(/oil) 모니터링
- **수동**: `/update` `/mineral` `/oil` 입력 시 응답 끝에 급변 항목 함께 표시
- 국채 수익률(/rate)은 가격이 아닌 수익률이라 5% 룰 제외
- 양식: `🔺 급등 / ▼ 급락 {종목} {변동률}% 변동!`

## 셋업

1. `.env.example`을 `.env`로 복사
2. Telegram BotFather에서 봇 토큰 발급 → `TELEGRAM_TOKEN`
3. 본인 user_id 확인 → `TELEGRAM_CHAT_ID`
4. Anthropic Console에서 API 키 발급 → `ANTHROPIC_API_KEY`
5. `pip install -r requirements.txt`
6. `python bot.py`

## 배포 (Google Cloud Always Free)

상세는 [배포가이드_GCP.md](배포가이드_GCP.md) 참고. 요약:
- e2-micro VM (us-west1, Ubuntu 22.04)
- `deploy/install.sh`로 systemd 서비스 등록
- 무료, 무중단, 자동 재시작

## 디렉토리

```
telegram-bot/
├── bot.py                  # 메인 봇 코드
├── requirements.txt        # 의존성
├── .env.example            # 시크릿 템플릿
├── .gitignore              # .env, venv, sent_links.json 제외
├── deploy/
│   ├── install.sh          # GCP/Oracle Ubuntu 자동 설치 스크립트
│   └── newsbot.service     # systemd 서비스 유닛
├── 배포가이드.md            # Oracle Cloud 배포 가이드 (대안)
└── 배포가이드_GCP.md         # Google Cloud 배포 가이드 (현재 사용 중)
```

## 보안

- `.env` 절대 커밋 금지 (`.gitignore`로 차단)
- `sent_links.json` 개인 활동 기록 제외
- `venv/` 환경 의존적 제외
