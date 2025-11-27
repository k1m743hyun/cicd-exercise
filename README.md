# CI/CD Exercise

GitLab과 GitLab Runner를 활용한 CI/CD 파이프라인 실습 프로젝트입니다.

## 프로젝트 구조

```
cicd-exercise/
├── docker-compose.yml      # GitLab + GitLab Runner 설정
├── config.toml             # GitLab Runner 설정 파일
└── test/                   # 샘플 Spring Boot 애플리케이션
    ├── .gitlab-ci.yml      # CI/CD 파이프라인 정의
    ├── Dockerfile          # 애플리케이션 Docker 이미지 빌드
    ├── build.gradle        # Gradle 빌드 설정
    └── src/                # 소스 코드
```

## 기술 스택

- **CI/CD**: GitLab EE 15.0.1, GitLab Runner
- **애플리케이션**: Spring Boot 2.7.18, Java 8
- **빌드 도구**: Gradle 7.6
- **런타임**: Tomcat 8.5.24
- **컨테이너**: Docker, Podman

## 인프라 구성

### GitLab 서버
- 포트: 80 (HTTP), 443 (HTTPS), 2222 (SSH)
- 호스트명: `gitlab.local`
- 성능 최적화 설정 적용 (소규모 팀용)

### GitLab Runner
- Executor: Docker
- 기본 이미지: `gradle:7.6-jdk8`
- 태그: `docker`, `gradle`

## 시작하기

### 1. GitLab 실행

```bash
docker-compose up -d
```

### 2. 초기 root 비밀번호 확인

```bash
podman exec gitlab cat /etc/gitlab/initial_root_password
```

### 3. GitLab Runner 등록

```bash
podman exec gitlab-runner gitlab-runner register \
  --non-interactive \
  --url "http://gitlab" \
  --registration-token "<YOUR_TOKEN>" \
  --executor "docker" \
  --docker-image "gradle:7.6-jdk8" \
  --description "docker-runner" \
  --tag-list "docker,gradle" \
  --docker-privileged \
  --docker-network-mode "cicd-exercise_gitlab-network" \
  --docker-volumes "/cache" \
  --docker-volumes "/var/run/docker.sock:/var/run/docker.sock" \
  --clone-url "http://gitlab"
```

## CI/CD 파이프라인

### 스테이지

1. **build**: Gradle로 WAR 파일 빌드
2. **package**: Docker 이미지 생성
3. **deploy**: 환경별 컨테이너 배포

### 브랜치 전략

- `deploy/*` 브랜치로 푸시 시 package, deploy 스테이지 실행
- 예: `deploy/dev`, `deploy/prod`

### 환경별 배포

브랜치 이름에서 환경을 추출하여 해당 포트로 배포:
- `deploy/dev` → `DEV_PORT` 환경변수 사용
- `deploy/prod` → `PROD_PORT` 환경변수 사용

## 샘플 애플리케이션

### 엔드포인트

| 경로 | 설명 |
|------|------|
| `GET /` | 메시지, 프로필, 타임스탬프 반환 |
| `GET /health` | 헬스체크 |

### 로컬 빌드

```bash
cd test
./gradlew clean build
```

### 응답 예시

```json
{
  "message": "Hello from CI/CD!",
  "profile": "dev",
  "timestamp": "2025-11-27T10:30:00"
}
```
