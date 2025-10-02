# CH.3 레디스 기본 개념

### 레디스의 자료 구조

#### string
- string에는 최대 512MB의 문자열 데이터를 저장할 수 있으며 이진 데이터를 포함하는 모든 종류의 문자열이 binary-safe하게 처리되기 때문에 JPEG 이미지와 같은 바이트 값, HTTP 응답값 등의 다양한 데이터를 저장하는 것도 가능하다.

```shell

> SET hello world
OK

> GET hello
"world"
```
- SET과 함께 NX 옵션을 사용하면 지정한 키가 없을 때에만 새로운 키를 저장한다.
```shell

> SET hello newval NX
(nil)
```
- XX 옵션을 사용하면 반대로 키가 이미 있을 때에만 새로운 값으로 덮어 쓰며 새로운 키를 생성하진 않도록 동작한다.
- INCR, INCRBY와 같은 커맨드를 이용하면 string 자료 구조에 저장된 숫자를 원자적으로(atomic) 조작할 수 있다.
```shell

> SET counter 100
OK

> INCR counter
(integer) 101

> INCR counter
(integer) 102

> INCRBY counter 50
(integer) 152
```
- DECR, DECRBY 커맨드는 동일한 방식으로 데이터를 감소시키는 커맨드다.
- ⭐️ 커맨드가 원자적이라는 것은 같은 키에 접근하는 여러 클라이언트가 경쟁 상태(race condition)를 발생시킬 일이 없음을 의미한다.
- 커맨드를 수행하는 타이밍이나 순서에 따라 이미 실행한 커맨드가 무시되거나 같은 커맨드가 중복 처리돼 수행 결과가 달라지는 일은 발생하지 않음을 뜻한다.
- MSET, MGET 커맨드를 이용하면 한 번에 여러 키를 조작할 수 있다.
```shell

> MSET a 10 b 20 c 30
OK

> MGET a b c
1) "10"
2) "20"
3) "30"
```
- MSET과 MGET과 같은 커맨드를 적절하게 사용해 네트워크 통신시간을 줄인다면 전반적으로 서비스의 응답 속도를 확실하게 향상시킬 수 있게 된다.

