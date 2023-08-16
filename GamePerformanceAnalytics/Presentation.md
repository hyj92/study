---
marp: true
html: true
paginate: true
_class: [invert, lead, head]
style: |
    h2 {
        font-weight: 500;
        text-align: center;
    }
    h4 {
        text-align: center;
        font-weight: 400;
        font-size: 20px;
    }
---
# **GPA v2 중간리뷰**
퍼블리싱 플랫폼개발팀 
한유진

---
# Index
- 배경
- 목표
- 유지 & 개선 내용
- 일정
- 현재 진행 상태
- 느낌점

---
# 배경
---
# 배경
- GPA(Game Performance Analytics)
  - 게임 성능 데이터를 수집/집계/리포팅하는 시스템
  - NC소프트에 각 개별 게임마다 성능 측정을 따로 하고 있었음
  - 2020년부터 개발되어 2021년 론칭, 약 1년간 운영
- 2021년 GPA 개발 인력 전체 이동
  - 신규 개발자 유입, 히스토리 전무
  - 방향성 상실 및 사용자 감소

---

# 배경 - 기능 개편
---
# 배경 - 기능 개편
1. **기존 기능 개선**
- 사용자 인터뷰를 통해 편의성/가시성 등의 문제 개선
- 개발 도중 홀딩된 기능 개발 재개
2. **신규 기능 추가**
- 기존 수동 유저 관리 -> 유저 관리 기능, 권한 기능 추가
- 게임 별이 아닌 전체 데이터 지표 추가
- 세션 별로 더 자세한 성능 footprint 기능 추가

---
# 배경 - 시스템 개편

---
# 배경 - 시스템 개편
1. **성능 개선**
- 처리량 개선
  - 기존은 게임별로 약 100~1000개의 랜덤한 디바이스를 선별하여 데이터 수를 샘플링하고 있음
  - 더 의미 있는 지표를 위해 더 많은 디바이스의 필요성
- 데이터 로드 성능 개선
  - 캐싱이 되지 않으면 처음 호출 시 1분 이상 로드 되는 성능 이슈

---
# 배경 - 시스템 개편
2. **운영 리소스 감축**
  - Dev/Live 환경 통일
    - 기존은 Dev는 NCCloud, Live는 GCP 사용
    - 2벌의 다른 시스템 운영
    - 다른 환경으로 인해 Dev/Live 개별적인 이슈 파악 필요
  - Kafka, Redis, Elasticsearch 모두 수동 운영 기반
    - GCE로 구성되어 직접 OS 별 dependency와 binary 를 설치
    - 개별 노드가 auto-scaling 불가
    - 이슈가 있는 경우 직접 쉘에 접근하여 확인 및 구성 작업 필요
  - Managed service 및 컨테이너 기반 서비스 적극 사용 -> 운영 리소스 축소

---
# 배경 - 시스템 개편
3. **가지치기와 정리**
  - 불필요 컴포넌트/코드 정리
    - legacy compute resource
    - 미사용, 불필요 코드 및 repository
  - 문서화
    - 기존 시스템 아키텍처, 서비스 환경, 기술 채택 등의 히스토리 문서 부재
    - 신규 개발자의 큰 랜딩 비용


---
<style scoped>
    h3, p {
      text-align: center
    }
</style>
# 목표
## 시스템 전체 신규 개발 하여 문제점 모두 해결

### 기능 개편안
```1. 기존 기능 개선```
```2. 더 많은 인사이트를 줄 수 있는 신규 기능 개발```

### 시스템 개편
**```1. 성능 개선```
```2. 운영 리소스 감축```
```3. 시스템 정리```**

---
<style scoped>
    h3, p {
      text-align: center
    }
</style>
# 목표

### 시스템 개편
**```1. 성능 개선```
```2. 운영 리소스 감축```
```3. 시스템 정리```**


---
# 기존

![](./gpa_arch_asis.png)

---

# 기존

![](./gpa_arch_asis_layer.jpeg)

---

# 교훈과 벤치마킹

---
# 교훈과 벤치마킹

![bg right:50%](./gpa_arch_asis_1.png)

1. Elastic Search 사용
    - Elastic Search 의 풍부한 기능을 사용하여 다양한 쿼리
    - 여러 예외 케이스에 쉽게 적용 가능

---
# 교훈과 벤치마킹

![bg right:50%](./gpa_arch_asis_7.png)

2. KStream 와 같이 Streaming App 으로 데이터 집계/수집 컴포넌트 사용
3. 오래 걸리는 집계 처리는 모두 Redis 캐싱 사용
  - 성능 이슈 최소화 

---

# 교훈과 벤치마킹

![bg right:50% height:80%](./gpa_arch_asis_2.png)

4. 유저 인증 처리는 Spring Gateway 서버에 위임
    - 이로인해 권한 처리도 인증 서버에서 처리 가능
    - 인증 로직을 모듈화 하지 않아도 한 곳에서 처리
