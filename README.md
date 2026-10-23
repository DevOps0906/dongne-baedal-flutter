# dongne-baedal-flutter

동네배달의 Flutter 앱 레포입니다. 한 앱 안에 **고객 모드**와 **라이더 모드**가 있습니다.

| 모드 | 화면 |
|---|---|
| 고객 | 홈, 검색, 찜, 주문내역, 마이, 가게 상세, 장바구니, 결제, 주문 현황 지도 |
| 라이더 | 운행 홈, 배차 요청, 진행 중 배달, 배달 완료, 수익 |

기획·디자인·토큰·DB 스키마는 `spec/` 서브모듈(`dongne-baedal-spec`)에 있습니다.
**작업 전에 `spec/docs/04-dev-instructions.md`(개발지시서)를 먼저 읽습니다.**

> 현재 상태: 개발 전. 코드는 개발지시서 `APP-001`부터 시작합니다.

---

## 개발 방법

### 1. 기술 스택

| 영역 | 선택 |
|---|---|
| SDK | Flutter 3.32 / Dart 3.8, minSdk 26 |
| 구조 | Clean Architecture (data / domain / presentation) |
| 상태 | Riverpod 2.6 |
| 라우팅 | GoRouter 14 (고객 5탭 셸, 라이더 모드 분리) |
| 모델 | Freezed, json_serializable |
| 백엔드 | supabase_flutter (Auth, DB, Realtime, Storage) |
| 로그인 | 휴대폰 OTP, 카카오·네이버(Edge Function → JWT), 애플 |
| 지도·위치 | flutter_naver_map, geolocator, 백그라운드 위치 서비스 |
| 푸시 | firebase_messaging |
| 보안 저장 | flutter_secure_storage |

패키지명: `com.nomad.dongnebaedal`

### 2. 받기와 실행

```bash
git clone --recurse-submodules https://github.com/DevOps0906/dongne-baedal-flutter.git
cd dongne-baedal-flutter
git submodule update --init      # 서브모듈 없이 받았다면

# APP-001 이후
flutter pub get
dart run tool/gen_tokens.dart    # spec/tokens/tokens.json → lib/core/theme/tokens.g.dart
dart run build_runner build -d   # Freezed 등 코드 생성
flutter run \
  --dart-define=SUPABASE_URL=... \
  --dart-define=SUPABASE_ANON_KEY=... \
  --dart-define=NAVER_MAP_CLIENT_ID=...
```

로컬 DB는 spec 레포에서 `supabase start` 후 나온 값을 넣습니다.

### 3. spec 갱신

```bash
git submodule update --remote spec
dart run tool/gen_tokens.dart
git commit -am "chore: spec 갱신"
```

`lib/core/theme/tokens.g.dart`는 생성 파일이라 직접 고치지 않습니다.

### 4. 폴더 구조

```
lib/core/theme/        tokens.g.dart(생성), app_theme.dart
lib/core/router/       GoRouter
lib/core/utils/        breakpoint.dart (~767 / 768~1199 / 1200~)
lib/features/          auth, address, home, store, cart, order, review, my
lib/features/rider/    라이더 모드
tool/gen_tokens.dart
spec/                  서브모듈
```

### 5. 라이더 위치 전송 기준

| 항목 | 값 |
|---|---|
| 전송 간격 | 이동 중 3~5초, 정차 시 15초 |
| 실시간 전달 | Supabase Realtime Broadcast |
| DB 저장 | 30초마다 `rider_locations` upsert |
| 권한 | 운행 시작 시 백그라운드 위치 권한 요청, 운행 종료 시 전송 중단 |
| 버튼 | 주요 버튼 높이 56 이상, 본문 17px |

### 6. 작업 규칙

- 티켓 하나에 브랜치 하나: `feat/APP-041-location-stream`
- 커밋: Conventional Commits, 제목에 티켓 번호
- PR 전에 `flutter analyze && flutter test`
- PR에 휴대폰·태블릿 캡처 첨부
- 색·글자 크기·간격은 `tokens.g.dart`만 사용

### 7. 앱 티켓 요약

| 단계 | 티켓 |
|---|---|
| P0 | APP-001 생성·토큰, APP-002 구조·라우팅, APP-003 인증 |
| P1 | APP-010~015 고객 탐색 |
| P2 | APP-020~022 장바구니·결제·주문내역 |
| P4 | APP-040~043 라이더 모드, 위치 전송, 주문 현황 지도 |

자세한 완료 기준은 `spec/docs/04-dev-instructions.md`에 있습니다.
