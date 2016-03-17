## 목차

Web Server 보안 고려 사항

## 자세한 server 정보 숨기기

운영하는 서버의 자세한 정보는 공격자에게 유용한 정보를 제공합니다. 다음과 같이 curl 명령어로 서버가 보내오는 HTTP 응답 헤더를 살펴 봅시다.

```
curl -I myhost.com
```

```
HTTP/1.1 200 OK
Date: Thu, 17 Mar 2016 05:54:33 GMT
Server: Apache/2.2.24 (Unix) mod_ssl/2.2.24 OpenSSL/0.9.8e-fips-rhel5 PHP/5.2.17 mod_jk/1.2.41
X-Powered-By: PHP/5.2.17
Content-Length: 1861
Content-Type: text/html
```

현재 사용하는 서버 제품(Apache)과 버전(2.2.24), 모듈(mod_ssl, PHP, mod_jk ..)과 버전등 굉장히 상세한 정보를 보여주고 있으며 공격자는 해당 제품과 버전의 취약점을 찾아서 공격할 수 있습니다.



### server 정보 가리기

apache httpd 는 httpd.conf 에 다음과 같이 설정하면 표시 정보를 최소화할 수 있습니다.

```
ServerTokens Prod
ServerSignature Off
```

nginx 는 nginx.conf 의 http 항목에 다음 내용을 추가합니다.

```
http {
	server_tokens off;
	
```

### X-Powered-By 헤더 제거

apache httpd 의 경우 Header 지시자를 사용하면 브라우저로 전송하기 전에 모든 X-Powered-By 헤더를 삭제할 수 있습니다. (추천 방법)

```
Header unset X-Powered-By
```

PHP 는 php.ini 의 다음 항목을 off 로 설정하면 됩니다.

```
expose_php = Off
```

## 참고 자료
* [apache ServerTokens Directive](https://httpd.apache.org/docs/2.4/mod/core.html#servertokens)
* 