#### list
- 일반적으로 알고 있는 다른 배열처럼 인덱스를 이용해 데이터에 직접 접근할 수도 있으며, 일반적으로 list는 서비스에서 스택과 큐로서 사용된다.
- LPUSH 커맨드는 list의 왼쪽(head)에 데이터를 추가하며, RPUSH 커맨드는 list의 오른쪽(tail)에 데이터를 추가한다.
- LRANGE 커맨드를 이용하면 list에 들어있는 데이터를 조회할 수 있다.
```shell

> LPUSH mylist E
(integer) 1

> RPUSH mylist B
(integer) 2

> LPUSH mylist D A C B A
(integer) 7

> LRANGE mylist 0 -1
1) "A"
2) "B"
3) "C"
4) "A"
5) "D"
6) "E"
7) "B"

> LRANGE mylist 0 3
1) "A"
2) "B"
3) "C"
4) "A"
```
- ⭐️ 인덱스는 음수가 될 수 있으며 가장 오른쪽(tail)에 있는 아이템의 인덱스는 -1, 그 앞의 인덱스는 -2이다.
- 위의 예제와 같이 0부터 -1까지의 아이템을 출력하라는 것은 전체 데이터를 출력하라는 의미를 갖는다.
- LPOP 커맨드를 사용하면 list에 저장된 첫 번째 아이템을 반환하는 동시에 list에서 삭제한다. 숫자와 함께 사용하면 지정한 숫자만큼의 아이템을 반복해서 반환한다.
```shell

> LPOP mylist
"A"

> LPOP mylist 2
1) "B"
2) "C"
```
- LTRIM 커맨드는 시작과 끝 아이템의 인덱스를 인자로 전달받아 지정한 범위에 속하지 않은 아이템은 모두 삭제하지만, LPOP과 같이 삭제되는 아이템을 반환하지는 않는다.
```shell

> LRANGE mylist 0 -1
1) "A"
2) "D"
3) "E"
4) "B"

> LTRIM mylist 0 1
OK

> LANGE mylist 0 -1
1) "A"
2) "D"
```
- LPUSH와 LTRIM 커맨드를 함께 사용하면 고정된 길이의 큐를 쉽게 유지할 수 있다.
- 로그는 계속해서 쌓이는 데이터이기 때문에 주기적으로 로그 데이터를 삭제해 저장 공간을 확보하는 것이 일반적이다.
```shell

> LPUSH logdata <data>
> LTRIM logdata 0 999
```
- 데이터의 개수가 1,001개가 되기 전까지는 1,000번 인덱스가 없으므로 LTRIM 커맨드를 사용해도 아무런 동작이 일어나지 않는다. 하지만 데이터가 1,001개가 되는 순간부터는 1,000 번째 인덱스, 즉 가장 처음 들어온 데이터를 삭제한다.
- 로그 데이터를 일단 쌓은 뒤 주기적으로 배치 처리를 이용해 삭제하는 것보다 위와 같은 방식으로 삭제하는 것이 훨씬 효율적이다.
- list에서 tail의 데이터를 삭제하는 작업은 O(1)로 동작 하기 때문에 굉장히 빠르게 처리되며, 배치 처리시마다 삭제할 데이터를 검색하는 것보다 훨씬 효율적이다.
- list의 양 끝에 데이터를 넣고 빼는 LPUSH, RPUSH, LPOP, RPOP 커맨드는 O(1)로 처리할 수 있어 매우 빠른 실행이 가능하다. 하지만 인덱스나 데이터를 이용해 list의 중간 데이터에 접근할 때에는 O(n)으로 처리되며, list에 저장된 데이터가 늘어남에 따라 성능은 저하된다.
- LINSERT 커맨드는 원하는 데이터의 앞이나 뒤에 데이터를 추가할 수 있다.
```shell

> LANGE mylist 0 -1
1) "A"
2) "B"
3) "C"
4) "D"

> LINSERT mylist BEFORE B E
(integer) 5

> LANGE mylist 0 -1
1) "A"
2) "E"
3) "B"
4) "C"
5) "D"
```
- LSET커맨드는 지정한 인덱스의 데이터를 신규 입력하는 데이터로 덮어 쓴다. 만약 list의 범위를 벗어난 인덱스를 입력하면 에러를 반환한다.
```shell

> LSET mylist 2 F
OK
> LRANGE mylist 0 -1
1) "A"
2) "E"
3) "F"
4) "C"
5) "D"
```
- LINDEX 커맨드를 사용하면 원하는 인덱스의 데이터를 확인할 수 있다.
```shell

> LINDEX mylist 3
"C"
```
#### hash
- 레디스에서 hash는 필드-값 쌍을 가진 아이템의 집합이다.
- 레디스에서 데이터가 key-value 쌍으로 저장되는 것처럼, 하나의 hash 자료구조 내에서 아이템은 필드-값 쌍으로 저장된다. 필드는 하나의 hash 내에서 유일하며, 필드와 값 모두 문자열 데이터로 저장된다. 
- hash는 객체를 표현하기 적절한 자료구조이기 때문에 관계형 데이터베이스의 테이블 데이터로 변환하는 것도 간편하다.
- 칼럼이 고정된 관계형 데이터베이스의 테이블과 달리, hash에서 필드를 추가하는 것은 간단하다.
- ⭐️ 관계형 데이터베이스 테이블에 데이터를 저장할 때에는 미리 합의된 칼럼 데이터를 저장할 수밖에 없는데, hash에는 새로운 필드에 데이터를 저장할 수 있기 때문에 조금더 유연한 개발이 가능하다.
- HSET 커맨드를 사용하면 hash에 아이템을 저장할 수 있으며, 한 번에 여러 필드-값 쌍 을 저장할 수도 있다.
```shell

> HSET Product:123 Name "Happy Hacking"
(integer) 1

> HSET Product:123 TypeID 35
(integer) 1

> HSET Product:123 Version 2002
(integer) 1

> HSET Product: 234 Name "Track Ball" TypeID 32
(integer) 2
```
- HMGET 커맨드를 이용하면 하나의 hash 내에서 다양한 필드의 값을 가져올 수 있다.
- HGETALL 커맨드는 hash 내의 모든 필드-값 쌍을 차례로 반환한다.
```shell

> HGET Product:123 TypeID
"35"

> HMGET Product: 234 Name TypeID
1) "Track Ball"
2) "32"

> HGETALL Product: 234
1) "Name"
2) "Track Ball"
3) "TypeID"
4) "32"
```

