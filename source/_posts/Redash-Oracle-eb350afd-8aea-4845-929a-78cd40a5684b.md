---
title: Redash에 Oracle 가능하도록 설정
categories: [DW/BI,Redash]
date: Sep 12, 2018 9:18 PM
permalink: enable-oracle-dbms-on-redash
tags: [Oracle,Redash]
updated: Oct 05, 2019 1:14 AM
description: Oracle DBMS의 경우 접속 드라이버를 다른 프로그램에서 배포하지 못하게 하는 것으로 알고 있다. 대부분 프로그램을 다운받고, 오라클 드라이버는 오라클 공홈에서 다운받아야 한다. Redash도 그래서인지 Oracle기능은 disable된 상태로 배포가 되는데, 이를 사용할 수 있도록 변경하는 방법을 연구
---

# Redash는 설치환경에 따라 셋팅 방법이 다름

- 환경을 셋팅하여 dockerfile로 생성
- provisioning script로 생성된 곳에 직접 셋팅

여기서는 provisioning script로 생성된 환경에 셋업

## 전체방법

- redash는 python으로 만들어진 앱이라, 결국 cx_Oracle 패키지로 oracle에 연결하는 것임.
- oracle instant client로 cx_Oracle을 사용할 수 있도록 하면 됨
- 그 외에 설정할 것은 그리 많지 않음... (과연)

# Oracle instant client 설치

## 드라이버 다운로드

오라클 드라이버를 다운받으려면 로그인도 하고 귀찮으니 누가 git에 올려놓은 것을 활용

    $ git clone https://github.com/leonnleite/instant-client-oracle.git

cx_Oracle에 대한 installation은 다음 링크를 참조하면 된다:

