## 목차

Application Server 보안 고려 사항

## apache tomcat

## PHP-FPM

### cgi.fix_pathinfo 설정 문제
 
nginx 와 PHP-FPM 을 같이 사용할 경우 PHP 가 PATH_INFO 환경 변수를 제대로 처리하지 못해서 서버에 hack.gif 라는 파일에 PHP 코드를 작성한 후에 이를 서버에 업로드하고 http://example.com/hack.gif/noexist.php 같이 호출하면 hack.gif 가 PHP 코드로 실행되는 심각한 보안 문제가 있었습니다.

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
* 