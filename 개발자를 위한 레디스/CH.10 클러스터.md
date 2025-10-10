# CH.10 클러스터

## 레디스 클러스터와 확장성
### 스케일 업 vs 스케일 아웃
- 확장성(scalability)은 운영중인 시스템에서 증가하는 트래픽에 유연하게 대응할 수 있는 능력을 뜻한다.
- 사용자나 데이터의 증가로 시스템이 처리할 수 있는 트래픽이 많아져야 할 때 용량 및 성능을 늘리기 위해 시스템의 확장이 필요한데, 이때 리소스를 투입하는 방식에 따라 스케일 업과 스케일 아웃으로 구분할 수 있다.
- 스케일 업이란 서버의 하드웨어를 높은 사양으로 업그레이드하는 것을 말한다.
- 서버에 디스크를 추가하거나 CPU나 메모리를 업그레이드함으로써 서버 능력을 증강 시키기 때문에 이를 수직 확장(Vertical scaling)이라고도 한다.
- 스케일 아웃은 장비를 추가해 시스템을 확장시키는 방식을 말한다.
- 비슷한 사양의 서버를 추가로 연결해 용랑뿐만 아니라 처리량도 나눠 성능을 높일 수 있다.
- 서버의 사양이 증가하는 것이 아니라 대수가 증가하는 것이므로 이를 수평 확장(horizontal scaling)이라고도 한다.
- 스케일 업 방식이 좀 더 간단하고 비용도 적게 들지만 하드웨어 허용 범위 내에서만 확장이 가능하기 때문에 업그레이드하는 데 한계가 있다.
- 스케일 아웃을 사용했을 때에는 장비를 추가하는 만큼 성능의 확장이 가능하지만, 데이터가 여러대의 서버에 분산 처리돼야하므로 분산처리에 대한 로직이 추가 개발돼야 한다.
### 레디스에서의 확장성
- 레디스를 운영하는 도중 키의 이빅션(eviction)이 자주 발생한다면 서버의 메모리를 증가시키는 스케일 업을 고려할 수 있다.
- 키의 이빅션은 레디스 인스턴스의 max-memory 만큼 데이터가 차 있을 때 또다시 데이터를 저장할 때 발생하는 것이므로, 서버의 메모리를 늘리고 레디스 인스턴스의 max-memory 값을 증가시키는 스케일 업을 통해 더 많은 데이터를 저장할 수 있다.
- 레디스는 단일 스레드로 동작하기 때문에 서버에 CPU를 추가한다고 해도 여러 CPU코어를 동시에 활용할 수 없다.
- 그러나 데이터를 여러 서버로 분할해 관리하면 다수의 서버에서 요청을 병렬로 처리할 수 있으므로, 서버 대수를 늘림으로써 처리랑을 선형적으로 확장시킬 수 있다.
### 레디스 클러스터의 기능
- 레디스를 클러스터 모드로 사용하면 추가적인 애플리케이션 아키텍처의 변경없이 여러 레디스 인스턴스간 수평확장이 가능해지며, 데이터의 분산처리와 복제, 자동 페일오버 기능 또한 사용할 수 있다.
#### 데이터 샤딩
- 데이터 저장소를 수평확장하며 여러 서버간에 데이터를 분할하는 데이터베이스 아키텍처 패턴을 샤딩이라 한다.
- 클러스터에서 데이터는 키를 이용해 샤딩되며 하나의 키는 항상 하나의 마스터 노드에 매핑된다.
- 클러스터의 모든 노드는 키가 저장돼야 할 노드를 알고 있기 때문에 클라이언트가 다른 노드에 데이터를 쓰거나 읽으려 할 때 키가 할당된 마스터 노드로 연결을 리디렉션한다.
- 클러스터에서 노드가 추가/변경되지 않는 이상 하나의 키는 특정 마스터에 매핑된다. 
- 매번 레디스에 키를 저장할 노드를 질의하지 않게 하기위해 클라이언트에서는 클러스터 내에서 특정키가 어떤 마스터에 저장돼 있는지의 정보를 캐싱할 수 있다.
- 이를 이용해 키를 찾아오는 시간을 단축시킬 수 있다.
#### 고가용성
- 클러스터는 각각 최소 3대의 마스터, 복제본 노드를 갖도록 구성하는 것이 일반적이며, 하나의 클러스터 구성에 속한 각 노드는 서로를 모니터링한다.
- 마스터 노드에 장애가 발생하면 이를 인지한 다른 노드들이 마스터에 연결됐던 복제본 노드를 마스터로 자동 페일오버시키기 때문에 사용자의 추가적인 개입없이 레디스의 가용성을 증가시킬 수 있다.
- 또한 마스터에 연결된 복제본의 개수를 파악해 잉여 복제본을 필요한 노드에 연결시키는 복제본 마이그레이션 작업을 수행하기도 한다.
- 모든 레디스 클리스터 노드는 다른 레디스 클러스터 노드에서 들어오는 연결을 수신하기 위한 추가 TCP 포트가 열려 있다.
- 클라이언트로부터 커맨드를 받는 TCP 포트와 독립되게 동작하며, 구성 파일에서 `cluster_bus_port` 값을 정의하지 않는다면 일반적으로 일반 포트에 10000을 더한 값으로 자동 설정된다.
- 클러스터는 모든 노드가 TCP 연결을 사용해 다른 모든 노드와 연결돼 있는 풀 메쉬(full-mesh) 토폴로지 형태다.
- 풀 메쉬 토폴로지 형태로 구성된 레디스 클러스터 구조이지만 노드 간 너무 많은 메시지를 교환하는 오버헤드는 걱정하지 않아도 된다.
- 가십 프로토콜과 구성 업데이트 메커니즘을 이용해 클러스터가 정상적인 상태에서는 노드 간 너무 많은 메시지를 교환하지는 않는다.
## 레디스 클러스터 동작 방법
### 해시슬롯을 이용한 데이터 샤딩
- 클러스터 구조에서 모든 데이터는 해시슬롯에 저장된다.
- 3대의 마스터 노드로 클러스터를 구성했을 때 해시슬롯은 분배된다.
  - 첫 번째 마스터 노드는 0부터 5460까지의 해시슬롯을 포함
  - 두 번째 마스터 노드는 5461부터 10922까지의 해시슬롯을 포함
  - 세 번째 마스터 노드는 10923부터 16383까지의 해시슬롯을 포함