#### Set
- 레디스에서 set은 정렬되지 않은 문자열의 모음이다.
- 하나의 set 자료구조 내에서 아이템은 중복해서 저장되지 않으며 교집합, 합집합, 차집합 등의 집합 연산과 관련한 커맨드를 제공하기 때문에 객체 간의 관계를 계산하거나 유일한 원소를 구해야 할 경우에 사용될 수 있다.
- SADD 커맨드를 사용하면 set에 아이템을 저장할 수 있으며, 한 번에 여러 개의 아이템을 저장하는 것도 가능하다.
```shell

> SADD myset A
(integer) 1

> SADD myset A A A CB D DEFF FFG
(integer) 6

> SMEMBERS myset
1) "D"
2) "F"
3) "C"
4) "G"
5) "B"
6) "A"
7) "E"
```
- SMEMBERS 커맨드는 set 자료구조에 저장된 전체 아이템을 출력하는데, 이때 데이터를 저장한 순서와 관계없이 랜덤한 순서로 데이터가 출력되는 것을 확인할 수 있다.
- SREM 커맨드를 이용하면 set에서 원하는 데이터를 삭제할 수 있으며, SPOP 커맨드는 set 내부의 아이템 중 랜덤으로 하나의 아이템을 반환하는 동시에 set에서 그 아이템을 삭제한다.
```shell

> SREM myset B
(integer) 1

> SPOP myset
"E"
```
- set에서의 합집합은 SUNION, 교집합은 SINTER, 차집합은 SDIFF 커맨드로 수행할 수 있다.
```shell

> SINTER set:111 set:222
1) "D"
2) "E"

> SUNION set:111 set:222
1) "D"
2) "H"
3) "F"
4) "C"
5) "G"
6) "B"
7) "A"
8) "E"

> SDIFF set:111 set:222
1) "B"
2) "C"
3) "A"
```
#### sorted Set
- 레디스에서 sorted set은 스코어(score) 값에 따라 정렬되는 고유한 문자열의 집합이다.
- 저장될 때부터 스코어 값으로 정렬돼 저장된다. 같은 스코어를 가진 아이템은 데이터의 사전 순으로 정렬돼 저장된다.
- ⭐️ 배열에서 인덱스를 사용하는 것이 더 일반적이기 때문에 레디스에서도 list에서 인덱스를 다루는 것이 더 빠를 것이라고 생각할 수 있지만, 인덱스를 이용해 아이템에 접근할 일이 많다면 list가 아닌 sorted set을 사용하는 것이 더 효율적이다.
- ZADD 커맨드를 사용하면 sorted set에 아이템을 저장할 수 있으며, 스코어-값 쌍으로 입력해야 한다.
```shell

> ZADD score:220817 100 user:B
(integer) 1

> ZADD score:220817 150 user:A 150 user:C 200 user:F 300 user:E
(integer) 4
```
- 만약 저장하고자 하는 데이터가 이미 sorted set에 속해 있다면 스코어만 업데이트 되며, 업데이트된 스코어에 의해 아이템이 재정렬된다.
- 지정한 키가 존재하지 않을 때에는 sorted set 자료 구조를 새로 생성하며, 키가 이미 존재하지만 sorted set이 아닐 경우에는 오류를 반환한다.

