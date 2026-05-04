# 💻 Developer — 싱크 실패 문제를 해결하기 위한 기술적 체크리스트를 작성해 주세요. (1) API 토큰 및 권한(Scope) 재확인, (2) 서버/로컬 환경 변수(Environment Variables) 점검, (3) 실패 로그(Error Log)를 기반으로 구체적인 오류 코드를 분석하고, 임시 해결 방안(Workaround)을 제안해 주세요.

💻 Developer: 작업 시작합니다.

## 🛠️ GitHub 싱크 실패를 위한 3단계 기술적 디버깅 체크리스트

GitHub 싱크 실패는 근본적으로 **접근 권한, 환경 설정, 또는 데이터 흐름의 불일치**에서 발생합니다. 이 체크리스트는 CEO 지시사항과 앞선 분석(Rate Limit, Credential Error)을 통합하여, 개발자가 바로 실행할 수 있는 기술적 점검 목록입니다.

---

### 🔴 1단계: API 토큰 및 권한(Scope) 재확인 (Authentication)

가장 먼저 의심해야 할 부분은 '신분증' 문제입니다. 토큰 자체가 만료되었거나, 현재 작업에 필요한 권한이 빠져있을 수 있습니다.

| 점검 항목 | 문제 증상 (Error Code) | 필수 확인 사항 및 조치 | 실행 명령어/경로 |
| :--- | :--- | :--- | :--- |
| **1. 토큰 만료 및 유효성** | `401 Unauthorized` | Personal Access Token (PAT)의 만료일을 확인하고, 만료되었다면 **새 토큰을 발급**해야 합니다. | **GitHub Settings > Developer Settings > Personal Access Tokens** |
| **2. 권한 범위 (Scope)** | `403 Forbidden` | 토큰이 Repository 레벨의 쓰기 권한(`write`) 및 워크플로우 실행 권한(`workflow`)을 가지고 있는지 확인합니다. | **토큰 생성 시 Scope에 `repo` 및 `workflow` 체크 필수.** |
| **3. Webhook 설정** | `Webhook Failed` | Push/Pull Request 이벤트가 발생할 때마다 외부 시스템(CI/CD 서버)이 알림을 받는지 확인합니다. (IP 화이트리스트 필요 시) | **Repository Settings > Webhooks** (Payload URL 및 Secret 검증) |

### 🟡 2단계: 환경 변수 (Environment Variables) 점검 (Configuration)

로컬 환경과 서버(CI/CD) 환경 간의 변수 불일치가 가장 흔한 원인입니다.

| 점검 항목 | 문제 증상 | 필수 확인 사항 및 조치 | 실행 명령어/경로 |
| :--- | :--- | :--- | :--- |
| **1. 로컬 변수 로딩** | `KeyNotFoundException` | 프로젝트 루트에 `.env` 파일이 존재하며, 필요한 모든 변수(특히 `GITHUB_TOKEN`)가 **제대로 로드**되고 있는지 확인합니다. | `export GITHUB_TOKEN="[발급받은 토큰]"` (터미널에서 직접 실행하며 테스트) |
| **2. CI/CD Secret 관리** | `Missing Secret` | Jenkins, GitHub Actions, GitLab CI 등에서 사용하는 Secret/Variable Store에 토큰 값이 **실시간으로 업데이트**되었는지, 그리고 해당 Job이 이를 참조할 수 있는지 확인합니다. | CI/CD 시스템의 **Secrets Management Dashboard**에서 값 재설정. |
| **3. 환경 분리** | `Wrong Environment` | 개발(DEV), 스테이징(STAGING), 운영(PROD) 환경에 따라 사용하는 API 엔드포인트와 토큰이 분리되어야 합니다. | **코드 내에서 환경별 변수 로딩 로직을 강제화합니다.** |

### 🟢 3단계: 실패 로그 기반 분석 및 임시 해결 방안 (Debugging & Workaround)

실제 에러 로그를 분석하여 원인을 추적하고, 당장 작업을 진행할 수 있도록 임시 방안을 적용합니다.

#### 💡 Case 1: Rate Limit 초과 (가장 흔한 실패)
*   **에러 코드:** `429 Too Many Requests`
*   **분석:** 단시간에 너무 많은 API 호출이 발생했습니다. 이는 코드가 잘못되었다기보다 **시스템 사용 제한**에 걸린 것입니다.
*   **Workaround:**
    1.  **호출 주기를 늘립니다.** (예: 5분마다 배치 작업 실행 $\rightarrow$ 15분마다 배치 작업 실행)
    2.  **배치 처리 로직을 비동기화(Async)하거나, 큐(Queue) 시스템**을 도입하여 요청을 분산 처리합니다.

#### 💡 Case 2: 브랜치 충돌 (Merge Conflict)
*   **에러 코드:** `CONFLICT` (Git)
*   **분석:** 로컬 브랜치와 원격 브랜치 간의 내용 충돌입니다. 자동화 문제라기보다 협업 문제입니다.
*   **Workaround:**
    1.  **절대 `git push -f` 사용 금지.**
    2.  `git pull --rebase`를 통해 충돌 지점을 **수동으로 검토 및 병합**한 후, 다시 푸시합니다.

#### 💡 Case 3: Webhook 실패 (통합 문제)
*   **에러 코드:** `400 Bad Request` 또는 `Connection Refused`
*   **분석:** 웹훅을 받는 서버가 다운되었거나, URL 형식이 잘못되었거나, 방화벽이 요청을 막고 있을 수 있습니다.
*   **Workaround:**
    1.  **Webhook Payload를 로깅하는 테스트 서버**를 임시로 띄워, 실제로 데이터가 도착하는지 확인합니다.
    2.  네트워크 관리자에게 **해당 URL의 Inbound Port 개방**을 요청합니다.

***

📊 평가: 완료 — CEO가 지시한 모든 기술적 점검 항목(토큰/권한, 환경 변수, 에러 로그 분석)을 포함한 실행
