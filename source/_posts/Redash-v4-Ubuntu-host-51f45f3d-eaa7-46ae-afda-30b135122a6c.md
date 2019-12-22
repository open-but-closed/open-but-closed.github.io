---
title: Redash(v4) Ubuntu(host)에 설치하기
categories: [DW/BI,Redash]
date: Aug 10, 2018 7:03 PM
permalink: install-redash-v4-on-ubuntu-host
tags: [Redash,cloud-computing]
updated: Oct 05, 2019 12:56 AM
description: Re:dash를 공짜로 받은 Digital Ocean에 설치해보자.
---

*참고: 이 버전(v4)까지는 host에 직접 설치할 수 있는 provisioning script를 제공했다. 이때 docker버전도 존재했는데, 아예 v5버전 이후에는 docker방식으로만 설치가 가능하다. 아주 바람직하다고 봄*

*참고2: 기본적으로 VM (Droplet)이 생성되었다는 전제로 시작*

# 설치

설치 파일 다운로드

    root@redash-001:~#
    root@redash-001:~# curl -O https://raw.githubusercontent.com/getredash/redash/master/setup/ubuntu/bootstrap.sh
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100  3739  100  3739    0     0   8879      0 --:--:-- --:--:-- --:--:--  8902
    root@redash-001:~# ls
    bootstrap.sh

권한을 `chmod`로 바꾸고 `bootstrap.sh`를 실행

    root@redash-001:~# chmod 755 *.sh
    root@redash-001:~# ./bootstrap.sh
    Hit:1 https://repos.sonar.digitalocean.com/apt main InRelease
    Hit:2 http://mirrors.digitalocean.com/ubuntu xenial InRelease
    Get:3 http://security.ubuntu.com/ubuntu xenial-security InRelease [107 kB]
    Get:4 http://mirrors.digitalocean.com/ubuntu xenial-updates InRelease [109 kB]
    Get:5 http://mirrors.digitalocean.com/ubuntu xenial-backports InRelease [107 kB]
    Fetched 323 kB in 2s (118 kB/s)
    ...
    ...
    ...
    tests/test_utils.py
    tests/__pycache__/
    webpack.config.js
    Traceback (most recent call last):
      File "/usr/bin/pip", line 11, in <module>
        sys.exit(main())
      File "/usr/lib/python2.7/dist-packages/pip/__init__.py", line 215, in main
        locale.setlocale(locale.LC_ALL, '')
      File "/usr/lib/python2.7/locale.py", line 581, in setlocale
        return _setlocale(category, locale)
    locale.Error: unsupported locale setting
    root@redash-001:~#

뭔가 실패 ㅠㅠ 계속 에러난 부분이

    perl: warning: Falling back to a fallback locale ("en_US.UTF-8").
    locale: Cannot set LC_CTYPE to default locale: No such file or directory
    locale: Cannot set LC_ALL to default locale: No such file or directory

`LC_ALL`등을 셋업해서 다시 시도

    root@redash-002:~# locale
    locale: Cannot set LC_CTYPE to default locale: No such file or directory
    locale: Cannot set LC_ALL to default locale: No such file or directory
    LANG=en_US.UTF-8
    LANGUAGE=
    LC_CTYPE=UTF-8
    LC_NUMERIC="en_US.UTF-8"
    LC_TIME="en_US.UTF-8"
    LC_COLLATE="en_US.UTF-8"
    LC_MONETARY="en_US.UTF-8"
    LC_MESSAGES="en_US.UTF-8"
    LC_PAPER="en_US.UTF-8"
    LC_NAME="en_US.UTF-8"
    LC_ADDRESS="en_US.UTF-8"
    LC_TELEPHONE="en_US.UTF-8"
    LC_MEASUREMENT="en_US.UTF-8"
    LC_IDENTIFICATION="en_US.UTF-8"
    LC_ALL=
    root@redash-002:~# export LC_ALL=en_US.UTF-8
    root@redash-002:~# export LANGUAGE=en_US.UTF-8
    root@redash-002:~# chmod 755 bootstrap.sh
    root@redash-002:~# ./bootstrap.sh
    ...
    ...
    ...
    Length: 453 [text/plain]
    Saving to: ‘/etc/nginx/sites-available/redash’
    
    /etc/nginx/sites-available/redash     100%[=========================================================================>]     453  --.-KB/s    in 0s
    
    2018-08-21 15:19:48 (48.2 MB/s) - ‘/etc/nginx/sites-available/redash’ saved [453/453]
    

