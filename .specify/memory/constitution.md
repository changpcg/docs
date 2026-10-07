<!--
Sync Impact Report
- Version change: (template) → 1.0.0
- Modified principles: 템플릿 자리표시자 5개 → 원칙 6개로 새로 정의
  I. 서버가 최종 판단한다 / II. 설치 없이 돈다 / III. 회원은 한 곳, 연결은 서명으로 /
  IV. 데이터는 지키고 스키마는 더하기만 / V. 요구사항 ID로 추적한다 / VI. 한국어·모든 화면·모든 사람
- Added sections: 기술·보안 제약, 개발 흐름과 품질 관문, Governance
- Removed sections: 없음
- Templates: plan/spec/tasks 템플릿은 실행 시 이 헌법을 읽으므로 수정하지 않음
- Follow-up TODOs: 없음 (Ratified = Spec Kit 도입일 2026-10-07)
-->

# 나만의 블로그 Constitution

## Core Principles

### I. 서버가 최종 판단한다 (NON-NEGOTIABLE)

- 권한·입력 검증·횟수 제한·허용 목록은 MUST 서버(블로그 서버·회원 서버)에서 강제한다.
  화면 검사는 빠른 안내용일 뿐이며, 화면 검사만으로 막는 기능은 완료로 보지 않는다.
- 모든 SQL은 MUST 자리표시자 바인딩을 쓴다(PHP는 PDO prepare + 에뮬레이션 끔).
- 모든 출력은 MUST 이스케이프하고, 마크다운 본문은 DOMPurify를 거친다.
- 비밀번호는 MUST 해시로만 저장한다(PHP bcrypt, 블로그 PBKDF2-SHA256 10만 회 + 솔트).
- 비밀번호·토큰·키는 MUST NOT 주소(쿼리스트링)·로그·화면에 노출된다.

근거: 이 서비스는 방문자 댓글·업로드·SNS 로그인을 받으므로, 화면을 우회한 요청이
기본 공격 경로다.

### II. 설치 없이 돈다

- 블로그 서버는 MUST Python 3.9 표준 라이브러리만 쓴다(pip 패키지 금지).
- 회원 서버는 MUST PHP 8 + pdo_sqlite·mbstring(SNS 로그인만 curl)만 쓴다(Composer 금지).
- 화면은 바닐라 JS로 만들고, 외부 라이브러리는 jsDelivr CDN의 marked·DOMPurify·
  highlight.js·Pretendard로 한정한다. 새 CDN 의존성은 헌법 개정(MINOR) 없이 추가하지 않는다.
- 데이터는 SQLite 파일(blog.db, php-auth/db/sqlite.db)과 uploads/ 폴더에 둔다.
- 그림(미니룸 등)은 코드 안 SVG로 그리며 외부 이미지 주소를 쓰지 않는다.

근거: 누구나 `python3`와 `php -S`만으로 켤 수 있어야 하고, 의존성이 적을수록 공격 면과
고장 지점이 줄어든다.

### III. 회원은 한 곳, 연결은 서명으로

- 회원 계정은 MUST 회원 서버(PHP) 한 곳에서만 만든다. 블로그 자체 로그인은 관리자(admin)와
  아직 연결하지 않은 예전 블로그 계정에만 허용한다.
- 두 서버 사이의 신뢰는 MUST sso.key로 만든 HMAC-SHA256 서명만으로 성립한다.
  서명된 표(입장권·로그아웃 표 등)는 만료 시간을 갖고, 상태를 바꾸는 표는 1회용이어야 한다.
- 계정 연결은 MUST 회원 번호로만 한다. 이메일·이름으로 자동 병합하지 않는다.
- 상태를 바꾸는 요청(로그인·로그아웃·가입·삭제)은 MUST POST + CSRF 확인을 거친다.
  GET 링크 클릭만으로 상태가 바뀌면 안 된다.

근거: 서버가 둘이라 둘 사이의 경계가 가장 위조되기 쉬운 곳이다.

### IV. 데이터는 지키고 스키마는 더하기만

- 기능이 늘어도 MUST 기존 blog.db·회원 DB를 그대로 열 수 있어야 한다. 스키마 변경은
  필요한 칸·표를 자동으로 더하는 방식만 쓰고, 기존 글·댓글·계정을 지우거나 바꾸지 않는다.
- 서버를 다시 켜도 글·댓글·블로그 로그인(2주)이 유지되어야 한다.
- 백업 범위는 blog.db·uploads/·php-auth/db/·php-auth/oauth.config.php이며, 회원 DB와
  blog.db는 MUST 함께 백업·복원한다.
