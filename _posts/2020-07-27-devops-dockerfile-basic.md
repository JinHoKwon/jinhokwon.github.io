---
title: Dockerfile 그리고 DockerHub
tags: [devops, docker]
comments: true
categories: devops
header:
  teaser: "/assets/images/docker/docker_logo.png"
---
본 문서에서는 Dockerfile 의 기본적인 사용 방법 및 Docker Hub에 Push 하는 방법에 대해서 설명하고 있습니다.
<br/>
<br/>

## 1. Dockerfile



Docker 이미지를 만들기 위해서 필요한 명령어를 나열한 문서를 Dockerfile 이라고 하며,

Dockerfile을 빌드하면 Docker 이미지가 생성이 됩니다.

<br/>



#### 1-1. Dockerfile 디렉토리 생성

```sh
# mkdir -p /tmp/test/dockerfile
# cd /tmp/test/dockerfile
```

<br/>

#### 1-2. Dockerfile 생성

`java -version` 명령어로 자바 버전만 출력하는 간단한 Dockerfile을 생성합니다.

```dockerfile
FROM openjdk:11-jdk-slim
ENTRYPOINT ["java", "-version"]
```

* **FROM** : Docker baseimage
* **ENTRYPOINT** : Docker run 시점에 실행할 명령어

<br/>

#### 1-3. Dockerfile 빌드

```sh
# docker build --no-cache . -t jinhokwon/jdk11:v1.0
```

<br/>

#### 1-4. Docker images

```sh
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jinhokwon/jdk11     v1.0                73650d9ebb25        6 minutes ago       402MB
```

<br/>

#### 1-5. Dockerfile 실행

```sh
# docker run -i -t -d --name jdk11 jinhokwon/jdk11:v1.0 
9cb75b0c79a6e0cbc315f41eb6322e63e57262a68bdd4cf2c6d2fd8e6caabf72
```

<br/>

#### 1-6. Dockerfile 실행 로그 확인

```sh
# docker logs jdk11
openjdk version "11.0.8" 2020-07-14
OpenJDK Runtime Environment 18.9 (build 11.0.8+10)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.8+10, mixed mode)
```

<br/>

#### 1-7. Dockerfile container 확인

```sh
# docker ps -a
CONTAINER ID   IMAGE                  COMMAND          CREATED          STATUS                      PORTS  NAMES
9cb75b0c79a6   jinhokwon/jdk11:v1.0   "java -version"  52 seconds ago   Exited (0) 51 seconds ago          jdk11
```



<br/>

<br/>

## 2. Docker Hub

Dockerfile로 만들어진 이미지는 로컬에서만 확인이 가능하고, 

외부에서 공유 받으려면, 다음과 같이 Docker Hub 와 같은 Docker Registry를 활용해야 합니다.

<br/>

#### 2-1. Docker login

Docker Hub에 로그인할 계정 정보를 입력합니다.

```sh
# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: jinhokwon
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
```



이 때, 올바르게 로그인 되었다면, `/root/.docker/config.json`에 로그인 인증 정보가 기록이 됩니다.

```json
{
    "auths": {
        "https://index.docker.io/v1/": {
            "auth": "amluaG9rd29uOm5ld2FzNDMxODY="
        }
    },
    "HttpHeaders": {
        "User-Agent": "Docker-Client/19.03.12 (linux)"
    }
}
```





<br/>

#### 2-1. Push docker image

```sh
# docker push jinhokwon/jdk11:v1.0
The push refers to repository [docker.io/jinhokwon/jdk11]
e232d268a4cd: Pushed
421e7764601c: Pushed
4739e510e849: Pushed
95ef25a32043: Pushed
v1.0: digest: sha256:a9097da941d14d997f27446bf9591222f4daca53df6b8c6f6979fba5edbc7686 size: 1160
```

<br/>

#### 2-2. Local docker image 삭제

```sh
# docker rmi jinhokwon/jdk11:v1.0
Untagged: jinhokwon/jdk11:v1.0
Untagged: jinhokwon/jdk11@sha256:a9097da941d14d997f27446bf9591222f4daca53df6b8c6f6979fba5edbc7686
Deleted: sha256:73650d9ebb25a61d738ddf10122412e09f54ae7bc82ab62dc978110ed7aae20a
```

<br/>

#### 2-3. Pull remote docker image

```sh
# docker pull jinhokwon/jdk11:v1.0
v1.0: Pulling from jinhokwon/jdk11
6ec8c9369e08: Already exists
3aa4e9b77806: Already exists
6d8b5d3bc409: Already exists
c52c1e05e8da: Already exists
Digest: sha256:a9097da941d14d997f27446bf9591222f4daca53df6b8c6f6979fba5edbc7686
Status: Downloaded newer image for jinhokwon/jdk11:v1.0
docker.io/jinhokwon/jdk11:v1.0
```

<br/>

#### 2-4. Docker images

```sh
# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
jinhokwon/jdk11     v1.0                73650d9ebb25        12 minutes ago      402MB
```

<br/>