- XX: 아이템이 이미 존재할 때에만 스코어를 업데이트한다.
- NX: 아이템이 존재하지 않을 때에만 신규 삽입하며, 기존 아이템의 스코어를 업데이트하지 않는다.
- LT: 업데이트하고자 하는 스코어가 기존 아이템의 스코어보다 작을 때에만 업데이트한다. 기존에 아이템이 존재하지 않을 때에는 새로운 데이터를 삽입한다.
- GT: 업데이트하고자 하는 스코어가 기존 아이템의 스코어보다 클 때에만 업데이트한다. 기존에 아이템이 존재하지 않을 때에는 새로운 데이터를 삽입한다.

- ZRANGE 커맨드를 사용하면 sorted set에 저장된 데이터를 조회할 수 있으며, start와 stop이라는 범위를 항상 입력해야 한다.
```shell

ZRANGE key start stop [BYSCORE | BYLEX] [REV] [LIMIT offset count]
[WITHSCORES]
```
- ZRANGE 커맨드는 기본적으로 인덱스를 기반으로 데이터를 조회하기 때문에 start와 stop인자에는 검색하고자 하는 첫 번째와 마지막 인덱스를 전달한다.
- WITHSCORE 옵션을 사용하면 데이터와 함께 스코어 값이 차례대로 출력되며, REV 옵션을 사용하면 데이터는 역순으로 출력된다.
```shell

> ZRANGE score: 220817 1 3 WITHSCORES
1) "user:A"
2) "150"
3) "user:C"
4) "150"
5) "user:F"
6) "200"

> ZRANGE score: 220817 1 3 WITHSCORES REV
1) "user:F"
2) "200"
3) "user:C"
4) "150"
5) "user:A"
6) "150"
```
- ZRANGE 커맨드에 BYSCORE 옵션을 사용하면 스코어를 이용해 데이터를 조회할 수 있다.
- start, stop 인자 값으로는 조회하고자 하는 최소, 최대 스코어를 전달해야 하며, 전달한 스코어를 포함한 값을 조회한다.
```shell

> ZRANGE score:220817 100 150 BYSCORE WITHSCORES
1) "user:B"
2) "100"
3) "user:A"
4) "150"
5) "user:C"
6) "150"
```
- 인수로 전달하는 스코어에 ( 문자를 추가하면 해당 스코어를 포함하지 않는 값만 조회할 수 있다.
```shell

> ZRANGE score:220817 (100 150 BYSCORE WITHSCORES
1) "user:A"
2) "150"
3) "user:C"
4) "150"

> ZRANGE score: 220817 100 (150 BYSCORE WITHSCORES
1) "user:B"
2) "100"
```
- 스코어의 최솟값과 최댓값을 표현하기 위해 infinity를 의미하는 -inf, +inf라는 값을 사용한다.
```shell

> ZRANGE score:220817 200 +inf BYSCORE WITHSCORES
1) "user:F"
2) "200"
3) "user: E"
4) "300"
```
- ZRANGE <key>-inf +inf BYSCORE 커맨드는 sorted set에 저장된 모든 데이터를 조회하겠다는 것을 의미한다.
```shell

> ZRANGE score:220817 +inf 200 BYSCORE WITHSCORES REV
1) "user:E"
2) "300"
3) "user:F"
4) "200"
```
- sorted set에 데이터를 저장할 때 스코어가 같으면 데이터는 사전 순으로 정렬된다고 설명했다.
- 이러한 특성을 이용해 스코어가 같을 때 BYLEX 옵션을 사용하면 사전식 순서를 이용해 특정 아이템을 조회할 수 있다.
```shell

> ZRANGE mySortedSet b (f BYLEX
banana
candy
dream
egg
```
- start와 stop에는 사전 순으로 비교하기 위한 문자열을 전달해야 하며, 이때 반드시 (나 [ 문자를 함께 입력해야 한다.
- 사전식 문자열의 가장 처음은 - 문자로, 가장 마지막은 + 문자로 대체할 수 있기 때문에 ZRANGE <key> - + BYLEX 커맨드는 sorted set에 저장된 모든 데이터를 조회하겠다는 것을 의미한다.
- 문자열은 ASCII 바이트 값에 따라 사전식으로 정렬되기 때문에, 한글 문자열도 이 기준에 따라 정렬하거나 사전식으로 검색할 수 있다.
```shell

> ZRANGE SSet 0 -1 WITHSCORES
김치볶음밥
0
납작만두
도토리묵
라면
0
묵사발
빈대떡
삼겹살
0

> ZRANGE SSet (나 (바 BYLEX
납작만두 도토리묵
라면
묵사발
```
#### 비트맵
- string 자료 구조에 bit 연산을 수행할 수 있도록 확장한 형태다.
- 비트맵을 사용할 때의 가장 큰 장점은 저장 공간을 획기적으로 줄일 수 있다는 것이다.
- SETBIT로 비트를 저장할 수 있으며, GETBIT 커맨드로 저장된 비트를 조회할 수 있다.
- 한 번에 여러 비트를 SET 하려면 BITFIELD 커맨드를 사용하면 된다.
```shell

> SETBIT mybitmap 2 1
(integer) 1

> GETBIT mybitmap 2
(integer) 1

> BITFIELD mybitmap SET u1 6 1 SET U1 10 1 SET u1 14 1
1) (integer) 1
2) (integer) 1
3) (integer) 1
```
- BITCOUNT 커맨드를 이용하면 1로 설정된 비트의 개수를 카운팅할 수 있다.
```shell

> BITCOUNT mybitmap
(integer) 4
```
#### Hyperloglog
- 집합의 원소 개수인 카디널리티를 추정할 수 있는 자료구조다.
- ⭐️ 대량 데이터에서 중복되지 않는 고유한 값을 집계할 때 유용하게 사용할 수 있는 데이터 구조다.
- hyperloglog 자료구조는 저장되는 데이터 개수에 구애받지 않고 계속 일정한 메모리를 유지할 수 있으며, 중복되지 않는 유일한 원소의 개수를 계산할 수 있다.
- 하나의 hyperloglog 자료 구조는 최대 12KB의 크기를 가지며, 레디스에서 카디널리티 추정의 오차는 0.81%로, 비교적 정확하게 데이터를 추정할 수 있다.
- PFADD 커맨드로 hyperloglog에 아이템을 저장할 수 있으며, PFCOUNT 커맨드로 저장된 아이템의 개수, 즉 카디널리티를 추정할 수 있다.
```shell

> PFADD members 123
(integer) 1

> PFADD members 500
(integer) 1

> PFADD members 12
(integer) 1

> PFCOUNT members
(integer) 3
```
#### Geospatial
- 경도, 위도 데이터 쌍의 집합으로 간편하게 지리 데이터를 저장할 수 있는 방법이다.
```shell

> GEOADD travel 14.399698913595286 50.09924276349484 prague
(integer) 1

> GEOADD travel 127.0016985 37.5642135 seoul -122.43454762275572
37.78530362582044 SanFrancisco
(integer) 2
```
- GEOPOS 커맨드를 이용하면 저장된 위치 데이터를 조회할 수 있으며, GEODIST 커맨드를 사용하면 두 아이템 사이의 거리를 반환할 수 있다.
```shell

> GEOPOS travel prague
1) 1)"14.39969927072525024"
2)"50.09924150927290043"

> GEODIST travel seoul prague KM
"8252.9957"
```
- ⭐️ GEOSEARCH 커맨드를 이용하면 특정 위치를 기준으로 원하는 거리내에 있는 아이템을 검색할 수 있다.
- BYRADIUS 옵션을 사용하면 반경 거리를 기준으로, BYBOX 옵션을 사용하면 직사각형 거리를 기준으로 데이터를 조회할 수 있다.
#### stream
- 레디스를 메시지 브로커로서 사용할 수 있게 하는 자료구조다.
- 데이터를 계속해서 추가하는 방식(append-only)으로 저장하므로, 실시간 이벤트 혹은 로그성 데이터의 저장을 위해 사용할 수 있다.
### 레디스에서 키를 관리하는 법
#### 키의 자동 생성과 삭제
- 키의 생성과 삭제 3가지 공통적 규칙
  1. ⭐️ 키가 존재하지 않을 때 아이템을 넣으면 아이템을 삽입하기 전에 빈 자료구조를 생성한다.
  ```shell
  > DEL mylist
  (integer) 1
  
  > LPUSH mylist 1 2 3
  (integer) 3
  ```
  - 키가 존재하지 않을 때 LPUSH 커맨드를 사용해 데이터를 입력하면 명시적으로 키를 생성하는 작업을 하지 않아도 mylist라는 이름의 list 자료 구조가 생성된다.
  - 저장하고자 하는 키에 다른 자료 구조가 이미 생성돼 있을 때 아이템을 추가하는 작업은 에러를 반환한다.
  ```shell
  > SET hello world
  OK
  
  > LPUSH hello 1 2 3
  (error) WRONGTYPE Operation against a key holding the wrong kind of value
  
  > TYPE hello
  string
  ```
  2. ⭐️ 모든 아이템을 삭제하면 키도 자동으로 삭제된다(stream은 예외).
  ```shell
  > LPUSH mylist 1 2 3
  (integer) 3
  
  > EXISTS mylist
  (integer) 1
  
  > LPOP mylist
  "3"
  
  > LPOP mylist
  "?"
  
  > LPOP mylist
  "1"
  
  > EXISTS mylist
  (integer) 0
  ```
  3. ⭐️ 키가 없는 상태에서 키 삭제, 아이템 삭제, 자료구조 크기 조회같은 읽기 전용 커맨드를 수행하면 에러를 반환하는 대신 키가 있으나 아이템이 없는 것처럼 동작한다.
  ```shell
  > DEL mylist
  (integer) 0
  
  > LLEN mylist
  (integer) 0
  
  > LPOP mylist
  (nil)
  ```
#### 키와 관련된 커맨드
#### EXISTS
- EXISTS는 키가 존재하는지 확인하는 커맨드다. 키가 존재하면 1을, 존재하지 않으면 0을 반환한다.
```shell

> SET hello world
OK

> EXISTS hello
(integer) 1

> EXISTS world
(integer) 0
```
#### KEYS
- KEYS 커맨드는 레디스에 저장된 모든 키를 조회하는 커맨드다. 매칭되는 패턴에 해당하는 모든 키의 list를 반환한다.
  - h?llo에는 hello, hallo가 매칭될 수 있다.
  - h*llo에는 hllo, heeeello가 매칭될 수 있다.
  - h[ae]llo에는 hello, hallo가 매칭될 수 있지만, hillo는 매칭되지 않는다.
  - h[^e]llo에는 hallo, hbllo가 매칭될 수 있지만, hello는 매칭되지 않는다.
  - h[a-b]llo에는 hallo, hbllo만 매칭될 수 있다
- KEYS는 위험한 커맨드다. 레디스에 100만 개의 키가 저장돼 있다면 모든 키의 정보를 반환한다. 레디스는 싱글 스레드로 동작하기 때문에 실행시간이 오래 걸리는 커맨드를 수행하는 동안 다른 클라이언트에서 들어오는 다른 모든 커맨드는 차단 된다.
#### SCAN
- SCAN은 KEYS를 대체해 키를 조회할 때 사용할 수 있는 커맨드다.
- ⭐️ KEYS는 한 번에 모든 키를 반환하는 커맨드로, 잘못 사용하면 문제가 발생할 수 있지만, SCAN 커맨드는 커서를 기반으로 특정 범위의 키만 조회할 수 있기 때문에 비교적 안전하게 사용할 수 있다.
```shell

> SCAN 0
1) "17"
2) 1) "key:12"
   2) "key:8"
   3) "key:4"
   4) "key:14"
   5) "key:16"
   6) "key:17"
   7) "key:15"
   8) "key:10"
   9) "key:3"
   10) "key:7"
   11) "key:1"

> SCAN 17
1) "0"
2) 1) "key:5"
   2) "key:18"
   3) "key:0"
   4) "key:2"
   5) "key:19"
   6) "key:13"
   7) "key:6"
   8) "key:9"
   9) "key:11"
```
- 처음으로 SCAN 커맨드를 사용해 키를 조회할 때에는 커서에 0을 입력해줬다. 첫 번째로 반환되는 값은 다음 SCAN 커맨드를 사용할 때 인수로 사용해야 하는 커서 위치다.
- 그 다음으로 반환되는 데이터는 저장된 키의 list다.
- 두 번째 SCAN 커맨드는 다음 커서 값으로 0을 반환했으며, 이는 레디스에 저장된 모든 키를 반환해서 더 이상 검색할 키 값이 없다는 것을 의미한다.
- 클라이언트는 반환되는 첫 번째 인자가 0이 될 때까지 SCAN을 반복적으로 사용해 레디스에 저장된 모든 키를 확인할 수 있다.
- 기본적으로 한 번에 반환되는 키의 개수는 10개 정도지만 COUNT 옵션을 사용하면 이 개수를 조정할 수 있다. 하지만 데이터는 정확하게 지정한 개수만큼 출력되지는 않는데, 레디스는 메모리를 스캔하며 데이터가 저장된 형상에 따라 몇 개의 키를 더 읽는 것이 효율적이라고 판단되면 1~2개의 키를 더 읽은 뒤 함께 반환하기도 한다.
- MATCH 옵션을 사용하면 KEYS에서처럼 입력한 패턴에 맞는 키 값을 조회한다. 하지만 이때 반환되는 값은 사용자가 의도하는 것과는 약간 다를 수 있다.
```shell

> KEYS *11*
1) "key:118"
2) "key:110"
3) "key:111"
4) "key:114"
5) "key:112"
6) "key:119"
7) "key:116"
8) "key:115"
9) "key:11"
10) "key:117"
11) "key:113"
```
- SCAN 커맨드와 MATCH 옵션을 이용해 키 값을 조회할 때에는 한 번에 패턴에 매칭된 여러 개의 키 값이 반환되지 않는다.
- 적은 수의 결과가 반환되거나 혹은 빈 값이 반환될 수 있다.
```shell

> SCAN 0 match *11*
1) "48"
2) 1) "key:115"

> SCAN 48 match *11*
1) "52"
2) (empty array)

> SCAN 52 match *11*
1) "190"
2) 1) "key:111"
```
- TYPE 옵션을 이용하면 지정한 타입의 키만 조회할 수 있다.
```shell

> SCAN 0 TYPE zset
1) "48"
2) (empty array)

> SCAN 48 TYPE zset
1) "128"
2) (empty array)

> SCAN 0 TYPE zset count 1000
1) "0"
2) 1) "zkey"
   2) "geokey"
```
- SCAN과 비슷한 커맨드로 SSCAN, HSCAN, ZSCAN이 있다. 각각 set, hash, sorted set에서 아이템을 조회하기 위해 사용되는 SMEMBERS, HGETALL, ZRANGE WITHSCORE를 대체 해서 서버에 최대한 영향을 끼치지 않고 반복해서 호출할 수 있도록 사용할 수 있는 커맨드다.
#### SORT
- list, set, sorted set에서만 사용할 수 있는 커맨드로, 키 내부의 아이템을 정렬해 반환한다.
- LIMIT 옵션을 사용하면 일부 데이터만 조회할 수 있으며, ASC/DESC 옵션을 사용하면 정렬 순서를 변경할 수 있다.
- 정렬할 대상이 문자열일 경우 ALPHA 옵션을 사용하면 데이터를 사전 순으로 정렬해 조회할 수 있다.
```shell

> LPUSH mylist a
(integer) 1

> LPUSH mylist b
(integer) 2

> LPUSH mylist c
(integer) 3

> LPUSH mylist hello
(integer) 4

> SORT mylist
(error) ERR One or more scores can't be converted into double

> SORT mylist ALPHA
1) "a"
2) "b"
3) "c"
4) "hello"
```
#### RENAME / RENAMENX
- RENAME, RENAMENX 커맨드 모두 키의 이름을 변경하는 커맨드다. 하지만 RENAMENX 커맨드는 오직 변경할 키가 존재하지 않을 때에만 동작한다.
```shell

> SET a apple
OK

> SET b banana
OK

> RENAME a aa
OK

> GET aa
"apple"

> SET a apple
OK

> SET b banana
OK

> RENAMEEX a b
(integer) 0

> get b
"banana"
```
#### COPY
```shell

COPY source destination [DB destination-db] [REPLACE]
```
- Source에 지정된 키를 destination 키에 복사한다.
- Destination에 지정한 키가 이미 있는 경우 에러가 반환되는데, REPLACE 옵션을 사용하면 destination 키를 삭제한 뒤 값을 복사하기 때문에 에러가 발생하지 않는다.
```shell

> SET B BANANA
OK

> COPY B BB
(integer) 1

> GET B
"BANANA"

> GET BB
"BANANA"
```
#### TYPE
- 지정한 키의 자료 구조 타입을 반환한다.
#### OBJECT
- 키에 대한 상세 정보를 반환한다.
- 사용할 수 있는 Subcommand 옵션으로는 ENCODING, IOLETIME 등이 있으며 해당 키가 내부적으로 어떻게 저장됐는지, 혹은 키가 호출되지 않은 시간이 얼마나 됐는지 등을 확인할 수 있다.

#### 키의 삭제
#### FLUSHALL
- 레디스에 저장된 모든 키를 삭제한다.
- 기본적으로 FLUSHALL 커맨드는 SYNC한 방식으로 동작해 모든 데이터가 삭제된 경우에만 OK를 반환해서 커맨드가 실행되는 도중에는 다른 응답을 처리할 수 없다.
- ASYNC 옵션을 사용하면 Flush는 백그라운드로 실행되고, 커맨드가 수행됐을 때 존재했던 키만 삭제해서 flush되는 중 새로 생성된 키는 삭제되지 않는다.

#### DEL
- 키와 키에 저장된 모든 아이템을 삭제하는 커맨드다. 기본적으로 동기적으로 작동한다.
#### UNLINK
- DEL과 비슷하게 키와 데이터를 삭제하는 커맨드다. 하지만 이 커맨드는 백그라운드에서 다른 스레드에 의해 처리되며, 우선 키와 연결된 데이터의 연결을 끊는다.
- set, sorted set과 같이 하나의 키에 여러 개의 아이템이 저장된 자료구조의 경우 1개의 키를 삭제하는 DEL 커맨드를 수행하는 것은 레디스 인스턴스에 영향을 끼칠 가능성이 존재한다.
- 키에 저장된 아이템이 많은 경우 DEL이 아니라 UNLINK를 사용 해 데이터를 삭제하는 것이 좋다.

#### 키의 만료 시간
#### EXPIRE
- 키가 만료될 시간을 초 단위로 정의할 수 있다.
- NX: 해당 키에 만료 시간이 정의돼 있지 않을 경우에만 커맨드 수행
- XX: 해당 키에 만료 시간이 정의돼 있을 때에만 커맨드 수행
- GT: 현재 키가 가지고 있는 만료 시간보다 새로 입력한 초가 더 클 때에만 수행
- LT: 현재 키가 가지고 있는 만료 시간보다 새로 입력한 초가 더 작을 때에만 수행

#### EXPIREAT
- 키가 특정 유닉스 타임스탬프에 만료될 수 있도록 키의 만료 시간을 직접 지정한다.
#### EXPIRETIME
- 키가 삭제되는 유닉스 타임스탬프를 초 단위로 반환한다.
- 키가 존재하지만 만료시간이 설정돼 있지 않은 경우에는 1을, 키가 없을 때에는 -2를 반환한다.
#### TTL
- 키가 몇 초 뒤에 만료되는지 반환한다.
- 키가 존재하지만 만료 시간이 설정돼 있지 않은 경우에는 1을, 키가 없을 때에는 -2를 반환한다