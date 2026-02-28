# 오픈클로(OpenClaw) 보안 취약점 및 상업용 위험성 리서치 보고서

> **작성일**: 2026년 2월 28일
> **연구 깊이**: Deep (3-4 hops)
> **신뢰도**: 높음 (다수의 보안 기업 및 공식 CVE 데이터 기반)

---

## Executive Summary

오픈클로(OpenClaw)는 2025년 11월 오스트리아 개발자 피터 스타인버거가 개인 프로젝트로 시작한 오픈소스 AI 에이전트로, 2026년 1월 GitHub 스타 18만 개를 돌파하며 폭발적으로 성장했다. 그러나 이 급속한 성장과 함께 **심각한 보안 위기**가 동시에 발생했다.

**핵심 결론**: 오픈클로는 현재 상태로는 **기업 상업 환경에서 사용하기에 부적합**하다. CVE-2026-25253(CVSS 8.8) 원격 코드 실행 취약점, 클로허브(ClawHub)의 대규모 공급망 공격(악성 스킬 800개 이상), API 키 평문 저장, 인증 우회 등 다층적 보안 문제가 존재하며, 네이버·카카오·당근을 포함한 주요 기업들이 사내 사용 금지령을 내렸다. Microsoft와 CrowdStrike 등 글로벌 보안 기업들도 기업 환경에서의 사용을 강력히 경고하고 있다.

---

## 1. 오픈클로(OpenClaw) 개요

### 1.1 프로젝트 정의

오픈클로는 LLM(거대언어모델)을 기반으로 사용자의 PC 화면을 인식하고 마우스와 키보드를 조작하여 실제 작업을 수행하는 **오픈소스 AI 에이전트**이다.

- **개발자**: 피터 스타인버거 (오스트리아, iOS 개발자 출신)
- **최초 출시**: 2025년 11월 (원래 이름: ClaudeBot → Clawdbot → MoltBot → OpenClaw)
- **라이선스**: MIT License (상업적 사용 자유)
- **GitHub 스타**: 180,000+ (2026년 2월 기준)
- **현 상태**: OpenAI가 창시자 영입, 독립 재단에서 오픈소스 유지 예정

### 1.2 핵심 기능

- PC 화면 인식 및 마우스/키보드 자동 조작
- WhatsApp, Slack, iMessage, 이메일 등 메신저 연동
- 일정 관리, 항공편 예약, 파일 정리 등 자동화
- ChatGPT, Gemini, Claude 등 다양한 LLM 연동 가능
- 스킬(Skill) 마켓플레이스인 ClawHub를 통한 기능 확장

---

## 2. 보안 취약점 상세 분석

### 2.1 CVE-2026-25253: 원격 코드 실행 (RCE) 취약점 [치명적]

| 항목 | 내용 |
|------|------|
| **CVE 번호** | CVE-2026-25253 |
| **CVSS 점수** | 8.8 (High) |
| **CWE 분류** | CWE-669 (잘못된 리소스 전송) |
| **발견자** | Mav Levin (depthfirst 연구팀) |
| **패치 버전** | v2026.1.29 (2026년 1월 30일) |

#### 공격 메커니즘

1. **게이트웨이 URL 조작**: 컨트롤 UI가 쿼리 스트링의 `gatewayUrl` 값을 검증 없이 신뢰
2. **인증 토큰 탈취**: 악성 URL 클릭 시 오픈클로가 공격자 서버로 인증 토큰 자동 전송
3. **WebSocket Origin 미검증**: 오픈클로 서버가 WebSocket origin 헤더를 검증하지 않아 Cross-Site WebSocket Hijacking 가능
4. **보안 가드레일 해제**: 탈취한 토큰으로 사용자 확인 절차를 비활성화하고 임의 셸 명령 실행

#### 핵심 위험

- **localhost에서도 취약**: 인터넷에 노출하지 않더라도, 피해자의 브라우저를 통해 로컬 네트워크에 침투 가능
- **1-Click 공격**: 악성 링크 하나만 클릭하면 PC 제어권 탈취

### 2.2 API 키 평문 저장 취약점

