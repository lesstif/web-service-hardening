# SSL/TLS/HTTPS 적용

<!-- toc -->

## SSL/TLS/HTTPS 란

TLS(Transport Layer Security)는 인터넷 상에서 통신할 때 주고받는 데이터를 보호하기 위한 표준화된 암호화 프로토콜입니다.

TLS는 넷스케이프사에 의해 개발된 SSL(Secure Socket Layer) 3.0 버전을 기반으로 하며, 현재는 2008년 발표된 TLS버전 1.2가 최종 버전입니다.

TLS는 전송계층(Transport Layer)의 암호화 방식이기 때문에 HTTP뿐만 아니라 FTP, XMPP등 
응용 계층(Application Layer)프로토콜의 종류에 상관없이 사용할 수 있다는 장점이 있으며기본적
으로 인증(Authentication), 암호화(Encryption), 무결성(Integrity)을 지원합니다.

![TLS 아키텍처](https://www.lesstif.com/download/attachments/18219486/image2014-7-30%2023%3A29%3A18.png?version=1&modificationDate=1406730379000&api=v2 "TLS 아키텍처")

SSL에서 TLS로 이름이 변경된 지 오래됐지만 아직도 사람들은 TLS대신 SSL이라는 표현을 더 많이 
사용하고 있으며 실제로 SSL/TLS의 오픈소스 구현체 프로젝트의 명칭은 아직도 OpenSSL이기도 합니다. 

또한 SSL/TLS의 가장 주된 적용 대상이 HTTPS다 보니 SSL/TLS를 HTTPS와 혼용하는 경우도 많습니다. 

이 문서에서는 SSL 이라고 할 경우 SSL 프로토콜, TLS 는 TLS 프로토콜을 의미하며, 보안이 적용된 HTTP는 HTTPS로 지칭하겠습니다.

SSL/TLS 를 사용하면 [중간자 공격](GLOSSARY.md)[^1]과 Packet Spoofing 을 통한 도감청을 막을 수 있으며 통신하는 상대방이 맞는지 인증할 수 있습니다.  [^2]


## TLS HandShake

SSL/TLS 세션은 다음 핸드셰이크 과정을 거친 후에 구축됩니다. 

![TLS 핸드셰이크 절차](https://www.lesstif.com/download/attachments/18219486/image2014-10-24%2013%3A9%3A16.png?version=1&modificationDate=1414123436000&api=v2 "TLS 핸드셰이크 절차")



1. 클라이언트와 서버는 헬로 메시지로 기본적인 정보를 송수신 (1, 2)

1. 서버는 서버가 사용하는 SSL/TLS 인증서를 전달 (3, 4)

1. 클라언트는 암호화 통신에 사용할 대칭키를 생성하고 사이를 서버에 전달(5). 이 과정을 키 교환(Key Exchange) 라고 하며 디피-헬만 키 교환(Diffie–Hellman key exchange) 또는 RSA 를 많이 사용. 

1. 클라이언트는 암호화 통신에 사용 가능한 암호 알고리즘과 해시 알고리즘 목록을 서버에 전달. (6, 7)

1. 서버도 알고리즘 목록을 교환후 핸드쉐이크가 종료되며 이제 클라이언트와 서버는 암호화 통신에 필요한 대칭키를 서로 보유.(8, 9)

위 과정이 끝나면 SSL 세션이 구축되며 실제 암호화 통신을 시작할 수 있습니다.



## 인증서 발급 받기

HTTPS 용 인증서를 발급받으려면 Verisign 이나 thawte, Comodo 같은 인증서 발급 기관을 통해서 절차에 따라 사이트 인증을 마친 후에 발급받아야 하며 이 과정에서 일정한 비용이 발생합니다.

개인 사이트라면 무료로 인증서를 발급하는 프로젝트인 [Lets' Encrypt](https://letsencrypt.org/) 를 통해 3개월 유효기간 인증서를 받을 수 있습니다.

* [Lets' Encrypt 에서 무료로 발급받기](https://blog.outsider.ne.kr/1178) 

AWS 를 사용한다면 [AWS Certificate Manager](https://aws.amazon.com/ko/certificate-manager/) 를 사용하여 무료로 인증서를 발급받을 수 있습니다.

* 현재 US East (Northern Virginia)리전에서만 사용 가능합니다.


## nginx 웹 서버 설정하기

nginx 의 가상 호스트에 다음과 같이 ssl 설정을 추가해 주면 됩니다.

```
server {
    listen       443;
    server_name  example.com
    root         html;
 
 
    ssl                  on;
    ssl_certificate      /etc/pki/tls/certs/example.com.chained.crt;
    ssl_certificate_key  /etc/pki/tls/private/example.com.key;
    ssl_session_timeout  5m;
    # SSLv2, SSLv3는 보안에 취약하므로 사용하지 마세요
    ssl_protocols  TLSv1.2 TLSv1.1 TLSv1;
    # 사용하지 않을 암호 알고리즘은 !로 명시적으로 지정할 수 있습니다.(블랙리스트 방식) 	
    ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
    ssl_prefer_server_ciphers   on;
    location ~ /\.ht {
         deny  all;
    }
}
```


## apache 웹 서버 설정하기

RHEL/CentOS 의 아파치 웹 서버는 /etc/httpd/conf.d/ssl.conf 에 다음과 같이 가상호스트를 설정합니다.

```
<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com
    SSLEngine on
    # Dropping SSLv2, SSLv3, ref: POODLE
    SSLProtocol all -SSLv2 -SSLv3 

    SSLCipherSuite ALL:!ADH:!EXPORT:!SSLv2:RC4+RSA:+HIGH:+MEDIUM:+LOW
    #SSLCipherSuite HIGH:!aNULL:!MD5

    SSLCertificateFile /etc/pki/tls/certs/example.com.crt
    SSLCertificateKeyFile /etc/pki/tls/private/example.com.key
    #SSLCertificateChainFile /etc/pki/tls/certs/server-chain.crt

    ErrorLog logs/www-ssl-error_log
    TransferLog logs/www-ssl-access_log
    CustomLog logs/ssl_request_log \
        "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"
</VirtualHost>

```


## SSL/TLS 보안 강화하기

사이트에 적용한 SSL/TLS 를 더 견고하게 하기 위한 권장 설정.

### 최신 버전의 TLS 사용

SSL 은 보안 취약점이 있으므로 사용하지 말고 TLS 를 사용해야 하며 TLS 도 최신 버전(TLS 1.2)을 사용하는 것이 좋습니다. ([TLS 1.1 이상은 openssl 1.0.1 이상이 필요.](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols))

만약 예전 IE 를 사용하는 고객이 많아서 TLS 1.2 를 강제하기 곤란하다면 다음과 같이 v1, v1.1, v1.2 를 다 사용하도록 하면 브라우저의 지원 여부에 따라 자동으로 적절한 TLS 버전을 사용하여 세션이 구성됩니다. 

nginx 는 아래와 같이 사용할 버전을 지정할 수 있습니다.

```
ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
```

**apache httpd** 는 아래와 같이 사용할 버전을 지정할 수 있습니다.

```
# Dropping SSLv2, SSLv3, ref: POODLE
SSLProtocol all -SSLv2 -SSLv3 
```

또는 사용할 프로토콜 버전을 명시적으로 지정해도 됩니다.

```
SSLProtocol TLSv1 TLSv1.1 TLSv1.2
```

### 강력한 알고리즘 사용

TLS 는 암호화 통신을 위해 사용할 알고리즘을 협상후 결정하는데 RC4 나 Triple DES 같은 오래된 알고리즘을 사용하면 암호화 통신을 하는 이유가 반감됩니다.

크롬의 경우 TLS V1.2 를 사용하더라도 예전 알고리즘이 사용 가능하면 사이트 정보 보기에서 아래와 같은 메시지를 출력하게 됩니다.

알고리즘 설정은 사용하지 않을 취약한 알고리즘을 명시적으로 지정하는 블랙리스트 방식보다는 사용할 강력한 알고리즘을 지정하는 화이트리스트 방식을 권장합니다.

**nginx** 는 아래와 같이 설정할 경우 강력한 알고리즘을 사용하게 됩니다.

```
ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
```

**apache httpd** 는 아래와 같이 설정하면 강력한 알고리즘을 사용하게 됩니다.

```
SSLCipherSuite HIGH:!aNULL:!MD5
```

## 참고 자료

* [mod_ssl 로 보안 강화하기](https://www.lesstif.com/pages/viewpage.action?pageId=18219486)
* [nginx에 HTTPS/SSL 적용하기](https://www.lesstif.com/pages/viewpage.action?pageId=27984443)





[^1]: ARP 스푸핑을 통한 피해 및 모범 대응 사례- http://blog.bandisoft.com/132
[^2]: 백신 프로그램은 HTTPS 패킷을 검사하기 위해 백신 회사가 발급한 root 인증 기관 인증서를 브라우저에 신뢰하는 인증기관으로 추가하고 TLS 인증서를 발급해서 HTTPS 를 통해 오가는 데이타를 검사합니다. 이 방식은 좋은 용도지만 중간자 공격과 동일합니다.

