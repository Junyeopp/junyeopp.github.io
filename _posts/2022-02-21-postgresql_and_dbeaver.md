---
title: PostgreSQL + DBeaver 시작하기
data: 2022-02-21 01:00:00 +09:00
categories: [DB]
tag: [SQL, DB]
---

[데이터 분석을 위한 SQL 레시피](https://www.hanbit.co.kr/store/books/look.php?p_code=B8585882565) 실습환경을 설정한 과정을 정리하였습니다.

# PostgreSQL

1. [posgres Docker Image](https://hub.docker.com/_/postgres)를 받아줍니다.
    ```bash
    docker pull postgres
    ```

2. 이제 컨테이너를 만들어줍니다.
    ```bash
    docker run --name postgres-demo -e POSTGRES_PASSWORD=junyeop -p 5432:5432 -v /Users/junyeop/Documents/try_everything/SQL_Recipe_sample-code:/mnt -d postgres
    ```
    5432포트를 연결하고 sql파일들이 있는 폴더를 마운트하여 postgres-demo라는 이름의 컨테이너를 만들었습니다.

3. container로 접속해서 DB를 하나 만들어줍니다.
    ```bash
    docker exec -it postgres-demo bash

    psql -U postgres

    CREATE DATABASE demodb;
    ```
    ![dbeaver0]({{ site.baseurl }}/assets/images/2022-02-21-postgresql_and_dbeaver0.png)

# DBeaver

1. [여기](https://dbeaver.io/download/)서 알맞은 버전을 설치해줍니다.

2. PostgreSQL로 연결해줍니다.
    ![dbeaver1]({{ site.baseurl }}/assets/images/2022-02-21-postgresql_and_dbeaver1.png)

3. Driver가 없으면 설치해줍니다.
    ![dbeaver2]({{ site.baseurl }}/assets/images/2022-02-21-postgresql_and_dbeaver2.png)

4. 생성한 DB정보와 비밀번호를 입력하고 연결해줍니다.
    ![dbeaver3]({{ site.baseurl }}/assets/images/2022-02-21-postgresql_and_dbeaver3.png)

5. 도커 컨테이너에서 sql파일을 이용해서 테이블을 만들어줍니다.
    ![dbeaver4]({{ site.baseurl }}/assets/images/2022-02-21-postgresql_and_dbeaver4.png)

6. 그러면 DBeaver에서도 테이블이 생성된 것을 확인할 수 있습니다.
    ![dbeaver5]({{ site.baseurl }}/assets/images/2022-02-21-postgresql_and_dbeaver5.png)