[https://cx-oracle.readthedocs.io/en/latest/installation.html](https://cx-oracle.readthedocs.io/en/latest/installation.html)

> If your database is on a remote computer, then download the free Oracle Instant Client “Basic” or “Basic Light” package for your operating system architecture. Use the RPM or ZIP packages, based on your preferences.

결국 basic만 있으면 됨

    $ mkdir /opt/oracle
    $ unzip instantclient-basic-linux.x64-12.2.0.1.0.zip -d /opt/oracle

## 환경셋팅

중간에 잘 안되어서 다음과 같은 셋팅도 어디서 보고 하긴 했음. 버전별로 파일이 다를 수 있으니 심볼릭 링크로 버전 파일을 맞춰줘야 함

    $ cd /opt/oracle
    
    # 심볼릭 링크 생성
    # 버전에 대한 심볼릭 링크이므로 버전 잘 확인(이것때문에 고생ㅠㅠㅠ)
    $ ln -s /opt/oracle/instantclient_12_2 /opt/oracle/instantclient
    $ ln -s /opt/oracle/instantclient/libclntsh.so.12.1 /opt/oracle/instantclient/libclntsh.so
    
    # 환경변수 (LD_LIBRARY_PATH까지 해주면 됨...)
    $ export ORACLE_HOME=/opt/oracle/instantclient
    $ export LD_LIBRARY_PATH=/opt/oracle/instantclient
    $ export TNS_ADMIN=$ORACLE_HOME/network/admin
    $ mkdir -p /opt/oracle/instantclient/network/admin
    

뭔가 파일 연결이 잘 안되어서... 검색으로 다음 해결방안을 찾음

    $ echo /opt/oracle/instantclient_12_2 > /etc/ld.so.conf.d/oracle-instantclient.conf
    $ ldconfig

# Python Oracle connector(cx_Oracle) 설치

Redash는 Python프로그램이라, connector를 위해 cx_Oracle 패키지를 설치한다. 설명버전으로는 패키지 5.3으로 하라고 했는데 잦은 에러가 발생해서 그냥 최신(6.4.1)로 해보니 정상적으로 동작한다;;

## pip로 cx_Oracle설치

    # cx_Oracle 설치
    # 과거에는 특정버전 (==5.3)으로 하라고 했는데, 오히려 과거버전 설치 시 에러나서 안됨
    # 오히려 최신버전 (==6.4.1)으로 하니, 실제 프로그램에서도 에러 안나고 잘됨
    $ pip install cx_Oracle
    Collecting cx_Oracle
      Using cached https://files.pythonhosted.org/packages/3b/09/6b10675a6db7c7da1b8d23225f0a95b2a45248c56a1e8f711d59809278d3/cx_Oracle-6.4.1-cp27-cp27mu-manylinux1_x86_64.whl
    Installing collected packages: cx-Oracle
    Successfully installed cx-Oracle-6.4.1
    
    # 관련 패키지가 있어야 하나바
    $ apt install libaio1 libaio-dev

## 접속 테스트

    $ python
    Python 2.7.12 (default, Dec  4 2017, 14:50:18)
    [GCC 5.4.0 20160609] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>>
    >>> import cx_Oracle
    >>> DB_HOST = "<<host주소>>"
    >>> DB_PORT = 1521
    >>> DB_SID = "ORCLCDB"
    >>> DB_USER = "<<계정입력>>"
    >>> DB_PWD = "<<암호입력>>"
    >>> dsn_tns = cx_Oracle.makedsn(DB_HOST, DB_PORT, DB_SID)
    >>> conn = cx_Oracle.connect(DB_USER, DB_PWD, dsn_tns)
    >>>
    

쿼리 날리는것 까지는 생략

cx_Oracle.makedns의 경우 : [https://cx-oracle.readthedocs.io/en/latest/module.html#cx_Oracle.makedsn](https://cx-oracle.readthedocs.io/en/latest/module.html#cx_Oracle.makedsn)

> cx_Oracle.makedsn(host, port, sid=None, service_name=None, region=None, sharding_key=None, super_sharding_key=None)

이러니, sid=로 하면 됨

# redash 추가 수정 부분

## 환경변수 (.env)에 Oracle 설정

Oracle을 사용하려면 `REDASH_ADDITIONAL_QUERY_RUNNERS`을 추가로 셋업하도록 되어 있음

    $ cd /opt/redash
    
    # 맨 마지막줄 ADDITIONAL_QUERY_RUNNERS임
    $ cat .env
    export REDASH_LOG_LEVEL="INFO"
    export REDASH_REDIS_URL=redis://localhost:6379/0
    export REDASH_DATABASE_URL="postgresql:///redash"
    export REDASH_COOKIE_SECRET=ngbGYiCcIIrmSm9gr52LBpA7jSi7PpDe
    export REDASH_ADDITIONAL_QUERY_RUNNERS=redash.query_runner.oracle

## `oracle.py`소스 일부 수정

원래 화면이 다음과 같음

![](/images/Capture2018-09-12at11-0f3f4803-8f47-4859-a359-99412b2b1981.25.27.jpg)

DSN Service Name에 Service name이 들어가야 하는데, SID가 들어가니 접속이 안되는 것임. 아...

이거 ㅠㅠ. 그래서 그냥 소스를 수정

[https://github.com/getredash/redash/blob/master/redash/query_runner/oracle.py](https://github.com/getredash/redash/blob/master/redash/query_runner/oracle.py)

    class Oracle(BaseSQLQueryRunner):
    
    ...
    
        @classmethod
        def configuration_schema(cls):
            return {
                "type": "object",
                "properties": {
                    "user": {
                        "type": "string"
                    },
                    "password": {
                        "type": "string"
                    },
                    "host": {
                        "type": "string"
                    },
                    "port": {
                        "type": "number"
                    },
                    "servicename": {
                        "type": "string",
                        "title": "DSN Service Name"
                    }
                },
                "required": ["servicename", "user", "password", "host", "port"],
                "secret": ["password"]
            }
    
    ...
    
        def __init__(self, configuration):
            super(Oracle, self).__init__(configuration)
    
            dsn = cx_Oracle.makedsn(
                self.configuration["host"],
                self.configuration["port"],
                service_name=self.configuration["servicename"])
    
            self.connection_string = "{}/{}@{}".format(self.configuration["user"], self.configuration["password"], dsn)

여기보면 `service_name=...` 부분을 `sid=...`로 수정

            dsn = cx_Oracle.makedsn(
                self.configuration["host"],
                self.configuration["port"],
                sid=self.configuration["servicename"])
    
            self.connection_string = "{}/{}@{}".format(self.configuration["user"], self.configuration["password"], dsn)

## 서비스 리스타트

    $ supervisorctl restart all
    redash_server: stopped
    redash_celery: stopped
    redash_celery_scheduled: stopped
    redash_celery: started
    redash_server: started
    redash_celery_scheduled: started

# 작업 완료

접속해보면 Oracle이 떡하니 생겨있음

![](/images/Capture2018-09-12at11-c0a1749a-4d9f-4c22-99e6-fd2b5b7f1850.36.28.jpg)

설정하고 접속하면 됩니당. 

![](/images/Capture2018-09-12at12-198c3c4b-d55b-473c-9b15-54eb6fe25fc9.06.07.jpg)

접속 완료될때의 기쁨 ㅠㅠ

# 참조

- Configure Oracle in Redash - Help! : [https://discuss.redash.io/t/configure-oracle-in-redash-help/663](https://discuss.redash.io/t/configure-oracle-in-redash-help/663)
- Dockerfile로 deply : [https://gist.github.com/nihonzaru/7d311dcb2fe4e21cc38459963d27ccaa](https://gist.github.com/nihonzaru/7d311dcb2fe4e21cc38459963d27ccaa)
- Redash で Oracle 接続の Docker を作ってみた : [https://qiita.com/sutoh/items/d19c787069ff43aeebcf](https://qiita.com/sutoh/items/d19c787069ff43aeebcf)