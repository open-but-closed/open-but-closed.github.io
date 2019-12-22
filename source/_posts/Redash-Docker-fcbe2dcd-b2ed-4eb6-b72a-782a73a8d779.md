---
title: Redash Docker로 이미지 생성
categories: [DW/BI,Redash]
date: Aug 31, 2018 11:21 PM
permalink: remake-redash-docker-images
tags: [Docker,Redash]
updated: Oct 05, 2019 1:55 AM
description: 결국 Redash는 docker버전으로 관리하기로 했다. 또 내부망 서버에는 인터넷이 제한적인 환경이라 docker로 관리하는 부분이 더 맞을듯. 외부에서 모든 셋업을 마치고 docker image를 만들어서 내부의 docker에 설치해보려고 하다.
---

# 전체순서

- [https://redash.io/help/open-source/dev-guide/docker](https://redash.io/help/open-source/dev-guide/docker)
    1. Docker, Docker compose 설치
    2. Node.js 설치

## Docker-CE 설치

- [http://nobase-dev.tistory.com/34](http://nobase-dev.tistory.com/34)

        ## 패키지 업데이트
        $ sudo apt-get update
        ...
        ## HTTPS를 통해 repository를 사용할 수 있도록 패키지 설치
        $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
        ...
        ## Docker 공식 GPG 키 추가
        ## Key fingerprint 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88 확인
        $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
        $ sudo apt-key fingerprint 0EBFCD88
        pub   4096R/0EBFCD88 2017-02-22
        Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
        uid                  Docker Release (CE deb) <docker@docker.com>
        sub   4096R/F273FCD8 2017-02-22
        ...
        ## stable repository 사용시 아래 커맨드 추가
        ## arch 값을 amd64, armhf, ppc64el, s390x 등으로 맞춰서 사용
        $ sudo add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
           $(lsb_release -cs) \
           stable"
        ...
        ## 패키지 업데이트
        $ sudo apt-get update
        ...
        ## 옵션 1. Docker CE 최신 버전 설치
        $ sudo apt-get install docker-ce
        ...
        ## 옵션 2. Docker CE 특정 버전 설치
        $ apt-cache madison docker-ce
        docker-ce | 17.12.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
        $ sudo apt-get install docker-ce=<VERSION>
        ...
        ## hello-world 이미지를 실행하여 Docker CE가 올바르게 설치되었는지 확인
        $ sudo docker run hello-world

## Docker Composer 설치

    root@redash-003:~# docker-compose -v
    The program 'docker-compose' is currently not installed. You can install it by typing:
    apt install docker-compose
    root@redash-003:~# apt install docker-compose
    ...
    ...
    root@redash-003:~# docker-compose -v
    docker-compose version 1.8.0, build unknown

## Node.js, npm 설치

    root@redash-003:~# sudo apt-get install nodejs
    ...
    ...
    root@redash-003:~# sudo apt-get install npm
    ...
    ...
    root@redash-003:~# nodejs -v
    v4.2.6

## redash 소스 다운로드

    root@redash-003:~# git clone https://github.com:getredash/redash.git
    Cloning into 'redash'...
    remote: Not Found
    fatal: repository 'https://github.com:getredash/redash.git/' not found
    root@redash-003:~# git clone https://github.com/getredash/redash.git
    Cloning into 'redash'...
    remote: Counting objects: 34272, done.
    remote: Compressing objects: 100% (2/2), done.
    remote: Total 34272 (delta 0), reused 0 (delta 0), pack-reused 34270
    Receiving objects: 100% (34272/34272), 11.01 MiB | 5.79 MiB/s, done.
    Resolving deltas: 100% (24142/24142), done.
    Checking connectivity... done.
    root@redash-003:~# cd redash

## docker-compose 실행

Dev환경 파일`docker-compose.yml`을 이용할 경우 : 왠지 잘 설치는 되고 돌아가는듯 보이지만 잘 안됨ㅠㅠ 

그래서 Prod환경 파일`docker-compose.production.yml`을 이용 : 잘 됨

- Postgres 볼륨 위치

        postgres:
            image: postgres:9.5.6-alpine
            volumes:
              - "/opt/postgres-data:/var/lib/postgresql/data"
            restart: always

- `REDASH_COOKIE_SECRET` 변경

        server:
            image: redash/redash:latest
            command: server
            depends_on:
              - postgres
              - redis
            ports:
              - "5000:5000"
            environment:
              PYTHONUNBUFFERED: 0
              REDASH_LOG_LEVEL: "INFO"
              REDASH_REDIS_URL: "redis://redis:6379/0"
              REDASH_DATABASE_URL: "postgresql://postgres@postgres/postgres"
              REDASH_COOKIE_SECRET: <<modified>>
              REDASH_WEB_WORKERS: 4

최종 수행

    root@redash-003:~/redash# docker-compose -f docker-compose.production.yml run --rm server create_db
    Creating redash_redis_1
    Creating redash_postgres_1
    Pulling server (redash/redash:latest)...
    latest: Pulling from redash/redash
    23a6960fe4a9: Pull complete
    e9e104b0e69d: Pull complete
    cd33d2ea7970: Pull complete
    534ff7b7d120: Pull complete
    7d352ac0c7f5: Pull complete
    c3c5a6207a0b: Pull complete
    554d68c17a7f: Pull complete
    17eadc3a433b: Pull complete
    20010883713d: Pull complete
    af7c42e7bf80: Pull complete
    3b4147361b03: Pull complete
    696785a1d19b: Pull complete
    0df8c7aae127: Pull complete
    ef1a3b509125: Pull complete
    a3ed95caeb02: Pull complete
    Digest: sha256:ec3fa4b94cef267f89ef1d9fde7d37facd59de58ea7c60b073226c12aa7dab4c
    Status: Downloaded newer image for redash/redash:latest
    [2018-08-22 06:41:25,332][PID:1][INFO][root] Generating grammar tables from /usr/lib/python2.7/lib2to3/Grammar.txt
    [2018-08-22 06:41:25,364][PID:1][INFO][root] Generating grammar tables from /usr/lib/python2.7/lib2to3/PatternGrammar.txt
    [2018-08-22 06:41:28,863][PID:1][INFO][alembic.runtime.migration] Context impl PostgresqlImpl.
    [2018-08-22 06:41:28,865][PID:1][INFO][alembic.runtime.migration] Will assume transactional DDL.
    [2018-08-22 06:41:28,900][PID:1][INFO][alembic.runtime.migration] Running stamp_revision  -> 969126bd800f

## 작업확인

    root@redash-003:~/redash# docker ps -a
    CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                         NAMES
    710d03e179a6        redash/nginx:latest     "nginx -g 'daemon of…"   44 seconds ago      Up 42 seconds       0.0.0.0:80->80/tcp, 443/tcp   redash_nginx_1
    7477d7fb3453        redash/redash:latest    "/app/bin/docker-ent…"   45 seconds ago      Up 43 seconds       0.0.0.0:5000->5000/tcp        redash_server_1
    b88528c588f2        redash/redash:latest    "/app/bin/docker-ent…"   45 seconds ago      Up 43 seconds       5000/tcp                      redash_worker_1
    a01fd9887ef8        postgres:9.5.6-alpine   "docker-entrypoint.s…"   13 minutes ago      Up 13 minutes       5432/tcp                      redash_postgres_1
    9cbb150b3d43        redis:3.0-alpine        "docker-entrypoint.s…"   13 minutes ago      Up 13 minutes       6379/tcp                      redash_redis_1

# Docker Image로 생성

링크 : [http://www.kwangsiklee.com/2017/10/redash-docker-설치/](http://www.kwangsiklee.com/2017/10/redash-docker-%EC%84%A4%EC%B9%98/) 

를 보면, docker 설치 후 이미지 추출하여 추후 사고?에 대비하는 것 같은데...

    root@redash-003:~/backup# docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    redash_server       latest              29d3dd8312e9        24 hours ago        1.07GB
    redash_worker       latest              29d3dd8312e9        24 hours ago        1.07GB
    <none>              <none>              f0743943bac3        26 hours ago        1.07GB
    redash/redash       latest              bec561ff0cc0        3 months ago        1.07GB
    redis               3.0-alpine          856249f48b0c        14 months ago       12.6MB
    redash/base         latest              9d04584f371a        14 months ago       566MB
    postgres            9.5.6-alpine        cc38b642ca58        15 months ago       36.9MB
    redash/nginx        latest              76abf32984e9        2 years ago         134MB
    root@redash-003:~/backup# docker ps -a
    CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                         NAMES
    710d03e179a6        redash/nginx:latest     "nginx -g 'daemon of…"   24 hours ago        Up 24 hours         0.0.0.0:80->80/tcp, 443/tcp   redash_nginx_1
    7477d7fb3453        redash/redash:latest    "/app/bin/docker-ent…"   24 hours ago        Up 24 hours         0.0.0.0:5000->5000/tcp        redash_server_1
    b88528c588f2        redash/redash:latest    "/app/bin/docker-ent…"   24 hours ago        Up 24 hours         5000/tcp                      redash_worker_1
    a01fd9887ef8        postgres:9.5.6-alpine   "docker-entrypoint.s…"   24 hours ago        Up 24 hours         5432/tcp                      redash_postgres_1
    9cbb150b3d43        redis:3.0-alpine        "docker-entrypoint.s…"   24 hours ago        Up 24 hours         6379/tcp                      redash_redis_1
    
    
    root@redash-003:~/backup# docker save redash/redash > redash.tar
    root@redash-003:~/backup# ls -al
    total 1078672
    drwxr-xr-x 2 root root       4096 Aug 23 06:35 .
    drwx------ 9 root root       4096 Aug 23 06:35 ..
    -rw-r--r-- 1 root root 1104546816 Aug 23 06:48 redash.tar
    
    
    root@redash-003:~/backup# docker load < redash.tar
    Loaded image: redash/redash:latest
    
    
    root@redash-003:~/backup# docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    redash_server       latest              29d3dd8312e9        25 hours ago        1.07GB
    redash_worker       latest              29d3dd8312e9        25 hours ago        1.07GB
    <none>              <none>              f0743943bac3        26 hours ago        1.07GB
    redash/redash       latest              bec561ff0cc0        3 months ago        1.07GB
    redis               3.0-alpine          856249f48b0c        14 months ago       12.6MB
    redash/base         latest              9d04584f371a        14 months ago       566MB
    postgres            9.5.6-alpine        cc38b642ca58        15 months ago       36.9MB
    redash/nginx        latest              76abf32984e9        2 years ago         134MB
    root@redash-003:~/backup# docker ps -a
    CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                         NAMES
    710d03e179a6        redash/nginx:latest     "nginx -g 'daemon of…"   24 hours ago        Up 24 hours         0.0.0.0:80->80/tcp, 443/tcp   redash_nginx_1
    7477d7fb3453        redash/redash:latest    "/app/bin/docker-ent…"   24 hours ago        Up 24 hours         0.0.0.0:5000->5000/tcp        redash_server_1
    b88528c588f2        redash/redash:latest    "/app/bin/docker-ent…"   24 hours ago        Up 24 hours         5000/tcp                      redash_worker_1
    a01fd9887ef8        postgres:9.5.6-alpine   "docker-entrypoint.s…"   24 hours ago        Up 24 hours         5432/tcp                      redash_postgres_1
    9cbb150b3d43        redis:3.0-alpine        "docker-entrypoint.s…"   24 hours ago        Up 24 hours         6379/tcp                      redash_redis_1

걍 백업의 개념인가;;