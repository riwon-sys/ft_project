프리랜서 개발자와 기업간의 중개를 맡는 중개 플랫폼
DevConnect — Admin Web & API




DevConnect는 관리자(Admin)가 기업(Company), 개발자(Developer), 평가(Crating/Drating), 프로젝트(Project/ProjectJoin) 를 한 곳에서 통합 관리하는 관리자 전용 플랫폼입니다.
React Admin Web + Spring Boot API + MySQL + Redis + JWT 구조로 동작합니다.

✨ 핵심 특징

관리자 전용: Admin 로그인 후 접근 가능. 기업/개발자용 앱은 별도(Flutter)이며 Admin Web에는 접근 불가.

도메인 통합 관리: Company / Developer / Crating / Drating / Project / ProjectJoin 전체 조회·상세·수정·삭제.

상태 기반 정책: Company state 변경(0~2) 지원, 삭제/수정 조건 분기.

FormData & JSON 혼용

Company 수정: @ModelAttribute CompanyDto + file 필드(이미지 업로드)

Company 상태변경: @RequestBody CompanyDto { cno, state } (0~2만 허용)

대시보드: 통계 카드, 최근 승인 리스트, Recharts 차트, Redis 로그인 상태 표시.

프론트엔드 가이드: Joy UI 테마, axiosInstance 토큰 자동첨부, PrivateRoute/RoleRoute 보호 라우팅.

🏗 아키텍처
[React Admin Web]  →  [Spring Boot API]  →  [MySQL]
         │                   │
         │ JWT               └→ [Redis] (최근 로그인/캐시 등)
         ▼
      RBAC(Admin)

🧰 기술 스택

Backend: Java 17+, Spring Boot 3.4.x (Web, Data JPA, Validation), Lombok, JWT

DB/Cache: MySQL 8.0, Redis 7.x

Frontend(Admin): React 18 (Vite), Joy UI/MUI, Recharts, Axios

Auth: JWT(LocalStorage), Authorization: Bearer <token>

DevOps: Docker(선택), GitHub Actions(선택)

📂 디렉토리 구조(현재 기준)
devconnect-admin-web/
├─ src
│  ├─ api
│  │  ├─ axiosInstance.js
│  │  ├─ adminApi.js
│  │  ├─ companyApi.js
│  │  ├─ developerApi.js
│  │  ├─ cratingApi.js
│  │  ├─ dratingApi.js
│  │  └─ project[Join]Api.js (필요 시)
│  ├─ components
│  │  ├─ Sidebar.jsx
│  │  ├─ Header.jsx
│  │  └─ StatusBadge.jsx
│  ├─ layouts
│  │  └─ AdminLayout.jsx
│  ├─ pages/admin
│  │  ├─ AdminLogin.jsx
│  │  ├─ AdminSignup.jsx
│  │  ├─ AdminDashboard.jsx
│  │  ├─ AdminList.jsx
│  │  ├─ AdminUpdate.jsx
│  │  ├─ CompanyList.jsx
│  │  ├─ CompanyDetail.jsx
│  │  ├─ DeveloperList.jsx
│  │  ├─ DeveloperDetail.jsx
│  │  ├─ CratingList.jsx
│  │  ├─ CratingDetail.jsx
│  │  ├─ DratingList.jsx
│  │  ├─ DratingDetail.jsx
│  │  └─ Project[List|Detail].jsx / ProjectJoin[List|Detail].jsx
│  ├─ routes
│  │  ├─ PrivateRoute.jsx
│  │  └─ RoleRoute.jsx
│  └─ utils
│     └─ tokenUtil.js
├─ App.jsx
└─ index.js


규칙: admin 영역 제외 도메인은 List.jsx + Detail.jsx 2개 파일로 관리.

🚀 빠른 시작
0) 선행 설치

Node 20+, MySQL 8, Redis 7, (백엔드 Java 17+ & Gradle)

1) 환경 변수

프론트(Web) — ./.env

VITE_API_BASE_URL=http://localhost:8080/api
VITE_JWT_STORAGE_KEY=DEVCONNECT_JWT


백엔드 — application.yml 예시

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/devconnect?useSSL=false&serverTimezone=Asia/Seoul
    username: devconnect
    password: devconnect
  jpa:
    hibernate:
      ddl-auto: update
    open-in-view: false
jwt:
  secret: change-me-long-random
redis:
  host: localhost
  port: 6379

2) 프론트 실행
npm i
npm run dev
# http://localhost:5173

3) 백엔드 실행(예시)
./gradlew bootRun
# http://localhost:8080

4) 기본 플로우

AdminSignup → AdminLogin

토큰 저장(LocalStorage 키: VITE_JWT_STORAGE_KEY)

보호 라우팅(PrivateRoute/RoleRoute) 통과

대시보드·도메인 관리 화면 접근

🔐 인증 & 라우팅

JWT: 로그인 성공 시 토큰 발급 → axiosInstance가 자동으로 Authorization: Bearer <token> 헤더 추가

관리자 전용: React Admin은 Admin만 접속 가능. 기업/개발자용 앱은 별도

