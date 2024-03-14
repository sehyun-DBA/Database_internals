# Disk-Based Structures

## Hard Disk Drives

과거 전통적인 알고리즘들은  spinning 디스크가 지속적인 저장 매체로 가장 널리 사용되던 시절이다. 

이후에 플래시 드라이브와 같은 저장 매체의 새로운 발전은 새로운 하드웨어의 기능을 활용하기 위해 기존 알고리즘을 수정하거나 새로운 알고리즘을 개발하는 추세이며,  요즘엔 비휘발성 바이트 주소 지정 저장 장치와 최적화된 새로운 데이터 구조가 등장하고 있다.

이처럼 저장매체의 변화는 곧 새로운 알고리즘과 로직의 시작이라고 할 수 있다.

spinning 디스크에서는 디스크 회전 및 기계적인 헤드 이동으로 인해서 랜덤 읽기의 비용이 증가하지만 한 번에 비용이 많이 든 부분을 마치면 연속적인 바이트(즉, 순차 작업)를 읽거나 쓰는 것은 비교적 저렴하다.

<img width="367" alt="스크린샷 2024-03-14 오후 2 14 30" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/67dcb008-c83c-4035-9427-64c4035b5854">


spinning 드라이브의 가장 작은 전송 단위는 섹터이므로 어떤 작업이 수행될 때 적어도 전체 섹터를 읽거나 쓸 수 있습니다. 

섹터 크기는 일반적으로 512byte에서 4kb까지 범위가 있습니다.

## Solid State Drives(SSD)

SSD는 spinning disk처럼 정보를 읽기 위한 위치 조절하는 head가 존재하지 않고, 일련의 문자열로 연결된 메모리 셀로 구성되어 있다.

참고로, 문자열은 배열로 결합되고, 배열은 페이지로 결합되며, 페이지는 블록으로 결합된다.

- 문자열 방식의  장점
    1. **고속의 랜덤 액세스:** SSD는 페이지 단위에서 데이터에 액세스할 수 있기 때문에 랜덤 읽기 및 쓰기 속도가 빠릅니다. 따라서 특정 데이터에 빠르게 액세스하는 데 소요되는 시간이 줄어듭니다.
    2. **병렬성과 효율성:** SSD는 여러 페이지를 동시에 처리하고 병렬로 읽거나 쓸 수 있습니다. 이는 데이터 처리 성능을 향상시키고, 입출력 작업을 효율적으로 수행할 수 있도록 합니다.
    3. **내구성과 안정성:** SSD는 기계 부품이 없기 때문에 물리적 충격에 강하고, 데이터를 안정적으로 보존할 수 있습니다. 또한, 데이터의 일부를 저장하는 방식과 페이지, 블록 간의 복사 기술을 사용하여 데이터 손실을 최소화할 수 있습니다.
    4. **고밀도 저장 및 공간 효율성:** SSD는 블록과 페이지를 효율적으로 구성하여 고밀도의 데이터를 저장할 수 있습니다. 또한, 효율적인 가용 공간 관리를 통해 빈 공간을 최소화하고 저장 용량을 최대한 활용할 수 있습니다.
    5. **에너지 효율성:** 기계 부품이 없고 램프 소비 전력이 낮기 때문에 SSD는 일반적으로 에너지를 효율적으로 사용하며, 이는 더 낮은 전력 소비와 냉각 요구를 의미합니다.

사용된 정확한 기술에 따라 한 셀은 데이터의 한 비트 또는 여러 비트를 저장할 수 있습니다. 

페이지의 크기는 장치에 따라 다르지만 일반적으로 2에서 16킬로바이트 사이이고, 블록에는 일반적으로 64에서 512개의 페이지가 포함되어 있습니다.

블록은 평면으로 구성되어 있으며, 마지막으로 평면은 die에 배치됩니다. 

<img width="620" alt="스크린샷 2024-03-14 오후 2 14 42" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/248eb66a-a56a-4fb7-8b6a-757c2f723853">

플래시 메모리 컨트롤러에서 페이지 ID를 물리적 위치로 매핑하고, 빈 페이지, 기록된 페이지, 폐기된 페이지를 추적하는 역할을 하는 부분이다. 

또한,  Flash Translation Layer(FTL)은 garbage collection하면서 안전하게 소거할 수 있는 블록을 찾습니다. 

일부 블록은 여전히 유효한 페이지를 포함할 수 있습니다. 이럴 땐, FTL은 그 블록에서 유효한 페이지를 새 위치로 이동하고 페이지 ID를 해당 위치로 다시 매핑하여 더 이상 사용되지 않는 블록을 지워서 기록할 수 있도록 만듭니다.

