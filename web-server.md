# Web 서버

<!-- toc -->

웹 서버는 인터넷에서 가장 중요한 인프라 SW중 하나로 HTTP 를 기반으로 컨텐츠를 사용자에게 전달하고 사용자의 입력을 받아서 처리합니다.

과거의 정적 컨텐츠에서 동적 컨텐츠로 변화되면서 다양한 웹용 언어와 프레임워크가 탄생하였고 이런 도구들은 웹 서버 위에서 바로 동작하는 경우가 많았습니다.

PHP를 처리하는 mod_php, Perl 스크립트를 처리하는 mod_perl, python 용 mod_python 등이 이런 확장 모듈입니다.

이런 모듈은 웹 서버를 바로 활용할 수 있으므로 설치와 설정이 쉽고 사용하기도 쉬운 장점이 있었지만 서버스의 규모가 커지고 보안이 중요시 되면서 다음과 같은 문제점이 나타났습니다.

1. 확장성 문제
 
 규모가 커질 경우 Web/WAS/DBMS 등  Tier 별로 확장 전략이 필요하지만 한 서버에서 Web/WAS 기능을 다 수행할 경우 Tier 별 확장이 어렵습니다.
 
1. 보안 문제

 웹 서버는 DMZ에 위치하므로 해킹에 대비하여 권한을 낮추고 내부 네트워크 접근을 최소화 해야 합니다. 
 
 웹 서버를 해킹하고 이를 통해 내부 네트워크에 침입해서 2차 피해를 입거나 또는 웹 서버를 경유지로 하여 스팸 메일을 보내거나 외부 서버에 연결하는 등의 공격이 많아 졌습니다.
 
 이런 이유로 SELinux 에서는 DMZ에 위치하는 서비스들(Web, FTP, Mail)를 엄격하게 통제하며 미리 허가되지 않은 작업은 모두 차단합니다.
 
 그러므로 웹 서버는 정적 컨텐츠를 전송하거나 또는 내부 네트워크에 있는 Application Server 의 Reverse Proxy 로 사용하는 게 안정성과 보안 측면에서 좋습니다.
 
 ![Reverse Proxy](
 https://www.lesstif.com/download/attachments/20776817/image2014-7-19%2022%3A39%3A39.png?version=1&modificationDate=1405777029000&api=v2 "Reverse Proxy")

그러면 웹 서버를 견고하게 하기 위한 설정 방법을 알아 봅시다.

## server 정보 숨기기

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

### Server Token Off

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

### X Powered By 헤더 제거

apache httpd 의 경우 Header 지시자를 사용하면 브라우저로 전송하기 전에 모든 X-Powered-By 헤더를 삭제할 수 있습니다. (추천 방법)

```
Header unset X-Powered-By
```

PHP 는 php.ini 의 다음 항목을 off 로 설정하면 됩니다.

```
expose_php = Off
```

## 중요 파일 접근 차단

웹 서버의 컨텐츠 디렉터리에는 URL Re-writing 을 처리하는 *.htaccess* 나 워드프레스의 *wp-config.php*, 또 git이나 subversion의 형상 관리 메타 정보(.git, .svn)등의 중요 정보가 있을 수 있습니다.

공격자는 이런 파일을 내려 받아서 서버의 정보를 파악할 수 있으므로 다음 설정으로 중요 파일을 보호할 수 있습니다.

**httpd 2.2**

```
<DirectoryMatch .*\.(git|svn)/.*>
    Order deny,allow
    Deny From All
</DirectoryMatch>
    
<FilesMatch "^(htaccess|wp-config)">
    Order deny,allow
    Deny From All
</FilesMatch>

<FilesMatch "\.(inc|ini|conf|cfg)$">
    Order deny,allow
    Deny From All
</FilesMatch>
```

**httpd 2.4**

```
<DirectoryMatch .*\.(git|svn)/.*>
    Require all denied
</DirectoryMatch>
    
<FilesMatch "^(htaccess|wp-config)">
    Require all denied
</FilesMatch>

<FilesMatch "\.(inc|ini|conf|cfg)$">
    Require all denied
</FilesMatch>
```

**nginx**

```
location ~ /\.(ht|git|svn) {
    deny all;
}
location ~ /wp-conf* {
    deny all;
}
location ~ /.*\.(inc|ini|conf|cfg)$ {
    deny all;
}
```

## 관리자 서비스 접근 제한

웹 애플리케이션에도 추가되어야 하지만 웹 서버는 관리자 영역등 특정 url 을 차단할 수 있습니다.

다음은 아파치 톰캣의 관리자 context 인 /manager 에 접근을 차단하는 예제입니다.

**httpd 2.2**

```
<Location /manager>
    Order deny,allow
    Deny from all
    Allow from 127.0.0.1 192.168.0.0/24
</Location>
```

**httpd 2.4** 는 다음과 같이 설정합니다.

```
<Location /manager>
    Require host  127.0.0.1 192.168.0.0/24
</Location>
```

**nginx**

```
location /manager {
    satisfy any;

    allow 192.168.0.0/24;
    deny  all;
}
```

## 참고 자료
* [apache ServerTokens Directive](https://httpd.apache.org/docs/2.4/mod/core.html#servertokens)
* [nginx RESTRICTING ACCESS](https://www.nginx.com/resources/admin-guide/restricting-access/)