- 레디스에 입력되는 모든 키는 하나의 해시슬롯에 매핑되며, 이때 다음 해시함수는 다음과 같다.
```text
HASH_SLOT = CRC16(key) mod 16384
```
- 키를 CRC16으로 먼저 한번 암호화한 다음 16384라는 값으로 나눈 나머지 값을 이용해 해시슬롯이 결정된다.
- 해시슬롯은 마스터 노드 내에서 자유롭게 옮겨질 수 있으며, 옮겨지는 중에도 데이터는 정상적으로 접근할 수 있다.
### 해시태그
- 클러스터를 사용할 때에는 다중키 커맨드를 사용할 수 없다.
- 다중키 커맨드는 한 번에 여러 키에 접근해 데이터를 가져오는 커맨드다.
```shell

MGET user1:name user2:name
```
- 레디스 클러스터에서는 서로 다른 해시슬롯에 속한 키에 대해서는 다중키 커맨드를 사용할 수 없다.
- 해시태그라는 기능을 사용하면 이런 문제를 해결할 수 있다.
- 키에 대괄호를 사용하면 전체키가 아닌 대괄호 사이에 있는 값을 이용해 해시될 수 있다. 이를 해시태그라 한다.
```shell

user:{123}:profile
user:{123}:account
```
### 자동 재구성
- 센티널 구조에서는 고가용성을 위해 센티널 인스턴스를 추가로 띄워야 했으며, 별개의 센티널 인스턴스가 레디스 노드를 감시하는 구조였다면, 클러스터 구조에서는 데이터를 저장하는 일반 레디스 노드가 서로 감시한다는 점에서 차이가 있다.
- 재구성은 총 두 가지다. 마스터 노드에 장애가 발생했을 때 복제본 노드를 마스터로 승격시키는 자동 페일오버와 잉여 복제본 노드를 다른 마스터에 연결시키는 복제본 마이그레이션이 있다.
#### 자동 페일오버
```text
   (장애)
┌ - - - - ┐      ┌─────────┐      ┌─────────┐
|  6001   |      │  6002   │      │  6003   │
|  Redis  |      │  Redis  │      │  Redis  │
└ - - - - ┘      └─────────┘      └─────────┘
     │                │                │
     ↓                ↓                ↓
┌─────────┐      ┌─────────┐      ┌─────────┐
│  6005   │      │  6006   │      │  6004   │
│  Redis  │      │  Redis  │      │  Redis  │
└─────────┘      └─────────┘      └─────────┘

┌─────────┐      ┌─────────┐      ┌─────────┐
│  6001   │      │  6002   │      │  6003   │
│  Redis  │      │  Redis  │      │  Redis  │
└─────────┘      └─────────┘      └─────────┘
     │                │                │
     ↓                ↓                ↓
┌ - - - - ┐      ┌─────────┐      ┌─────────┐
|  6005   |      │  6006   │      │  6004   │
|  Redis  |      │  Redis  │      │  Redis  │
└ - - - - ┘      └─────────┘      └─────────┘
   (승격)
```
- 6001번 마스터에 장애가 발생하면 6005번 복제본은 다른 마스터 노드들에게 페일오버를 시도해도될지 투표를 요청한다.
- 6005 노드에 또 다시 장애가 발생하면 어떻게 될까? 클러스터 내의 마스터가 하나라도 정상상태가 아닐 경우 전체 클러스터를 사용할 수 없게 된다.
```shell

cluster-require-full-coverage yes
```
- 해당 옵션의 기본값은 yes로, 레디스 클러스터에서 일부 해시슬롯을 사용하지 못하게 되면, 즉 일부 노드만 다운된 경우라도 데이터의 정합성을 위해 클러스터의 전체 상태가 fail이 돼, 문제가 생긴 해시슬롯을 포함한 전체 해시슬롯에 대한 데이터의 조작도 실패한다. 
- 만약 가용성이 중요한 서비스에서 클러스터 노드의 다운타임을 줄이고 싶다면 자동 복제본 마이그레이션이 가능하도록 아무 마스터 노드에 복제본을 하나 더 추가하는 것을 고려하는 것이 좋다.
#### 자동 복제본 마이그레이션
```text
   (장애)
┌ - - - - ┐      ┌─────────┐            ┌─────────┐
|  6001   |      │  6002   │            │  6003   │
|  Redis  |      │  Redis  │            │  Redis  │
└ - - - - ┘      └─────────┘            └─────────┘
     │                │                   │     │
     ↓                ↓                   ↓     ↓
┌─────────┐      ┌─────────┐      ┌─────────┐ ┌─────────┐
│  6005   │      │  6006   │      │  6004   │ │  6007   │
│  Redis  │      │  Redis  │      │  Redis  │ │  Redis  │
└─────────┘      └─────────┘      └─────────┘ └─────────┘

┌─────────┐      ┌─────────┐            ┌─────────┐
│  6001   │      │  6002   │            │  6003   │
│  Redis  │      │  Redis  │            │  Redis  │
└─────────┘      └─────────┘            └─────────┘
     │                │                   │     │
     ↓                ↓                   ↓     ↓
┌ - - - - ┐      ┌─────────┐      ┌─────────┐ ┌─────────┐
|  6005   |      │  6006   │      │  6004   │ │  6007   │
|  Redis  |      │  Redis  │      │  Redis  │ │  Redis  │
└ - - - - ┘      └─────────┘      └─────────┘ └─────────┘
   (승격)
   
┌─────────┐      ┌─────────┐            ┌─────────┐
│  6001   │      │  6002   │            │  6003   │
│  Redis  │      │  Redis  │            │  Redis  │
└─────────┘      └─────────┘            └─────────┘
     │                │                   │     │
     ↓                ↓                   ↓     ↓
┌ - - - - ┐      ┌─────────┐      ┌─────────┐ ┌ - - - - ┐
|  6005   |      │  6006   │      │  6004   │ │  6007   │
|  Redis  |      │  Redis  │      │  Redis  │ │  Redis  │
└ - - - - ┘      └─────────┘      └─────────┘ └ - - - - ┘
     ↑_____________________________________________⏌
                    복제본 마이그레이션
```
- 레디스 클러스터는 총 7개의 노드로 구성돼 있으며 6001, 6002 마스터는 각각 1개의 복제본을, 6003 노드는 2개의 복제본을 가지고 있다.
- 6001 노드에 장애가 발생하면 6005 노드가 마스터로 승격된다.
- 6005 마스터는 복제본이 없으며, 6002 노드는 1개의 복제본, 6003 노드는 2개의 복제본을 갖고 있다.
- 이 상황에서 레디스 클러스터는 각 마스터에 연결된 복제본 노드의 불균형을 파악해 6003에 연결돼 있는 2개의 복제본 중 하나의 복제본을 6005의 복제본이 되도록 이동시킨다.
- 이를 복제본 마이그레이션(replica migration)이라 한다.
- 복제본 마이그레이션은 모든 마스터가 적어도 1개 이상의 복제본에 의해 복제되는 것을 보장하며, 이를 이용해 클러스터 전체의 안정성을 향상시킨다.
```shell

cluster-allow-replica-migration yes
cluster-migration-barrier 1
```
- 마이그레이션은 `cluster-allow-replica-migration` 옵션이 yes일 때 동작하며, 기본값은 yes다.
- `cluster-migration-barrier`는 복제본을 마이그레이션하기 전 마스터가 가지고 있어야 할 최소 복제본의 수를 의미한다.
## 레디스 클러스터 실행하기
- 레디스를 클러스터 모드로 사용하려면 최소 3개의 마스터 노드가 있어야 한다.
- 실제 운영목적으로 사용할 때는 3개의 마스터에 각각 복제본을 추가해 총 6개의 노드로 클러스터를 구성하는 것이 일반적이다.
### 클러스터 초기화
```shell

cluster-enabled yes
```
- `cluster-enabled`설정을 yes로 변경해 레디스를 클러스터 모드로 변경한 다음 각기 다른 서버 6대에 레디스를 실행시키자.
```shell

redis-cli -cluster create [host:port] --cluster-replicas 1
```
- `--cluster-replicas 1` 옵션은 각 마스터마다 1개의 복제본을 추가할 것임을 의미한다.
```shell

$ src/redis-cli --cluster create 192.168.0.11:6379 192.168.0.22:6379
192.168.0.33:6379 192.168.0.44:6379 192.168.0.55:6379 192.168.0.66:6379
--cluster-replicas 1 -a nhncloud
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master [1] -> Slots 5461 - 10922
Master [2] -> Slots 10923 - 16383
Adding replica 192.168.0.55:6379 to 192.168.0.11:6379
Adding replica 192.168.0.66:6379 to 192.168.0.22:6379
Adding replica 192.168.0.44:6379 to 192.168.0.33:6379
M: 5a15c78c1ca9f39aceab55357d69c193756a1445 192.168.0.11:6379
   slots: [0-5460] (5461 slots) master
M: 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05 192.168.0.22:6379
   slots: [5461-10922] (5462 slots) master
M: ab1b4edfa9085b104fc3fd9f3f9d5a740f7dea66 192.168.0.33:6379
   slots: [10923-16383] (5461 slots) master
S: 52e66ec38afe31063a9821f03a9dab9ae3cdf9dd 192.168.0.44:6379
   replicates ablb4edfa9085b104fc3fd9f3f9d5a740f7dea66
S: c7fa336489d69e3dc9e5068374a19ca9376e9c20 192.168.0.55:6379
   replicates 5a15c78c1ca9f39aceab55357d69c193756a1445
```
- 해시슬롯은 마스터에만 할당되며 복제본 노드에는 할당되지 않음을 알 수 있다.
- 복제본 노드는 마스터와 동일한 데이터를 저장하기 때문에 해시슬롯 내부의 데이터를 동일하게 저장하긴 하지만, 해시슬롯을 할당받진 않는다.
```shell

slots: [0- 5460] (5461 slots) master <- 마스터 노드
slots: (0 slots) slave <- 복제본 노드
```
### 클러스터 상태 확인하기
- CLUSTER NODES 커맨드를 이용해 현재 클러스터의 상태를 확인할 수 있다.
```shell

<id> <ip:port@cport> <flags> <master> <ping-sent> <pong-recv> <config-epoch>
<link-state> <slot> <slot>...<slot>
```
### redis-cli를 이용해 클러스터 접근하기와 리디렉션
- 레디스 클러스터에 접속하기 위해서는 클러스터 모드를 지원하는 레디스 클라이언트가 필요하다.
```shell

$ redis-cli
127.0.0.1:6379> set user:1 true
(error) MOVED 10778 192.168.0.22:6379
```
- user:1이라는 키를 입력하려 시도했으나, 해당 키가 들어가야 할 해시슬롯은 10778이며, 그 해시슬롯을 가지고 있는 레디스는 ip가 192.168.0.22인 마스터 노드라는 응답을 받았다.
- 일반적인 클라이언트를 이용해 데이터를 넣을 때에는 데이터가 저장될 수 있는 노드가 정해져 있고 해당 노드에만 키에 대한 커맨드를 수행시킬 수 있음을 의미한다.
- 이러한 불편함을 줄이기 위해 Jedis, Redisson 등의 레디스 클라이언트들은 클러스터 모드 기능을 제공한다.
- redis-c1i를 사용한다면 -c 옵션을 추가해 클러스터 모드로 사용할 수 있고, 이 경우에 리디렉션 기능이 제공된다.
```shell

$ redis-cli -c
127.0.0.1:6379> set user:1 true
-> Redirected to slot [10778] located at 192.168.0.22:6379
OK
192.168.0.22:6379>
```
- 데이터를 저장하자 Redirected됐다는 메시지가 뜬 뒤, 데이터가 정상적으로 저장돼 OK 응답을 받은 것을 확인할 수 있다.
- 그 뒤 연결 ip는 로컬을 의미하는 주소인 127.0.0.1에서 실제 데이터가 저장된 노드인 192.168.0.22로 변경돼 있다.
- redis-cli라는 클라이언트가 연결을 192.168.0.11에서 저장하고자 하는 키가 있는 192.168.0.22로 자동으로 옮겼다는 것을 의미한다.
- 대부분의 레디스 클라이언트는 이렇게 리디렉션한 정보를 캐싱해 맵을 생성하게 되는데, 다음번 같은 키에 대해 커맨드를 수행해야 할 경우 이번과 같이 에러를 반환해서 커넥션을 옮겨가는 과정을 거치지 않고 캐싱된 노드로 바로 커맨드를 보낼 수 있게 해 클러스터의 성능을 향상시킬 수 있게 된다.
### 페일오버 테스트
#### 커맨드를 이용한 페일오버 발생(수동 페일오버)
- 수동으로 페일오버시키려면 페일오버시키고자 하는 마스터에 1개 이상의 복제본이 연결돼 있어야 한다.
- 페일오버를 발생시킬 복제본 노드에서 `cluster failover` 커맨드를 실행하면 페일오버를 발생시킬 수 있다.
```text

┌───────────────────┐      ┌───────────────────┐      ┌───────────────────┐
│ 192.168.0.11:6379 │      │ 192.168.0.22:6379 │      │ 192.168.0.33:6379 │
│     [ Redis ]     │      │     [ Redis ]     │      │     [ Redis ]     │
│     (Master)      │      │     (Master)      │      │     (Master)      │
└───────────────────┘      └───────────────────┘      └───────────────────┘
          │                          │                          │
          ↓                          ↓                          ↓
┌───────────────────┐      ┌───────────────────┐      ┌───────────────────┐
│ 192.168.0.55:6379 │      │ 192.168.0.66:6379 │      │ 192.168.0.44:6379 │
│     [ Redis ]     │      │     [ Redis ]     │      │     [ Redis ]     │
│      (Slave)      │      │      (Slave)      │      │      (Slave)      │
└───────────────────┘      └───────────────────┘      └───────────────────┘
```
```shell

192.168.0.55:6379> INFO REPLICATION
# Replication
role: slave
master_host: 192.168.0.11
master_port:6379
.
.
.
192.168.0.55:6379> CLUSTER FAILOVER
OK
```
```shell

192.168.0.55:6379> INFO REPLICAITON
# Replication
role: master
connected_slaves: 1
slave®: ip=192.168.0.11,port=6379, state=online,offset=613998,1ag=0
```
- 수동 페일오버가 진행되는 동안 기존 마스터에 연결된 클라이언트는 잠시 블락된다. 
- 페일오버를 시작하기 전 복제 딜레이를 기다린 뒤, 마스터의 복제 오프셋을 복제본이 따라잡는 작업이 완료되면 페일오버를 시작한다.
- 페일오버가 완료되면 클러스터의 정보를 변경하고, 모든 작업이 완료되면 클라이언트는 새로운 마스터로 리디렉션된다.
#### 마스터 동작을 중지시켜 페일오버 발생(자동 페일오버)
```shell

$ redis-cli -h <master-host> -p <master-port> shutdown
```
- 클러스터 구조에서 복제본은 redis.conf에 지정한 `cluster-node-timeout` 시간 동안 마스터에서 응답이 오지 않으면 마스터의 상태가 정상적이지 않다고 판단해 페일오버를 트리거한다.
## 레디스 클러스터 운영하기
### 클러스터 리샤딩
- 마스터 노드가 가지고 있는 해시슬롯 중 일부를 다른 마스터로 이동하는 것을 리샤딩이라 한다.
- 리샤딩은 redis-cli에서 `cluster reshard` 옵션을 이용해 수행할 수 있다.
```shell

$ redis-cli --cluster reshard 192.168.0.66 6379
>>> Performing Cluster Check (using node 192.168.0.66:6379)
S: f6c15801602a4f5e89458945362ce3e6cf1d6cd3 192.168.0.66:6379
slots: (0 slots) slave
replicates 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05
M: 5a15c78c1ca9f39aceab55357d69c193756а1445 192.168.0.11:6379
slots: [0-5460] (5461 slots) master
1 additional replica(s)
M: 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05 192.168.0.22:6379
slots: [5461-10922] (5462 slots) master
1 additional replica(s)
M: ab1b4edfa9085b104fc3fd9f3f9d5a740f7dea66 192.168.0.33:6379
slots:[10923-16383] (5461 slots) master
1 additional replica(s)
S: 52e66ec38afe31063a9821f03a9dab9ae3cdf9dd 192.168.0.44:6379
slots: (0 slots) slave
replicates ablb4edfa9085b104fc3fd9f3f9d5a740f7dea66
S: c7fa336489d69e3dc9e5068374a19ca9376e9c20 192.168.0.55:6379
slots: (0 slots) slave
replicates 5a15c78c1ca9f39aceab55357d69c193756a1445
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)?
```
- 클러스터에 속한 여러 노드 중 하나의 노드를 지정하면 해당 노드가 속한 클러스터 구조를 파악한 뒤 연결된 다른 노드의 정보를 찾아와 다음과 같이 보여준다.
- 마스터 노드뿐만 아니라 레플리카 노드 중 하나를 지정하더라도 리샤딩 동작은 동일하게 수행된다.
```shell
How many slots do you want to move (from 1 to 16384)? 100
What is the receiving node ID?
```
- 첫 번째로, 이동시킬 슬롯의 개수를 정해야 한다.
- 이 해시슬롯을 받을 노드의 ID를 입력한다.
```shell

How many slots do you want to move (from 1 to 16384)? 100
What is the receiving node ID? ablb4edfa9085b104fc3fd9f3f9d5a740f7dea66
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1:
```
- all을 입력한다면 모든 마스터 노드에서 조금씩 이동할 것을 의미한다.
- 해시슬롯을 가져올 마스터 ID를 지정하고 싶다면 하나씩 입력한 뒤 done을 입력해주면 된다.
```shell

Source node #1: all
Ready to move 100 slots.
  Source nodes:
    M: 5a15c78c1ca9f39aceab55357d69c193756а1445 192.168.0.11:6379
      slots: [0-5460] (5461 slots) master
      1 additional replica(s)
    M: 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05 192.168.0.22:6379
      slots: [5461-10922] (5462 slots) master
      1 additional replica(s)
  Destination node:
    M: ab1b4edfa9085b104fc3fd9f3f9d5a740f7dea66 192.168.0.33:6379
      slots: [10923-16383] (5461 slots) master
      1 additional replica(s)
      
Resharding plan:
    Moving slot 5461 from 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05
    Moving slot 5462 from 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05
    Moving slot 5463 from 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05
    Moving slot 5464 from 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05
.
.
.
Moving slot 5509 from 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05
    Moving slot 5510 from 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05
    Moving slot 5511 from 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05
    Moving slot 0 from 5a15c78c1ca9f39aceab55357d69c193756a1445
    Moving slot 1 from 5a15c78c1ca9f39aceab55357d69c193756a1445
    Moving slot 2 from 5a15c78c1ca9f39aceab55357d69c193756a1445
    Moving slot 3 from 5a15c78c1ca9f39aceab55357d69c193756a1445
.
.
.
Moving slot 44 from 5a15c78c1ca9f39aceab55357d69c193756a1445
    Moving slot 45 from 5a15c78c1ca9f39aceab55357d69c193756a1445
    Moving slot 46 from 5a15c78c1ca9f39aceab55357d69c193756a1445
    Moving slot 47 from 5a15c78c1ca9f39aceab55357d69c193756а1445
    Moving slot 48 from 5a15c78c1ca9f39aceab55357d69c193756a1445
Do you want to proceed with the proposed reshard plan (yes/no)?
```
- 해시슬롯을 이동시킬 노드의 입력이 끝나면 리샤딩이 진행될 소스와 데스티네이션의 마스터 노드 정보를 확인할 수 있으며, 리샤딩 플랜을 보여준다.
- `cluster check` 커맨드는 `cluster nodes`보다 조금 더 자세한 구성을 확인할 수 있다.
```shell

$ redis-cli --cluster check 192.168.0.22 6379
192.168.0.22:6379 (73abfbb3...) -> 1 keys | 5411 slots | 1 slaves.
192.168.0.11:6379 (5a15c78c...) > 0 keys | 5412 slots | 1 slaves.
192.168.0.33:6379 (ablb4edf...) -> 0 keys | 5561 slots | 1 slaves.
[OK] 1 keys in 3 masters.
0.00 keys per slot on average.
>>> Performing Cluster Check (using node 192.168.0.22:6379)
M: 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05 192.168.0.22:6379
    slots: [5512-10922] (5411 slots) master
  1 additional replica(s)
S: c7fa336489d69e3dc9e5068374a19ca9376e9c20 192.168.0.55:6379
    slots: (0 slots) slave
    replicates 5a15c78c1ca9f39aceab55357d69c193756a1445
S: 52e66ec38afe31063a9821f03a9dab9ae3cdf9dd 192.168.0.44:6379
    slots: (0 slots) slave
    replicates ablb4edfa9085b104fc3fd9f3f9d5a740f7dea66
M: 5a15c78c1ca9f39aceab55357d69c193756а1445 192.168.0.11:6379
    slots: [49-5460] (5412 slots) master
    1 additional replica(s)
S: f6c15801602a4f5e89458945362ce3e6cf1d6cd3 192.168.0.66:6379
    slots: (0 slots) slave
    replicates 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05
M: ab1b4edfa9085b104fc3fd9f3f9d5a740f7dea66 192.168.0.33:6379
    slots: [0-48], [5461-5511], [10923-16383] (5561 slots) master
    1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```
