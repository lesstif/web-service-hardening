### 목차 

* [방화벽으로 ssh 포트 보호](#방화벽으로-ssh-포트-보호)
* [fail2ban 사용](#fail2ban-사용)
* [강화된 인증 사용](#강화된-인증-사용)
  * [공개키 인증](#공개키-인증)
  * [2단계 인증](#2단계-인증)
* [베스천 호스트 사용](#베스천-호스트-사용)
* [참고 자료](#참고-자료)

원격지 연결에 사용하는 텔넷(telnet) 이나 rlogin 프로토콜은 오래되었고 보안상 취약한 프로토콜로 이제는 사용하면 안 됩니다.

ssh 는 암호화를 사용하여 세션을 보호하고 암호, 공개키, 챌린지 응답등 다양한 인증 방식을 제공하고 있으며 포트 포워딩을 제공하므로 X-Windows가 필요해도 ssh 포트만 열어도 사용할 수 있습니다.

많은 리눅스 배포판들이 오래전부터 텔넷 서버를 제거하고 ssh 서버를 기본 원격 로그인 서버로 사용했습니다.

그러면 sshd를 더 안전하게 사용할 수 있는 방법을 알아 봅시다.

### 방화벽으로 ssh 포트 보호

ssh의 기본 포트인 22 번 포트는 모든 IP에 대해서 열지 말고 허용된 곳에서만 연결을 허용하도록 해야 합니다.

다음은 192.168.10 대역에서 ssh 연결을 허용하는 iptables 의 설정 예제입니다.

```
-A INPUT -s 192.168.10.0/24 -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
```

### fail2ban 사용

Brute force Attack을 막기 위해 일정 횟수용 이상 접근을 시도한 IP 는 [fail2ban](http://www.fail2ban.org/) 을 사용하여 차단할 수 있습니다.

fail2ban은 침입 방지 시스템(IPS; Intrusion prevention system) 으로 */var/log/auth.log* 나 */var/log/apache/access.log* 를 모니터링하여 로그인을 시도한 IP 를 차단합니다.

다른 솔루션과 차이점은 iptables 같은 커널 레벨의 방화벽을 사용하여 로우 레벨에서 차단하는 점이며 ssh외에 http 나 기타 프로토콜에도 사용할 수 있습니다.

### 강화된 인증 사용

ssh의 암호 인증은 암호가 유출되면 무력화되므로 다른 방식의 인증을 사용하는 것이 좋습니다.
암호 인증을 사용하지 않으려면 */etc/ssh/sshd_config* 에 다음 항목을 설정하고 *service sshd restart* 를 실행하면 됩니다.

```
PasswordAuthentication no
```

**암호 인증을 중지하기전에 공개키나 2단계 인증을 설정**해 두어야 합니다.

#### 공개키 인증

공개키 방식의 인증은 키 쌍을 보유하고 있어야 하므로 암호 방식보다는 안전합니다.
*/etc/ssh/sshd_config* 에 다음과 같이 설정되어 있으면 공개키 인증이 가능합니다.(기본 설정)


```
PubkeyAuthentication yes
```

접근이 허용된 공개키는 원격 사이트의  *~/.ssh/authorized_keys* 에 저장되어 있습니다.

*ssh-copy-id* 명령어를 사용하면 원격지에 사용할 공개키를 등록할 수 있습니다.

```
ssh-copy-id -i ~/.ssh/id_rsa.pub myloginid@myhost.com
```

myloginid 는 로그인하려는 id 이고 myhost.com은 로그인하려는 서버의 주소입니다.


공개키 방식의 인증도 키쌍이 유출되면 다른 이가 접근 가능하며 더욱 견고하게 하려면 *~/.ssh/authorized_keys* 에 다음과 같이 *from* 키워드에 해당 공개키로 접근 가능한 client IP를 기술해 주면 됩니다.

```
from="192.168.10.2" ssh-rsa AAAAB3NzaC1yc2EA...AQWqz myemail@myhost.com
``` 

#### 2단계 인증

2단계 인증(2 factor authentication)을 사용하면 OTP(One Time Password)같이 추가 인증 수단을 통해 ssh 로 로그인할 수 있으므로 더욱 안전합니다.

제일 쉽게 사용할 수 있는 OTP는 [google authenticator](https://github.com/google/google-authenticator) 이며 ssh 서버에 설치한 후에 스마트 폰에 app을 설치하여 휴대폰에서 생성된 1회용 비밀번호로 원격지에 로그인할 수 있습니다.

더 자세한 내용은 [google authenticator 를 사용하여 Linux ssh 에 OTP 적용하기](https://www.lesstif.com/pages/viewpage.action?pageId=24444948) 을 참고하세요.

### 베스천 호스트 사용

Bastion 호스트는 보호된 네트워크에서 유일하게 외부에 노출되는 내외부 네트워크의 연결 호스트를 의미합니다.

운영등의 이유로 외부에서 내부 서버에 ssh 로 연결해야 할 필요가 있을 경우 바로 들어갈 수 있도록 하지 말고 베스천 호스트를 구성한 후에 이 서버를 통해서만 내부에 접근할 수 있도록 구성해야 합니다. 

![베스천 호스트](http://cloudacademy.com/blog/wp-content/uploads/2015/11/aws-bastion-host-1.png "베스천 호스트")

특히 AWS 등의 크라우드를 사용하는 경우 베스천 호스트를 통해 private network 에 위치한 서버에 연결하는 것을 권장합니다.


### 참고 자료
* [fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page)
* [Hardening ssh Servers](https://feeding.cloud.geek.nz/posts/hardening-ssh-servers/)
* [AWS Security: Bastion Host, NAT instances and VPC Peering](http://cloudacademy.com/blog/aws-bastion-host-nat-instances-vpc-peering-security/)