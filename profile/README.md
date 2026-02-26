# CatchMe

> 실종 반려동물 대응을 위해 **신고 -> 탐색 -> 추천 -> 소통 -> 케어**를 하나로 연결한 통합 서비스

## 한 줄 요약
CatchMe는 실종/목격 제보를 지도, AI, 실시간 채팅으로 통합해 탐색 시간을 줄이고 재회 가능성을 높입니다.

## 문제 정의
실종 대응 정보가 커뮤니티, 지도, 메신저로 흩어져 초기 대응이 늦어집니다.  
목격 제보 신뢰도 판단과 보호자-목격자 소통에도 시간이 소모됩니다.  
CatchMe는 이 흐름을 하나의 서비스로 묶어 골든타임 손실을 줄입니다.

## 시스템 아키텍처 핵심 구조
- **Client**: 신고, 탐색, 채팅, 예약 UX
- **Application API**: 인증/게시글/지도/채팅/예약 도메인 처리
- **AI Inference**: 이미지 기반 품종/색상 추론
- **Data Layer**: 사용자/게시글/채팅/예측 결과 저장

설계 철학: 사용자 경험은 단일하게 유지하고, AI·실시간 처리는 독립적으로 확장합니다.

## 시스템 다이어그램
```mermaid
flowchart LR
    U["User (Mobile/Web)"] --> FE["Client App\n신고/탐색/채팅/예약"]

    FE --> API["Application API\n인증 · 게시글 · 지도 · 예약"]
    FE <--> WS["Realtime Gateway\nSTOMP/WebSocket"]

    API --> DB["Service Database\n유저 · 게시글 · 채팅 · 병원 · 예약"]
    API --> FS["File Storage\n업로드 이미지/미디어"]

    API --> AI["AI Inference Service\nCLIP 기반 이미지 추론"]
    AI --> DB

    API --> MAP["Map/Geo Provider\n주소-좌표 변환"]

    WS --> DB
```

## 핵심 기능
1. 실종/목격 통합 게시 관리
2. AI 추론 기반 연관 제보 추천
3. 지도 중심 탐색(제보/보호소/병원)
4. 실시간 채팅으로 즉시 소통
5. 수의사 조회 및 예약 연계
6. JWT 기반 인증/접근 제어

## 기술 스택
- **Client**: React, Vite, React Router, SockJS, STOMP, Kakao Map SDK
- **Backend**: Java 17, Spring Boot, Spring Data JPA, Spring Security, WebSocket/STOMP, JWT
- **AI**: FastAPI, PyTorch, Hugging Face Transformers(CLIP)
- **Infra/Data**: MariaDB, Redis, File Storage, Geo API

## Quick Start
### 필수 요구사항
- Git 2.40+
- Java 17
- Python 3.10+
- Node.js 18+ (웹 프론트 사용 시)
- Android Studio + Android SDK (모바일 프론트 사용 시)
- MySQL 8.0+ 또는 MariaDB 10.6+
- Redis 7+

### 프로젝트 클론
```bash
git clone <YOUR_GITHUB_REPO_URL>
cd <YOUR_REPO_DIR>
```

### 환경 변수 (`backend-deploy/.env`)
```env
DB_URL=jdbc:mysql://localhost:3306/luno?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Seoul
DB_USERNAME=root
DB_PASSWORD=1234

GOOGLE_CLIENT_ID=your_google_client_id
jwt.secret=your_base64_jwt_secret

CLOVA_STT_INVOKE_URL=your_clova_stt_url
CLOVA_SECRET_KEY=your_clova_secret
AI_SERVER_BASE_URL=http://localhost:8000
```

추가 파일: `backend-deploy/src/main/resources/firebase-key.json`

### 외부 의존성 실행
```bash
docker run -d --name luno-mysql -e MYSQL_ROOT_PASSWORD=1234 -e MYSQL_DATABASE=luno -p 3306:3306 mysql:8.0
docker run -d --name luno-redis -p 6379:6379 redis:7
```

### 실행
```bash
# 1) AI
cd AI-main
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install sentence-transformers
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 2) Backend
cd ../backend-deploy
./gradlew clean build
./gradlew bootRun

# 3) Frontend (Android)
cd ../frontend-develop
./gradlew assembleDebug

# 4) Frontend (Web, 포함된 경우)
cd ../Frontend-main
npm install
npm run dev
```

실행 순서: MySQL/Redis -> DB 스키마 -> AI -> Backend -> Frontend

## 향후 개선
- 이미지+텍스트+거리/시간 결합 랭킹 고도화
- 악성 제보 탐지와 운영 검수 워크플로우 강화
- API/AI/채팅 전 구간 메트릭·트레이싱 체계 강화
