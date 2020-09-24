## Redis migration

### 서버 접속 정보 정리

Source : localhost 16379

Target : localhost 26379

```sh
# docker run -d -p 26379:6379 --name redis6 redis:6.0 --appendonly no --port 6379
# docker run -d -p 36379:6379 --name redis61 redis:6.0 --appendonly no --port 6379
```

<br/>

### 샘플 데이터 입력

```sh
# docker exec -i -t redis6 redis-cli -h localhost -p 6379 
> set key1 value1
> save
> config get dir
1) "dir"
2) "/data"
```

<br/>

### 데이터 마이그레이션

데이터 마이그레이션 대상 서버의 appendonly 옵션이 no로 설정되어 있어야 합니다.

```sh
# docker cp redis6:/data/dump.rdb .
# docker stop redis61
# docker cp dump.rdb redis61:/data/
# docker start redis61
```

<br/>

### 데이터 마이그레이션 체크

```sh
# docker exec -i -t redis61 redis-cli -h localhost -p 6379 
localhost:6379> keys *
1) "key1"
```

<br/>

### appendonly 설정 복원

```sh
# docker exec -i -t redis61 redis-cli -h localhost -p 6379 
localhost:6379> config set appendonly yes
localhost:6379> quit
```

<br/>

### docker restart

```sh
# docker exec -i -t redis6 redis-cli -h localhost -p 6379            
localhost:6379> config get appendonly
1) "appendonly"
2) "yes"
localhost:6379localhost:6379> quit                                                              
```



