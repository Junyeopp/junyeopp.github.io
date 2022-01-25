---
title: Let’s Encrypt의 DST Root CA X3 만료
data: 2021-10-07 19:00:00 +09:00
categories: [Tools]
tag: [Other]
---
Let’s Encrypt에서 발급받은 인증서를 사용하던 중 인증에 문제가 발생하여 알아보니 Root CA 하나가 만료되어 발생한 문제였습니다.

Let's Encrypt가 2015년에 서비스를 시작할 때, root certificate(A self-signed certificate controlled by a certificate authority)로 IdenTrust라는 곳의 DST Root X3를 통해서 cross-signed된 ISRG Root X1을 사용하였습니다.

이후 2021-10-01부터는 DST Root CA X3를 만료하고, ISRG Root X1을 사용한다고 합니다. (일부 Android device에 대해서는 2024년 9월까지 DST Root CA X3 지원)

- 참고자료
    - [DST Root CA X3 Expiration (September 2021)](https://letsencrypt.org/docs/dst-root-ca-x3-expiration-september-2021/)
    - [Production Chain Changes](https://community.letsencrypt.org/t/production-chain-changes/150739)
    - [안전한 SSL/TLS를 운영하기 위해 알아야 하는 것들](https://engineering.linecorp.com/ko/blog/best-practices-to-secure-your-ssl-tls/)