- `~/.openclaw/credentials/` 디렉토리에 API 키가 **평문 Markdown 및 JSON** 파일로 저장
- 삭제된 키도 `.bak` 파일에 잔존
- 정보 탈취 악성코드(AMOS, RedLine, Lumma, Vidar)의 주요 표적
- 비밀번호/토큰 복잡도 정책 부재 — "a" 같은 단순 문자도 유효한 패스워드로 허용

### 2.3 인증 우회 취약점

- 인터넷에 노출된 오픈클로 인스턴스: **42,665개** 확인 (보안 연구원 Maor Dayan 조사)
- 이 중 5,194개가 실제 취약한 것으로 검증
- **93.4%가 인증 우회 취약점** 보유
- 인증이 필수이나, 토큰 복잡도 정책이 없어 브루트포스 공격에 취약

### 2.4 프롬프트 인젝션 공격

- 이메일, 문서, 웹페이지, 이미지에 삽입된 악성 콘텐츠가 AI 에이전트의 행동을 조작 가능
- **실증 사례**: Archestra.AI CEO Matvey Kukuy가 이메일에 프롬프트 인젝션을 삽입하여 오픈클로가 연결된 PC에서 **개인 키(private key)를 탈취**하는 시연 성공
- AI 에이전트가 시스템 권한을 가진 상태에서 프롬프트 인젝션에 취약하면, 전통적 보안 경계가 무력화

### 2.5 ClawHub 공급망 공격 (ClawHavoc 캠페인)

| 항목 | 수치 |
|------|------|
| **발견된 악성 스킬** | 최초 341개 → 이후 800개 이상 (레지스트리의 ~20%) |
| **최초 14개 업로드** | 2026년 1월 27-29일 (공식 레지스트리에 3일간) |
| **주요 페이로드** | Atomic macOS Stealer (AMOS) |
| **공격 방식** | 가짜 사전 요구사항으로 AMOS 설치 유도 |
| **탈취 대상** | 오픈클로 API 키 포함 전반적 자격 증명 |

### 2.6 Moltbook 데이터 유출 사건

오픈클로 생태계의 소셜 네트워크 플랫폼인 Moltbook에서:
- Supabase 백엔드 설정 오류로 API가 오픈 데이터베이스에 노출
- **약 35,000개 이메일 주소** 유출
- **약 1,500,000개 에이전트 토큰** 유출
- 누구나 해당 에이전트의 제어권 탈취 가능

### 2.7 보안 감사 결과 요약

2026년 1월 보안 감사에서:
- **총 512개 취약점** 확인
- **8개는 치명적(Critical)** — 인증 및 시크릿 관리 영역
- 기본적인 보안 설계 원칙이 미흡한 상태

---

## 3. 상업용 사용 시 위험성 분석

### 3.1 기업 보안 인증 부재

| 인증/규격 | 상태 |
|-----------|------|
| SOC 2 | 미보유 |
| ISO 27001 | 미보유 |
| HIPAA | 미준수 |
| GDPR 컨트롤러 | 자체 구현 필요 |
| 감사 로깅 | 미지원 |
| RBAC(역할 기반 접근 제어) | 미지원 |
| 데이터 암호화 표준 | 미흡 |
| 네트워크 세분화 | 미지원 |

### 3.2 주요 기업의 사용 금지 현황

#### 국내
- **네이버**: 사내망 및 업무용 기기에서 사용 금지/제한 내부 지침 공지
- **카카오**: 동일하게 사내 사용 금지령
- **당근마켓**: 동일하게 사내 사용 금지령
- **시행일**: 2026년 2월 8일 일제히 시행

#### 글로벌
- **Microsoft AI 안전팀**: "OpenClaw는 아직 기업용으로 쓰기엔 보안이 미흡하다"는 내부 메모를 전사 공유
- **중국 공업정보화부(MIIT)**: OpenClaw가 사이버 공격 통로가 될 수 있다고 공식 경고

### 3.3 글로벌 보안 기업의 위험 평가

