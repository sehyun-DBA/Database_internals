# Managing Variable-Size Data

페이지에서 항목을 제거할 때 실제 셀을 제거하지 않아도 되며, 다른 셀들을 재배열하여 해제된 공간을 채울 필요가 없지만 해당 셀은 삭제되었다고 표시될 수 있고, 메모리의 가용성 목록은 해제된 메모리 양과 해제된 값의 포인터로 업데이트될 수 있습니다. 

가용성 목록은 해제된 세그먼트의 오프셋과 그 크기를 저장하면서 새 셀을 삽입할 때, 먼저 가용성 목록을 확인하여 해당 셀이 맞을 수 있는 세그먼트가 있는지 확인합니다. 

SQLite는 미사용 세그먼트를 "freeblocks"이라고 하며, 페이지 헤더에 첫 번째 freeblock에 대한 포인터를 저장합니다. 또한 페이지 내에서 사용 가능한 총 바이트 수도 저장하여 페이지를 다시 조각화한 후 새 요소를 삽입할 수 있는지 빠르게 확인할 수 있습니다.

![스크린샷 2024-03-23 오후 3 54 48](https://github.com/sehyun-DBA/Database_internals/assets/160465819/41a99df4-8e4e-445b-8348-d3db621d9b31)

Fit는 다음과 같은 전략을 기반으로 계산됩니다:

***첫 번째 적합 (First fit)***
공간을 재사용한 후 남은 공간이 다른 셀을 적합하게 너무 작을 수 있어 더 큰 오버헤드를 유발하여  공간 낭비가 될 수 있다.

***최적의 적합 (Best fit)***
최적의 적합에서는 삽입 후 남은 공간이 가장 작은 세그먼트를 찾으려고 합니다.

만약 새 셀을 삽입하기에 충분한 연속적인 바이트를 찾을 수 없지만 충분히 조각화된 바이트가 있을 경우, 활성 셀이 읽혀져 다시 작성되어 페이지를 조각화하고 새로운 쓰기를 위한 공간을 회수합니다. 

조각화 후에도 충분한 공간이 없는 경우 "오버플로우 페이지"를 생성해야 합니다.

요약하면, B-트리 레이아웃을 단순화하기 위해 각 노드가 단일 페이지를 차지한다고 가정합니다. 페이지는 고정 크기 헤더, 셀 포인터 블록 및 셀로 구성되고, 셀은 키를 보유하고 자식 노드나 관련 데이터 레코드를 나타내는 페이지에 대한 포인터를 보유합니다.

 B-트리는 간단한 포인터 계층 구조를 사용하고, 트리 파일에서 자식 노드를 찾기 위한 페이지 식별자 및 페이지 내의 셀을 찾기 위한 셀 오프셋입니다.

# Versioning

데이터베이스 시스템은 지속적으로 발전하며, 개발자들은 기능을 추가하고 버그 및 성능 문제를 수정하기 위해 노력하기에  이진 파일 형식이 변경될 수 있습니다. 

대부분의 경우, 어떤 저장 엔진 버전도 역 호환성을 위해 현재 및 하나 이상의 이전 형식을 지원해야 하고, 이를 지원하기 위해서는 파일의 버전을 확인할 수 있어야 합니다.

이를 수행하는 방법은 여러 가지가 있습니다. 

예를 들어, Apache Cassandra는 파일 이름에 버전 접두사를 사용합니다. 이렇게 하면 파일을 열지 않아도 파일의 버전을 확인할 수 있고, 4.0 버전부터 데이터 파일 이름은 "na" 접두사를 가지며, 예를 들어 "na-1-big-Data.db"입니다. 

이전 파일은 다른 접두사를 가지고 있습니다. 3.0 버전으로 작성된 파일은 "ma" 접두사를 가집니다.

또 다른 방법으로는 버전을 별도의 파일에 저장하는 것입니다. 예를 들어, PostgreSQL은 버전을 "PG_VERSION" 파일에 저장합니다.

또한 버전을 직접 인덱스 파일 헤더에 저장할 수 있습니다. 

이 경우, 헤더의 일부(또는 전체 헤더)는 버전 간에 변경되지 않는 형식으로 인코딩되어야 합니다. 파일이 인코딩된 버전을 확인한 후에는 버전별 리더를 생성하여 내용을 해석할 수 있습니다.3

# Checksumming

디스크에 있는 파일은 소프트웨어 버그나 하드웨어 고장으로 인해 손상되거나 오염될 수 있으므로 사전에 식별하고 오염된 데이터가 다른 하위 시스템이나 노드로 전파되는 것을 피하기 위해 우리는 체크섬과 순환 중복 검사(CRC)를 사용할 수 있습니다.
일부 소스들은 암호화된 해시 함수, CRC 및 체크섬 사이에 차이를 두지 않을 수 있지만 대부분의 경우 대량의 데이터를 작은 수로 축소하지만 사용 사례, 목적 및 보장이 다릅니다.

체크섬은 가장 약한 보증 형태를 제공하며 여러 비트의 오염을 감지할 수 없고, 보통 이들은 XOR 및 패리티 검사 또는 합산을 사용하여 계산됩니다.
CRC는 폭발적인 오류(예: 연속된 여러 비트가 손상된 경우)를 검출하는 데 도움이 되며, 구현에는 일반적으로 룩업 테이블 및 다항식 나눗셈이 사용됩니다. 멀티 비트 오류는 통신 네트워크 및 저장 장치의 장애의 상당한 비율이 이 방식으로 표시되므로 감지하는 것이 중요합니다.

비암호화 해시 및 CRC는 데이터 변조를 확인하는 데 사용되어서는 안되고, 항상 보안을 위해 설계된 강력한 암호화 해시를 사용해야 합니다. 

CRC의 주요 목표는 데이터에 의도하지 않은 무작위 변경이 없었는지 확인하는 것입니다. 이러한 알고리즘은 공격 및 의도적인 데이터 변경에 대해 설계되지 않았습니다.

디스크에 데이터를 쓰기 전에 해당 데이터의 체크섬을 계산하고 데이터와 함께 기록합니다. 다시 읽을 때, 체크섬을 다시 계산하고 기록된 것과 비교합니다. 체크섬 불일치가 있으면 오염이 발생했다는 것을 알 수 있으며 읽은 데이터를 사용해서는 안 됩니다.

전체 파일에 대한 체크섬을 계산하는 것은 종종 현실적이지 않으며 매번 전체 내용을 읽을 가능성이 적기 때문에 페이지 체크섬은 일반적으로 페이지에서 계산되어 페이지 헤더에 배치됩니다. 이렇게 하면 체크섬이 더 견고해질 수 있으며 오염이 단일 페이지에 포함된 경우 전체 파일을 버릴 필요가 없습니다.
