---
title: Dockerfile 명령어 요약
tags: [devops, docker]
comments: true
categories: devops
header:
  teaser: "/assets/images/docker/docker_logo.png"
---
본 문서에서는 Dockerfile 에서 자주 사용하는 명령어들에서 설명하고 있습니다.
<br/>







#### 1-1. FROM

어떤 이미지를 기반으로 이미지를 생성할지 베이스 이미지를 설정합니다.

```dockerfile
FROM centos:7
```

<br/>

#### 1-2. MAINTAINER

Dockerfile을 관리하는 담당자의 정보를 기록합니다.

```dockerfile
MAINTAINER Jinhokwon <Jinhokwon@gmail.com>
```

<br/>





#### 1-3. COPY

로컬 파일을 이미지 내부로 복사합니다. 

```dockerfile
COPY README.md /README.md
```

<br/>



#### 1-4. ADD

로컬 파일을 이미지 내부로 복사합니다.

이 때, 로컬 파일 형식이 압축 파일인 경우, 압축을 해제하고 풀어서 복사합니다.

```dockerfile
ADD hello.tgz / 
```

<br/>



#### 1-5. RUN

dockerfile 빌드 시점에 실행될 명령어를 기록합니다. 

```dockerfile
FROM openjdk:11-jdk-slim
RUN echo "aaa"
RUN echo "bbb"
ENTRYPOINT ["java", "-version"]
```

이 때, 빌드시점에 실행되었던 `RUN` 명령어 만큼 이미지 레이어가 생성이 되며, <br/>

생성된 레이어 정보는 `docker history` 명령어로 조회 할 수 있습니다.

```sh
# docker build --no-cache . -t jinhokwon/jdk11:v1.0

# docker history jinhokwon/jdk11:v1.0
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
71e42e1d7dcb        2 minutes ago       /bin/sh -c #(nop)  ENTRYPOINT ["java" "-vers…   0B
0400d89dbb93        2 minutes ago       /bin/sh -c echo "bbb"                           0B
b401a8287e35        2 minutes ago       /bin/sh -c echo "aaa"                           0B
3b3ff3a752cf        4 days ago          /bin/sh -c #(nop)  CMD ["jshell"]               0B
<missing>           4 days ago          /bin/sh -c set -eux;   dpkgArch="$(dpkg --pr…   324MB
<missing>           4 days ago          /bin/sh -c #(nop)  ENV JAVA_URL_VERSION=11.0…   0B
<missing>           4 days ago          /bin/sh -c #(nop)  ENV JAVA_BASE_URL=https:/…   0B
<missing>           4 days ago          /bin/sh -c #(nop)  ENV JAVA_VERSION=11.0.8      0B
<missing>           4 days ago          /bin/sh -c { echo '#/bin/sh'; echo 'echo "$J…   27B
<missing>           4 days ago          /bin/sh -c #(nop)  ENV PATH=/usr/local/openj…   0B
<missing>           4 days ago          /bin/sh -c #(nop)  ENV JAVA_HOME=/usr/local/…   0B
<missing>           4 days ago          /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B
<missing>           4 days ago          /bin/sh -c set -eux;  apt-get update;  apt-g…   8.78MB
<missing>           5 days ago          /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>           5 days ago          /bin/sh -c #(nop) ADD file:6ccb3bbcc69b0d44c…   69.2MB
```

<br/>



#### 1-6. VOLUME

호스트와 공유할 디렉토리를 설정합니다. 

```dockerfile
VOLUME /tmp
```

<br/>

#### 1-7. EXPOSE

외부로 노출할 포트번호를 설정합니다.

```dockerfile
EXPOSE 8080
```

<br/>

#### 1-8. ENV

도커 컨테이너 환경변수를 설정합니다.

```dockerfile
ENV myEnv myVal
```



<br/>

#### 1-9. LABEL

도커 이미지의 라벨 정보를 설정합니다.

```sh
LABEL author="JinhoKwon"
```

이렇게 설정된 라벨 정보는 `docker inspect` 명령어로 확인이 가능합니다.

##### 전체 라벨을 확인하는 경우

![dockerfile_label_config_labels](/assets/images/docker/dockerfile_label_config_labels.png)

##### 특정 라벨을 확인하는 경우

![dockerfile_label_config_labels_author](/assets/images/docker/dockerfile_label_config_labels_author.png)



<br/>

#### 1-10. ENTRYPOINT

컨테이너가 실행하는 시점에 실행될 명령어를 설정합니다.

Dockerfile 내에서 단 한번만 설정할 수 있습니다.

```dockerfile
FROM openjdk:11-jdk-slim
ENTRYPOINT ["echo", "first"]
```



##### 이미지 빌드후, 실행결과

```sh
# docker build --no-cache . -t jinhokwon/jdk11:v1.0
# docker run -i -t -d --name jdk11 jinhokwon/jdk11:v1.0 second third
# docker logs jdk11
first second third
```





<br/>



#### 1-11. CMD

컨테이너가 실행하는 시점에 실행될 명령어를 설정합니다.

Dockerfile 내에서 여러번 설정할 수 있지만, <br/>

실제로는 가장 마지막 설정된 `CMD` 명령어만 실행됩니다.

```dockerfile
FROM openjdk:11-jdk-slim
CMD echo first second
CMD echo param1 param2
```

<br/>

##### 이미지 빌드후, 실행결과 확인

```sh
# docker build --no-cache . -t jinhokwon/jdk11:v1.0
# docker run -i -t -d --name jdk11 jinhokwon/jdk11:v1.0
# docker logs jdk11
param1 param2
```

<br/>

##### CMD를 오버라이드 하여 실행한 경우

```sh
# docker rm jdk11
# docker run -i -t -d --name jdk11 jinhokwon/jdk11:v1.0 echo param3 param4
# docker logs jdk11
param3 param4
```

<br/>

#### 1-12. ENTRYPOINT와 CMD가 함께 사용되는 경우

ENTRYPOINT와 CMD가 함께 사용되는 경우,

CMD에 입력된 값들은 ENTRYPOINT의 파라미터로 설정이 됩니다.

```dockerfile
FROM openjdk:11-jdk-slim
ENTRYPOINT ["echo", "first"]
CMD ["second", "third"]
```

##### 이미지 빌드후, 실행결과 확인

```sh
# docker build --no-cache . -t jinhokwon/jdk11:v1.0
# docker run -i -t -d --name jdk11 jinhokwon/jdk11:v1.0
# docker logs jdk11
first second third
```

<br/>

#### 1-13. ARG

Dockerfile 빌드 시점에 사용할 파라미터를 설정합니다.

다음 예제는 `--build-arg` 로 전달받은 NAME값을 ENTRYPOINT로 다시 전달하는 예제입니다.

```dockerfile
FROM openjdk:11-jdk-slim
ARG NAME
ENV NAME ${NAME}
ENTRYPOINT echo "welcome ${NAME}"
```

이 때, ENTRYPOINT를 SHELL 방식으로 사용해야 `${NAME}`이 올바르게 치환됩니다.<br/>

##### 이미지 빌드후, 실행결과 확인

```sh
# docker build --build-arg NAME=JinhoKwon --no-cache . -t jinhokwon/jdk11:v1.0
# docker run -i -t -d --name jdk11 jinhokwon/jdk11:v1.0
# docker logs jdk11
welcome JinhoKwon
```