보호 라우팅:

PrivateRoute: 토큰 존재 검증

RoleRoute: Admin 권한 검증

src/api/axiosInstance.js

import axios from "axios";
const instance = axios.create({ baseURL: import.meta.env.VITE_API_BASE_URL });

instance.interceptors.request.use((config) => {
  const key = import.meta.env.VITE_JWT_STORAGE_KEY;
  const token = localStorage.getItem(key);
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

export default instance;

🗂 주요 API 개요(실행 백엔드에 맞게 조정)
Auth / Admin
POST   /api/admin/signup
POST   /api/admin/login            -> { token, admin }
GET    /api/admin/list
PUT    /api/admin/update
PUT    /api/admin/delete           // ※ 보안정책상 DELETE 대신 PUT 유지(리원님 규칙)
GET    /api/admin/dashboard/stats  // 대시보드 통계 (Redis 로그인 수 등)

Company
GET    /api/company/list           ?page=&size=&keyword=&state=
GET    /api/company/detail?cno=#
PUT    /api/company/update         // @ModelAttribute CompanyDto (+ file: 이미지)
PUT    /api/company/state          // @RequestBody { cno, state }  // state: 0~2만 허용
PUT    /api/company/delete         // 정책상 PUT 사용


중요 규칙

update는 FormData 기반(이미지 업로드 지원)

state는 0~2 외 값(예: 9) 사용 시 오류 가능

백엔드 수정 불가 → 프론트가 API 스펙에 맞춰야 함

Developer
GET    /api/developer/list
GET    /api/developer/detail?dno=#
PUT    /api/developer/update       // FormData 방식으로 통일 가능(운영 규칙에 따름)
PUT    /api/developer/delete

Crating (기업 평가)
GET    /api/crating
GET    /api/crating/view?crno=#
POST   /api/crating
PUT    /api/crating
DELETE /api/crating?crno=#

Drating (개발자 평가)
GET    /api/drating
GET    /api/drating/view?drno=#
POST   /api/drating
PUT    /api/drating
DELETE /api/drating?drno=#

Project / ProjectJoin (관리자 직권 기준)
GET/PUT/DELETE /api/project[...]
GET/PUT/DELETE /api/admin/project-join[...]

📊 대시보드(예시 구성)

통계 카드: 총 기업/개발자/프로젝트/평가 수

최근 승인 리스트: Company/Developer/Project 최신 승인 항목

차트(Recharts): 월별 참여 추이, 로그인 통계

Redis 연동: 최근 로그인 수/상태 키 조회

🗃 DB & Redis(개요)

MySQL: company, developer, project, project_join, crating, drating, admin, state_codes 등

인덱스 권장: company(state, created_at), developer(created_at), crating(crstate, create_at) 등

Redis 키 예시:

RECENT_LOGIN_ADMIN:{adminId}

대시보드 카운터/캐시(필요 시 TTL 적용)

🧪 스크립트

Frontend

npm run dev
npm run build
npm run preview


Backend

./gradlew test
./gradlew bootJar

🛡 품질/보안

비밀키/DB 비밀번호는 .env & GitHub Secrets로 관리 (커밋 금지)

비밀번호 해시(BCrypt), 입력값 Validation

CORS: 필요한 Origin만 허용

업로드 파일: MIME 검증(필요 시 백엔드 필터 추가)

삭제는 정책상 PUT 사용(리원님 운영 원칙)

🧭 트러블슈팅 FAQ

Company 상태 변경 400/500
→ state 값이 0~2인지 확인. 9 등은 금지.

이미지 업로드 실패
→ update는 FormData + file 필드로 전송했는지 확인.

401/403
→ 토큰 만료 또는 Role 미일치. 로컬 스토리지 키(VITE_JWT_STORAGE_KEY) 점검.

대시보드 수치 미표시
→ Redis 연결/키 존재 여부 확인.

🗺 로드맵(예시)

 회사/개발자 상세의 활동 로그 탭

 고급 검색/정렬/CSV 내보내기(Company/Developer/Project)

 알림센터(멘션/승인/상태변경)

 SSO(OAuth2/PASS 실명 인증)

 감사 로그 & 권한 세분화(Viewer/Mid/Super Admin)

🧾 부록: Gradle 의존성(백엔드 예시)
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.4.4'
    id 'io.spring.dependency-management' version '1.1.7'
}

java { toolchain { languageVersion = JavaLanguageVersion.of(17) } }

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'io.jsonwebtoken:jjwt-api:0.12.6'
    runtimeOnly  'io.jsonwebtoken:jjwt-impl:0.12.6'
    runtimeOnly  'io.jsonwebtoken:jjwt-jackson:0.12.6'
    runtimeOnly  'com.mysql:mysql-connector-j'
    implementation 'org.springframework.security:spring-security-crypto:6.4.4'
    compileOnly  'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

📜 라이선스

MIT (필요 시 변경)

👤 Maintainer

김리원 — Admin 전용 React + Spring Boot 풀스택
문의/이슈는 GitHub Issues로 부탁드립니다.
