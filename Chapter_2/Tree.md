
|목차|내용|
|------|---|
|1|[1.Binary Search Trees](#Binary-Search-Trees)|
|2|[2.Ubiquitous B-Trees](#Ubiquitous-B-Trees)|


# Binary Search Trees

1장에선 저장 구조(가변/불변)에 따라서 DBMS의 설계와 구현에 영향을 미치는 것을 설명했고,  대부분의 가변 저장 구조는 장소 직접 업데이트 메커니즘을 사용하며 삽입, 삭제 또는 업데이트 작업 중에 데이터 레코드는 대상 파일의 위치에 직접 업데이트된다.

Storage 엔진은 종종 동일한 데이터 레코드의 여러 버전을 데이터베이스에 유지 할 수 있고, 다중 버전 동시성 제어 또는 슬롯이 있는 페이지 조직 사용하는 경우이다

1. **다중 버전 동시성 제어 (MVCC):**
    - **개념:** 다중 버전 동시성 제어는 여러 트랜잭션이 동시에 데이터를 읽고 쓸 수 있도록 하는 동시성 제어 기술로  각 트랜잭션은 자신만의 데이터 버전을 보게 되므로 서로간의 간섭이 최소화된다.
    - **B-Tree에서의 적용:** B-Tree에서는 삽입, 삭제, 업데이트 작업 시 새로운 버전의 노드를 생성하고 트랜잭션은 해당 버전을 참조하므로 여러 트랜잭션이 동시에 작동할 때도 각각의 트랜잭션이 자신의 일관된 데이터 버전을 유지한다.
2. **슬롯이 있는 페이지 조직:**
    - **개념:** 슬롯이 있는 페이지 조직은 페이지 내에 슬롯을 사용하여 레코드를 저장하는 방식이므로 각 슬롯은 하나의 레코드를 가리키며, 레코드의 크기가 가변적일 때 유용하다.
    - **B-Tree에서의 적용:** B-Tree에서는 각 노드가 페이지에 저장되고, 페이지 내의 슬롯을 사용하여 여러 레코드를 저장하므로 이는 효율적인 공간 활용을 가능케 하고, 페이지 조직의 변경이 필요할 때 기존의 슬롯 구성을 업데이트하여 새로운 데이터를 삽입하거나 기존 데이터를 삭제하므로 이는 동시성을 향상시키고 I/O 작업을 최적화하는 데 도움이 됩니다.
    

B-Tree는 Rudolph Bayer와 Edward M. McCreight에 의해서 1971년에 소개되었으며 시간이 지남에 따라 인기를 얻었습니다.

현재는 가장 인기 있는 저장 구조 중 하나는 B-Tree입니다. 많은 오픈 소스 데이터베이스 시스템은 B-Tree를 기반으로 하며, 이들은 여러 사용 사례를 대다수로 처리하는 것으로 입증되었습니다.

<img width="529" alt="스크린샷 2024-03-14 오후 2 09 04" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/c149b7e8-ef19-4814-b09a-7c59f2a0451f">


- B-Tree 인기 이유
    1. **다양한 사용 사례 대응**: B-Tree는 다양한 사용 사례에 효과적으로 대응할 수 있는 유연한 구조를 가지고 있기에  범용적으로 사용되며, 삽입, 삭제, 검색 등의 작업에서 효율적으로 동작한다.
    2. **포괄적인 쿼리 성능**: B-Tree는 범위 검색 및 범위 쿼리에 특히 효과적이므로 범위 쿼리에 대한 높은 성능은 데이터베이스에서 다양한 쿼리 유형을 처리할 때 중요하다.
    3. **높은 평균 및 최악 케이스 성능**: B-Tree는 삽입, 삭제 및 검색 작업의 평균 및 최악 케이스에서 모두 효율적으로 동작하는 특징때문에 다양한 작업 부하에서 일관된 성능을 제공한다.
    4. **인덱싱**: B-Tree는 데이터를 인덱싱하는 데 적합한 구조를 제공하므로 데이터베이스에서 빠른 검색을 지원하며, 인덱스 작업에 대한 성능 향상을 제공한다.
    5. **다중 버전 동시성 제어 지원**: B-Tree는 다중 버전 동시성 제어(MVCC)를 비롯한 다양한 동시성 제어 메커니즘을 지원하는 데 효과적이기에 여러 트랜잭션이 동시에 데이터를 수정할 때 데이터 일관성을 유지하는 데 중요하다.
    6. **장애 복구**:  트랜잭션 로그와 함께 사용되어 데이터 손실을 방지하고 시스템이 무결성을 유지한다.
    7. **업데이트의 효율성**: B-Tree는 기존 레코드를 직접 업데이트할 수 있는 불변성의 장점을 살려 빠르게 업데이트를 처리한다.

B-Tree에 대해 자세히 들어가기 전에 전통적인 검색 트리(예: 이진 검색 트리, 2-3 트리 및 AVL 트리)에 대한 대안을 고려해야 하는 이유에 대해 먼저 설명합니다.

## Binary Search Trees

이진 검색 트리(BST)는 효율적인 키-값 조회를 위해 사용되는 정렬된 인메모리 데이터 구조이고 여러 노드로 구성되어 있다.

각 트리 노드는 키, 이 키와 관련된 값, 그리고 두 개의 자식 포인터로 표현된다.   BST는 단일 노드인 루트 노드에서 시작하여 트리에는 단 하나의 루트만 존재한다. 

각 노드는 검색 공간을 왼쪽 및 오른쪽 서브트리로 분할하고, 노드 키가 해당 왼쪽 서브트리에 저장된 키보다 크고 오른쪽 서브트리에 저장된 키보다 작다는 이진 트리 노드 불변성을 보여줍니다

<img width="563" alt="스크린샷 2024-03-14 오후 2 09 18" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/5f5b1c5d-e66f-4f24-837b-113ece87d8f8">

트리의 루트에서부터 left 레벨의 끝까지 왼쪽 포인터를 따라가면 트리 내에서 가장 작은 키를 가진 노드와 그와 관련된 값을 찾을 수 있다

마찬가지로 오른쪽 포인터를 따라가면 트리 내에서 가장 큰 키를 가진 노드와 그와 관련된 값을 찾을 수 있으며 값은 트리의 모든 노드에 저장한다. 

검색은 루트 노드에서 시작하며, 검색된 키가 더 높은 레벨에서 찾아졌다면 트리의 하단 레벨에 도달하기 전에 종료된다.

### Tree Balancing

Insert 작업은 특정한 패턴을 따르지 않으며, 원소를 삽입하는 과정에서 트리가 balance를 잃을 수 있습니다 (즉, 한 쪽 브랜치가 다른 쪽보다 더 길어지는 상황이 발생할 수 있습니다). 

최악의 경우는 (b)에 나와 있는 것처럼, 극단적인 트리가 형성되어 로그 복잡도 대신 선형 복잡도가 나타나는 비균형한 트리가 됩니다.(정말 극단적인 예시)

<img width="536" alt="스크린샷 2024-03-14 오후 2 09 32" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/39fae063-5d29-4e1e-b5f3-969466c42794">


위의 그림은 balance 문제를 약간 과장했을 수 있지만, 중요한 사실인 “트리가 balance(이하: 균형)를 유지해야 하는 이유”를 설명합니다. 

모든 항목이 트리의 한 쪽에 모이는 것은 어느 정도 불가능하지만, 적어도 일부 항목은 트리의 한쪽에 모이게 되면 검색이 크게 느려지는 경우가 있다.

균형 잡힌 트리는 트리 내의 항목 총 개수 N에 대해 log2 N의 높이를 가지며, 두 서브트리 간의 높이 차이가 1보다 크지 않은 것으로 정의되기에 균형을 유지하지 않으면, 이진 검색 트리 구조의 성능 이점을 잃게 되며, 삽입 및 삭제 순서가 트리 모양을 결정하게 됩니다.

균형잡힌 트리(a)에서는 왼쪽 또는 오른쪽 노드 포인터를 따라가면 평균적으로 검색 공간이 절반으로 줄어들어 조회 복잡도가 로그 함수로 표현됩니다: O(log2 N). 

그러나 트리가 균형을 유지하지 않으면 최악의 경우(b) 복잡도는 O(N)으로 상승하며, 모든 요소가 트리의 한 쪽에 모일 수 있는 상황이 발생한다.

새로운 요소를 트리의 한 쪽 가지에 추가하여 그 가지를 더 길게 만들고 나머지 가지는 비워둘 대신(b), 각 작업 후에 트리는 균형을 맞추고, 노드를 재구성하여 트리 높이를 최소화하고 각 쪽에 있는 노드 수를 제한하는 방식으로 유지된다.

b와 같은 방식을 균형있게 유지하는 방법은 “노드가 추가 혹은 제거된 후 회전 단계 수행”이다.

<img width="543" alt="스크린샷 2024-03-14 오후 2 09 46" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/67ad129d-3a56-4571-9c42-3f8f587290b3">

만약 삽입 작업이 한 가지 가지를 균형을 깨뜨리게 만든다면 (한 가지 가지의 연속된 두 노드가 하나의 자식만 갖는 경우), 중간에 위치한 노드를 중심으로 노드를 회전시킬 수 있습니다. Figure 2-4에 나와 있는 예제에서 회전 중에 중간 노드(3), 회전 피벗이라고도 하는 이 노드는 한 수준 높아지고, 그 부모는 그의 오른쪽 자식이 된다.

(*최대한 이렇게 하고 싶진 않을 것 같다.*)

### Trees for Disk-Based Storage

위에서 언급했듯이, 균형이 깨진 트리는 최악의 경우 복잡도가 O(N)이고, 균형 잡힌 트리는 평균적으로 O(log2 N)이다.

BST를 디스크 기반 데이터 구조로 할 경우, **low fanout, relocate nodes, and update pointers rather frequently**로 인해서 유지 보수 비용의 증가로 비실용적이다.

**혹시 BST를 디스크 기반으로 할 경우 생길 문제**

1. 지역성(locality)
    
    > 요소가 무작위로 추가되기 때문에 새로 생성된 노드가 부모 근처에 작성될 보장이 없고, 이는 노드 자식 포인터가 여러 디스크 페이지에 걸쳐 있을 수 있다는 것을 의미하므로 트리 레이아웃을 수정하고 페이지 단위의 이진 트리를 사용하여 상황을 어느 정도 개선할 수 있습니다.
    > 
2. 자식 포인터를 따라가는 비용(feat. 트리의 높이:N)
    
    > 이진 트리는 팬아웃이 두 개면서 높이는 트리 내의 요소 수의 이진 로그이며, 검색된 요소를 찾기 위해 O(log2 N)의 시행이 필요하므로, 동일한 수의 디스크 전송이 수행되어야 한다.  즉, 인메모리 데이터 구조로는 유용하지만 작은 노드 크기로 외부 저장소에는 적합하지 않습니다.
    > 
    - 자세한 설명 with 예시
      
        1. **팬아웃과 이진 트리 높이:**
            - 이진 트리의 팬아웃은 두 개입니다.
            - 즉, 각 노드는 최대 두 개의 자식을 가질 수 있습니다.
            - 트리의 높이는 트리 내의 요소 수의 이진 로그입니다.
            - 이는 트리가 완전히 균형잡혀 있을 때 트리의 높이가 최소화됨을 의미합니다.
            - 예를 들어, 8개의 요소가 있는 이진 트리는 높이가 3이 됩니다 (log2(8) = 3).
        2. **검색 비용과 디스크 전송:**
            - 이진 트리에서 특정 요소를 찾기 위해서는 트리의 높이에 비례하는 횟수의 비교가 필요합니다.
            - 따라서 이진 트리의 검색 복잡도는 O(log2 N)입니다.
            - 검색된 요소를 찾기 위해서는 해당 요소가 위치한 노드까지 따라가야 합니다.
            - 이는 트리의 높이에 해당하는 디스크 전송 횟수를 의미합니다.

**그럼에도 BST를 디스크 기반으로 하기 위한 요건**

- 이웃하는 키들의 지역성을 향상시키기 위한 높은 팬아웃.
- 탐색 중 디스크 탐색 횟수를 줄이기 위한 낮은 높이.

> 디스크 상의 BST 구현은 지역성이 내장되어 있지 않기 때문에 비교 횟수만큼 디스크 시트를 필요로 하므로 위의 요인들을 고려하면 디스크 구현에 더 적합한 트리 버전이 된다.


<br>

# Ubiquitous B-Trees

B-트리는 도서관의 거대한 Catalog룸으로 생각할 수 있습니다. 

먼저 올바른 캐비닛을 선택해야 하고, 그 캐비닛 안의 올바른 선반을 선택한 다음, 그 선반의 올바른 서랍을 선택한 다음, 서랍 안의 카드를 찾아서 검색한 항목을 찾아야 합니다. 

마찬가지로 B-트리는 검색된 항목을 빠르게 찾을 수 있도록 도와주는 계층 구조를 구축합니다.
"이진 검색 트리"에서 논의했듯이 B-트리는 균형 잡힌 검색 트리의 기초를 쌓고 있으며 더 높은 팬아웃(더 많은 자식 노드)과 작은 높이를 가지고 있습니다.
각 노드는 하나의 키에만 책임이 있으며 범위를 두 부분으로 나눕니다. 이 세부 정보 수준은 충분하고 직관인 동시에 B-트리 노드는 종종 직사각형으로 그려지고 포인터 블록도 명시적으로 표시되어 자식 노드와 분리 키 간의 관계를 강조합니다.

<img width="559" alt="스크린샷 2024-03-14 오후 2 10 45" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/985939a4-1b46-42bd-b0de-e8453e21a405">


두 구조(원 모형과 직사가형 모형) 모두 유사한 포인터 추적 의미론을 가지고 있으며 균형을 유지하는 방식에서 차이가 나타납니다. 

도식 2-8은 그것을 보여주고 BST와 B-트리 간의 유사성을 시사하고, 양쪽 모두에서 키가 트리를 서브트리로 나누고 트리를 탐색하며 검색된 키를 찾기 위해 사용된다.

<img width="559" alt="스크린샷 2024-03-14 오후 2 10 55" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/32570dd9-2361-406a-befc-8753bce11b17">

 B-트리 노드 내의 키는 순서대로 저장되어 정렬된다.

이로 인해 검색된 키를 찾기 위해 이진 검색과 유사한 알고리즘을 사용하고, 이것은 또한 B-트리에서의 조회가 로그 복잡성을 가진다는 것을 의미합니다. 

*로그 복잡성*

> 데이터의 양이 증가해도 조회 속도가 상대적으로 느리게 증가한다는 의미로 주로 이진 검색과 관련된다. 이는 간단히 말하면 데이터의 크기가 두 배로 증가할 때마다 조회 속도가 상용 로그에 비례하여 늘어난다는 것을 의미합니다.
> 

예를 들어, B-트리에서 로그 복잡성을 가진 조회를 수행한다면, 트리의 높이에 비례하는 횟수로 원하는 항목을 찾을 수 있습니다. 트리의 높이가 log(N)이라면 N개의 항목 중 하나를 찾기 위해 최대 log(N)번의 탐색이 필요하게 됩니다. 이는 효율적인 데이터 검색을 가능하게 하며, 데이터의 크기가 증가하더라도 조회 속도가 빠르게 증가하지 않는 특징을 가지게 됩니다.

각 비교마다 디스크 탐색을 수행해야 한다면 우리를 상당히 느리게 만들 것이지만, B-트리 노드는 수십 개 또는 수백 개의 항목을 저장하기 때문에 각 수준 이동당 디스크 탐색을 한 번만 수행하므로  쿼리와 범위 쿼리를 효율적으로 실행할 수 있습니다. 대부분의 쿼리 언어에서 등호(=) 프레디케이트로 표현되는 점 쿼리는 단일 항목을 찾습니다. 반면에 비교(<, >, ≤, 및 ≥) 프레디케이트로 표현되는 범위 쿼리는 여러 데이터 항목을 순서대로 조회하는 데 사용됩니다.

## B-Tree Hierarchy

B-트리는 여러 노드로 구성되며 각 노드는 최대 N개의 키와 N+1개의 자식 노드를 가자고, 이러한 노드들은 논리적으로 세 그룹으로 나뉜다.

1. 루트 노드
    - 부모가 없으며 트리의 꼭대기에 위치한 노드
2. 리프 노드
    - 자식 노드가 없는 가장 하위 레벨의 노드
3. 내부 노드
    - 루트와 리프를 연결하는 나머지 모든 노드로 일반적으로 여러 수준의 내부 노드.
        
        <img width="518" alt="스크린샷 2024-03-14 오후 2 11 14" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/94f57e83-1980-49e6-9cce-4c9e0697c457">


B-트리는 고정 크기의 페이지를 구성하고 탐색하는 데 사용하는 기술에 기반하며  노드의 용량과 실제로 보유한 키의 수 간의 관계를 나타내는 용어는 '점유도(occupancy)'라고 합니다.

B-트리의 팬아웃(Fanout)은 각 노드가 가질 수 있는 자식 노드의 수 또는 각 노드에 저장된 키의 수로 B-트리의 효율성과 성능에 영향을 미치는 중요한 특성 중 하나입니다. 높은 팬아웃은 각 내부 노드가 많은 수의 자식을 가지고 있음을 의미하는데 이로 인해 트리의 높이가 낮아지고, 효율적인 데이터 검색이 가능해집니다. 

또한, 높은 팬아웃은 구조적 변경(노드의 삽입 또는 삭제) 시에도 비용을 낮추어주는 역할을 합니다.

**팬아웃이 높을수록 각 노드당 저장할 수 있는 키의 수가 많아지므로, 더 많은 데이터를 한 번에 처리할 수 있게 되며 특히 디스크 I/O 등의 비용이 큰 작업에서 성능 향상을 가져올 수 있습니다. B-트리의 설계와 성능에 영향을 미치는 중요한 조절 가능한 매개변수 중 하나이며, 높은 팬아웃은 B-트리의 장점 중 하나로 꼽힌다.**

## Separator Keys

B-트리 노드에 저장된 키는 인덱스 항목, 분리자 키 또는 분할 셀등으로 불리며 이러한 키들은 트리를 서브트리로 나누어 해당하는 키 범위를 보관한다. 

키는 정렬된 순서로 저장되어 이진 검색을 허용하므로 서브트리는 키를 찾아 해당 키와 연관된 포인터를 상위 레벨에서 하위 레벨로 따라가면서 찾아갑니다.

노드의 첫 번째 포인터는 첫 번째 키보다 작은 항목을 보유하는 서브트리를 가리키며, 노드의 마지막 포인터는 마지막 키보다 크거나 같은 항목을 보유하는 서브트리를 가리킵니다. 다른 포인터들은 두 키 사이의 서브트리를 참조합니다: Ki-1 ≤ Ks < Ki, 여기서 K는 키의 집합이고 Ks는 서브트리에 속하는 키입니다. 

<img width="560" alt="스크린샷 2024-03-14 오후 2 11 26" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/ee7e6c56-1a2d-496c-bb2d-d831cf803176">

일부 B-트리 변형은 범위 스캔을 간소화하기 위해 종종 리프 레벨에 형성된 형제 노드 포인터를 갖고, 이러한 포인터는 다음 sibling노드를 찾기 위해 부모 노드로 돌아가지 않도록 도와줍니다. 일부 구현은 양방향으로 포인터를 가지고 있어 리프 레벨에서 이중 연결 리스트를 형성하며 역방향 순회가 가능하게 합니다.

B-트리를 독특하게 만드는 것은 위에서 아래로가 아니라 아래에서 위로 구성되므로 리프 노드의 수가 증가함에 따라 내부 노드와 트리의 높이가 증가합니다.

B-트리는 미래의 삽입 및 업데이트를 위해 노드 내부에 여분의 공간을 확보하므로 트리 저장 공간 이용률은 50%로 떨어질 수 있지만, 일반적으로 훨씬 높습니다. 높은 점유도는 B-트리의 성능에 부정적인 영향을 미치지 않습니다.

## B-Tree Lookup Complexity

B-트리 조회 복잡성은 두 가지 관점

- 조회 동안 수행된 블록 전송 수와 비교 횟수

전송 횟수 측면에서 로그의 밑(base)은 노드 당 키의 수인 N으로 각 새로운 레벨마다 노드 수가 K배 늘어나며, 자식 포인터를 따라가면 검색 공간이 N배 감소합니다. 조회 중에는 최대 logK M(M은 B-트리의 총 아이템 수) 페이지가 검색되고, 루트에서 리프로 이어지는 경로에서 따라가야 하는 자식 포인터의 수는 트리의 높이인 h와 동일합니다.

**바교 횟수 관점에서 로그의 밑은 2이며, 각 노드 내에서 키를 찾는 것은 이진 검색을 사용하므로 복잡성은 log2 M입니다. 각 비교는 검색 공간을 절반으로 줄이므로 복잡성은 log2 M입니다.**

**예시 1: B-트리 효율성 좋음**

가정:

- B-트리의 각 노드는 최대 3개의 키를 포함 (N=3)
- B-트리에는 총 81개의 아이템이 저장 (M=81)
1. **블록 전송 수 관점:**
    - B-트리의 높이는 log₃(81) ≈ 4입니다. (81:3^4)
    - 각 레벨마다 노드 수가 3배씩 늘어나므로, 루트 레벨에는 1개의 노드, 두 번째 레벨에는 3개, 세 번째 레벨에는 9개, 네 번째 레벨에는 27개의 노드가 있습니다.
    - 최종적으로, 전체 B-트리에는 40개의 노드가 있습니다. 따라서 최대 log₃(40) ≈ 3 페이지만 검색됩니다.
2. **비교 횟수 관점:**
    - 각 노드에서 키를 찾을 때마다 이진 검색을 사용하므로 각 노드에서 log₂(3) ≈ 2번의 비교가 필요합니다.
    - 노드의 수는 높이에 따라 지수적으로 증가하므로, 루트에서 리프로 이어지는 경로에서 전체 비교 횟수는 약 2 * 4 = 8번

> 블록 전송 수와 비교 횟수 모두 상대적으로 낮게 유지되며 B-트리의 효율성이 나타남
> 

**예시 1: B-트리 효율성 안 좋음**

가정:

- B-트리의 각 노드는 최대 2개의 키를 포함할 수 있습니다. (N=2)
- B-트리에는 총 15개의 아이템이 저장되어 있습니다. (M=15)
1. **블록 전송 수 관점:**
    - B-트리의 높이는 log₂(15) ≈ 4입니다. (16: 2^4)
    - 각 레벨마다 노드 수가 2배씩 늘어나므로, 루트 레벨에는 1개의 노드, 두 번째 레벨에는 2개, 세 번째 레벨에는 4개, 네 번째 레벨에는 8개의 노드가 있습니다.
    - 최종적으로, 전체 B-트리에는 15개의 노드가 있습니다. 따라서 최대 log₂(15) ≈ 4 페이지만 검색됩니다.
2. **비교 횟수 관점:**
    - 각 노드에서 키를 찾을 때 이진 검색을 사용하므로 각 노드에서 log₂(2) = 1번의 비교가 필요합니다.
    - 노드의 수는 높이에 따라 지수적으로 증가하므로, 루트에서 리프로 이어지는 경로에서 전체 비교 횟수는 약 4 * 1 = 4번입니다.

> B-트리의 효율성이 떨어지며, 특히 블록 전송 수 관점에서 높은 높이와 노드의 수로 인해 I/O 비용이 상당히 증가하므로 따라서 B-트리의 팬아웃 및 노드 크기와 같은 파라미터를 신중하게 선택하는 것이 중요
> 

## B-Tree Lookup Algorithm

 B-트리의 조회, 삽입 및 삭제에 대한 알고리즘을 알아보려고 합니다.

B-트리에서 항목을 찾으려면 루트에서 리프까지의 단일 순회를 수행해야 합니다.  검색의 목적은 찾으려는 키 또는 해당 키의 이전 키를 찾는 것으로 정확한 일치를 찾는 것은 점 쿼리, 업데이트 및 삭제에 사용되며, 이전 키를 찾는 것은 범위 스캔 및 삽입에 유용합니다.

알고리즘은 루트에서 시작하여 이진 검색을 수행합니다. 

검색된 키를 루트 노드에 저장된 키와 비교하고, 검색 값보다 큰 첫 번째 분리자 키를 찾을 때까지 반복합니다. 이로써 검색된 서브트리가 위치합니다.

인덱스 키는 두 인접한 키 사이의 경계를 나타내며 트리를 여러 서브트리로 나눈 후서브트리를 찾으면 해당하는 포인터를 따라가고 동일한 검색 프로세스를 계속 진행하여 목표 리프 노드에 도달합니다. 

여기서 검색된 키를 찾거나 이전 키를 찾아 존재하지 않음을 결론 짓습니다.

각 레벨에서 트리의 더 자세한 뷰를 얻습니다. 가장 트리의 루트에서 시작하여 키가 더 정확하고 자세한 범위를 나타내는 다음 레벨로 이동하며 최종적으로 데이터 레코드가 위치한 리프에 도달합니다.

쿼리에서는 검색된 키를 찾거나 찾지 못한 후에 검색이 수행됩니다. 범위 스캔에서는 가장 가까운 찾은 키-값 쌍에서 시작하여 형제 포인터를 따라가 범위의 끝에 도달하거나 범위 조건이 충족될 때까지 계속됩니다.

위와 같은 알고리즘은 장점도 있지만 단점도 존재한다.

### B-Tree 단점

1. **복잡한 구현:** B-트리의 분할 및 병합과 같은 동적인 조작을 처리해야 하므로 코드가 복잡
2. **메모리 사용량:** B-트리는 내부 노드에 많은 포인터를 유지하고 있어 메모리 사용량이 크고 이는 높은 팬아웃과 노드 크기로 인한 것이며, 작은 메모리 시스템에서는 이를 고려
3. **쓰기 연산 성능:** B-트리는 쓰기 연산(삽입 및 삭제)이 수행될 때 많은 구조적 변경이 필요하므로 쓰기 연산이 상대적으로 느리며  빈번한 쓰기 연산이 발생하는 상황에서 성능이 감소
4. **높은 구조적 변경 비용:** B-트리는 균형을 유지하기 위해 구조적 변경이 필요하고, 노드의 분할 및 병합이 빈번하게 발생하면 연산의 비용이 증가
5. **저장공간 이용도:** B-트리는 각 노드에 여분의 공간을 확보하여 삽입 및 업데이트를 수용하므로 실제 데이터의 저장 공간 이용도가 낮을 수 있습니다.
6. **높은 팬아웃 관리 어려움:** 높은 팬아웃을 가진 B-트리를 효과적으로 관리하려면 적절한 조절이 필요하며, 이를 위한 최적의 구성을 찾는 것이 어려움

## Counting Keys

최적의 페이지 크기를 나타내는 장치 종속적인 자연수 k가 언급됩니다. 이 경우 페이지는 k와 2k 사이의 키를 보유할 수 있으며 부분적으로 채워져 있을 수 있으며 적어도 k + 1 및 최대 2k + 1개의 자식 노드 포인터를 보유합니다.

루트 페이지는 1에서 2k 사이의 키를 보유할 수 있습니다. 나중에 숫자 l이 소개되고, 이로써 모든 비리프 페이지가 l + 1개의 키를 가질 수 있다고 합니다.

## B-Tree Node Splits

B-트리에 값을 삽입하려면 먼저 대상 리프를 찾고 삽입 지점을 찾기

- B-Tree 알고리즘 사용
- leaf is located, 키와 값이 해당 리프에 추가

B-트리에서 업데이트는 조회 알고리즘을 사용하여 대상 리프 노드를 찾고 기존 키에 새 값을 연결함으로써 작동하며 대상 노드에 충분한 공간이 없을 경우 해당 노드는 오버플로우되었다고 말하며 새 데이터를 수용하기 위해 두 부분으로 분할한다.

- 리프 노드의 경우: 노드가 최대 N 개의 키-값 쌍을 보유할 수 있으며 하나의 키-값 쌍을 추가하면 최대 용량 N을 초과하는 경우.
- Non-리프 노드의 경우: 노드가 최대 N + 1 개의 포인터를 보유할 수 있으며 하나의 포인터를 추가하면 최대 용량 N + 1을 초과하는 경우.

분할은 새 노드를 할당하고 분할되는 노드의 절반의 요소를 이동시킨 다음 새로운 노드의 첫 번째 키와 포인터를 부모 노드에 추가함으로써 수행됩니다. 

분할이 수행되는 지점을 분할 지점(또는 중간 지점)이라고 하며 분할 지점 이후의 모든 요소는 새로 생성된 sibling 노드로 전송되며 나머지 요소는 분할되는 노드에 남습니다.

부모 노드가 가득 차 있고  키와 새로 생성된 노드의 포인터에 대한 공간이 없는 경우 부모 노드도 분할되어야 합니다. 

이 작업은 루트까지 재귀적으로 전파될 수 있습니다.

트리가 최대 용량에 도달하면(즉, 분할이 루트까지 전파됨), 루트 노드를 분할해야 합니다. 루트 노드가 분할되면 분할 지점 키를 보유하는 새로운 루트가 할당됩니다. 이전 루트(이제 절반의 항목만 보유)은 새로 생성된 형제와 함께 다음 레벨로 강등되어 트리의 높이가 1 증가합니다. 

트리 높이는 루트 노드가 분할되고 새로운 루트가 할당되거나 두 노드가 병합되어 새로운 루트가 형성될 때만 변경됩니다. 리프 및 내부 노드 수준에서는 트리가 수평으로만 성장합니다

<img width="556" alt="스크린샷 2024-03-14 오후 2 11 45" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/4e1c54e3-8aea-4233-bbc5-4fb4702f787a">

> shows a fully occupied leaf node during insertion of the new
element 11. We draw the line in the middle of the full node, leave half the elements in the node, and move the rest of elements to the new one. A split point value is placed into the parent node to serve as a separator key.
> 

<img width="544" alt="스크린샷 2024-03-14 오후 2 11 52" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/83b759a4-6d4a-45f6-823e-8fda1f0eb8ae">

> shows the split process of a fully occupied nonleaf (i.e., root
or internal) node during insertion of the new element 11. To perform a split, we first create a new node and move elements starting from index N/2 + 1 to it. The split point key is promoted to the parent.
> 

비리프 노드 분할은 항상 아래 레벨에서 전파된 분할의 표현으로  새로 생성된 노드를 가리키는 추가 포인터가 있습니다. 

부모에 충분한 공간이 없는 경우 부모도 분할되어야 합니다.
리프 노드든 비리프 노드든 분할 여부는 중요하지 않습니다. 리프 분할의 경우 키는 해당 값과 함께 이동됩니다. 분할이 완료되면 두 개의 노드가 생성되며 삽입을 마치기 위해 올바른 노드를 선택해야 합니다. 

이를 위해 구분자 키 불변식을 사용할 수 있습니다. 삽입된 키가 승격된 키보다 작으면 분할된 노드에 삽입 작업을 마치고, 그렇지 않으면 새로 생성된 노드에 삽입합니다.

**4단계로 분할 노드**

1. 새로운 노드를 할당합니다.
2. 분할되는 노드의 절반의 요소를 새로운 노드로 복사합니다.
3. 새로운 요소를 해당 노드에 배치합니다.
4. 분할된 노드의 부모에서 구분자 키와 새 노드를 가리키는 포인터를 추가합니다.

## B-Tree Node Merges

삭제는 먼저 대상 리프를 찾아서 수행됩니다. 

—> 리프가 찾아지면 해당 키와 연결된 값이 제거됩니다.

주변 노드가 값이 너무 적을 경우(즉, 그들의 점유가 임계값 이하로 떨어질 경우), 형제 노드는 병합됩니다. 이 상황을 언더플로우라고 합니다. 

**언더플로우 시나리오**

- 두 인접한 노드가 공통 부모를 가지고 있고 그들의 내용이 단일 노드에 맞는 경우, 그들의 내용을 병합(연결)해야 합니다.
- 그들의 내용이 단일 노드에 맞지 않는 경우, 키가 재분배되어 균형을 맞춥니다

두 노드가 병합되는 조건

- 리프 노드의 경우: 노드가 최대 N 개의 키-값 쌍을 보유할 수 있으며 두 인접 노드의 키-값 쌍의 합이 N 이하이면 병합
- 비리프 노드의 경우: 노드가 최대 N + 1 개의 포인터를 보유할 수 있으며 두 인접 노드의 포인터의 합이 N + 1 이하이면 병합

이를 위해 하나의 sibling node로부터 요소를 다른 sibling node로 이동시킵니다. 일반적으로 오른쪽 sibling node에서 왼쪽으로 요소를 이동시키지만, 키 순서가 유지되는 한 그 반대로 수행한다.

<img width="552" alt="스크린샷 2024-03-14 오후 2 12 05" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/744c481c-454f-496b-b822-4f6a4a803919">


> shows the merge during deletion of element 16. To do this,
we move elements from one of the siblings to the other one. Generally, elements from the right sibling are moved to the left one, but it can be done the other way around as long as the key order is preserved.
> 

<img width="590" alt="스크린샷 2024-03-14 오후 2 12 16" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/6a3b118b-383f-44ed-b425-ba4f6eebaa6b">

> shows two sibling nonleaf nodes that have to be merged
during deletion of element 10. If we combine their elements, they fit into one node, so we can have one node instead of two. During the merge of nonleaf nodes, we have to pull the corresponding separator key from the parent (i.e., demote it). The number of pointers is reduced by one because the merge is a result of the propagation of the pointer deletion from the lower level, caused by the page removal. Just as with splits, merges can propagate all the way to the root level.
> 

### 노드 병합 단계

1. 오른쪽 노드의 모든 요소를 왼쪽 노드로 복사합니다.
2. 부모에서 오른쪽 노드 포인터를 제거하거나(또는 비리프 병합의 경우 강등시킵니다).
3. 오른쪽 노드를 제거합니다.