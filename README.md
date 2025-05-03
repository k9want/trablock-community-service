[![CI (feat. Java CI with Gradle)](https://github.com/TravelLaboratory/travel-laboratory-was/actions/workflows/ci.yml/badge.svg)](https://github.com/TravelLaboratory/travel-laboratory-was/actions/workflows/ci.yml)
[![Deploy to Production](https://github.com/TravelLaboratory/travel-laboratory-was/actions/workflows/main-deploy.yml/badge.svg)](https://github.com/TravelLaboratory/travel-laboratory-was/actions/workflows/main-deploy.yml)
# 여행 계획과 일정 공유 커뮤니티 서비스, Trablock

### 📽️ 시연영상
[https://www.youtube.com/watch?v=7-nYAZsZRtk](https://www.youtube.com/watch?v=7-nYAZsZRtk)

<br>


### 🔖 프로젝트 개요

- **주제** : 여행 계획을 작성하고 리뷰를 추가하며 이를 다른 사용자들과 공유할 수 있는 여행 계획 및 일정 공유 커뮤니티 서비스
- **개발 프로세스** :  (외주) 프로젝트 설계 및 기능 개발 → (개인) 테스트 기반 로직 정합성 점검 및 성능 개선 반복
- **참고사항** : 타 부트캠프 수강생들을 위한 API 서버를 외주 개발 및 정산 후 개인 프로젝트로 전환

<br>


### 📚 기술 스택

<img width="600" alt="기술스택" src="https://github.com/user-attachments/assets/befdc4b0-aea2-4cfe-8a5c-356247bf5bdc" />

<br>
<br>


### 🌏 서버 아키텍쳐

<img width="825" alt="모니터링서버 구축" src="https://github.com/user-attachments/assets/a1f5bbb7-3604-417e-997c-0ac27eecabf0" />
<br>
<br>


### 🔗 담당 핵심 기능 (대표 기능 중심)
#### 1️⃣ 홈 화면 조회

- 홈 화면에서는 **트렌딩 여행 계획**과 **인기 여행 계획**을 목록으로 제공합니다.
- 각 여행 계획에는 **작성자, 여행지 태그, 장소, 날짜, 제목, 대표 이미지** 등의 정보를 함께 제공하여, 다양한 여행 계획들을 한눈에 비교하고 확인할 수 있습니다.

#### 2️⃣ 일정 작성
- 여행 전체 일정의 동선을 지도에서 직관적으로 확인할 수 있으며, 일정 블록별 위치가 마커로 표시되어 여행 흐름을 쉽게 파악할 수 있습니다.
- 각 블록에는 **상세 정보와 개별 비용**이 포함되며, **전체 일정의 총 경비도 함께 제공**되어 예산 계획에 참고할 수 있습니다.

#### 3️⃣ 일정 상세 조회
- 여행 일정은 **일반(숙소, 식당, 관광지, 액티비티), 교통, 기타 3가지 유형**으로 구성된 블록 단위로 작성할 수 있습니다.
- 각 블록별로 비용을 입력할 수 있으며, 사용자는 드래그 앤 드롭 방식으로 블록의 순서를 변경할 수 있습니다.
- 일정에는 대표 커버 이미지 등록이 가능하며, 위치 정보를 포함한 일정 블록들은 지도 위에 시각화되어 여행 동선을 확인할 수 있습니다.

<br>

### 🚀 개선 사항
### 1️⃣ 홈 배너 조회 성능 개선 – 캐싱 전략 도입

**문제 상황**

- 실시간성이 요구되지 않는 홈 배너 데이터를 매 요청마다 DB에서 조회하여 불필요한 I/O 발생
- 자주 호출되는 API임에도 기대한 성능을 내지 못해 서비스 품질 저하 우려

**해결 방법**

- **로컬 캐시와 글로벌 캐시를 비교 분석**
    - 로컬 캐시는 인스턴스 재시작 시 캐시 리셋, 운영 상태 확인 어려움
    - **Redis 기반 글로벌 캐시**는 모니터링 및 추적 용이, 추후 다중 인스턴스 확장에도 대응 가능
- Redis 기반 **글로벌 캐싱 구조 채택** + TTL 설정으로 적절한 캐시 만료 정책 설계
- 실시간성이 요구되지 않는 배너 데이터를 **Redis 캐시에 저장** 캐시 미스 시에만 DB 조회 → **DB 부하 제거 + 캐시 적중률 향상**

**결과**


### 2️⃣ 일정 상세 조회 최적화 – 테이블 정규화 및 인덱스 설계

**문제 상황**

- **일정 데이터를 단일 테이블로 관리**하며 Null 컬럼이 증가하고, 타입별 데이터 구조가 섞이며 **확장성 저하**
- 일정 상세 API 호출 시 **100만 건 이상의 데이터 대상 풀 스캔 발생**
- 필터 조건 및 JOIN 연산에서 성능 저하 발생 → **평균 응답 시간 3.9초 측정**

**해결 방법**

- 일정 데이터를 **일반/교통/기타 일정으로 구분하여 Dtype 기반 정규화**
- 실행 계획 분석 기반으로 쿼리 튜닝 및 인덱스 설계하여 **풀 스캔 제거**

**결과**


### 3️⃣ 배포 프로세스 개선 – GitHub Actions 기반 자동화 구축

**문제 상황**

- 초기 수동 배포 환경으로 인해 배포 누락·시간 소요 발생
- Docker 이미지 용량 증가로 빌드/배포 지연

**해결 방법**

- CI/CD 도구로 **GitHub Actions + AWS CodeDeploy** 선택
- **멀티 스테이지 빌드**, 슬림 베이스 이미지 적용 → 이미지 최적화
- Gradle 캐시를 활용해 빌드 속도 개선

**결과**

- 일관된 자동 배포 파이프라인 구축
- 빌드 시간 단축 및 운영 실수 방지

### 🚀 트러블 슈팅
### 1️⃣ 요청/응답 로깅 체계 구축 – 디버깅 속도 향상 및 에러 추적 자동화

**문제 상황**

- 이미지 업로드 실패 또는 외부 API 요청 실패 시 원인 추적이 어려움
- 운영 중 장애 발생 시 재현과 대응에 시간 소요

**해결 방법**

- **AOP 기반으로 S3 업로드 성공/실패 시점 로깅**
- Filter를 통해 **요청/응답 바디 및 헤더 자동 기록**
- 요청 ID 기반으로 로그를 트래킹 가능하도록 구조 설계

**결과**

- **에러 발생 시점의 컨텍스트 확보 가능**
- 디버깅 시간 단축, 운영 안정성 향상

  

### ⛓ Link
* [API 명세서 링크](https://docs.google.com/spreadsheets/d/1dbc9NR9iWJA5QqbcnxBggA11os2IF-6P/edit?gid=403544037#gid=403544037)
* [ERD 링크](https://www.erdcloud.com/d/WAacjyYwg2zGhtC98)
