---
title: Google Cloud Platform에 Oracle Database 설치하기 (with Docker)
categories: [Database,Oracle]
date: Mar 11, 2019 10:09 PM
permalink: install-oracle-database-with-docker-on-google-cloud-platform
tags: [Oracle,Docker,GCP]
updated: Mar 11, 2019 10:09 PM
description: Oracle DBMS를 Docker를 통해 설치해보도록 한다.
---

# 개요

Oracle DBMS를 Docker를 통해 설치해보려고 한다. 테스트 용도로 가끔 Oracle을 설치하곤 하는데, Docker로 설치하면 뭔가 관리가 쉽지 않을까?라는 기대에서 한번 설치하게 되었다. (설치할 당시엔 Docker를 거의 사용해보지 않은 상태기도 했고) 

## 그런데 Docker를 사용해서 Oracle을 설치할 필요가 있을까?

Docker를 사용하는 최대 장점은 격리화라는 생각이다. 호스트 OS와 개별 프로그램의 OS를 분리할 수 있다. 그래서 OS 패치등에서 좀더 자유롭고, 프로그램의 OS를 별도로 셋팅이 가능하다. MySQL이나 PostgreSQL등도 간편?한 Docker 이미지가 존재한다.

하지만 Oracle DBMS는 좀 달라 보인다. 단순 테스트 용도라면 좋아 보인다만... 실제 운영에 사용할 DBMS라면 호스트 OS에 직접 설치하는 것이 장점이 많아 보인다. DBMS라는 것은 미들웨어중에서도 OS에 가장 가까운 미들웨어라 OS와 운명을 같이해 보인다. OS를 별도로 업그레이드 하거나, (데이터를 떼어 놓은 채로)DBMS 엔진만을 업그레이드 하는 경우도 없으니 말이다. 

어쨌건 가끔 SQL도 실행해 보고 할 목적으로 만들어 보자.

# Google Cloud VM 생성

## VM 생성

Docker기반으로 설치하려고 마음을 먹었으니, OS 이미지를 아예 Container-Optimized OS로 사용해 보려고 한다. Chromium기반으로 만든 것 같다.

- 지역/영역 - 우리의 친구 도쿄
- CPU/메모리 - 4 vCPUs & 15 GB - 월마다 $147정도 나가니 Free Credit으로는 2달 정도 사용 가능할 것 같다.
- Image - Container-Optimized OS 67 (stable) - 그냥 한번 선택해봤음.
- 디스크 - SSD 100GB (SSD가 과연 필요할까 하면서도...)

![](/images/Capture2018-07-12at6-878f4091-c3b8-4636-b1ee-cfe08143fa2a.10.54PM.jpg)

조금 있으면 잘 생성됨

![](/images/Capture2018-07-12at6-c59b6fa8-dbb5-4175-8089-84b4cc688f5c.11.39PM.jpg)

## OS 셋업

SSH를 눌러 접속하거나, metadata에 SSH 접속정보를 추가해서 접속한다. 들어가보면 이미 Docker가 설치되어 있다.

    $ docker --version
    Docker version 17.03.2-ce, build f5ec1e2

(참고로, Docker 특화되어 있는 리눅스라 docker 관련 툴만 설치되어 있다. 즉, 호스트 OS에는 vi같은 기본 툴도 깔려 있지 않아 사용이 매우 불편했다. ~~나중에는 사용 안할 것임~~)

이후 확인해보니 몇가지 셋업을 더 했었던 것 같다.

- 사용자 생성 : user:`oracle`, group: `oracle`
- 디렉토리 생성 : Oracle관련 디렉토리를 `/home/oracle/oradata`로 설치하려 했던 것 같다. 참고로, 퍼미션?문제가 있어서 `oradata`는 퍼미션을 그냥 777로 주었음;

# Oracle DBMS 설치 (w/ Docker)

## Docker Store에 이미지 확인

Oracle사는 2017년 DockerCon에서 Oracle 12.1 Database Docker 이미지를 발표했다. Docker Store에 보면 Oracle DBMS 이미지가 존재한다. 회원 가입과 인증이 필요하다. Docker Hub는 Github처럼 공개된 이미지가 있는 것 같고, Docker Store는 판매 목적의 이미지가 있는 것으로 스스로 생각해보았다.

