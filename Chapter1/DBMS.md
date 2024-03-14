# DBMS Architecture

![스크린샷 2024-02-04 오전 11.19.38.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/0c6a30f7-0581-4dec-a684-19dc4d80e412/932c9178-a0c9-4d9a-86d9-a6e1505a1ae8/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-02-04_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_11.19.38.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/0c6a30f7-0581-4dec-a684-19dc4d80e412/6950ab17-0dfc-482b-9c6c-5e67fcf74de3/Untitled.png)

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
