
|목차|내용|
|------|---|
|1|[1.Motivation](#Motivation)|
|2|[2.Binary Encoding](#Binary-Encoding)|



# Motivation

파일 형식을 만드는 것은 데이터 블록을 할당하고 고정 크기의 기본 형식과 구조를 사용하고, 더 큰 메모리 청크나 변수 크기의 구조 참조를 위해 포인터를 사용합니다.

관리되지 않는 메모리 모델을 가진 언어는 우리가 필요할 때마다 (합리적인 한도 내에서) 더 많은 메모리를 할당할 수 있게 해주는데 연속적인 메모리 세그먼트가 있는지, 그것이 단편화되어 있는지 여부, 그리고 해제한 후에 무엇이 발생하는지에 대해 걱정할 필요가 없습니다. 

반면에 디스크에서는 가비지 컬렉션과 단편화에 대한 관리를 우리가 스스로 처리해야 합니다.

효율적인 디스크 저장 데이터 구조를 위해서는 데이터를 빠르게 액세스할 수 있는 방식으로 디스크에 데이터를 배치하고 영구 저장 매체의 특성을 고려해 이진 데이터 형식을 찾고 데이터를 효율적으로 직렬화 및 역직렬화하는 방법을 고민해야 합니다.

C와 같은 저수준 언어를 추가 라이브러리 없이 사용한 적이 있는 사람은 이러한 제약을 알고 있을 것입니다. 구조체는 미리 정의된 크기를 가지며 명시적으로 할당되고 해제됩니다. 메모리 할당 및 추적을 수동으로 구현하는 것은 더 어렵습니다. 왜냐하면 미리 정의된 크기의 메모리 세그먼트로만 작업할 수 있으며 이미 해제된 세그먼트와 아직 사용 중인 세그먼트를 추적해야하기 때문입니다.

메인 메모리에 데이터를 저장할 때 메모리 레이아웃과 관련된 대부분의 문제는 존재하지 않거나 쉽게 해결되거나 제3자 라이브러리를 사용하여 해결할 수 있습니다. 예를 들어, 가변 길이 필드와 오버사이즈 데이터를 처리하는 것은 메모리 할당 및 포인터를 사용하므로 훨씬 더 간단합니다. 

CPU 캐시 라인, 프리페칭 및 기타 하드웨어 관련 세부 사항을 활용하기 위해 특수한 메인 메모리 데이터 레이아웃을 설계하는 경우도 있지만, 이는 주로 최적화 목적으로 수행됩니다.

# Binary Encoding

디스크에 데이터를 효율적으로 저장하려면, compact하고 직렬화 및 역직렬화가 쉬운 형식을 사용하여 데이터를 인코딩해야 합니다. 

malloc과 free와 같은 기본 요소가 없기 때문에 오직 읽기와 쓰기만 가능하므로, 우리는 다르게 액세스를 생각하고 데이터를 준비해야 합니다.

여기서는 효율적인 페이지 레이아웃을 만들기 위해 사용되는 주요 원칙(::떠한 이진 형식에도 적용되며, 파일 및 직렬화 형식 또는 통신 프로토콜을 만들 때 유사한 지침을 사용)에 대해 논의합니다. 
레코드를 페이지로 구성하기 전에 이진 형식으로 키와 데이터 레코드를 어떻게 나타낼지, 여러 값을 더 복잡한 구조로 어떻게 결합할지, 그리고 가변 크기 유형과 배열을 어떻게 구현할지 이해해야 합니다.

### Primitive Types

키와 값은 integer, date 또는 string과 같은 유형을 가지며, raw 바이너리 형식으로 표현되어(직렬화 및 역직렬화)질 수 있습니다.
대부분의 numeric 데이터 유형은 고정 크기의 값으로 표현됩니다. 멀티바이트 숫자 값과 작업할 때 인코딩과 디코딩 모두에 동일한 바이트 순서(endianness)를 사용하는 것이 중요합니다. endianness은 바이트의 순차적인 순서를 결정합니다.

**Big-endian**
순서는 가장 중요한 바이트(MSB)에서 시작하여 중요도가 감소하는 순서대로 바이트가 이어집니다. 다시 말해, MSB가 가장 낮은 주소를 갖습니다.

**Little-endian**
순서는 가장 덜 중요한 바이트(LSB)에서 시작하여 중요도가 증가하는 순서대로 바이트가 이어집니다.
그림 3-1은 이를 설명합니다. 16진수 32비트 정수 0xAABBCCDD는 AA가 MSB인 경우의 빅엔디안 및 리틀엔디안 바이트 순서로 나타납니다.

![스크린샷 2024-03-09 오후 2.49.55.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/0c6a30f7-0581-4dec-a684-19dc4d80e412/b4c74127-e15d-453e-8d39-243ead1b59fd/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-03-09_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.49.55.png)

예를 들어, 대응하는 바이트 순서를 갖는 64비트 정수를 재구성하려면, RocksDB는 목표 플랫폼 바이트 순서를 식별하는 데 도움이 되는 플랫폼별 정의를 가지고 있습니다. 

목표 플랫폼 endiannes이 값의 endianness과 일치하지 않는 경우 (EncodeFixed64WithEndian은 kLittleEndian 값을 조회하고 그 값을 값의 엔디안과 비교합니다), EndianTransform은 값을 역순으로 바이트 단위로 읽고 결과에 추가합니다.

레코드는 숫자, 문자열, 부울 및 그들의 조합과 같은 기본 유형으로 구성되지만 그러나 데이터를 네트워크를 통해 전송하거나 디스크에 저장할 때는 바이트 시퀀스만 사용할 수 있습니다. 이는 레코드를 보내거나 쓰기 위해서는 이를 직렬화(바이트의 해석 가능한 시퀀스로 변환)하고, 수신하거나 읽은 후에 사용하려면 역직렬화(바이트 시퀀스를 원래 레코드로 다시 변환)해야 한다는 것을 의미합니다.

이진 데이터 형식에서는 항상 더 복잡한 구조의 기본 요소로 시작합니다. 서로 다른 숫자 유형은 크기에서 차이가 있을 수 있습니다. byte 값은 8 비트, short는 2 바이트 (16 비트), int는 4 바이트 (32 비트), long은 8 바이트 (64 비트)입니다.

부동 소수점 숫자(예: float 및 double)는 부호, 분수 및 지수로 표현됩니다. IEEE 표준 이진 부동 소수점 산술 (IEEE 754) 표준은 널리 받아들여진 부동 소수점 숫자 표현을 설명합니다. 32비트 float는 단정도 값을 나타냅니다. 예를 들어, 부동 소수점 숫자 0.15652는 그림 3-2에 나와 있는 바와 같이 이진 표현을 갖습니다. 처음 23비트는 분수를 나타내고, 다음 8비트는 지수를 나타내며, 1비트는 부호를 나타냅니다 (숫자가 음수인지 여부).

![스크린샷 2024-03-09 오후 2.58.51.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/0c6a30f7-0581-4dec-a684-19dc4d80e412/1b67dbfd-5ece-4fe4-a610-02219b84701a/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-03-09_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.58.51.png)

### Strings and Variable-Size Data

모든 원시 숫자 형식은 고정 크기를 갖으며, 더 복잡한 값을 조합하는 것은 C의 구조체(struct)와 매우 유사하다.기본값을 구조체로 결합하고 고정 크기 배열이나 다른 메모리 영역에 대한 포인터를 사용할 수 있고,

문자열 및 다른 가변 크기 데이터 형식(고정 크기 데이터 배열 포함)은 배열 또는 문자열의 길이를 나타내는 숫자 다음에 크기 바이트가 오는 형식으로 직렬화될 수 있습니다. 

문자열의 경우 이 표현은 종종 UCSD 문자열 또는 Pascal 문자열이라고 불립니다. 이러한 방식을 의사 코드로 표현하면 다음과 같다.

```c
String
{
    size uint_16
    data byte[size]
}
```

Pascal 문자열에 대한 대안으로 null 종료 문자열이 있고, reader는 문자열의 끝에 도달할 때까지 문자 단위로 문자열을 소비합니다. 이런 방식은 문자열의 길이를 상수 시간에 알 수 있으므로 문자열 내용을 반복하는 것보다 용이하며, 언어별 문자열은 메모리에서 크기 바이트를 잘라내어 바이트 배열을 문자열 생성자에 전달하여 구성할 수 있습니다.

### Bit-Packed Data: Booleans, Enums, and Flags

부울 값은 단일 바이트를 사용하거나 true와 false를 1과 0 값으로 인코딩하여 표현할 수 있습니다. 부울은 오직 두 가지 값만 가지므로, 전체 바이트를 사용하여 표현하는 것은 낭비적이며, 개발자들은 종종 각 부울이 하나의 비트를 차지하도록 여덟 개씩 부울 값을 묶어서 그룹화합니다. 이 때, 각 1 비트는 설정되어 있고, 각 0 비트는 설정되어 있지 않거나 비어 있는 것으로 표현됩니다.

열거형(Enum)은 정수로 표현될 수 있으며, 이진 형식과 통신 프로토콜에서 자주 사용됩니다. 열거형은 자주 반복되는 저 기수의 값을 나타내기 위해 사용됩니다. 예를 들어, B-트리 노드 유형을 열거형으로 인코딩할 수 있습니다:

```c
 enum NodeType {
    ROOT, // 0x00h
    INTERNAL, // 0x01h
    LEAF // 0x02h
};
```

다른 밀접한 관련 개념은 플래그(Flags)입니다. 플래그는 패킹된 부울과 열거형의 조합입니다. 플래그는 상호 배제적인 이름이 지정된 부울 매개변수를 나타낼 수 있습니다. 예를 들어, 페이지가 값 셀을 보유하는지 여부, 값이 고정 크기인지 또는 가변 크기인지, 이 노드와 관련된 오버플로 페이지가 있는지 여부를 나타내기 위해 플래그를 사용할 수 있습니다. 각 비트가 플래그 값 하나를 나타내므로, 마스크에 대해 2의 제곱 값만 사용할 수 있습니다(이진수에서 2의 제곱은 항상 하나의 설정된 비트를 가지기 때문에 사용할 수 있습니다. 예: 2^3 == 8 == 1000b, 2^4 == 16 == 0001 0000b 등):

```c

int IS_LEAF_MASK = 0x01h; // 비트 #1
int VARIABLE_SIZE_VALUES = 0x02h; // 비트 #2
int HAS_OVERFLOW_PAGES = 0x04h; // 비트 #3
```

패킹된 부울과 마찬가지로, 플래그 값은 비트 마스크 및 비트 연산자를 사용하여 패킹된 값에서 읽고 쓸 수 있습니다. 예를 들어, 플래그 중 하나를 담당하는 비트를 설정하기 위해, 비트 연산 OR (|)과 비트마스크를 사용할 수 있습니다. 비트마스크 대신 비트 시프트(<<)와 비트 인덱스를 사용할 수도 있습니다. 비트를 해제하기 위해 비트와 비트 연산 AND (&) 및 비트의 부정을 사용할 수 있습니다.

연산자(~)를 사용하여 비트 n이 설정되어 있는지 여부를 테스트할 수 있습니다. 비트 단위 AND의 결과를 0과 비교하여 확인할 수 있습니다:

```c

// 비트 설정
flags |= HAS_OVERFLOW_PAGES;
flags |= (1 << 2);

// 비트 해제
flags &= ~HAS_OVERFLOW_PAGES;
flags &= ~(1 << 2);

// 비트가 설정되어 있는지 여부 확인
is_set = (flags & HAS_OVERFLOW_PAGES) != 0;
is_set = (flags & (1 << 2)) != 0;

```

여기서 **`flags`**는 플래그들을 담고 있는 변수이며, **`HAS_OVERFLOW_PAGES`**와 같은 특정 플래그를 나타내는 상수로 설정됩니다. **`HAS_OVERFLOW_PAGES`**의 경우, 비트를 설정하려면 비트 OR(|) 연산을 사용하고, 비트를 해제하려면 비트 AND(&) 연산과 비트의 부정(~)을 사용합니다. 비트가 설정되어 있는지 여부를 확인하려면 비트 AND(&) 연산을 사용하고, 그 결과를 0과 비교하여 확인합니다.