- 192.168.0.33 IP의 노드는 0부터 48, 5461부터 5511, 10923부터 16383까지의 총 5,561개의 해시슬롯을 갖게 됐음을 알 수 있다.
### 클러스터 리샤딩- 간단 버전
- 운영상 클러스터 내에서 슬롯을 이동시킬 일이 자주있거나, 자동화를 하고 싶을 경우 위와 같은 커맨드를 이용해 사용자와의 인터렉션 없이 바로 슬롯을 이동시키는 방법도 존재한다.
- 커맨드를 실행하자마자 바로 데이터가 옮겨지기 때문에 중간에 취소와 확인이 어렵다.
```shell

redis-cli --cluster reshard <host>:<port> --cluster-from <node-id>
--cluster-to ‹node-id› --cluster-slots ‹number of slots> --cluster-yes
```
```shell

$ redis-cli --cluster reshard 192.168.0.11:6379 --cluster-from 5a15c78c1ca9f
39aceab55357d69c193756a1445 --cluster-to ab1b4edfa9085b104fc3fd9f3f9d5a740f7
dea66 --cluster-slots 100 --cluster-yes
>>> Performing Cluster Check (using node 192.168.0.11:6379)
M: 5a15c78c1ca9f39aceab55357d69c193756а1445 192.168.0.11:6379
    slots: [49-5460] (5412 slots) master
    1 additional replica(s)
M: 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05 192.168.0.22:6379
    slots: [5512-10922] (5411 slots) master
    1 additional replica(s)
S: f6c15801602a4f5e89458945362ce3eбcf1d6cd3 192.168.0.66:6379
    slots: (0 slots) slave
    replicates 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05
S: 52e66ec38afe31063a9821f03a9dab9ae3cdf9dd 192.168.0.44:6379
    slots: (0 slots) slave
    replicates ablb4edfa9085b104fc3fd9f3f9d5a740f7dea66
M: ab1b4edfa9085b104fc3fd9f3f9d5a740f7dea66 192.168.0.33:6379
    slots: [0-48], [5461-5511], [10923-16383] (5561 slots) master
    1 additional replica(s)
S: c7fa336489d69e3dc9e5068374a19ca9376e9c20 192.168.0.55:6379
    slots: (0 slots) slave
    replicates 5a15c78c1ca9f39aceab55357d69c193756a1445
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
Ready to move 100 slots.
    Source nodes:
        M: 5a15c78c1ca9f39aceab55357d69c193756а1445 192.168.0.11:6379
            slots: [49-5460] (5412 slots) master
            1 additional replica(s)
    Destination node:
        M: ab1b4edfa9085b104fc3fd9f3f9d5a740f7dea66 192.168.0.33:6379
            slots: [0-48], [5461-5511], [10923-16383] (5561 slots) master
            1 additional replica(s)
```
- `--cluster-yes` 커맨드는 모든 프롬프트에 자동으로 yes를 입력하겠다는 것을 의미한다.
- 위의 커맨드를 시작하자마자 자동으로 클러스터 리샤딩 작업이 진행된다.
```shell

$ redis-cli cluster nodes
5a15c78c1ca9f39aceab55357d69c193756а1445 192.168.0.11:6379@16379 master - 0
1670856942314 1 connected 149-5460
73abfbb3872609862c9fcc229cdf1c3a3c0f2d05 192.168.0.22:6379@16379 master - 0
1670856943819 2 connected 5512-10922
f6c15801602a4f5e89458945362ce3e6cf1d6cd3 192.168.0.66:6379@16379
myself, slave 73abfbb3872609862c9fcc229cdf1c3a3c0f2d05 0 1670856942000 2
connected
ab1b4edfa9085b104fc3fd9f3f9d5a740f7dea66 192.168.0.33:6379@16379 master - 0
1670856943000 7 connected 0-148 5461-5511 10923-16383
52e66ec38afe31063a9821f03a9dab9ae3cdf9dd 192.168.0.55:6379@16379 slave ab1b4
edfa9085b104fc3fd9f3f9d5a740f7dea66 0 1670856942615 7 connected
C7fa336489d69e3dc9e5068374a19ca9376e9c20 192.168.0.44:6379@16379 slave 5a15c
78c1ca9f39aceab55357d69c193756a1445 0 1670856942000 1 connected
```
- 192.168.0.33 IP의 노드가 가지고 있는 해시슬롯이 0-148, 5461-5511, 10923-16383으로 변경된 것을 알 수 있다.
### 클러스터 확장- 신규 노드 추가
### 노드 제거하기
### 레디스 클러스터로의 데이터 마이그레이션
### 복제본을 이용한 읽기 성능 향상

## 레디스 클러스터 동작 방법
### 하트비트 패킷
### 해시슬롯 구성이 전파되는 방법
### 노드 핸드셰이크
### 클러스터 라이브 재구성
### 리디렉션
### 장애 감지와 페일오버
### 복제본 선출
