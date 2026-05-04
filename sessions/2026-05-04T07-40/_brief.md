# 📋 작업 브리프

**원 명령:** 깃허브 싱크 실패가 자꾸 되는데 어떻게 해결하지

## 요약
깃허브 싱크 실패는 기술적 인프라 문제이므로, 문제의 원인 진단(Researcher)과 기술적 해결책 제시(Developer)가 핵심입니다. 실패의 범위와 비즈니스 영향도를 먼저 파악하고, 개발자에게 로그 분석 및 환경 점검을 지시해야 합니다.

## 분배
- **🔍 Researcher**: GitHub 싱크 실패의 일반적인 원인(API Rate Limit 초과, Credential 만료/오류, Branch Conflict, Webhook 설정 오류 등)을 목록화하고, 각 원인에 따른 자가 진단 질문 리스트를 작성하여 공유해 주세요.
- **💰 Business**: 싱크 실패가 현재 비즈니스에 미치는 영향을 분석해 주세요. (예: 어떤 기능 개발이 멈췄는지, 어떤 데이터가 최신화되지 않아 KPI 산출에 차질이 생기는지 등) 문제의 심각도(Severity)를 기준으로 우선순위를 매겨주세요.
- **💻 Developer**: 싱크 실패 문제를 해결하기 위한 기술적 체크리스트를 작성해 주세요. (1) API 토큰 및 권한(Scope) 재확인, (2) 서버/로컬 환경 변수(Environment Variables) 점검, (3) 실패 로그(Error Log)를 기반으로 구체적인 오류 코드를 분석하고, 임시 해결 방안(Workaround)을 제안해 주세요.
- **📱 Secretary**: 위에서 수집된 정보(원인, 영향도, 기술적 체크리스트)를 종합하여, 사용자에게 전달할 'GitHub Sync Failure Troubleshooting Checklist'를 단계별(Step 1, Step 2...)로 요약하고, 각 단계별 담당자(Who)와 예상 소요 시간(When)을 명시해 주세요.
