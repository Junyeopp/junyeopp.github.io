---
title: downtime없이 MongoDB Replica Set 설정 변경하기
data: 2022-01-06 22:00:00 +09:00
categories: [MongoDB]
tag: [dev]
---

# 상황

현재 상황은 docker로 MongoDB가 실행되고 있으며, [PSA](https://docs.mongodb.com/manual/core/replica-set-architecture-three-members/#primary-with-a-secondary-and-an-arbiter--psa-) 구조(Primary 1개, Secondary 1개, Arbiter 1개)의 Replica Set으로 구성되어 있는 상황입니다. 그리고 변경해야하는 사항들은 다음과 같습니다.

1. Replica Set의 host 주소를 IP에서 URI로 변경하기
2. `mongod.conf` 파일의 `security.keyFile`을 활성화하기
3. `mongod.conf` 파일의 `net.tls`를 활성화하기
4. `docker-compose.yaml` 파일의 `extra_hosts`의 주소를 IP에서 URI로 변경하기

# 해결

## 첫 번째 시도

1. Arbiter를 내린 후 `secrity.keyFile` 활성화, `net.tls` 활성화, `extra_hosts` 변경 후 restart 해보았습니다.

    → `lastHeartbeatMessage: 'Connection closed by peer’` 로 Arbiter에 연결되지 않았습니다. 

    → `security`가 활성화되면서 기존의 `security`가 비활성화된 Primary와 Secondary가 Aribiter 연결에 실패한 것이 아닐까 생각이 되어서 `net.tls` 활성화, `extra_hosts` 변경만 하였더니 정상적으로 연결되었습니다.


## 두 번째 시도

1. [MongoDB 문서](https://docs.mongodb.com/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set-without-downtime/)를 참고해서 downtime없이 `security.keyFile` 활성화를 시도하였습니다.

    → config파일의 `security.transitionToAuth` 를 `true`로 설정하고 `secrity.keyFile`을 설정 후 restart 하였습니다.

    → rs.status() 을 통해 정상적으로 서로 연결된 것을 확인하였고 문서를 따라 나머지 절차도 진행하였습니다.

    → 성공!

2. [MongoDB 문서](https://docs.mongodb.com/manual/tutorial/change-hostnames-in-a-replica-set/#change-hostnames-while-maintaining-replica-set-availability)를 참고해서 downtime없이 rs.reconfig()를 진행하였습니다.

    ```bash
    cfg = rs.config()
    cfg.members[0].host = "new hostname"

    rs.reconfig(cfg)
    ```

    → 위의 방법으로 Secondary와 Arbiter를 먼저 수정하였습니다.

    → Primary를 `rs.stepDown()`하여 Secondary로 변경 후 새로운 Primary에서 위의 방법으로 기존 Primary의 host도 변경하였습니다.

    → 성공!

3. [MongoDB 문서](https://docs.mongodb.com/manual/tutorial/upgrade-cluster-to-ssl/#upgrade-a-cluster-to-use-tls-ssl)를 참고해서 downtime없이 TLS 설정을 하였습니다.

    ```yaml
    net:
       tls:
          mode: allowTLS
          PEMKeyFile: <path to TLS/SSL certificate and key PEM file>
          CAFile: <path to root CA PEM file>
    ```

    → 먼저 config파일의 tls를 allowTLS모드로 변경후 Secondary, Arbiter, Primary 순으로 restart 해주었습니다.
    
    → `db.adminCommand( { setParameter: 1, tlsMode: "preferTLS" } )` 를 통해 tlsMode를 변경하였고, `db.adminCommand( { getParameter : 1, "tlsMode" : 1 } )` 를 통해 현재 tlsMode parameter가 `preferTLS`로 되어있는 것을 확인하였습니다.

    → 같은 방법으로 `requireTLS`모드로 변경하였습니다.

    → config의 `tls.mode`도 `requireTLS`로 변경하여 restart될 때 문제가 없도록 하였습니다.

    → 임의로 변경된 config파일로 restart 해보았때 정상적으로 작동하였습니다.

    → 성공!


# +
원했던대로 downtime없이 설정을 완료할 수 있었습니다. member 중 하나가 down되어도 되는 Replica Set의 장점을 잘 사용할 수 있었던 경험이었습니다.