- 삭제는 연관 데이터(댓글·공감·이웃·방문 기록)까지 일관되게 처리하고, 답글이 남은 댓글은
  "삭제된 댓글입니다" 자리로 남긴다.

근거: 개인 블로그의 글은 되돌릴 수 없는 사용자 자산이다.

### V. 요구사항 ID로 추적한다

- 모든 기능·수정은 MUST `requirements.md`의 ID(AUTH·BLOG·EXT·SEC·NFR·SOC)에
  연결된다. 새 기능은 새 ID를 받고, 표의 "완료 기준"이 곧 인수 테스트다.
- spec.md의 각 기능 요구사항은 관련 기존 ID를 적는다.
- 구현이 끝나면 MUST `requirements.md`(와 원본 Claude Docs 문서)의 표·남은 과제 체크를
  함께 갱신한다. 문서와 동작이 다르면 버그로 본다.

근거: 요구사항 정의서가 이 프로젝트의 단일 기준이며, Spec Kit 산출물은 그것을 바꾸는 단위다.

### VI. 한국어·모든 화면·모든 사람

- 화면 문구와 오류는 MUST 한국어로, 사용자가 다음에 할 일을 알 수 있게 쓴다.
- 375px 휴대폰에서 가로 넘침이 없어야 하고, 라이트·다크 모드 모두에서 읽혀야 한다.
- '동작 줄이기' 설정을 존중하고, 키보드 포커스 표시와 화면 읽기용 이름을 갖춘다.
- 디자인은 네오 브루탈리즘 카드 규칙(NFR-01)을 따른다.

근거: 사용자는 한국어 사용자이며 PC·휴대폰을 함께 쓴다.

## 기술·보안 제약

- 구성: 블로그 서버(Python, 기본 127.0.0.1:8000) + 회원 서버(PHP, 8080). 주소·포트는
  PORT·HOST·AUTH_URL·BLOG_URL 환경변수로만 바꾼다.
- 세션: HttpOnly 쿠키, 로그인 시 세션 번호 재발급, 블로그 쿠키 SameSite=Strict,
  PHP 쿠키 SameSite=Lax.
- 무차별 대입: 비밀번호를 받는 모든 입구는 횟수 제한 또는 지연을 MUST 갖는다.
- 업로드: 허용 목록 형식·크기만 받고, 저장 이름은 무작위, nosniff, 사진 외 파일은
  attachment로 내려준다.
- 비밀: DB·키 파일은 웹 폴더 밖, sso.key 권한 600, 외부 API 키는 등록 여부만 화면에 보인다.
- 외부 요청: 서버의 외부 호출은 8초 제한과 캐시를 두고, 실패 시 직전 값을 유지한다.
- 인터넷 공개(배포)는 현재 범위 밖이다. 배포를 다루는 기능은 HTTPS·Secure 쿠키·
  관리자 비밀번호 교체를 함께 다뤄야 한다.

## 개발 흐름과 품질 관문

1. `/speckit-specify` → spec.md: 무엇을·왜. 관련 요구사항 ID와 완료 기준을 적는다.
2. `/speckit-clarify`(필요 시) → 모호한 점을 질문으로 정리한다.
3. `/speckit-plan` → plan.md: Constitution Check에서 원칙 I~VI 위반 여부를 표로 확인한다.
   위반은 Complexity Tracking에 이유를 적어야만 허용된다.
4. `/speckit-tasks` → tasks.md, `/speckit-analyze`로 산출물 일관성 확인.
5. `/speckit-implement` → 구현 후 다음을 MUST 확인한다.
   - 각 완료 기준을 실제 요청(브라우저 또는 curl)으로 재현해 통과
   - 보안 항목은 화면을 우회한 직접 요청으로 거절되는지 확인
   - 기존 blog.db로 서버를 켜 데이터가 그대로인지 확인
   - requirements.md 갱신

## Governance

- 이 헌법은 다른 개발 관행보다 우선한다. plan.md의 Constitution Check는 매 기능마다 필수다.
- 개정: 변경 이유와 영향을 Sync Impact Report로 남기고 버전을 올린다.
  - MAJOR: 원칙 삭제·재정의(예: 표준 라이브러리 원칙 폐기)
  - MINOR: 원칙·섹션 추가, 허용 의존성 추가, 지침의 실질적 확대
  - PATCH: 문구·오타·설명 보완
- 리뷰: 모든 변경은 원칙 I(서버 강제)과 IV(데이터 보존)를 먼저 확인한다.
- 실행 중 참고 문서는 `requirements.md`이며, 원본은 Claude Docs 문서
  "나만의 블로그 요구사항 정의서"다.

**Version**: 1.0.0 | **Ratified**: 2026-10-07 | **Last Amended**: 2026-10-07
