# Web Application Server

WAS(Web Application Server) 보안 고려 사항

<!-- toc -->

## 일반 사용자로 구동

일반적으로 WAS 는 웹 서버 기능도 내장하고 있는 경우가 많지만  웹 서버는 apache httpd 나 nginx 같은 제품을 사용하고 Reverse Proxy 로 WAS 와 연결하는 것이 보안과 확장성 측면에서 더 좋습니다.

![Reverse Proxy](https://cloud.githubusercontent.com/assets/404534/14357003/e18f6cee-fd21-11e5-89a0-b2e70b96a518.png "Reverse Proxy")

WAS 는 웹 서버를 겸용하지 않을 경우 보통 1024 이후의 포트를 사용하므로 root 가 아닌 일반 사용자로 구동해야 합니다.

## apache tomcat

### 서버 정보 숨기기

아파치 톰캣은 기본적으로 아래와 같이 HTTP 헤더에 Server 정보를 전송합니다. 

```sh
$ curl -v -I localhost:8080

Server: Apache-Coyote/1.1
Content-Type: text/html;charset=UTF-8
Transfer-Encoding: chunked
```

이 정보를 가리려면 conf/server.xml 내의 Connector 에 Server 정보를 설정해 주고 재구동하면 됩니다.

```xml
<Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" 
               Server="Apache" />
```



톰캣은 404 나 500 같은 에러 발생시 기본 에러 페이지에 다음과 같이 버전을 표시합니다.

![톰캣 버전 노출](https://user-images.githubusercontent.com/404534/27223549-e09af85c-52cb-11e7-98ad-1b7b561b65d3.png)

정보를 숨기려면 다음 명령을 실행합니다.

```sh
cd $TOMCAT_HOME/lib
jar xvf catalina.jar
vi org/apache/catalina/util/ServerInfo.properties
```

이제 프로퍼티에서 server.info 를 *Apache* 으로 변경하고 저장합니다.

```
server.info=Apache
server.number=7.0.47.0
server.built=Oct 18 2013 01:07:38
```

톰캣을 다시 구동하고 에러 페이지에 가서 버전이 가려지는지 확인합니다.



사용중인 톰캣의 버전을 확인하려면 터미널에서 다음 명령어를 입력합니다.

```sh
$ bin/version.sh

Server version: Apache Tomcat/8.0.44
Server built:   May 10 2017 17:21:09 UTC
Server number:  8.0.44.0
OS Name:        Linux
OS Version:     4.9.27-14.31.amzn1.x86_64
```



### manager 기능 접근 제어

톰캣에는 host-manager 라는 관리자 기능이 포함되어 있고 기본 설정은 비활성화입니다.

이 기능을 사용하면 웹에서 새로운 Context 를 deploy 할 수 있기 때문에 host-manager Context 를 활성화하는 경우가 있습니다.

사용할 경우 default Context name(manager) 을 변경하고 암호를 새로 설정해 사용합니다.

conf/user.xml

```xml
<role rolename="manager"/>
<user username="darren" password="ReallyComplexPassword" roles="manager"/>
```

그리고 manager 권한으로 연결할 수 있는 IP 를 제한합니다.

conf/Catalina/localhost/manager.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context antiResourceLocking="false" privileged="true">
    <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="192\.168\.152\.\d+|127\.0\.0\.1"/>
</Context>
```

 기본 암호가 설정되어 있고 접근 IP 제한이 안 걸려 있을 경우 외부에서 악성코드를 war 에 담아서 deploy 해서 톰캣을 악성 코드 경유지로 사용할 우려가 있습니다.
이런 공격을 당했을 경우 tomcat 이 일반 사용자로 구동되었다면 해당 계정을 삭제하면 되지만 root 로 구동되었다면 운영체제부터 새로 설치해야 할 수 있습니다.


## PHP-FPM

### cgi.fix_pathinfo 설정 문제

nginx 와 PHP-FPM 을 동일한 서버에서 같이 사용할 경우 PHP 가 PATH_INFO 환경 변수를 제대로 처리하지 못해서 서버에 hack.gif 라는 파일에 PHP 코드를 작성한 후에 이를 서버에 업로드하고 http://example.com/hack.gif/noexist.php 같이 호출하면 hack.gif 가 PHP 코드로 실행되는 심각한 보안 문제가 있었습니다.

최근 버전의 PHP-FPM 은 이 문제가 해결되었고 이를 확인하려면 PHP-FPM 의 pool 설정 파일(배포판마다 다르며 /etc/php/7.0/fpm/pool.d/www.conf, /etc/php-fpm.d/www.conf 등에 위치합니다.)에 다음과 같이 설정되어 있으면 됩니다.

```
security.limit_extensions = .php .php3 .php4 .php5
```

만약 다음과 같이 빈 값이 설정되어 있다면 심각한 보안 문제가 발생할 수 있으므로 php-fpm 을 내리고 즉시 수정해야 합니다. 

```
security.limit_extensions =
```

이와 별개로 사용자 파일 업로드는 white-list 방식으로 파일 유형을 제한하고 임의의 경로에 업로드하지 못하도록 해야 합니다.

그리고 업로드 파일이나 기타 리소스는 URL 을 통해서 바로 접근하는 것을 차단하고 애플리케이션의 특정 기능을 통해서만 접근할 수 있도록 제한해야 합니다.



## 참고 자료
* [Is the PHP option 'cgi.fix_pathinfo' really dangerous with Nginx + PHP-FPM?](http://serverfault.com/questions/627903/is-the-php-option-cgi-fix-pathinfo-really-dangerous-with-nginx-php-fpm)