5. 역할별로 잘 분리된 컴포넌트
    - SDK/포털 서비스용 API 서버 분리

---
# 교훈과 벤치마킹

![bg right:50%](./gpa_arch_asis_3.png)

6. 데이터 전송 주기와 특징에 맞춘 로직 설계
    - 디바이스 정보 데이터와 성능 데이터 분리로 인해 데이터 머지 이슈
7. 다양한 모니터링 도구
    - 시스템 모니터링을 위한 다양한 도구(Kafdrop, Celrebro, Kibana...)를 사용
---
# 개선

---
# 개선

![](./gpa_arch_tobe.png)

---
# 개선

![](./gpa_arch_tobe_layer.jpeg)

---
# 개선 - 성능

---
# 개선 - 성능
![bg right width:600px](./gpa_arch_asis_4.png)
- AS-IS
  - 하루에 약 800만건의 record 수집
  - ```집계, 조회 방식```: 실시간 elastic search 에 쿼리
  - 샘플링 디바이스 개수 증가 시 Elastic Search scale-up 인프라 비용 기존 대비 약 2*n배 이상 예상 (SSD 및 클러스터링)
 
---
# 개선 - 성능
![bg right width:600px](./gpa_arch_tobe_1.png)
- TO-BE
  - ```집계 방식```: Bigquery로 기집계하는 방식
  - ```조회 방식```: RDB(MySQL)로 조회
  - 샘플링 디바이스 개수 증가 시 BigQuery 쿼리 비용만 증가 (현재 live 데이터 기준 하루 35,000원)
---
# 개선 - 운영 리소스 감축
---
# 개선 - 운영 리소스 감축
- Dev도 Live와 동일하게 GCP 로 이전
- 수동 운영 기반의 환경 모두 컨테이너화
  - GCE Elastic Search -> GKE Container 기반 Elastic Search (Helm)
  - 개별 GKE Scheduler -> GKE Airflow 통합
- Managed 서비스 대체
  - GCE Kafka -> Pubsub
  - GKE Redis -> MemoryStore
  - GKE Straming App -> Dataflow

---
# 개선 - 운영 리소스 감축
- Pubsub
  - Scaling 필요 없음, 모니터링 제공, 운영 필요 없음
- GKE Container 기반 Elastic Search (Helm)
  - 커맨드 한 줄로 설치, Auto-Scaling (HPA) 가능
- Dataflow
  - 서버리스 (클러스터 운영 필요 없음), Auto-Scaling, 셔플 연산 자동 최적화, 코드 이식성(Apache Beam)
- Airflow
  - 간단한 Python 코드로 매우 쉽게 스케줄링, 모니터링, DataSource 와 연동 가능

---
# 개선 - 시스템 정리
---
# 개선 - 시스템 정리
![bg right:50%](./gpa_arch_asis_5.png)
- 불필요 컴포넌트 약 7가지 정리
- 미사용 코드/데이터 구조 정리
- 문서화 부재, 테스트 코드 부재 -> 필수 적용
- 모든 웹 서비스 Nginx GKE 에서 Proxy, GraphQL 등 숨겨진 인프라 -> GCP Load Balancer 로 대체

---
# 일정
- 6월 2주차 ~ 11월(총 5개월 2주)
  - 6월(2주): 시스템 분석 및 기반 작업
  - 7~10월(4개월): 시스템 개발 
  - 11월(1개월): 개발 및 배포 환경 구성 및 live 배포
- [자세한 일정](https://wiki.ncsoft.com/pages/viewpage.action?pageId=753103117)
---
# 현재 진행중인 상태
- GCP 에 Dev 존 구축하면서 Live 용 데이터로 작업 및 검증 중
- 대략 서버 컴포넌트 구성은 70% 정도 진행도
- API 기능 개발 중
- 예상 일정보다 3주 빠름

---
# 느낀점
- 전체 개선이 쉽게 이루어지는 이유는 실시간으로 실서비스 데이터 파이프라인을 연결하여 사용하였기 때문
- 시스템을 전체적으로 개선하려면 시스템에 대한 이해도를 높이는 시간을 아낌없이 써야한다
- 문제, 배경을 정확하게 인지하고 이를 해결하기 위한 목표에 오해가 없어야 한다
- 시스템 유지를 위해 분기나 반기에 한 번씩은 구조를 자주 리뷰하면 좋겠다
- 어떤 시스템이든 배울 점은 있다, 불만만 가지지 말고 교훈을 찾자
  
---
#
<style scoped>
    h2 {
        font-size:100px !important;
        font-weight: 600 !important;
    }
</style>
</br></br>

## Q&A 

---
# Appendix 
- [Overview](https://wiki.ncsoft.com/pages/viewpage.action?pageId=748582940)
- [시스템 구조 개편](https://wiki.ncsoft.com/pages/viewpage.action?pageId=748583980)
- [데이터 집계 방식](https://wiki.ncsoft.com/pages/viewpage.action?pageId=770518325)

