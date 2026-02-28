# OpenClaw 보안 취약점 및 상업용 위험성 리서치

> 오픈클로(OpenClaw) AI 에이전트의 보안 취약점과 기업 환경에서의 사용 위험성을 분석한 리서치 보고서입니다.

[![GitHub Pages](https://img.shields.io/badge/Demo-GitHub%20Pages-blue?style=for-the-badge)](https://kyangkyongkyungkyang.github.io/openclaw-security-research/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

## Demo

**[데모 페이지 바로가기](https://kyangkyongkyungkyang.github.io/openclaw-security-research/)**

모바일 반응형으로 제작되어 어떤 디바이스에서든 최적화된 읽기 경험을 제공합니다.

## 핵심 발견사항

### 주요 보안 취약점

| # | 취약점 | 심각도 |
|---|--------|--------|
| 1 | **CVE-2026-25253** — 1-Click RCE (원격 코드 실행) | CVSS 8.8 |
| 2 | **API 키 평문 저장** — 자격증명 암호화 없이 저장 | High |
| 3 | **ClawHavoc 공급망 공격** — ClawHub 스킬 ~20%가 악성코드 | High |
| 4 | **프롬프트 인젝션** — 이메일로 개인 키 탈취 실증 | High |
| 5 | **인증 우회** — 노출 인스턴스 93.4%가 인증 우회 가능 | High |

### 기업 사용 금지 현황

- **네이버 / 카카오 / 당근** — 2026.02.08 사내 사용 전면 금지
- **Microsoft** — "표준 워크스테이션에서 실행 부적합"
- **CrowdStrike** — "강력한 AI 백도어가 될 수 있음"
- **Palo Alto Networks** — "2026년 최대 내부자 위협"

### 최종 결론

> 기업 환경에서의 오픈클로 사용은 현 시점에서 **강력히 비권장**됩니다.

## 프로젝트 구조

```
openclaw-security-research/
├── index.html          # 모바일 반응형 데모 페이지
├── research.md         # 전체 리서치 보고서 원본
└── README.md           # 프로젝트 소개
```

## 출처

본 리서치는 아래 보안 기업 및 공식 CVE 데이터를 기반으로 작성되었습니다:

- [NVD - CVE-2026-25253](https://nvd.nist.gov/vuln/detail/CVE-2026-25253)
- [Microsoft Security Blog](https://www.microsoft.com/en-us/security/blog/2026/02/19/running-openclaw-safely-identity-isolation-runtime-risk/)
- [CrowdStrike](https://www.crowdstrike.com/en-us/blog/what-security-teams-need-to-know-about-openclaw-ai-super-agent/)
- [The Hacker News](https://thehackernews.com/2026/02/openclaw-bug-enables-one-click-remote.html)
- [Kaspersky](https://www.kaspersky.com/blog/moltbot-enterprise-risk-management/55317/)

전체 출처 목록은 [리서치 보고서](research.md#6-출처-sources)에서 확인하세요.

## 라이선스

MIT License
