# Keycloak
이 프로젝트는 Keycloak 설정 저장소입니다. Docker Compose를 통해 즉시 가동 가능한 인증 환경을 제공합니다.
# 로그인 테스트 방법
keycloak, user 서버 받기
1. user db 내에서 마스터 계정 생성 (id:master,pw:master)
```
INSERT INTO p_user (
    user_id, 
    username, 
    password, 
    email, 
    first_name, 
    last_name, 
    role, 
    status, 
    slack_id, 
    keycloak_id, 
    created_at, 
    updated_at, 
    deleted_at
) VALUES (
    'a7f0395c-6498-4add-a94f-0148e220df6f',
    'master', 
    '$2a$10$s4MV01AfqohFiFCMaZXZjeZx9XV90E2zpYR6zrLwdjh5/uyqMENBW', 
    'master@naver.com', 
    'first', 
    'last', 
    'MASTER', 
    'APPROVED', 
    NULL, 
    'a7f0395c-6498-4add-a94f-0148e220df6f',
    CURRENT_TIMESTAMP, 
    NULL, 
    NULL
) ON CONFLICT (username) 
DO UPDATE SET 
    user_id = EXCLUDED.user_id,
    deleted_at = NULL;
```
2. master login해서 accesstoken 복사해두기

post http://localhost:8090/api/v1/auth/login

body에 raw로
```
{
    "username": "master", 
    "password": "master"
}
```
<img width="1090" height="764" alt="image" src="https://github.com/user-attachments/assets/b1cbc707-83b7-4978-9423-2fa1cee71b0c" />

4. 업체 회원 등은 회원가입 api를 통해 회원가입을 우선 진행해야 합니다.

post http://localhost:8090/api/v1/users

body에 raw로 대충 이런 형식으로
```
{
    "username": "company123",
    "password": "Password123!",
    "firstName": "길동",
    "lastName": "홍",
    "email": "company@test.com",
    "slackId": "U12345678"
}
```
<img width="1085" height="670" alt="image" src="https://github.com/user-attachments/assets/20161e59-6025-48a3-8e6e-547612f00587" />

5. 업체 회원 가입 승인

patch http://localhost:8090/api/v1/admin/users/{4번에서 가입한 user의 user_id}

master로 로그인해서 받은 응답의 accesstoken을 헤더에 넣고 X-User-UserId에는 master계정의 user_id값을 넣음
```
key:Authorization,value:Bearer 2번에서복사한토큰값
key:X-User-UserId,value:a7f0395c-6498-4add-a94f-0148e220df6f
```
body에 raw로
```
{
    "status": "APPROVED"
}
```
<img width="1087" height="654" alt="image" src="https://github.com/user-attachments/assets/fc6cdbb3-98b8-4cbd-aee9-4fc812c88db4" />

6. 로그인

post http://localhost:8090/api/v1/auth/login

body에 raw로 아까 4번에서 가입한 아이디와 비번으로
```
{
    "username": "company123", 
    "password": "Password123!"
}
```
<img width="1091" height="755" alt="image" src="https://github.com/user-attachments/assets/128649b1-63da-4891-a6af-eb7e1d84be8f" />

# 각 서버별 security 설정 방법
게이트웨이에서 전파한 헤더를 어떻게 쓸지는 자유인것 같긴 하지만 user서버는 이렇게 구성하였고 모든 서버 동일하게 해도 무방할 것 같습니다
헤더 값 받는 필터
https://github.com/Gu-Pang/user-server/blob/dev/src/main/java/org/gupang/user/Presentation/Filter/HeaderAuthenticationFilter.java
security config
https://github.com/Gu-Pang/user-server/blob/dev/src/main/java/org/gupang/user/Infrastructure/Config/SecurityConfig.java

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