| 기업 | 평가 내용 |
|------|-----------|
| **Microsoft** | "표준 개인 또는 기업 워크스테이션에서 실행하기에 부적합. 반드시 완전 격리된 환경(전용 VM, 별도 물리 시스템)에서만 평가해야 함" |
| **CrowdStrike** | "직원들이 업무 노트북에 설치하면 적대자를 위한 '강력한 AI 백도어'로 작동할 수 있음" |
| **Palo Alto Networks** | "2026년 최대의 내부자 위협이 될 가능성" |
| **Kaspersky** | "비전문가 사용자가 설치할 수 있는 가장 위험한 소프트웨어 중 하나" |
| **Cisco** | "개인 AI 에이전트는 보안의 악몽" |
| **Sophos** | "기업 AI 보안에 대한 경고탄" |

### 3.4 GDPR 및 컴플라이언스 위험

- 오픈클로는 SaaS가 아닌 **인프라 소프트웨어**
- 자체 호스팅 시: 사용자가 **데이터 컨트롤러**(및 잠재적 프로세서) 역할
- 적법한 근거, DPIA(데이터 보호 영향 평가), 프로세스를 사용자가 직접 처리해야 함
- 비관리 에이전트를 기업 데이터에 연결하면:
  - GDPR/HIPAA 등 대부분의 컴플라이언스 표준 위반
  - **'Shadow AI' 위험** 발생 — 민감 데이터의 도난 또는 변조 가능

### 3.5 라이선스 위험 (낮음)

- 오픈클로는 **MIT 라이선스** 사용
- 상업적 사용에 법적 제한 없음
- 수정, 배포, 상업적 통합 자유
- 단, AGPL 라이선스 우려와는 무관하므로 **라이선스 자체의 위험은 낮은 편**

### 3.6 운영 리스크

| 위험 유형 | 상세 |
|-----------|------|
| **데이터 유출** | 프롬프트 인젝션을 통한 기업 기밀 유출 가능 |
| **공급망 오염** | ClawHub의 ~20%가 악성 스킬 — 검증 없이 설치 시 말웨어 감염 |
| **권한 상승** | AI 에이전트가 시스템 수준 권한으로 동작 시 공격 표면 극대화 |
| **감사 추적 불가** | 감사 로깅 미지원으로 사후 사고 분석 어려움 |
| **통제 불가능** | 에이전트가 자율적으로 행동하므로 예측 불가능한 동작 발생 가능 |
| **제3자 API 비용** | 종량제 API 비용이 통제 없이 증가할 수 있음 |

---

## 4. 종합 위험 매트릭스

| 위험 영역 | 심각도 | 발생 가능성 | 종합 위험도 |
|-----------|--------|-------------|-------------|
| 원격 코드 실행 (RCE) | 치명적 | 높음 | **극히 높음** |
| API 키/자격증명 탈취 | 높음 | 높음 | **매우 높음** |
| 공급망 공격 (악성 스킬) | 높음 | 높음 | **매우 높음** |
| 프롬프트 인젝션 | 높음 | 중간 | **높음** |
| 데이터 유출/프라이버시 | 높음 | 중간 | **높음** |
| 컴플라이언스 위반 | 중간 | 높음 | **높음** |
| 인증 우회 | 높음 | 중간 | **높음** |
| 라이선스 위험 | 낮음 | 낮음 | **낮음** |

---

## 5. 권고사항

### 5.1 즉시 조치 (기업)

1. **사내 사용 전면 금지** 또는 엄격한 통제 정책 수립
2. 기존 설치 인스턴스 탐지 및 제거 (SecureClaw 등 탐지 도구 활용)
3. 업무 기기에서의 비인가 AI 에이전트 설치 모니터링 체계 구축

### 5.2 불가피하게 평가가 필요한 경우

1. **완전 격리된 환경**에서만 운영 (전용 VM 또는 별도 물리 시스템)
2. **비권한 전용 계정** 사용
3. **비민감 데이터**만 접근 허용
4. 지속적 모니터링 및 재구축 계획 수립
5. 최소한 v2026.1.29 이상으로 업데이트
6. ClawHub 스킬은 반드시 코드 리뷰 후 설치

### 5.3 개인 사용자