HDD와 SSD 모두에서 우리는 개별 바이트가 아닌 메모리 덩어리를 다루고 있기 때문에 대부분의 운영 체제는 블록 장치 추상화를 가지고 있습니다 

이는 내부 디스크 구조를 숨기고 I/O 작업을 내부적으로 버퍼링하여 블록 장치에서 단일 워드를 읽을 때 해당 워드를 포함하는 전체 블록을 읽는  무시할 수 없는 제약이며, 디스크 기반 데이터 구조를 다룰 때 항상 고려해야 할 사항입니다.

정리하자면,  가비지 컬렉션이 일반적으로 백그라운드에서 수행되지만, 그 영향은 특히 랜덤 및 정렬되지 않은 쓰기 워크로드의 경우에는 쓰기 성능에 부정적으로 작용할 수 있기에 전체 블록에만 기록하고 이후에 동일한 블록에 대한 연속적인 쓰기를 결합하는 것은 필요한 I/O 작업 수를 줄이는 데 도움이 될 수 있습니다. 

- SSD 기능 정리
    1. **Flash Memory Cells:**
        - SSD의 기본 저장 매체는 플래시 메모리 셀입니다.
        - 각 셀은 여러 비트의 데이터를 저장할 수 있으며, 이것은 다양한 플래시 기술에 따라 달라집니다.
    2. **Flash Translation Layer (FTL):**
        - FTL은 SSD 내에서 가장 중요한 부분 중 하나로, 논리적인 페이지 주소를 물리적인 블록 및 페이지 주소로 매핑합니다.
        - 비휘발성 메모리를 사용하면서도 데이터를 일관되게 관리하고 읽고 쓰기 작업을 최적화하는 데 도움이 됩니다.
        - 쓰기 및 소거 작업의 효율적인 관리를 담당하며, 가비지 컬렉션을 통해 사용하지 않는 데이터 블록을 식별하고 정리합니다.
    3. **Memory Cells Organization:**
        - 셀들은 문자열, 배열, 페이지, 블록 등의 계층적인 구조로 조직됩니다.
        - 이 계층 구조를 통해 데이터를 효율적으로 저장하고 관리할 수 있습니다.
    4. **Garbage Collection:**
        - SSD는 가비지 컬렉션을 수행하여 더 이상 필요하지 않은 데이터 블록을 식별하고 해제합니다.
        - 사용 중인 페이지를 새로운 위치로 이동하고 매핑을 업데이트하여 가비지 수집 후에 더 이상 사용되지 않는 블록을 해제합니다.
    5. **Wear Leveling:**
        - 플래시 메모리는 한 번에 제한된 횟수의 쓰기 작업만 수행할 수 있습니다. 이를 방지하기 위해 웨어 레벨링이라는 기술이 사용됩니다.
        - 웨어 레벨링은 쓰기 작업을 균등하게 분산하여 각 셀이 고르게 소모되도록 합니다.
    6. **Bad Block Management:**
        - SSD는 특정 블록이 불량일 경우에 대비하여 불량 블록 관리를 수행합니다.
        - 불량 블록은 다른 영역으로 대체되며, 사용자 데이터의 안전성을 보장합니다.
    7. **Read and Write Operations:**
        - SSD는 읽기 및 쓰기 작업을 효율적으로 수행할 수 있도록 설계되었습니다.
        - 빠른 랜덤 액세스 속도를 제공하며, 쓰기 작업은 일반적으로 블록 단위로 수행됩니다.
    8. **Error Correction:**
        - SSD는 데이터 무결성을 유지하기 위해 에러 수정 코드 (ECC) 및 다른 오류 보정 기술을 사용합니다.

## On-Disk Structures

온디스크 구조는 대상 저장소의 특성을 고려하여 설계되었으며 일반적으로 더 적은 디스크 액세스를 위해 최적화됩니다. 이를 위해 지역성을 향상시키고, 구조의 내부 표현을 최적화하며, 페이지 밖 포인터의 수를 줄이는 방법을 사용할 수 있습니다.

"이진 탐색 트리"에서 우리는 높은 팬아웃과 낮은 높이가 온디스크 데이터 구조에 대한 최적의 속성이라는 결론을 내린 것처럼 방금 포인터에서 오는 추가 공간 오버헤드와 균형 조정의 결과로 이러한 포인터를 다시 매핑하는 유지 관리 오버헤드를 논의했습니다.

B-트리는 이러한 아이디어를 결합합니다: 노드 팬아웃을 증가시키고, 트리 높이, 노드 포인터의 수 및 균형 조정 작업의 빈도를 줄입니다.

<img width="553" alt="스크린샷 2024-03-14 오후 2 14 57" src="https://github.com/sehyun-DBA/Database_internals/assets/160465819/d215a2a6-798b-4e8b-87f8-ae055b315977">