- [https://store.docker.com/images/oracle-database-enterprise-edition](https://store.docker.com/images/oracle-database-enterprise-edition)

![](/images/Capture2018-07-12at6-10398338-7abf-4be3-9156-9d24f3f0e0a9.31.13PM.jpg)

~~근데 공식 이미지임에도 평점이 높지 않네요; → 최근 좋아진듯 보임~~

**Log in**을 하고 **Proceed to Checkout**을 하면 각종 개인정보(이메일과 전화번호)를 요구한다. Docker Store에 개인정보를 판매한 후 pull image 명령어 링크를 확인. `docker pull store/oracle/database-enterprise:12.2.0.1`로 되어있는 부분을 복사.

![](/images/Capture2018-07-12at6-fab802f9-ed3f-41ea-bce8-7a5528001d6c.34.45PM.jpg)

말대로 한번 수행해보면...

    $ docker pull store/oracle/database-enterprise:12.2.0.1
    Error response from daemon: repository store/oracle/database-enterprise not found: does not exist or no pull access

음... 에러가;;; 로그인을 해야 한다. `docker login` 이후 id 및 pw를 입력하면 된다.

    $ docker login
    Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker
    .com to create one.
    Username: kangbu 
    Password: 
    Login Succeeded

이미지를 pull 한다.

    $ docker pull store/oracle/database-enterprise:12.2.0.1
    12.2.0.1: Pulling from store/oracle/database-enterprise
    4ce27fe12c04: Pull complete 
    9d3556e8e792: Pull complete 
    fc60a1a28025: Pull complete 
    0c32e4ed872e: Pull complete 
    b465d9b6e399: Pull complete 
    Digest: sha256:40760ac70dba2c4c70d0c542e42e082e8b04d9040d91688d63f728af764a2f5d
    Status: Downloaded newer image for store/oracle/database-enterprise:12.2.0.1

다운로드를 완료하고, 도커 이미지를 확인

    $ docker images
    REPOSITORY                         TAG                 IMAGE ID            CREATED             SIZE
    store/oracle/database-enterprise   12.2.0.1            12a359cd0528        10 months ago       3.44 GB

## Oracle DBMS Docker 컨테이너 생성

설명에 보면 다음을 설정 할 수 있다.

- `DB_SID` Oracle의 SID임. 설정하지 않으면 `ORCLCDB`
- `DB_PDB` Oracle PDB의 이름임. 설정하지 않으면 `ORCLPDB1`
- `DB_MEMORY` Oracle을 구동하기 위한 메모리 사이즈임. SGA와 PGA를 합친 값. 기본 값은 **2GB**
- `DB_DOMAIN` 데이터베이스 서버를 위한 도메인. 기본 값은 `localdomain`

환경변수는 기본 값으로 하고 다른 옵션만 추가

- 컨테이너 이름 : kbora
- 포트 1521, 5500을 host로 노출
- 이미지의 `ORCL`디렉토리를 `/home/oracle/oradata`로 연결

다음과 같이 수행

    $ docker run -d -it --name kbora
    > -P --env-file env \
    > -p 1521:1521 -p 5500:5500 \
    > -v /home/oracle/oradata:/ORCL \
    > store/oracle/database-enterprise:12.2.0.1
   
    ...
    ...

근데 설치되다가 멈췄다?

    CONTAINER ID        IMAGE                                       COMMAND                  CREATED             STATUS                     PORTS               NAMES
    1299793d554a        store/oracle/database-enterprise:12.2.0.1   "/bin/sh -c '/bin/..."   5 minutes ago       Exited (1) 4 minutes ago                       kbora

로그 확인

    $ docker logs kbora

    Setup Oracle Database
    Oracle Database 12.2.0.1 Setup
    Fri Jul 13 10:01:59 UTC 2018
    
    Check parameters ......
    log file is : /home/oracle/setup/log/paramChk.log
    paramChk.sh is done at 0 sec
    
    untar DB bits ......
    log file is : /home/oracle/setup/log/untarDB.log
    untarDB.sh is done at 45 sec
    
    config DB ......
    log file is : /home/oracle/setup/log/configDB.log
    mkdir: cannot create directory '/ORCL/u01': Permission denied
    /home/oracle/setup/configDBora.sh: line 56: cd: /u01/app/oracle/product/12.2.0/dbhome_1/dbs/: No such file or directory
    /home/oracle/setup/configDBora.sh: line 58: initORCLCDB.ora: Permission denied
    /home/oracle/setup/configDBora.sh: line 63: initORCLCDB.ora: Permission denied
    /home/oracle/setup/configDBora.sh: line 64: initORCLCDB.ora: Permission denied
    mkdir: cannot create directory '/u01/app/oracle/diag': File exists
    Fri Jul 13 10:02:44 UTC 2018
    Start Docker DB configuration
    Call configDBora.sh to configure database
    Fri Jul 13 10:02:46 UTC 2018
    Configure DB as oracle user
    Setup Database directories ...
    
    SQL*Plus: Release 12.2.0.1.0 Production on Fri Jul 13 10:02:47 2018
    
    Copyright (c) 1982, 2016, Oracle.  All rights reserved.
    
    ERROR:
    ORA-12547: TNS:lost contact
    
    
    Enter user-name: SP2-0306: Invalid option.
    Usage: CONN[ECT] [{logon|/|proxy} [AS {SYSDBA|SYSOPER|SYSASM|SYSBACKUP|SYSDG|SYSKM|SYSRAC}] [edition=value]]
    where <logon> ::= <username>[/<password>][@<connect_identifier>]
          <proxy> ::= <proxyuser>[<username>][/<password>][@<connect_identifier>]
    Enter user-name: Enter password: 
    ERROR:
    ORA-12547: TNS:lost contact
    
    
    SP2-0157: unable to CONNECT to ORACLE after 3 attempts, exiting SQL*Plus
    update password
    
    Enter password for SYS: 
    
    OPW-00029: Password complexity failed for SYS user : Password must contain at least 8 characters.
    create pdb : ORCLPDB1
    
    SQL*Plus: Release 12.2.0.1.0 Production on Fri Jul 13 10:03:01 2018
    
    Copyright (c) 1982, 2016, Oracle.  All rights reserved.
    
    ERROR:
    ORA-12547: TNS:lost contact
    
    
    Enter user-name: SP2-0306: Invalid option.
    Usage: CONN[ECT] [{logon|/|proxy} [AS {SYSDBA|SYSOPER|SYSASM|SYSBACKUP|SYSDG|SYSKM|SYSRAC}] [edition=value]]
    where <logon> ::= <username>[/<password>][@<connect_identifier>]
          <proxy> ::= <proxyuser>[<username>][/<password>][@<connect_identifier>]
    Enter user-name: SP2-0306: Invalid option.
    Usage: CONN[ECT] [{logon|/|proxy} [AS {SYSDBA|SYSOPER|SYSASM|SYSBACKUP|SYSDG|SYSKM|SYSRAC}] [edition=value]]
    where <logon> ::= <username>[/<password>][@<connect_identifier>]
          <proxy> ::= <proxyuser>[<username>][/<password>][@<connect_identifier>]
    SP2-0157: unable to CONNECT to ORACLE after 3 attempts, exiting SQL*Plus
    Reset Database parameters
    
    SQL*Plus: Release 12.2.0.1.0 Production on Fri Jul 13 10:03:01 2018
    
    Copyright (c) 1982, 2016, Oracle.  All rights reserved.
    
    ERROR:
    ORA-12547: TNS:lost contact
    
    
    Enter user-name: SP2-0306: Invalid option.
    Usage: CONN[ECT] [{logon|/|proxy} [AS {SYSDBA|SYSOPER|SYSASM|SYSBACKUP|SYSDG|SYSKM|SYSRAC}] [edition=value]]
    Configure DB as oracle user
    where <logon> ::= <username>[/<password>][@<connect_identifier>]
          <proxy> ::= <proxyuser>[<username>][/<password>][@<connect_identifier>]
    Enter user-name: Enter password: 
    ERROR:
    ORA-12547: TNS:lost contact
    SP2-0157: unable to CONNECT to ORACLE after 3 attempts, exiting SQL*Plus
    LSNRCTL for Linux: Version 12.2.0.1.0 - Production on 13-JUL-2018 10:03:01
    Copyright (c) 1991, 2016, Oracle.  All rights reserved.
    Starting /u01/app/oracle/product/12.2.0/dbhome_1/bin/tnslsnr: please wait...
    TNSLSNR for Linux: Version 12.2.0.1.0 - Production
    System parameter file is /u01/app/oracle/product/12.2.0/dbhome_1/admin/ORCLCDB/listener.ora
    Log messages written to /u01/app/oracle/diag/tnslsnr/1299793d554a/listener/alert/log.xml
    Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=0.0.0.0)(PORT=1521)))
    Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
    Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=0.0.0.0)(PORT=1521)))
    STATUS of the LISTENER
    ------------------------
    Alias                     LISTENER
    Version                   TNSLSNR for Linux: Version 12.2.0.1.0 - Production
    Start Date                13-JUL-2018 10:03:01
    Uptime                    0 days 0 hr. 0 min. 0 sec
    Trace Level               off
    Security                  ON: Local OS Authentication
    SNMP                      OFF
    Listener Parameter File   /u01/app/oracle/product/12.2.0/dbhome_1/admin/ORCLCDB/listener.ora
    Listener Log File         /u01/app/oracle/diag/tnslsnr/1299793d554a/listener/alert/log.xml
    Listening Endpoints Summary...
      (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=0.0.0.0)(PORT=1521)))
      (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
    The listener supports no services
    The command completed successfully
    DONE!
    Remove password info
    Docker DB configuration is complete !
    ERROR : config DB failed, please check log /home/oracle/setup/log/configDB.log for details!
    tail: cannot open '/u01/app/oracle/diag/rdbms/orclcdb/ORCLCDB/trace/alert_ORCLCDB.log' for reading: No such file or directory
    tail: no files remaining

음 `/ORCL/u01`을 생성 못해서 에러 난듯.

다시시도. oradata 퍼미션을 안바꿨었네;; chmod 777 로 바꾸고 다시 시도 (기존 컨테이너는 삭제함)

    $ docker run -d --name kbora -p 1521:1521 -p 5500:5500 -v /home/oracle/oradata:/ORCL --shm-size="10g" store/oracle/database-enterprise:12.2.0.1
    ...
    ...

정상 작동 함을 확인할 수 있음

    $ docker ps -a
    CONTAINER ID        IMAGE                                       COMMAND                  CREATED             STATUS                            PORTS                                            NAMES
    6ea333d7cdb4        store/oracle/database-enterprise:12.2.0.1   "/bin/sh -c '/bin/..."   3 seconds ago       Up 3 seconds (health: starting)   0.0.0.0:1521->1521/tcp, 0.0.0.0:5500->5500/tcp   kbora

로그 확인

    $ docker logs kbora
    Setup Oracle Database
    Oracle Database 12.2.0.1 Setup
    Fri Jul 13 15:12:07 UTC 2018
    
    Check parameters ......
    log file is : /home/oracle/setup/log/paramChk.log
    paramChk.sh is done at 0 sec
    
    untar DB bits ......
    log file is : /home/oracle/setup/log/untarDB.log
    untarDB.sh is done at 89 sec
    
    config DB ......
    log file is : /home/oracle/setup/log/configDB.log
    Fri Jul 13 15:13:36 UTC 2018
    Start Docker DB configuration
    Call configDBora.sh to configure database
    Fri Jul 13 15:13:39 UTC 2018
    Configure DB as oracle user
    Setup Database directories ...
    
    SQL*Plus: Release 12.2.0.1.0 Production on Fri Jul 13 15:13:39 2018
    
    Copyright (c) 1982, 2016, Oracle.  All rights reserved.
    
    Connected to an idle instance.
    
    SQL>
    File created.
    
    SQL> ORACLE instance started.
    
    Total System Global Area 1342177280 bytes
    Fixed Size		    8792536 bytes
    Variable Size		  352323112 bytes
    Database Buffers	  973078528 bytes
    Redo Buffers		    7983104 bytes
    Database mounted.
    Database opened.
    SQL>
    Database altered.
    
    SQL>
    NAME				     TYPE	 VALUE
    ------------------------------------ ----------- ------------------------------
    spfile				     string	 /u01/app/oracle/product/12.2.0
    						 /dbhome_1/dbs/spfileORCLCDB.or
    						 a
    SQL>
    NAME				     TYPE	 VALUE
    ------------------------------------ ----------- ------------------------------
    encrypt_new_tablespaces 	     string	 CLOUD_ONLY
    SQL>
    User altered.
    
    SQL>
    User altered.
    
    SQL> Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
    update password
    
    Enter password for SYS:
    create pdb : ORCLPDB1
    
    SQL*Plus: Release 12.2.0.1.0 Production on Fri Jul 13 15:14:32 2018
    
    Copyright (c) 1982, 2016, Oracle.  All rights reserved.
    
    
    Connected to:
    Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
    
    SQL>   2    3    4    5
    Pluggable database created.
    
    SQL>
    Pluggable database altered.
    
    SQL>
    Pluggable database altered.
    
    SQL> Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
    Reset Database parameters
    
    SQL*Plus: Release 12.2.0.1.0 Production on Fri Jul 13 15:14:54 2018
    
    Copyright (c) 1982, 2016, Oracle.  All rights reserved.
    
    
    Connected to:
    Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
    
    SQL>
    System altered.
    
    SQL> Disconnected from Oracle Database 12c Enterprise Edition Release 12.2.0.1.0 - 64bit Production
    
    LSNRCTL for Linux: Version 12.2.0.1.0 - Production on 13-JUL-2018 15:14:54
    
    Copyright (c) 1991, 2016, Oracle.  All rights reserved.
    
    Starting /u01/app/oracle/product/12.2.0/dbhome_1/bin/tnslsnr: please wait...
    
    TNSLSNR for Linux: Version 12.2.0.1.0 - Production
    System parameter file is /u01/app/oracle/product/12.2.0/dbhome_1/admin/ORCLCDB/listener.ora
    Log messages written to /u01/app/oracle/diag/tnslsnr/6ea333d7cdb4/listener/alert/log.xml
    Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=0.0.0.0)(PORT=1521)))
    Listening on: (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
    
    Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=0.0.0.0)(PORT=1521)))
    STATUS of the LISTENER
    ------------------------
    Alias                     LISTENER
    Version                   TNSLSNR for Linux: Version 12.2.0.1.0 - Production
    Start Date                13-JUL-2018 15:14:54
    Uptime                    0 days 0 hr. 0 min. 0 sec
    Trace Level               off
    Security                  ON: Local OS Authentication
    SNMP                      OFF
    Listener Parameter File   /u01/app/oracle/product/12.2.0/dbhome_1/admin/ORCLCDB/listener.ora
    Listener Log File         /u01/app/oracle/diag/tnslsnr/6ea333d7cdb4/listener/alert/log.xml
    Listening Endpoints Summary...
      (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=0.0.0.0)(PORT=1521)))
      (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
    The listener supports no services
    The command completed successfully
    
    DONE!
    Remove password info
    Docker DB configuration is complete !
    configDB.sh is done at 167 sec
    
    Done ! The database is ready for use .
    # ===========================================================================
    # == Add below entries to your tnsnames.ora to access this database server ==
    # ====================== from external host =================================
    ORCLCDB=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address>)(PORT=<port>))
        (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLCDB.localdomain)))
    ORCLPDB1=(DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=<ip-address>)(PORT=<port>))
        (CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCLPDB1.localdomain)))
    #
    #ip-address : IP address of the host where the container is running.
    #port       : Host Port that is mapped to the port 1521 of the container.
    #
    # The mapped port can be obtained from running "docker port <container-id>"
    # ===========================================================================
    ORCLPDB1(3):Database Characterset for ORCLPDB1 is AL32UTF8
    ORCLPDB1(3):Opatch validation is skipped for PDB ORCLPDB1 (con_id=0)
    2018-07-13T15:14:54.478929+00:00
    ORCLPDB1(3):Opening pdb with no Resource Manager plan active
    Pluggable database ORCLPDB1 opened read write
    Completed:     alter pluggable database ORCLPDB1 open
        alter pluggable database all save state
    Completed:     alter pluggable database all save state
    2018-07-13T15:14:54.744338+00:00
    ALTER SYSTEM SET encrypt_new_tablespaces='DDL' SCOPE=BOTH;
    2018-07-13T15:15:08.494065+00:00
    Thread 1 advanced to log sequence 5 (LGWR switch)
      Current log# 2 seq# 5 mem# 0: /u04/app/oracle/redo/redo002.log
    2018-07-13T15:15:09.496693+00:00
    TABLE SYS.WRP$_REPORTS: ADDED INTERVAL PARTITION SYS_P287 (3116) VALUES LESS THAN (TO_DATE(' 2018-07-14 01:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    TABLE SYS.WRP$_REPORTS_DETAILS: ADDED INTERVAL PARTITION SYS_P288 (3116) VALUES LESS THAN (TO_DATE(' 2018-07-14 01:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))
    TABLE SYS.WRP$_REPORTS_TIME_BANDS: ADDED INTERVAL PARTITION SYS_P291 (3115) VALUES LESS THAN (TO_DATE(' 2018-07-13 01:00:00', 'SYYYY-MM-DD HH24:MI:SS', 'NLS_CALENDAR=GREGORIAN'))

