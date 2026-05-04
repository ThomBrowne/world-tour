# 💻 Developer — 핵심 기술인 'AI Enrichment Layer'의 데이터 파이프라인 프로토타입 개발 범위를 확정합니다. 목표는 Purpose-POI 매핑의 기본 기능(최소한의 데이터 구조와 API 호출 흐름)을 구현하는 것입니다. 필요한 데이터 스키마와 API 엔드포인트 초안을 작성하고, PostGIS 기반의 데이터 모델링을 시작합니다.

💻 Developer: 작업 시작합니다.

[지시사항 요약]
1.  **목표:** Purpose-POI 매핑 기능을 가진 'AI Enrichment Layer' 데이터 파이프라인 프로토타입 개발.
2.  **결과물:** 데이터 스키마, API 엔드포인트 초안, PostGIS 기반 데이터 모델링 시작.

---

### 🛠️ 1. 데이터 모델링 (PostGIS Schema Definition)

핵심 목표인 'Purpose-POI 매핑'을 지원하기 위해 최소 3개의 엔티티(Purpose, POI, Mapping)를 정의하고, 공간 데이터를 위해 PostGIS의 `GEOMETRY` 타입을 사용합니다.

#### A. `purposes` 테이블 (사용자 목적 정의)
| Column Name | Data Type | Description | Constraints |
| :--- | :--- | :--- | :--- |
| `purpose_id` | `SERIAL` | Primary Key. (예: 1 - 휴식, 2 - 모험) | `PK` |
| `purpose_name` | `VARCHAR(100)` | 목적 이름 (예: "Relaxation", "Adventure") | `UNIQUE` |
| `description` | `TEXT` | 목적 상세 설명 | |
| `keywords` | `TEXT[]` | 목적과 관련된 핵심 키워드 배열 (AI 학습용) | |

#### B. `pois` 테이블 (관심 지점 정의)
| Column Name | Data Type | Description | Constraints |
| :--- | :--- | :--- | :--- |
| `poi_id` | `SERIAL` | Primary Key. | `PK` |
| `name` | `VARCHAR(255)` | POI 이름 | `NOT NULL` |
| `category` | `VARCHAR(100)` | POI의 일반 카테고리 (예: "카페", "박물관", "산") | |
| `address` | `VARCHAR(500)` | 주소 | |
| `coordinates` | `GEOMETRY(Point, 4326)` | **핵심: 위도/경도 기반 공간 데이터** | `SPATIAL INDEX` |
| `description` | `TEXT` | POI 상세 설명 | |

#### C. `purpose_poi_map` 테이블 (매핑 관계 정의)
이 테이블은 목적과 POI 간의 관계를 정의하며, AI Enrichment Layer의 핵심 로직이 작동하는 곳입니다.
| Column Name | Data Type | Description | Constraints |
| :--- | :--- | :--- | :--- |
| `map_id` | `SERIAL` | Primary Key. | `PK` |
| `purpose_id` | `INT` | 연결되는 Purpose ID (FK: `purposes`) | `FK` |
| `poi_id` | `INT` | 연결되는 POI ID (FK: `pois`) | `FK` |
| `relevance_score` | `FLOAT` | **Purpose와 POI의 관련성 점수 (0.0 ~ 1.0)** | `NOT NULL` |
| `reasoning_key` | `VARCHAR(255)` | 매핑의 근거가 된 핵심 키워드 (AI 출력용) | |

---

### 🌐 2. API 엔드포인트 초안 (Prototype Flow)

**Endpoint:** `/api/v1/enrichment/purpose-to-pois`
**Method:** `GET`
**Purpose:** 사용자가 제공한 목적 키워드 또는 `purpose_id`를 기반으로 가장 적합한 POI 목록을 반환합니다.

#### A. 요청 (Request Parameters)
| Parameter | Type | Required | Description | 예시 |
| :--- | :--- | :--- | :--- | :--- |
| `purpose_id` | `INT` | Yes | 조회할 목적의 ID. | `1` (휴식) |
| `lat` | `FLOAT` | No | 사용자 현재 위도 (근거리 검색 시 사용). | `37.5665` |
| `lon` | `FLOAT` | No | 사용자 현재 경도 (근거리 검색 시 사용). | `126.9780` |
| `limit` | `INT` | No | 반환할 POI 최대 개수. | `10` |

#### B. 응답 (Response Body - JSON)
```json
{
  "status": "success",
  "query_purpose": {
    "id": 1,
    "name": "휴식",
    "keywords": ["카페", "공원", "조용함"]
  },
  "results": [
    {
      "poi_id": 101,
      "name": "북촌 한옥마을 카페",
      "category": "카페",
      "distance_km": 1.2,
      "relevance_score": 0.95,
      "coordinates": "37.5700, 126.9800",
      "description": "조용한 분위기의 전통 카페입니다."
    },
    {
      "poi_id": 105,
      "name": "남산 공원 산책로",
      "category": "공원",
      "distance_km": 2.5,
      "relevance_score": 0.88,
      "coordinates": "37.5500, 126.9900",
      "description": "도심 속에서 자연을 즐기기 좋습니다."
    }
    // ... (limit 만큼 반복)
  ]
}
```

---

### 💻 3. 초기 구현 코드 (SQL/Pseudo-Code Draft)

PostGIS와 매핑 관계를 활용하여 근거리 및 관련성 높은 POI를 찾는 쿼리입니다.

```sql
-- 1. 데이터 초기화 (Sample Data Insert)
-- (이 부분은 백엔드 초기 마이그레이션 스크립트로 실행되어야 합니다.)
INSERT INTO purposes (purpose_id, purpose_name, description, keywords) VALUES
(1, '휴식', '도심 속에서 편안하게 재충전하는 목적', ARRAY['
