# DBMS Architecture

<img width="575" alt="스크린샷 2024-03-14 오후 1 47 33" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/da194e07-792a-4d3a-860e-bbf7e10dcb73">


DBMS는 Client/Server Model 사용하는데 데이터베이스 시스템 인스턴스는 서버의 역할하고 응용 프로그램 인스턴스는 Client 역할을 한다.

Client 요청은 Query 형태로 transport subsystem을 지나서 데이터베이스 클러스터와 통신한다.

- Cluster connection→Client → Query → Execution→Storage(순환적 구조)
    - Transport: Cluster and Client communication
        - access control check , Query문 이해(핵심 영역)
    - Processor: Query Parser → Optimizer
        - Query Parser : Find Query
            - Query Parser의 요소
                - **DML Compiler:** DML 명령어
                    - **Embedded DML Pre-compiler: Application program과 순차적으로 일련의 과정 수행**
                        - DML Comiler에 의해서 instruction 생성
                    - Low level Instruction.
                    
                    | SELECT | 데이터 조회 및 검색 |
                    | --- | --- |
                    | INSERT,UPDATE
                    DELETE | 데이터 삽입, 수정, 삭제등 데이터를 조작하는 명령어 |
                - **DDL Interpreter: DDL 명령어**
                    - Meta data.
                    
                    | CREATE, ALTER,
                    DROP,RENAME,
                    TRUNCATE | 데이터 구조를 정의하는 명령어
                     |
                    | --- | --- |
            - **Internal Statistics**와 **Data Placement** 를 바탕으로 효율적인 execution 방법 찾기
                - **Internal Statistics**
                    - index cardinality
                        - 인덱스가 갖는 고유한 값의 수로, 높을 수록 효과적인 인덱스이다.
                        - 통계 정보를 수집하여 옵티마이저의 쿼리 실행 계획 최적화를 도움.
                            
                            → 특정 컬럼에 대한 인덱스의 쿼리 최적화 기여도 측정
                            
                        - DB의 버전과 동작 방식에 따라서 설정이 다름.
                            
                            
                            |  | PostgreSQL | Oracle |
                            | --- | --- | --- |
                            | 통계 수집 | ANALYZE  | DBMS_STATS  |
                            | 통계 정보 | pg_stats | ALL_TAB_COLUMNS |
                    - approximate intersection size
                        - 정확한 계산이 어려울 경우 근사치 활용해서 효율성
                            - 대략적인 행의 수나 카디널리티 정보로 계산
                        - 대규모 데이터셋이나 분산시스템에서 유용
                            
                            
                            |  | PostgresSQL | Oracle |
                            | --- | --- | --- |
                            | 교집합 크기 | pg_statistic
                            (null_frac, n_distnict) | DBMS_STATS
                            GATHER_PLAN_STATISTICS |
                - **Data Placement**
                    - **Node Assignment**
                        - **Sharding**
                            - 데이터베이스가 여러 노드로 분할되어 있을 때, 각각의 노드에 특정 범위의 데이터를 할당
                            - 데이터의 균형있는 분산과 조회 성능을 향상
                        - **Replication**
                            - 데이터베이스에서 데이터를 여러 노드에 복제할 때, 어떤 노드가 원본이 되고 어떤 노드가 복제본이 되는지를 결정
                            - 데이터 가용성을 높이고 장애 복구를 지원
                        - **Partitioning**
                            - 데이터베이스의 특정 테이블을 파티션으로 나누어 각 파티션을 특정 노드에 할당
                            - 데이터 관리 및 쿼리 성능을 최적화
                        
                        |  | PostgresSQL | Oracle |
                        | --- | --- | --- |
                        | Sharding | Citus 툴을 이용하여 데이터를 여러 
                        노드에 분산 | Oracle Sharding 기능을 이용하여 물리적/논리적으로 분할하여 노드에 데이터 할당 |
                        | Replication | 스트리밍/로그 기반 복제를 지원 | Oracle Data Guard를 
                        사용하여 다수의 노드를 동기적/비동기적으로 복제 |
                        | Partitioning | 특정 기준에 따라 논리적으로 파티셔닝 진행 | 특정 기준에 따라 논리적으로 파티셔닝 진행 |
        - Query Optimizer: relational operations(→ query resolution)
            - Dependency tree and optimization
                
                > Optimizer가 쿼리 실행 계획 수립 시, DB 객체 간의 의존성을 나타내는 트리 구조.
                > 
                - index ordering
                    - 여러 인덱스 중 무슨 인덱스를 먼저 사용할지 결정
                        - PG: EXPLAIN 명령어로 실행계획 및 순서 확인
                        - Oracle: EXPLAIN PLAN 사용하여 확인
                - cardinality estimation
                    - 각 테이블이나 조인에 대한 예상 Rows를 추정하는 과정
                        - PG: ANALYZE
                        - Oracle: DBMS_STATS
                - choosing access methods
                    - 전체 테이블 스캔, 인덱스 스캔, 조인 등 데이터 접근 방법 결정
                        - PG: EXPLAIN
                        - Oracle: EXPLAIN PLAN
    - Execution: Execution Plan(=Query Plan)
        - sequence of operations → complete
            
            → different execution plans → efficiency
            
            → pick(by optimizer) → best available plan
            
    - Storage: Collections of Remote and Local operations
        - Remote execution
            - write and read data in cluster and replication
        - Local execution
            - Transaction Manager
                
                > 트랜잭션을 일정하게 조정하고, 논리적으로 일관성 없는 상태로 데이터베이스가 남지 않도록 보장
                > 
            - Lock Manager
                
                > 실행 중인 트랜잭션에 대한 데이터베이스 객체에 락을 걸어서 동시 작업이 물리적 데이터 무결성을 위배 X
                > 
            - Access methods
                
                > 디스크 상의 데이터에 대한 액세스와 조직화를 담당합니다. 힙 파일과 B-트리 또는 LSM 트리와 같은 저장 구조를 포함
                > 
            - Buffer Manager
                
                > 메모리에 데이터 페이지를 캐싱하여 성능을 향상
                > 
            - 복구 관리자 (Recovery Manager)
                
                > 작업 로그를 유지하고, 시스템이 실패한 경우에 시스템 상태를 복원
                > 
        
        *Transaction and Lock managers → concurrency control → Logical and Physical data integrity> ensuring concurrent operations을 가능한 효과적인 방법으로 execution*
        