`oradata`디렉토리를 확인해보면

    $ cd oradata
    $ ls -al
    total 24
    drwxrwxrwx 6 oracle oracle 4096 Jul 13 15:12 .
    drwxr-xr-x 4 oracle oracle 4096 Jul 13 15:39 ..
    drwxr-xr-x 3  54321  54321 4096 Aug  8  2017 u01
    drwxr-xr-x 3  54321  54321 4096 Aug  8  2017 u02
    drwxr-xr-x 3  54321  54321 4096 Aug  8  2017 u03
    drwxr-xr-x 3  54321  54321 4096 Aug  8  2017 u04

각각 내용을 확인해 보면

- u01 : dbs (init.ora)
- u02 : audit, oradata (dbs files)
- u03 : fast_recovery_area (archivelog, controlfile)
- u04 : redo (redofile)

처럼 저장된다.

각각의 파티션을 나눠 사이즈를 지정하는 방식도 있는듯. [이곳](https://blog.purestorage.com/docker-oracle-12c-and-persistent-storage/) 링크에 잘 나와있는 듯 하다.

## SQL Plus 접속 테스트

다음 명령어를 이용 `docker exec -it kbora bash -c "source /home/oracle/.bashrc; sqlplus /nolog"` 

    $ docker exec -it kbora bash -c "source /home/oracle/.bashrc; sqlplus /nolog"
    
    SQL*Plus: Release 12.2.0.1.0 Production on Sat Jul 14 17:08:38 2018
    
    Copyright (c) 1982, 2016, Oracle.  All rights reserved.
    
    SQL>

일단 sys의 비밀번호부터 변경

    SQL> conn /as sysdba
    Connected.
    SQL> alter user sys identified by <new-password>;
    User altered.
    SQL> show sga
    
    Total System Global Area 1342177280 bytes
    Fixed Size		    8792536 bytes
    Variable Size		  570426920 bytes
    Database Buffers	  754974720 bytes
    Redo Buffers		    7983104 bytes

# 기타 설정

## Oracle net 방화벽 열기

같은 프로젝트에서 사용 방화벽은 문제 없는데, 외부에서 접속하려면 VPC 설정에서 포트를 열어줘야 한다.

**VPC network** > **Firewall rules** 

![](/images/Capture2018-08-03at18-42d45501-0f42-4702-aaea-290bb84be515.52.31.jpg)

`CREATE FIREWALL RULE` 버튼 누르고 설정하면 됨

- Name : 룰 명
- Priority : 우선순위. 작을 수록 먼저 적용됨.
- Direction of Traffic : 방향. 들어오는 것이니 Ingress로
- Action on match : 해당하는 룰에 맞을 때 어떻게 할지. Allow로
- Targets
    - All instances in the network : 본 네트워크 전체에 대해
    - Specified target tags : 특정 tag에 대해
    - Specified service account : 특정 account 기준
- Source filter : IP ranges
- Source IP range : 0.0.0.0/0 (전체)
- Protocols and ports : Specified protocols and ports 로 하고 **tcp:1521**

![](/images/Capture2018-08-03at19-96a70e1a-fe94-439a-a4b8-1d7030871d9b.06.07.jpg)

다 입력하고 Create 버튼을 눌러주면 됨

## 컨테이너 OS 셋업

한참 쓰다보니 시간대가 안맞는다=_=; Docker를 잘 모를때라 호스트OS 설정을 계속 바꾸다 망했는데; 아마 컨테이너 구동 시 -e 환경변수로 넣어줘야 하는것 같은데, 컨테이너 OS에 직접 들어가서 수정하니 잘 작동 되어서(?) 그냥 쓰고 있다. (근데 왜 NLS_PARAMETER로 수정하지 않았었을까?)

즉,

1. `docker exec -it bash` 로 진입
2. /home/oracle의 .bashrc 에 지역 추가

        export TZ='Asia/Seoul'

3. 변경했으면 DB 재시작

## 참조

- [ORACLE DATABASE ON THE DOCKER STORE](http://gokhanatil.com/2017/04/oracle-database-on-the-docker-store.html)
- [Deploy Oracle Database as a Docker Container in Oracle Container Cloud Service](http://www.oracle.com/technetwork/articles/cloudcomp/deploy-database-in-container-cloud-3876722.html)
- [Oracle Brings Oracle’s Flagship Databases and Developer Tools to the Docker Store](https://www.oracle.com/corporate/pressrelease/docker-oracle-041917.html) : DockerCon 2017에서 Oracle 12.1 을 하겠다는 발표
- [DOCKER, ORACLE 12C AND PERSISTENT STORAGE](https://blog.purestorage.com/docker-oracle-12c-and-persistent-storage/)