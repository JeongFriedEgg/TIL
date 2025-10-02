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
- LPUSH 커맨드는 list의 왼쪽(head)에 데이터를 추가하며, RPUSH 커맨드는 list의 오른쪽(ail)에 데이터를 추가한다.
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
- 인덱스는 음수가 될 수 있으며 가장 오른쪽(tail)에 있는 아이템의 인덱스는 -1, 그 앞의 인덱스는 -2이다.
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
- 관계형 데이터베이스 테이블에 데이터를 저장할 때에는 미리 합의된 칼럼 데이터를 저장할 수밖에 없는데, hash에는 새로운 필드에 데이터를 저장할 수 있기 때문에 조금더 유연한 개발이 가능하다.
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
- ⭐️ 배열에서 인덱 스를 사용하는 것이 더 일반적이기 때문에 레디스에서도 list에서 인덱스를 다루는 것이 더 빠를 것이라고 생각할 수 있지만, 인덱스를 이용해 아이템에 접근할 일이 많다면 list가 아닌 sorted set을 사용하는 것이 더 효율적이다.
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