# SSL-TLS HTTPS 적용

<!-- toc -->

## SSL-TLS 와 HTTPS

TLS(Transport Layer Security)는 인터넷 상에서 통신할 때 주고받는 데이터를 보호하기 위한 표준화된 암호화 프로토콜입니다.

TLS는 넷스케이프사에 의해 개발된 SSL(Secure Socket Layer) 3.0 버전을 기반으로 하며, 현재는 [2018년 8월에 발표된 TLS버전 1.3](https://en.wikipedia.org/wiki/Transport_Layer_Security#TLS_1.3)이 최종 버전입니다.
(이전 버전인 1.2는 2008년에 발표되었으므로 10년만에 업그레이드 되었습니다.)

TLS는 전송계층(Transport Layer)의 암호화 방식이기 때문에 HTTP뿐만 아니라 FTP, XMPP등 
응용 계층(Application Layer)프로토콜의 종류에 상관없이 사용할 수 있다는 장점이 있으며 기본적
으로 인증(Authentication), 암호화(Encryption), 무결성(Integrity)을 지원합니다.

![TLS 아키텍처](https://www.lesstif.com/download/attachments/18219486/image2014-7-30%2023%3A29%3A18.png?version=1&modificationDate=1406730379000&api=v2 "TLS 아키텍처")

SSL에서 TLS로 이름이 변경된 지 오래됐지만 아직도 사람들은 TLS대신 SSL이라는 표현을 더 많이 
사용하고 있으며 실제로 SSL/TLS의 오픈소스 구현체 프로젝트의 명칭은 아직도 OpenSSL이기도 합니다. 

또한 SSL/TLS의 가장 주된 적용 대상이 HTTPS다 보니 SSL/TLS를 HTTPS와 혼용하는 경우도 많습니다. 

이 문서에서는 SSL 이라고 할 경우 SSL 프로토콜, TLS 는 TLS 프로토콜을 의미하며, 보안이 적용된 HTTP는 HTTPS로 지칭하겠습니다.

SSL/TLS 를 사용하면 [중간자 공격](GLOSSARY.md)<sup id="fnref1">[1](#footnote1)</sup>과 Packet Spoofing 을 통한 도감청을 막을 수 있으며 통신하는 상대방이 맞는지 인증할 수 있습니다. <sup id="fnref2">[2](#footnote2)</sup>


## TLS HandShake

SSL/TLS 세션은 다음 핸드셰이크 과정을 거친 후에 구축됩니다. 

![TLS 핸드셰이크 절차](https://www.lesstif.com/download/attachments/18219486/image2014-10-24%2013%3A9%3A16.png?version=1&modificationDate=1414123436000&api=v2 "TLS 핸드셰이크 절차")



1. 클라이언트와 서버는 헬로 메시지로 기본적인 정보를 송수신 (1, 2)

1. 서버는 서버가 사용하는 SSL/TLS 인증서를 전달 (3, 4)

1. 클라이언트는 암호화 통신에 사용할 대칭키를 생성하고 사이를 서버에 전달(5). 이 과정을 키 교환(Key Exchange) 라고 하며 디피-헬만 키 교환(Diffie–Hellman key exchange) 또는 RSA 를 많이 사용. 

1. 클라이언트는 암호화 통신에 사용 가능한 암호 알고리즘과 해시 알고리즘 목록을 서버에 전달. (6, 7)

1. 서버도 알고리즘 목록을 교환후 핸드셰이크가 종료되며 이제 클라이언트와 서버는 암호화 통신에 필요한 대칭키를 서로 보유.(8, 9)

위 과정이 끝나면 SSL 세션이 구축되며 실제 암호화 통신을 시작할 수 있습니다.

**openssl 로 TLS 정보 엿보기**

다음 openssl 명령어를 사용하면 현재 tls 의 자세한 정보를 볼 수 있습니다.

```bash
openssl s_client -connect google.com:443 
```

출력 결과를 보면 프로토콜 버전, 암호 알고리즘, 세션 정보등 상세한 정보를 확인할 수 있습니다.
```
subject=/C=US/ST=California/L=Mountain View/O=Google Inc/CN=*.google.com
issuer=/C=US/O=Google Inc/CN=Google Internet Authority G2
---
No client certificate CA names sent
Peer signing digest: SHA256
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 4548 bytes and written 443 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES128-GCM-SHA256
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
    Session-ID: 43AF49BE4F199E128046F88C1A591732E45AB2B87898F4D80FD232D1F52B46E2
    Session-ID-ctx: 
    Master-Key: 4A20442D07BB216DE4A61E7BD78350B5D1B92AEEDECE5E07229B267BA555D68543BF1EAC08D19F47E98AB3DB270D5438
    Key-Arg   : None
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 100800 (seconds)
    TLS session ticket:
```



## 인증서 발급 받기

HTTPS 용 인증서를 발급받으려면 Verisign 이나 thawte, Comodo 같은 인증서 발급 기관을 통해서 절차에 따라 사이트 인증을 마친 후에 발급받아야 하며 이 과정에서 일정한 비용이 발생합니다.

개인 사이트라면 무료로 인증서를 발급하는 프로젝트인 [Lets' Encrypt](https://letsencrypt.org/) 를 통해 3개월 유효기간 인증서를 받을 수 있습니다.

* [Lets' Encrypt 에서 무료로 발급받기](https://blog.outsider.ne.kr/1178) 

AWS 를 사용한다면 [AWS Certificate Manager](https://aws.amazon.com/ko/certificate-manager/) 를 사용하여 무료로 인증서를 발급받을 수 있습니다.

> [!NOTE] 
> 현재 US East (Northern Virginia)리전에서만 사용 가능합니다.

### CSR 생성과 발급 

공개키 기반(PKI) 은 공개키와 개인키 두 개의 키 쌍을 사용하며 개인키는 절대 유출되어서는 안되며 사용자만 소유하고 있어야 합니다.

인증서 발급 기관에 HTTPS 용 인증서를 신청하는 프로세스와 명령어(openssl 사용)는 일반적으로 다음과 같습니다. 

1. HTTPS 를 적용하려는 서버 또는 신청자의 PC 에서 키쌍을 생성
```
openssl genrsa -aes256 -out /etc/pki/tls/private/example.com.key 2048
```

1. 생성한 공개키를 넣어서 인증서 발급 요청(CSR; Certificate Signing Request) 파일을 만들고 개인키로 전자서명
```
openssl req -new -key /etc/pki/tls/private/example.com.key -out /etc/pki/tls/certs/example.com.csr
```

1. CSR 을 인증서 발급기관에 전송

1. 인증서 발급기관은 CSR 에 있는 사용자가 보낸 전자서명을 CSR 에 포함된 공개키로 서명 검증

1. 사용자 공개키 검증이 끝났으면 사용자의 공개키와 추가 정보(도메인, 이메일 주소등)를 추가하여 SSL 인증서 발급(정확히는 발급기관의 개인키로 전자서명)

1. 생성된 인증서를 웹 서버에 적용

PKI 에 익숙하지 않은 경우 국내의 HTTPS 인증서 대행 기관들이 키 쌍 생성도 대행해서 인증서를 발급해 주지만 개인키 유출 우려가 있으므로 직접 키 쌍을 생성하고 CSR 을 만들어서 발급 받는 것을 권장합니다.


### 서버에 인증서 올리기

인증서 발급이 완료되면 인증서 파일(확장자: .crt, .cert 등)을 서버에 전송하고 웹 서버 설정을 수정하면 됩니다.

RHEL/CentOS 는 인증서와 개인키를 저장하는 폴더가 다음으로 정해져 있습니다.

* 인증서 : */etc/pki/tls/certs/*
* 개인키 : */etc/pki/tls/private/*

특히 SELinux 를 사용한다면 cert_t 라는 context 가 부여되어 있어야 웹 서버가 읽을 수 있습니다.
만약 인증서와 개인키가 있는데 웹 서버가 못 읽는다면 *chcon* 명령어로 context 를 변경하면 됩니다.

```
chcon -t cert_t /etc/pki/tls/certs/*
chcon -t cert_t /etc/pki/tls/private/*
```

### 개인키 passphrase 제거

일반적으로 개인키는 유출되어도 안전하도록 [PBKDF2](/encryption.html#pbkdf) 기반으로 암호화되어 저장합니다.
그래서 웹 서버를 구동하려면 개인키의 암호를 입력해야 하는데 서비스를 운영할 때 문제가 될 수 있습니다.

암호화된 개인키는 *openssl rsa in* 명령어를 사용하여 해독할 수 있으며 해독된 개인키는 보안을 위해 *chmod 600* 으로 모드를 변경하는 게 좋습니다. 

```
cp  /etc/pki/tls/private/example.com.key  /etc/pki/tls/private/example.com.key.enc
openssl rsa -in  /etc/pki/tls/private/example.com.key.enc -out  /etc/pki/tls/private/example.com.key
```

## 웹 서버 설정하기

인증서 발급이 끝났다면 이제 웹 서버에 적용할 순서입니다. 

### nginx 웹 서버

nginx 의 가상 호스트에 다음과 같이 ssl 설정을 추가해 주면 됩니다.

```
server {
    listen 80;
    listen 443 ssl;
    server_name  example.com
    root         html;
    index index.html index.htm index.php;

    charset utf-8;
 
    ssl                  on;
    ssl_certificate      /etc/pki/tls/certs/example.com.crt;
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

**중요 키워드**

-  *listen 443 ssl* : SSL-TLS 서비스를 제공할 포트를 지정합니다.
-  *ssl on* : SSL-TLS 를 켭니다.
-  *ssl_protocols* : 사용할 SSL-TLS 버전을 지정합니다.
-  *ssl_ciphers* : 사용할 암호 알고리즘을 지정합니다.
-  *ssl_prefer_server_ciphers* : SSL-TLS 협상 과정에서 서버에 설정한 암호 알고리즘을 우선하며 off 일 경우 알고리즘을 약화시켜서 공격할 수가 있으므로 on 으로 설정합니다.


### apache 웹 서버

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
    SSLHonorCipherOrder on

    SSLCertificateFile /etc/pki/tls/certs/example.com.crt
    SSLCertificateKeyFile /etc/pki/tls/private/example.com.key
    #SSLCertificateChainFile /etc/pki/tls/certs/server-chain.crt

    ErrorLog logs/www-ssl-error_log
    TransferLog logs/www-ssl-access_log
    CustomLog logs/ssl_request_log \
        "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"
</VirtualHost>

```

**중요 키워드**

-  *VirtualHost *:443* : SSL-TLS 서비스를 제공할 포트를 지정합니다.
-  *SSLEngine on* : SSL-TLS 를 켭니다.
-  *SSLProtocol* : 사용할 SSL-TLS 버전을 지정합니다.
-  *SSLCipherSuite* : 사용할 암호 알고리즘을 지정합니다.
-  *SSLHonorCipherOrder on* : nginx 의 ssl_prefer_server_ciphers 와 동일한 의미입니다.

### 인증서 경로 구성

모든 인증서는 상위 발급 기관과 최상위 인증 기관(CA)이 있으며 verisign 같은 유명 CA 들의 인증서는 브라우저에 내장되어 있으므로 브라우저는 SSL-TLS 통신시 사이트의 인증서가 신뢰하는 CA에서 발급받았고 위변조 되지 않았는지 확인할 수 있습니다.

![인증 경로](https://cloud.githubusercontent.com/assets/404534/14711049/d5ddcc28-0812-11e6-975b-1c998d94320a.png "인증 경로")


만약 인증서 발급 기관의 인증서가 브라우저에 기본으로 포함되지 않아서 인증서 경로를 찾지 못했다면 브라우저가 SSL-TLS 연결을 하지 못하는 경우가 있습니다. 
예로 [AlphaSSL](https://www.alphassl.com/support/install-root-certificate.html) 인증서는 모바일 chrome 에서 문제가 발생할 수 있습니다.


이런 문제를 해결하기 위해서는 Intermediate CA certificate 라고 하는 인증 기관 인증서 체인 파일을 웹 서버에 설정해 주어야 합니다.

 아파치 httpd 에는 [SSLCACertificateFile](http://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslcacertificatefile) 라는 지시자가 있으며 여기에 PEM 형식으로 인코딩된 SSL 인증서 발급 기관의 인증서 체인 파일을 설정하면 됩니다.

```
SSLCACertificateFile /etc/pki/tls/certs/ca-bundle.crt
```

 nginx 는 해당 지시자가 없으므로 다음과 같이 사이트의 SSL 인증서와 CA 인증서를 하나의 파일로 만들어 주면 됩니다.
 
 ```sh
cat example.com.crt example-ca.crt example-rootca.crt > /etc/pki/tls/certs/example.com.chained.crt
 ```
 
 합쳐진 인증서 파일을 nginx 의 ssl_certificate 에 지정해 주면 SSL chain 을 구성할 수 있습니다.
 
 ```
 ssl_certificate      /etc/pki/tls/certs/example.com.chained.crt;
 ```
 
 > **Danger** nginx 는 chain 파일의 첫 번째 인증서를 개인키와 일치한다고 가정하므로 체인의 첫 번째 인증서가 사이트의 SSL 인증서가 아닐 경우 다음과 같이 "*key values mismatch*" 에러가 발생하며 구동이 안 됩니다.
 
    ```
    SSL_CTX_use_PrivateKey_file(" ... /example.com.key") failed
       (SSL: error:0B080074:x509 certificate routines:
       X509_check_private_key:key values mismatch)
    ``` 

### http -> https 전환

로그인이나 개인 정보 변경등의 일부 기능에만 https 를 적용하는 것 보다는 사이트 전체에 적용하는 것이 관리도 용이하고 보안도 강화됩니다.

일반 사용자는 사이트에 연결시에 주소창에 https 를 명시하지 않고 연결하는 경우가 많으므로 http 로 연결했을 경우 https 로 redirect 하도록 설정하는 게 필요합니다.

애플리케이션 프레임워크에서 필터나 미들웨어 방식으로 이런 기능을 제공하지만 웹 서버의 redirection 기능을 사용하는 것이 쉽습니다.

*apache http* 는 다음과 같이 설정합니다.

```
<VirtualHost *:80>
	ServerName example.com

	RewriteEngine on
	RewriteCond %{HTTPS}  !on
	RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R,L]
</VirtualHost>
```

*nginx* 는 다음과 같이 설정합니다.

```
server {
	listen       80;
	server_name	example.com;
	return 301 https://$http_host$request_uri;	
}
```

### SSL 가상호스트와 SNI

웹 서버의 멋진 기능중에 하나는 가상 호스트(Virtual Host)로 이 기능을 사용하여 추가 장비없이 새로운 웹 서비스를 시작할 수 있습니다.

웹 서버는 브라우저의 HTTP Request Header 에서 Host 헤더를 통해 어떤 가상 호스트의 컨텐츠를 서비스할지 결정합니다.

즉 다음과 같이 curl을 수행했을 경우

```sh
$ curl -v google.com
```

아래와 같이 **Host: google.com** 을 보내므로 웹 서버는 정확한 가상 호스트를 결정할 수 있습니다.

```sh
GET / HTTP/1.1
Host: google.com
User-Agent: curl/7.43.0
Accept: */*
```
   
하지만 SSL/TLS의 경우 HTTP보다 먼저 수행되므로 브라우저가 Host Header 를 보내기 전에 SSL handshaking이 이루어 지고 웹 서버는 첫 번째 SSL 가상 호스트에 설정된 서버 인증서를 전송합니다.
그래서 SSL/TLS 기반으로 여러 개의 가상 호스트를 설정했을 경우 브라우저에서 SSL 인증서 검증시 인증서와 HostName 이 다르다는 에러가 발생할 수 있습니다.

이런 문제를 해결하기 위해 "서버 이름 표시([SNI;Server Name Indication - RFC 4366](http://tools.ietf.org/html/rfc4366#page-9))" 규격이 있으며 대부분의 브라우저, 웹 서버, HTTPS 구현 라이브러리가 SNI 를 지원하므로 SSL/TLS에서도 가상 호스트를 사용할 수 있습니다.

> [!NOTE]  
> Windows XP 와 JDK 6 은 SNI 를 지원하지 않으며 전체 목록은 [위키피디아의 SNI](https://en.wikipedia.org/wiki/Server_Name_Indication#Support)에서 확인할 수 있습니다.


## SSL/TLS 보안 강화하기

사이트에 적용한 SSL/TLS 를 더 견고하게 하기 위한 권장 설정.

### 최신 버전의 TLS 사용

SSL 은 보안 취약점이 있으므로 사용하지 말고 TLS 를 사용해야 하며 TLS 도 최신 버전(TLS 1.3)을 사용하는 것이 좋습니다.

만약 예전 Browser 를 사용하는 고객이 많아서 최신 버전을 강제하기 곤란하다면 다음과 같이 v1, v1.1, v1.2, v1.3 을 다 사용하도록 하면 브라우저의 지원 여부에 따라 자동으로 적절한 TLS 버전을 사용하여 세션이 구성됩니다. 

nginx 는 아래와 같이 사용할 버전을 지정할 수 있습니다.

```
ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
```

> [!NOTE] 
> TLS1.3 은 [nginx 1.13.0](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols) 이상과 [OpenSSL 1.1.1](https://www.openssl.org/news/openssl-1.1.1-notes.html) 이상이 필요합니다.

**apache httpd** 는 아래와 같이 사용할 버전을 지정할 수 있습니다.

```
# Dropping SSLv2, SSLv3, ref: POODLE
SSLProtocol all -SSLv2 -SSLv3 
```

이보다 좋은 방법은 사용할 프로토콜 버전을 명시적으로 지정하는 것입니다.

```
SSLProtocol TLSv1 TLSv1.1 TLSv1.2 TLSv1.3
```
> [!NOTE] 
> TLS1.3 은 [apache 2.4.36](https://github.com/apache/httpd/blob/2.4.36/CHANGES) 이상과 [OpenSSL 1.1.1](https://www.openssl.org/news/openssl-1.1.1-notes.html) 이상이 필요합니다.

### 강력한 알고리즘 사용

TLS 는 암호화 통신을 위해 사용할 알고리즘을 협상후 결정하는데 RC4 나 Triple DES 같은 오래된 알고리즘을 사용하면 암호화 통신을 하는 이유가 반감됩니다.

크롬의 경우 TLS V1.3 를 사용하더라도 예전 알고리즘이 사용 가능하면 사이트 정보 보기에서 아래와 같은 메시지를 출력하게 됩니다.

![약한 알고리즘 사용 경고](https://cloud.githubusercontent.com/assets/404534/12735884/cd7c5eae-c98e-11e5-84d5-315927f8147b.png "약한 알고리즘 사용 경고")

알고리즘 설정은 사용하지 않을 취약한 알고리즘을 명시적으로 지정하는 블랙리스트 방식보다는 사용할 강력한 알고리즘을 지정하는 화이트리스트 방식을 권장합니다.

**nginx** 는 아래와 같이 설정할 경우 강력한 알고리즘을 사용하게 됩니다.

```
ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
```

**apache httpd** 는 아래와 같이 설정하면 강력한 알고리즘을 사용하게 됩니다.

```
SSLHonorCipherOrder On
SSLInsecureRenegotiation off
SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
```

## HSTS(HTTP Strict Transport Security)

사이트에 최신 버전의 TLS와 강력한 알고리즘을 사용하여 HTTPS 를 적용하면 중간에서 통신을 가로채도 감청이 거의 불가능합니다.

하지만 [중간에 공격자가 끼어 들어 클라이언트의 HTTPS 요청을 HTTP 로 전환](http://noplanlife.com/?p=1418)할 수 있다면 감청이 가능해 집니다.

![SSL strip 공격](http://noplanlife.com/wp/wp-content/uploads/2016/03/FG0LEHk-640x387.png "SSL strip 공격 - http://noplanlife.com/wp/wp-content/uploads/2016/03/FG0LEHk-640x387.png")

이는 중간자 공격(Man in the middle attack)의 일종으로 **"SSL strip 공격"** 이라고 부릅니다.

HSTS 는 이런 문제를 해결하기 위해 HTTP 헤더에 **"Strict-Transport-Security"** 가 있으면 브라우저는 무조건 HTTPS 로만 연결하도록 하여 "SSL strip 공격"을 방지하는 표준입니다.

> [!Warning]
> 사이트에 HSTS 를 적용하면 브라우저가 더 엄격하게 동작하므로 서버 설정에 주의를 기울여야 합니다. 예로 HSTS 사이트의 SSL 인증서가 잘못되었을 경우 "위험을 감수하고 연결" 옵션이 없어집니다.
> ![image](https://cloud.githubusercontent.com/assets/404534/15269802/dca102ec-1a45-11e6-9b46-685bf60b7098.png "잘못된 인증서가 설정된 HSTS 사이트")

HSTS 를 사용하려면 **"Strict-Transport-Security"** HTTP 헤더를 설정하면 되며 지시자로 세부적인 동작을 지정할 수 있습니다. 

 - **max-age = delta-seconds** : 브라우저에게 delta-seconds 로 지정된 시간(단위 초)만큼 HTTPS 를 사용하라는 의미입니다. 개발 단계에서는 값을 아주 작게 설정하고 안정화되면 크게 주는게 좋습니다.
 - **includeSubdomains** : HSTS 를 서브 도메인도 적용합니다. 예로 www.example.com에서 *includeSubdomains* 설정이 포함된 HSTS 헤더를 전송했다면 사용자가 http://mail.exampl.com 같이 서브 도메인에 연결할 때도 브라우저는 자동으로 https 연결로 전환합니다.
 - **preload** : 브라우저가 해당 사이트를 HSTS 적용 preload list 에 추가합니다.
 
다음은 apache httpd 의 HSTS 설정으로 하루(86400)동안 HSTS를 유지하며 서브 도메인에도 HSTS를 적용합니다.

```
Header always set Strict-Transport-Security "max-age=86400; includeSubdomains; preload"
```

nginx 는 add_header 지시자로 HSTS 를 설정하면 됩니다.

```
add_header Strict-Transport-Security "max-age=86400; includeSubdomains; preload";
```

### HSTS 설정 해제

preload 에 추가한 사이트는  max-age 기간동안 자동으로 https 로 연결하며 웹 서버의 HSTS 헤더를 삭제해도 사용자의 브라우저에는 설정이 유지됩니다.

여러 가지 이유로 해제가 필요하다면 사용자가 직접 브라우저의 설정을 수정해야 합니다.

**Chrome**

 크롬의 경우 해제하려면 다음 절차를 따르면 됩니다.

1. 주소창에 *chrome://net-internals/#hsts* 를 입력하여 설정에 들어갑니다.
1. **Delete Domain** 에 삭제할 주소를 입력하고 **Delete** 를 클릭합니다.
1. **Query Domain** 에 주소를 입력하고 **Query** 를 클릭해서 **Not Found** 가 나오는지 확인합니다.

![크롬 HSTS 해제](https://cloud.githubusercontent.com/assets/404534/14701735/bbf5749a-07e1-11e6-88ec-b172338c2d24.png "크롬 HSTS 해제")

## 요약

- 위에서 설명한 내용과 추가 설정을 웹 서버별로 상세히 정리해서 제공하는 **[Strong Ciphers for Apache, nginx and Lighttpd](https://cipherli.st/)** 사이트를 참고해서 실제 서버에 적용하세요.
- HSTS 는 일단 적용되면 **max-age 기간동안 자동 적용**되므로 테스트 환경에서 충분히 테스트를 거친 후에 운영 환경에 적용하세요.
- 적용이 완료되었다면 [온라인 SSL-TLS 사이트 분석 서비스](https://www.ssllabs.com/ssltest/analyze.html) 를 통해 견고하게 설정되었는지 확인해 보세요.

## 참고 자료

* [HTTP Strict Transport Security - OWASP](https://www.owasp.org/index.php/HTTP_Strict_Transport_Security)
* [STS(Strict Transport Security) 및 보안 쿠키 설정](https://developers.google.com/web/fundamentals/security/encrypt-in-transit/turn-on-strict-transport-security-and-secure-cookies?hl=ko)
* [The First Few Milliseconds of an HTTPS Connection](http://www.moserware.com/2009/06/first-few-milliseconds-of-https.html)
* [mod_ssl 로 보안 강화하기](https://www.lesstif.com/pages/viewpage.action?pageId=18219486)
* [nginx에 HTTPS/SSL 적용하기](https://www.lesstif.com/pages/viewpage.action?pageId=27984443)


<a name="footnote1" href="#fnref1">[1]</a>: ARP 스푸핑을 통한 피해 및 모범 대응 사례- http://blog.bandisoft.com/132
<a name="footnote2" href="#fnref2">[2]</a>: 백신 프로그램은 HTTPS 패킷을 검사하기 위해 백신 회사가 발급한 root 인증 기관 인증서를 브라우저에 신뢰하는 인증기관으로 추가하고 TLS 인증서를 발급해서 HTTPS 를 통해 오가는 데이타를 검사합니다. 이 방식은 좋은 용도지만 중간자 공격과 동일합니다.