뭔가 에러 없이 잘 설치된 듯 하다.

# 접속 테스트

- [http://178.128.57.213/](http://178.128.57.213/)

![](/images/Capture2018-08-22at00-cd555037-6e5a-4fcc-aab2-65ff45d0c200.29.29.jpg)

끝.

## https도 활성화하자

- [https://redash.io/help/open-source/admin-guide/https-ssl-setup](https://redash.io/help/open-source/admin-guide/https-ssl-setup)

cerificate가 있어야 해서... 그냥 패스

# 설치 이후 확인

서버에 어떤 식으로 설치되었는지 확인해보면,

## 설치된 프로그램

- python 및 패키지
- redis
- nginx
- postgreSQL

## 방화벽

    root@redash-002:~# netstat -nlpt
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
    tcp        0      0 127.0.0.1:9001          0.0.0.0:*               LISTEN      17137/python
    tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      13882/redis-server
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      17162/nginx -g daem
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1583/sshd
    tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN      13652/postgres
    tcp        0      0 127.0.0.1:5000          0.0.0.0:*               LISTEN      17166/gunicorn: mas
    tcp6       0      0 :::22                   :::*                    LISTEN      1583/sshd

## 계정

postgres, redis, redash가 생성되는 듯

    root@redash-002:~# cat /etc/passwd
    root:x:0:0:root:/root:/bin/bash
    daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
    bin:x:2:2:bin:/bin:/usr/sbin/nologin
    sys:x:3:3:sys:/dev:/usr/sbin/nologin
    sync:x:4:65534:sync:/bin:/bin/sync
    games:x:5:60:games:/usr/games:/usr/sbin/nologin
    man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
    lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
    mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
    news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
    uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
    proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
    www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
    backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
    list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
    irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
    gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
    nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
    systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
    systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
    systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
    systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
    syslog:x:104:108::/home/syslog:/bin/false
    _apt:x:105:65534::/nonexistent:/bin/false
    lxd:x:106:65534::/var/lib/lxd/:/bin/false
    messagebus:x:107:111::/var/run/dbus:/bin/false
    uuidd:x:108:112::/run/uuidd:/bin/false
    dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
    sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin
    pollinate:x:111:1::/var/cache/pollinate:/bin/false
    postgres:x:112:117:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
    redis:x:113:118::/var/lib/redis:/bin/false
    redash:x:114:65534:,,,:/home/redash:/bin/false

## PostgreSQL 접속 및 PW 변경

다음 링크에 따르면, 설치된 postgres의 경우 system authentication을 따르므로, 접속해서 바꾸라는

- 링크 : [https://discuss.redash.io/t/connnect-to-postgres-database/714](https://discuss.redash.io/t/connnect-to-postgres-database/714)

가능하면 다음과 같은 방식으로 접속 이후 변경하는 것을 추천함.

- 링크 : [https://serverfault.com/questions/110154/whats-the-default-superuser-username-password-for-postgres-after-a-new-install](https://serverfault.com/questions/110154/whats-the-default-superuser-username-password-for-postgres-after-a-new-install)

        sudo -u postgres psql postgres
        
        # \password postgres
        
        Enter new password:

psql로 확인해보면, `redash` 스키마와 `redash`라는 유저가 생성되어 있음.

    redash=> \l
                                      List of databases
       Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
    -----------+----------+----------+-------------+-------------+-----------------------
     postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
     redash    | redash   | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
     template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
               |          |          |             |             | postgres=CTc/postgres
     template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
               |          |          |             |             | postgres=CTc/postgres
    (4 rows)
    
    redash=> \dn
      List of schemas
      Name  |  Owner
    --------+----------
     public | postgres
    (1 row)
    
    redash=> \dg
                                       List of roles
     Role name |                         Attributes                         | Member of
    -----------+------------------------------------------------------------+-----------
     postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
     redash    |                                                            | {}