## Redis Oeprator

### 1. 차트 레포 세팅

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami

kubectl create namespace redis
```

### 2. 차트 설치

```bash
kubectl create namespace redis
helm install my-redis bitnami/redis \
  -n redis \
  -f 2-redis-values.yaml
```

Redis 클러스터 CRD(Custom Resource Definition) 작성: Redis 클러스터를 생성하기 위해 Redis 리소스를 정의하는 YAML 파일을 작성합니다. 아래의 내용을 ` 파일로 저장합니다.

&nbsp;

### 3. Redis 상태 확인

Redis 클러스터가 정상적으로 배포되었는지 확인합니다. 다음 명령어로 Pods의 상태를 확인합니다.

```bash
kubectl get pods -n redis
```

모든 Redis Pods가 Running 상태여야 합니다.

&nbsp;

### 4. Redis에 접근하기

Redis 클러스터에 접근하기 위해 Redis CLI를 사용할 수 있습니다. 다음 명령어로 Redis 클라이언트를 실행합니다.

```bash
kubectl run -it --rm --namespace redis redis-client --image=bitnami/redis:latest -- bash
```

Redis 클라이언트 내부에서 Redis 서버에 연결합니다:

```bash
redis-cli -u redis://my-redis:6379 -p 6379
```

&nbsp;

5. 데이터 확인 및 테스트

Redis에 데이터를 저장하고 확인합니다. 아래는 몇 가지 명령어입니다.

```bash
I have no name!@redis-client:/$ redis-cli -u redis://my-redis:6379 -p 6379 -a password
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
my-redis:6379> set key "Hello world"
OK
my-redis:6379> get key
"Hello world"
```

&nbsp;

6. 클러스터 모니터링

Redis Sentinel을 사용하여 클러스터의 상태를 모니터링할 수 있습니다. Sentinel의 로그를 확인하려면 다음 명령어를 사용합니다:

Sentinel이 포함된 Redis 파드에 접속: Sentinel이 포함된 Redis 파드에 접속합니다. 파드의 이름은 실제 실행 중인 파드의 이름으로 바꿉니다.

```bash
kubectl exec -it my-redis-node-0 -n redis -- bash
```

Sentinel CLI 사용: Sentinel CLI를 통해 Sentinel 상태를 확인합니다. Sentinel에 접근하기 위해 다음 명령어를 실행합니다.

> [!WARNING]
> `-a` 옵션의 값은 `values.yaml` 파일에 선언된 `auth.password` 값과 동일하게 세팅해서 실행합니다.

```bash
redis-cli -p 26379 -a password
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
```

Sentinel에 접속한 후, 다음과 같은 명령어를 사용하여 모니터링할 수 있습니다.

Sentinel 상태 확인:

```bash
SENTINEL masters
1)  1) "name"
    2) "mymaster"
    3) "ip"
    4) "my-redis-node-0.my-redis-headless.redis.svc.cluster.local"
    5) "port"
    6) "6379"
    7) "runid"
    8) "d82ccf5bbea9ae013872b7d589e1ee6044f3fbcb"
    9) "flags"
   10) "master"
   11) "link-pending-commands"
   12) "0"
   13) "link-refcount"
   14) "1"
   15) "last-ping-sent"
   16) "0"
   17) "last-ok-ping-reply"
   18) "530"
   19) "last-ping-reply"
   20) "530"
   21) "down-after-milliseconds"
   22) "60000"
   23) "info-refresh"
   24) "9362"
   25) "role-reported"
   26) "master"
   27) "role-reported-time"
   28) "491268"
   29) "config-epoch"
   30) "0"
   31) "num-slaves"
   32) "2"
   33) "num-other-sentinels"
   34) "2"
   35) "quorum"
   36) "2"
   37) "failover-timeout"
   38) "180000"
   39) "parallel-syncs"
   40) "1"
```

이 명령어는 Sentinel이 감시하는 모든 마스터의 상태를 보여줍니다. 출력에는 마스터의 이름, IP 주소, 포트, 현재 상태 등의 정보가 포함됩니다.

슬레이브 상태 확인:

```bash
SENTINEL slaves <master-name>
```

특정 마스터에 대한 슬레이브의 상태를 확인합니다. <master-name>은 이전 `SENTINEL masters` 명령어에서 확인한 마스터의 이름입니다.

센티넬 정보를 확인합니다.

```bash
SENTINEL sentinels mymaster
```

&nbsp;

7. 클러스터 삭제

헬름 차트를 삭제합니다. 쿠버네티스의 Redis 리소스가 같이 지워집니다.

```bash
helm uninstall my-redis -n redis
kubectl delete namespace redis
```

&nbsp;

`minikube` 클러스터를 삭제합니다.

```bash
# 중지
minikube stop

# 영구 삭제
minikube delete
```