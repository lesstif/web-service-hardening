### 목차 

* [SSL/TLS/HTTPS 란](#SSL/TLS/HTTPS 란)
* [HTTPS 인증서 발급 받기](#HTTPS 인증서 발급 받기)
* [nginx 웹 서버 설정하기](###nginx 웹 서버 설정하기)
* [apache 웹 서버 설정하기](###apache 웹 서버 설정하기)
  * [SSL/TLS 보안 강화하기](#SSL/TLS 보안 강화하기)


### SSL/TLS/HTTPS 란
TODO


### HTTPS 인증서 발급 받기

* [Lets' Encrypt 에서 무료로 발급받기](https://blog.outsider.ne.kr/1178)
AWS 를 사용한다면 [AWS Certificate Manager](https://aws.amazon.com/ko/certificate-manager/) 를 사용하여 무료로 인증서를 발급받을 수 있습니다.
* 현재 US East (Northern Virginia)리전에서만 사용 가능합니다.


### nginx 웹 서버 설정하기

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


### apache 웹 서버 설정하기

TODO


### SSL/TLS 보안 강화하기

TODO

#### 최신 버전의 TLS 사용하기

SSL 은 보안 취약점이 있으므로 사용하지 말고 TLS 를 사용해야 하며 TLS 도 최신 버전(TLS 1.2)을 사용하는 것이 좋습니다. ([TLS 1.1 이상은 openssl 1.0.1 이상이 필요합니다.](http://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols))

만약 예전 IE 를 사용하는 고객이 많아서 TLS 1.2 를 강제하기 곤란하다면 다음과 같이 v1, v1.1, v1.2 를 다 사용하도록 하면 브라우저의 지원 여부에 따라 자동으로 적절한 TLS 버전을 사용하여 세션이 구성됩니다. 

nginx 는 아래와 같이 사용할 알고리즘을 지정할 수 있습니다.
```
ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
```

#### 강력한 알고리즘 사용하기

TLS 는 암호화 통신을 위해 사용할 알고리즘을 협상후 결정하는데 RC4 나 Triple DES 같은 오래된 알고리즘을 사용하면 암호화 통신을 하는 이유가 반감됩니다.

크롬의 경우 TLS V1.2 를 사용하더라도 예전 알고리즘이 사용 가능하면 사이트 정보 보기에서 아래와 같은 메시지를 출력하게 됩니다.

알고리즘 설정은 사용하지 않을 취약한 알고리즘을 명시적으로 지정하는 블랙리스트 방식보다는 사용할 강력한 알고리즘을 지정하는 화이트리스트 방식을 권장합니다.

nginx 는 아래와 같이 설정할 경우 강력한 알고리즘을 사용하게 됩니다.
```
ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
```


### 참고 자료
* [nginx에 HTTPS/SSL 적용하기](https://www.lesstif.com/pages/viewpage.action?pageId=27984443)