1. 최신 버전(v2026.1.29 이상)으로 반드시 업데이트
2. 강력한 인증 토큰 설정
3. 인터넷에 인스턴스를 노출하지 않기
4. 의심스러운 스킬 설치 자제
5. API 키를 정기적으로 교체
6. 이메일 자동 처리 등 민감한 자동화 기능 주의

---

## 6. 출처 (Sources)

### CVE 및 기술적 취약점
- [NVD - CVE-2026-25253](https://nvd.nist.gov/vuln/detail/CVE-2026-25253)
- [The Hacker News: OpenClaw Bug Enables One-Click RCE](https://thehackernews.com/2026/02/openclaw-bug-enables-one-click-remote.html)
- [SOCRadar: CVE-2026-25253 RCE Analysis](https://socradar.io/blog/cve-2026-25253-rce-openclaw-auth-token/)
- [runZero: OpenClaw RCE Vulnerability](https://www.runzero.com/blog/openclaw/)

### 악성 스킬 및 공급망 공격
- [The Hacker News: 341 Malicious ClawHub Skills](https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html)
- [CyberPress: ClawHavoc 1,184 Malicious Skills](https://cyberpress.org/clawhavoc-poisons-openclaws-clawhub-with-1184-malicious-skills/)
- [Trend Micro: Atomic macOS Stealer Distribution](https://www.trendmicro.com/en_us/research/26/b/openclaw-skills-used-to-distribute-atomic-macos-stealer.html)

### 기업 보안 권고
- [Microsoft Security Blog: Running OpenClaw Safely](https://www.microsoft.com/en-us/security/blog/2026/02/19/running-openclaw-safely-identity-isolation-runtime-risk/)
- [CrowdStrike: What Security Teams Need to Know](https://www.crowdstrike.com/en-us/blog/what-security-teams-need-to-know-about-openclaw-ai-super-agent/)
- [Kaspersky: Key OpenClaw Risks](https://www.kaspersky.com/blog/moltbot-enterprise-risk-management/55317/)
- [Sophos: Enterprise AI Security Warning](https://www.sophos.com/en-us/blog/the-openclaw-experiment-is-a-warning-shot-for-enterprise-ai-security)
- [Cisco: Personal AI Agents Security Nightmare](https://blogs.cisco.com/ai/personal-ai-agents-like-openclaw-are-a-security-nightmare)

### 기업 금지 현황
- [포인트경제: 네이버·카카오·당근 오픈클로 금지령](https://www.pointe.co.kr/news/articleView.html?idxno=71344)
- [ZDNet Korea: 네카당이 금지한 오픈클로](https://zdnet.co.kr/view/?no=20260209143822)
- [AI매터스: AI 에이전트 보안 공포 확산](https://aimatters.co.kr/news-report/ai-news/37951/)

### 데이터 유출 및 생태계 위험
- [Adversa.AI: OpenClaw Security Guide 2026](https://adversa.ai/blog/openclaw-security-101-vulnerabilities-hardening-2026/)
- [Conscia: The OpenClaw Security Crisis](https://conscia.com/blog/the-openclaw-security-crisis/)
- [Giskard: Data Leakage & Prompt Injection](https://www.giskard.ai/knowledge/openclaw-security-vulnerabilities-include-data-leakage-and-prompt-injection-risks)
- [Bitsight: Exposed AI Agents](https://www.bitsight.com/blog/openclaw-ai-security-risks-exposed-instances)
- [ITWorld Korea: 오픈클로 생태계 통제 없는 에이전틱 AI](https://www.itworld.co.kr/article/4129149/)

### 프로젝트 일반 정보
- [나무위키: OpenClaw](https://namu.wiki/w/OpenClaw)
- [Open Network System: 기업 사용 안전성 분석](https://blog.open-network.co.kr/openclaw-enterprise-ai-security)
- [OpenClaw Wikipedia](https://en.wikipedia.org/wiki/OpenClaw)
- [OpenClaw 공식 보안 문서](https://docs.openclaw.ai/gateway/security)

---

*본 보고서는 2026년 2월 28일 기준 공개된 정보를 바탕으로 작성되었으며, 오픈클로 프로젝트의 빠른 개발 속도를 고려할 때 최신 패치 및 업데이트 상황을 지속적으로 확인할 필요가 있습니다.*
