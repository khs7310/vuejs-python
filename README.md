#  Tech Project Documentation

## 1. Python Backend

### 1-1. Python Backend 구조
```
python-backend/
├── app.py              # Flask 애플리케이션 메인 파일
├── requirements.txt    # Python 패키지 의존성 파일
└── Dockerfile         # Backend Docker 이미지 빌드 설정
```


#### 주요 기능
- Flask 기반 REST API 서버
- CORS 지원 (모든 출처 허용)
- 클라이언트 IP 정보 로깅
- 두 개의 엔드포인트 제공:
  - `/api/hello`: Hello 메시지 반환
  - `/api/test`: 연결 테스트 메시지 반환

### 1-2. Dockerfile 설명
```dockerfile
# Build stage
FROM python:3.11-slim as builder
WORKDIR /app
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    rm -rf /var/lib/apt/lists/*
COPY requirements.txt .
RUN python -m venv /opt/venv
RUN pip install --no-cache-dir -r requirements.txt

# Final stage
FROM python:3.11-slim
RUN groupadd -r appuser && useradd -r -g appuser appuser
WORKDIR /app
COPY --from=builder /opt/venv /opt/venv
COPY app.py .
RUN chown -R appuser:appuser /app
USER appuser
ENV PATH="/opt/venv/bin:$PATH"
CMD ["python", "app.py"]
```

#### Dockerfile 주요 특징
- Multi-stage 빌드 사용으로 최종 이미지 크기 최소화
- Python 3.11 slim 이미지 기반
- 보안을 위한 비루트 사용자 설정
- 가상환경 사용으로 의존성 격리

## 2. Vuejs Frontend

### 2-1. Vuejs Frontend 구조
```
vuejs-front/
├── src/
│   ├── App.vue         # 메인 Vue 컴포넌트
│   ├── main.js         # Vue 애플리케이션 진입점
│   └── components/     # Vue 컴포넌트 디렉토리
├── public/
│   └── index.html      # HTML 템플릿
├── package.json        # Node.js 의존성 및 스크립트
└── Dockerfile         # Frontend Docker 이미지 빌드 설정
```

#### 주요 기능
- Vue.js 기반 SPA (Single Page Application)
- Axios를 사용한 API 통신
- 반응형 UI 디자인
- 클라이언트 정보 표시 기능

### 2-2. Dockerfile 설명
```dockerfile
# Build stage
FROM node:20-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm config set registry https://registry.npmjs.org/ && \
    npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
RUN mkdir -p /var/cache/nginx && \
    mkdir -p /var/run && \
    mkdir -p /var/log/nginx && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    chown -R nginx:nginx /etc/nginx/conf.d && \
    touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid
USER nginx
EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
```

#### Dockerfile 주요 특징
- Multi-stage 빌드 사용
- Node.js Alpine 이미지로 빌드
- Nginx Alpine 이미지로 서빙
- 보안을 위한 비루트 사용자 설정
- 정적 파일 서빙 최적화

## 3. 실행 방법

### 3-1. 사전 요구사항
- Docker
- Docker Compose

### 3-2. 실행 명령어
```bash
# 프로젝트 루트 디렉토리에서
docker-compose up -d --build
```

### 3-3. 접속 정보
- Frontend: http://localhost:8080
- Backend API: http://localhost:5000

### 3-4. 로그 확인
```bash
# 전체 로그 확인
docker-compose logs -f

# Backend 로그만 확인
docker-compose logs -f backend

# Frontend 로그만 확인
docker-compose logs -f frontend
```

### 3-5. 서비스 중지
```bash
docker-compose down
``` 
