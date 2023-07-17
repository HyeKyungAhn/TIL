# Hash table
컴퓨터에 저장되는 결국 숫자(이진수)
이 숫자를 어떻게 해석하는냐가 문제

## Hash table이란?
- Hash table은 숫자를 저장하는 컴퓨터의 속성을 가장 잘 활용한 컨테이너
- 물체를 일렬로 세우지 않고 분류하기위한 바구니에 넣어두는 것(binning, hash table)
    - hash map은 물체를 바구니에 넣을 때, 바구니에 넣는 기준과 실제 넣는 물체가 다르다. 물류창고에서 어떤 상품을 제자리에 놔둬야할 때, 바코드를 찍어서 장소를 찾아 상품을 둬야한다. 기준을 key, 물체를 value라고 한다.
- 데이터를 저장할 때 같은 조건 시 언제나 같은 label을 반환하여 탐색속도를 높여줌


## Hash table의 특징
- array의 random access와 linked list의 가변길이 특성이라는 장점을 가지고 있지만 둘의 단점을 완화시키는 데이터 구조가 hash table
- hash table에서는 데이터 그 자체가 저장과 검색의 기준이 될 수 있다. hash table은 데이터 정렬과 분류가 다른 데이터 구조보다 불리함. 그래서 hash table을 사용할 때는 데이터의 정렬과 분류에 신경을 쓰지 않아도 되는 데이터를 다뤄야한다.

## Hash table의 구조
- hash table은 hash function, array로 이루어져있다.
- 데이터를 저장할 때, 데이터를 hash function에 돌려서 음수가 아닌 정수를 반환한다. 반환한 정수에 해당하는 array의 index에 데이터를 저장한다.

- hash map은 물체를 바구니에 넣을 때, 바구니에 넣는 기준과 실제 넣는 물체가 다르다. 물류창고에서 어떤 상품을 제자리에 놔둬야할 때, 바코드를 찍어서 장소를 찾아 상품을 둬야한다. 기준을 key, 물체를 value라고 한다.

## Hash function
- 입력값이 동일할 때 언제나 유니크한 값을 만드는 함수가 hash 함수
- object끼리 비교할 때 hash 함수를 사용하면, object의 모든 값을 비교하지 않고 유니크한 값으로만 비교할 수 있음. 물론 object간의 hash 값이 높은 확률로 다르지만, 간혹가다 같을 수 있다. 유니크한 값을 얻으려면 hash 알고리즘이나.. 이런 쪽으로 가야함

이런 유니크한 값을 어떻게 만드는가가 hash table의 핵심이다

값이 같을 때 객체가 같아야한다고 가정하면, 자바의 객체를 만들 때 해쉬 코드를 오버라이딩 해서 해쉬코드를 생성하는 알고리즘을 직접 작성해야한다. 만약 해쉬코드를 오버라이딩하지 않는다면 자바의 디폴트 알고리즘을 따라(메모리주소) 해쉬코드가 생성됨. 해쉬코드가 어떤 값을 반환하는가에 따라 어떤 빈에 들어갈지가 결정됨. 해쉬코드가 같은 두 오브잭트는 컨테이너 안에서 하나가 다른 하나를 덮어쓸 수 밖에 없음

### 좋은 hash function이란?
- 좋은 hash function은 hash되는 데이터’만’ 사용해야하고 hash되는 데이터 ‘전부’를 사용해야한다.
- deterministic, 결정론적이어야한다. 즉, 같은 input에는 항상 같은 output이 나와야한다.
- 균일하게 분포(uniformly)되어야 한다. 매번 함수를 실행할 때마다 같은 값이 나오지 않고, 넓은 범위의 값을 출력해야한다.
```
unsigned int hash(char* str)
{
    int sum = 0;
    for(int j = 0; str[j] != '\0'; j++)
    {
        sum += str[j];
    }
    return sum % HASH_MAX;
}
```
위의 예시에서는 스트링의 각 char의 asccii 코드의 값을 다 더해서 HASH_MAX( array의 길이)로 나눠서 나머지를 리턴함. hash_max로 나누지 않으면 segmentation fault(허용되지 않은 메모리 접근, outofindexException)를 범함

직접 hash code를 작성하고 싶어하지는 않는다 보통. 과학이 아닌 예술.. 에 해당하기 때문에 그냥 내걸 만들기 보다는 검색해서 사용하는게 좋다. 대신 출처는 표기하자.

## Hash tabel의 속도
hash table의 속도가 빠른 이유는 데이터를 저장할 index의 길이를 정해두고, hashcode를 나머지 연한사여 얻은 값과 동일한 index에 저장함. hash code자체로 값을 검색할 수 있기 때문에 검색하는데 시간이 덜든다. array에 저장된 모든 list를 검색할 필요가 없다는 것.

## collision 발생
- 다른 데이터를 hash function에 돌렸을 때 같은 ouput이 나오는 경우
    - 무한한 조합의 string에 비해 음수가 아닌 int인 hash 코드는 유한
- 다른 hash 코드의 인덱스가 동일할 때
    - array의 길이는 한정적이기 때문
- overwrite하지 않고 다른 데이터 모두를 array에 저장하고 싶다면?
    
    → linear probing

### hash 알고리즘
- hash 함수의 hash 알고리즘의 성능이 좋지 못한경우 O(1) → O{n}
- hash 알고리즘은 입력받은 키를 가지고 얼마나 고르게 데이터 구조에 나눠 저장할 수 있는가가 성능의 기준이다

## linear probing
- 한국어로는 선형검색법이라고 한다
- collision 발생 시 이미 값이 저장되었다면, plus 1한 index에 저장하는 방법
- data를 가지고 얻은 hash code에 해당하는 index에 정확히 저장되지는 않지만, 그 근처에 저장되어있으니 보다 쉽게 찾을 수 있다
- 하지만 collision이 계속 발생할 경우, hash table의 장점을 잃게 된다
    
    → clustering

### clustering
- array 에 연속된 값 두 개가 있다면, collision이 발생할 확률이 2배가 증가
- 값이 다 저장된다면 빈자리를 찾기 위해서 infinite loop

## chaining
- hash table에 array를 저장하는게 아니라, pointers of head of linked list를 저장함
- pointer를 저장하고는, 각 linked list에 여러 개의 값을 저장하면 collision 문제를 해결할 수 있다
- linked list는 array 보다 가변적이기 때문에 계속해서 크기를 늘려 값을 저장할 수 있다
- linked list를 간략하게 설명하자면, 새 값이 insert 될 때, 기존 값의 위치를 가리키는 정보를 새 값이 대신 저장하고, 그곳에 새 값의 위치 정보를 저장함.
- 이 chaining과 이를 활용한 hash table은 많은 양의 데이터를 저장하는데 매우 효과적임
- hash table이 저장하는 pointer의 갯수가 늘어날 수록 linked list의 검색에 걸리는 시간은 줄어든다.

## 출처
- https://www.youtube.com/watch?v=nvzVHwrrub0
- https://velog.io/@cyranocoding/Hash-Hashing-Hash-Table해시-해싱-해시테이블-자료구조의-이해-6ijyonph6o#해시-테이블hash-table
- https://www.youtube.com/watch?v=S7vni1hdsZE
- https://www.youtube.com/@eleanorlim/videos