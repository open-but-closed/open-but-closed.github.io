---
title: Redash Oracle Driver 설치 (Docker)
categories: [DW/BI,Redash]
date: Sep 10, 2019 9:00 PM
permalink: redash-add-oracle-driver-with-docker
tags: [Docker,Oracle,Redash]
updated: Oct 05, 2019 1:41 AM
description: Redash에 Oracle을 사용하려면 추가적으로 설정해야 하는 부분을 docker file에 적용해서, 해당 이미지에 아예 적용되어 있도록 수정함
---

## Oracle 드라이버 다운로드

역시 드라이버 다운받기 부터 시작

    $ git clone https://github.com/leonnleite/instant-client-oracle.git
    $ unzip instant-client-oracle.zip

## Dockerfile 변경

/redash 에 있는 Dockerfile 사용하되 일부 수정

- 다운받은 oracle driver와 tnsnames.ora를 추가해줌
- SID방식으로 접근하기 위해선 `redash/query_runner/oracle.py`파일 일부를 변경해줘야 함;;;

코드

    FROM redash/redash:5.0.1.b4850
    
    # apt 할 수 있게 root로 변경
    USER root
    RUN apt-get update  -y
    RUN apt-get install -y unzip
    RUN apt-get install -y libaio-dev libaio1
    RUN apt-get clean -y
    
    # Oracle instant client 설치, TNS파일 추가
    ADD instant-client-oracle/instantclient-basic-linux.x64-12.2.0.1.0.zip /tmp/instantclient-basic-linux.x64-12.2.0.1.0.zip
    ADD tnsnames.ora /tmp/tnsnames.ora
    
    RUN mkdir -p /opt/oracle/
    RUN unzip /tmp/instantclient-basic-linux.x64-12.2.0.1.0.zip -d /opt/oracle/
    
    ENV ORACLE_HOME=/opt/oracle/instantclient_12_2
    ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME
    ENV TNS_ADMIN=$ORACLE_HOME/network/admin
    
    RUN mkdir -p $ORACLE_HOME/network/admin
    RUN cp /tmp/tnsnames.ora $ORACLE_HOME/network/admin/
    
    ENV REDASH_ADDITIONAL_QUERY_RUNNERS=redash.query_runner.oracle
    
    # 추가적으로 requirements_oracle_ds를 실행
    COPY requirements_oracle_ds.txt ./
    
    RUN pip install --upgrade pip
    RUN pip install -r requirements_oracle_ds.txt
    
    # Oracle접속 시 Serivce_name 말고 SID로 접속하고 싶으면 아래 추가 (make_dsn의 파라미터 부분 변경)
    RUN sed -i "84s/service_name=//g" redash/query_runner/oracle.py
    
    USER redash
    
    ENTRYPOINT ["/app/bin/docker-entrypoint"]

## requirements_oracle_ds.txt

cx_Oracle 버전이 5.3.0이었는데, 최신버전도 되는듯 (위 libaio만 잘 맞추면 되는듯)

    # Requires installation of, or similar versions of:
    #    oracle-instantclient12.2-basic_12.2.0.1.0-1_x86_64.rpm
    #    oracle-instantclient12.2-devel_12.2.0.1.0-1_x64_64.rpm
    #cx_Oracle==5.3.0
    cx_Oracle==6.4.1

## docker-compose.yml

    version: '2'
    x-redash-service: &redash-service
      image: redash/redash:5.0.1.b4851o
      depends_on:
        - postgres
        - redis
      env_file: /opt/redash/env
      restart: always
    services:
      server:
        <<: *redash-service
        command: server
        ports:
          - "5000:5000"
          - "1555:1555"
        environment:
          REDASH_WEB_WORKERS: 4
      scheduler:
        <<: *redash-service
        command: scheduler
        environment:
          QUEUES: "celery"
          WORKERS_COUNT: 1
      scheduled_worker:
        <<: *redash-service
        command: worker
        environment:
          QUEUES: "scheduled_queries"
          WORKERS_COUNT: 1
      adhoc_worker:
        <<: *redash-service
        command: worker
        environment:
          QUEUES: "queries"
          WORKERS_COUNT: 2
      redis:
        image: redis:3.0-alpine
        restart: always
      postgres:
        image: postgres:9.5.6-alpine
        env_file: /opt/redash/env
        volumes:
          - /opt/redash/postgres-data:/var/lib/postgresql/data
        restart: always
      nginx:
        image: redash/nginx:latest
        ports:
          - "80:80"
        depends_on:
          - server

## tnsnames.ora

사실 설정 필요 없을듯 (직접 접속 방식이라)

    KBORA.WORLD=
      (DESCRIPTION =
        (ADDRESS_LIST =
            (ADDRESS =
              (PROTOCOL = TCP)
              (HOST = kbora.nohijacking.com)
              (PORT = 1521)
            )
        )
        (CONNECT_DATA =
           (SID = ORCLCDB)
        )
    )

## * 참고 : 접속 테스트

    $ docker exec -it redash_server_1 /bin/bash
    
    redash@be778126bd13:/app$ python
    
    Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
    [GCC 5.4.0 20160609] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>>
    >>> import cx_Oracle
    >>> DB_HOST = "host주소"
    >>> DB_PORT = 1521
    >>> DB_SID = "ORCLCDB"
    >>> DB_USER = "사용자명"
    >>> DB_PWD = "암호"
    >>> dsn_tns = cx_Oracle.makedsn(DB_HOST, DB_PORT, DB_SID)
    >>> conn = cx_Oracle.connect(DB_USER, DB_PWD, dsn_tns)
    >>>