- 참고 자료
    
    PG Index efficieny : [링크](https://distributedsystemsauthority.com/index-efficiency-and-maintenance-postgresql-12-high-performance-guide-part-5-12/)
    
    PG intersection Statiscs: [링크](https://www.postgresql.org/docs/current/catalog-pg-statistic.html)
    
    Citus- PG: [링크](https://medium.com/rate-labs/citus-postgres-%EB%B6%84%EC%82%B0-%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4-a-z-%EC%86%8C%EA%B0%9C-8f2fe3dd3428)


<br>

# Memory - Versus Disk-Based DBMS

데이터베이스 시스템은 주로 메모리와 디스크에 데이터를 저장한다.

메모리 기반 데이터베이스 관리 시스템은 주로 데이터를 메모리에 저장하고 회복 및 로깅을 위해 디스크를 사용하고, 디스크 기반 데이터베이스는 대부분의 데이터를 디스크에 저장하고 메모리를 디스크 캐싱이나 임시 저장소로 사용한다.

두 유형의 시스템은 어느 정도로든 디스크를 사용하지만, 메모리 기반 데이터베이스는 내용을 거의 전적으로 RAM에 저장합니다.

메모리 접근은 여전히 디스크 접근보다 몇 차례 빠르기 때문에 주로 메모리를 주요 저장 매체로 사용하는 것이 타당하지만 RAM 가격은 여전히 SSD 및 HDD와 비교할 때 높다.

메모리를 주요 데이터 저장소로 사용하는 데이터베이스는  상대적으로 낮은 액세스 비용 및 액세스 세분화 때문에 이를 수행하며  메모리에 대한 프로그래밍은 디스크에 대한 것보다 상당히 간단하다.

운영 체제는 메모리 관리를 추상화하고 임의 크기의 메모리 chunk를 할당하고 해제할 수 있게 하므로 디스크에서는 데이터 참조, 직렬화 형식, 해제된 메모리 및 단편화를 수동으로 관리해야 한다.

메모리 데이터베이스의 성장에 대한 주요 제한 요인은 RAM의 휘발성(즉, 내구성 부족) 및 비용으로  지속적이지 않기 때문에 소프트웨어 오류, 시스템 충돌, 하드웨어 장애 및 전원 차단으로 데이터 손실이 발생하는 경우도 있다.

그러므로, Non-Volatile Memory (NVM) 기술의 가용성과 인기가 증가하게 될 가능성이 있다.

NVM 저장소는 읽기 및 쓰기 지연 시간 간의 불균형을 줄이거나 완전히 제거하며 읽기 및 쓰기 성능을 더 향상시키며 바이트 주소 지원을 허용한다.

*기존의 RAM과는 달리 전원이 꺼져도 정보가 지속되어야 하는 경우에 사용*

### Durability in Memory-Based Stores

<img width="573" alt="스크린샷 2024-03-14 오후 1 48 18" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/d8a53c69-6a60-4c1a-9499-ae4b020e85ae">



**In-memory Database**

- 휘발성 데이터 손실 방지를 위한 백업 유지
- Sequential log file에 작업 기록 보유
- 백업 복사본 저장
    - 작업 시작 중 혹은 충돌 후 로그 소멸 방지
    - 정렬된 디스크 구조(비동기적)
    - I/O 작업 수 감소
    - 백업과 로그를 통해서 복원 가능

**log record**

- 일괄 처리로 백업 적용
- 특정 시점의 데이터베이스 스냅샷 보유
    - 로그 내용 폐기 가능(체크포인팅)
    - 백업 업데이트 전, Client block없이 복구 시간 감소

**디스크 기반 Database**

- 디스크 액세스에 최적화된 특수한 저장 구조
- 포인터를 비교적 빠르게 접근
- Cache & RAM

***Oracle vs Postgresql***

- Oracle
    - 데이터 복구
        - 트랜잭션 로깅 및 redo log —> 트랜잭션 변경사항 기록
    - 체크포인팅
        - 백업과 로그로 DB 복원
        - 로그 내용을 일정 시점에서 폐기하여 복구 시간 최소화
- PostgreSQL
    - WAL
        - 트랜잭션 변경사항을 로그에 기록
    - checkpointing
        - 로그 내용을 일정 시점에서 폐기하여 복구 시간 최소화
     
# Column- Versus Row-Oriented DBMS

<img width="541" alt="스크린샷 2024-03-14 오후 1 48 54" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/fc3f4499-1a2f-4c2c-bef9-1adf0d0ac756">

DBMS는 대부분 Table안에 열과 행으로 이뤄진 데이터 레코드 집합이고, 필드는 a signle value of some type이다.

Field in Same row는 same data type을 갖는다.

실제로, DDL을 생각해보면 열마다 type을 지정하는 것을 알 수 있고, 이런 열의 모음이 행을 형성한다.

데이터는 디스크에 어떻게 저장되는지에 따라서 행 지향(b) 또는 열 지향(a) 데이터베이스로 나뉜다.

### Row-Oriented Data Layout

- 데이터를 레코드 혹은 행에 저장하는 방식
    - 각 행이 동일한 필드 집합을 가진 tabular data
    - 행을 연속적으로 저장하는 방식
- 고유한 값들이 많아서 식별하기 유용한 경우 효과적인 방식
    - 동시에 각 field를 개별적으로 수정 가능
- 한 메모리 주소에 접근할 때 그 주소뿐만 아니라 해당 블록을 전부 캐시에 가져옴으로써 공간 지역성의 효율을 높임
    - 행으로 데이터 엑세스하는 시나리오에 유용
- 전체 사용자 레코드 조회 시 좋음
    - 여러 사용자의 개별 값을 조회 시, 비효율적.
        - 다른 필드의 데이터도 페이지로 로드.
- 예시
    
    ```java
    | ID | Name | Birth Date | Phone Number |
    | 10 | John | 01 Aug 1981 | +1 111 222 333 |
    | 20 | Sam | 14 Sep 1988 | +1 555 888 999 |
    | 30 | Keith | 07 Jan 1984 | +1 333 444 555 |
    ```
    

### Column-Oriented Data Layout

- 열 단위로 파티셔닝을 하고, 동일한 열의 값이 연속적으로 저장됨
    - 별도의 파일을 저장하는 느낌보다 파일 세그먼트에 저장
    - 전체 행을 소비하지 않음
- 집계를 계산하는 분석 WorfkFlow 적합
    - 여러 개의 필드와 함께 사용되어서 복잡한 집계 처리에 좋음
- 테이블 튜플 재구성
    - 조인, 필터링 및 다중 행 집계(메타데이터 열 수준 보존)
        - 각 값이 Key를 보유하므로 중복 발생 및 저장된 데이터 증가
        - Virtual Id를 사용하여 값을 다시 매핑하여 사용(Off-set)
- 대표적인 예시
    - Apache Parquet, Apache ORC…etc
- Table 얘시
    
    ```java
    | ID | Symbol | Date | Price |
    | 1 | DOW | 08 Aug 2018 | 24,314.65 |
    | 2 | DOW | 09 Aug 2018 | 24,136.16 |
    | 3 | S&P | 08 Aug 2018 | 2,414.45 |
    | 4 | S&P | 09 Aug 2018 | 2,232.32 |
    ```
    

### Distinctions and Optimizations

- 동일한 열의 여러 값을 한 번에 읽으면 캐시 활용과 계산 효율 향상
- 벡터화된 명령을 사용하여 단일 CPU명령으로 여러 데이터 포인트 처리
- 동일한 데이터 유형을 가진 값을 합계 저장하는 것은 더 나은 압축 비율 제공
    - (숫자-다른 숫자, 문자열-다른 문자열)

위의 예시와 같이, “행/열 지향 저장소 중 하나를 선택할 시” **접근 패턴 이해 필수**

→ 읽은 데이터가 레코드 단위로 소비 및 워크로드가 대부분 포인트 쿼리 및 범위 스캔

—> 행지향 저장소 방식이 더 좋음

→ 스캔이 여러 행을 걸치거나 일부 열에서 집계를 계산

—> 열 지향 접근 방식을 고려

### Wide Column Stores

각 행은 row key로 색인되고, 관련 열은 그룹화하여 계층적 색인을 가진 다차원 정렬로 데이터가 저장된다.

아래의 표처럼, contents 및 anchor와 같은 열 family는 별도의 디스크에 저장되고, 각 열은 column key의해 식별된다.

column Famuly

- Timestamp별로 여러 버전 저장
- 상위 항복과 매개 변수를 찾게 함
  
<img width="569" alt="스크린샷 2024-03-14 오후 1 49 20" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/95de6965-9504-418f-bbe9-226d79bc8c24">

