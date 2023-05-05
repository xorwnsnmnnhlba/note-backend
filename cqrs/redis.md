# Redis

### Redis
* Key-Value 형태의 비정형 데이터를 메모리에 저장하는 인메모리(In-Memory) 데이터베이스.
* DB, Cache, Streaming Engine, Message Broker로 활용되고 있음.
* HA 구성을 위해 Master-Slave 방식의 Redis Sentinel과 Multi-Master, Multi-Slave 방식의 Redis Cluster를 구축할 수 있음.
* 아래와 같이 Docker Container 형태로 설치 가능.
  * 환경설정에 필요한 redis.conf 파일은 GitHub Redis 공식 Repository에 있는 파일 이용.
```
$ ll
total 16
drwxr-xr-x  3 admin  staff    96B  5  2 10:05 conf
drwxr-xr-x@ 3 admin  staff    96B  5  2 21:32 data
-rwxr--r--  1 admin  staff   281B  5  2 20:56 docker-run.sh
-rw-r--r--  1 admin  staff    14B  5  2 09:38 redis.env

$ cat docker-run.sh
docker stop redis
docker rm redis

docker run \
--detach \
--restart=always \
--env-file=redis.env \
--publish 6379:6379 \
-v $(pwd)/conf/redis.conf:/usr/local/etc/redis/redis.conf \
-v $(pwd)/data:/data \
--name redis \
redis:7.0.11

$ cat redis.env
TZ=Asia/Seoul

$ ./docker-run.sh
```

* SasS 형태로 Amazon ElastiCache 등을 활용할 수 있음.

<br>

### In-Memory Database
* 디스크가 아닌 메모리에 데이터를 보관하고 있는 데이터베이스.
* 디스크에 저장되는 데이터베이스에 비해 데이터 접근 및 처리에 있어 월등히 빠른 속도를 가짐.
* 다만, 갑작스러운 장애 발생 시 메모리에 있는 데이터들이 모두 사라질 수 있으므로 메모리에 올라간 데이터를 주기적으로 백업해줘야 함. 대부분의 제품에서 데이터를 저장하는 주기를 설정할 수 있음.
* 대표적으로 Redis, Zookeeper 등등이 있음.

<br>

### NoSQL
* 

<br>

#### 참고
* 

<br>

#### 배워가는 것들
* 