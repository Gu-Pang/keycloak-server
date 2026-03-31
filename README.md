# Keycloak
이 프로젝트는 Keycloak 설정 저장소입니다. Docker Compose를 통해 즉시 가동 가능한 인증 환경을 제공합니다.
## Keycloak 설정 구조 (2026/03/31 기준)
### 1. Realm
- Realm : Gu-Pang
### 2. Clients
- api-gateway : API 게이트웨이 인증용
- all-services : 전체 서비스 공통 인증용
---
## 참고사항
도커로 실행 시 keycloack 설정들을 자동으로 import합니다.
```
docker-compose up -d
```
keycloak 설정 export 명령어 : 
```
docker exec -it keycloak /opt/keycloak/bin/kc.sh export --dir /opt/keycloak/data/import --realm Gu-Pang --users same_file
```
