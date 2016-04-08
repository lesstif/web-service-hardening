### 목차

<!-- toc -->

### SELiux 란

SELinux 는 레드햇 계열의 배포판의 사용자들이 가장 증오하는 기능일 겁니다.
검색 엔진에서 SElinux 를 치면 끄기가 자동 완성되니까요.

이것은 우리나라만이 아니라 외국도 마찬가지이며 SELinux 의 장점과 중단했을 때의  문제점을 널리 알리기 위한 [stop disabling SELinux](http://stopdisablingselinux.com/) 사이트도 생겨났습니다.

그러면 SELinux는 대체 무엇이며 왜 이리 사용자들의 증오를 받는 것일까요.

SELinux는 미국의 국가 안보국(NSA; National Security Agency; 또는 농담으로 No Such Agency 라고도 합니다.)에서 개발한 플라스크(Flask)라는 보안 커널을 리눅스에 이식한 커널 레벨의 보안 모듈입니다.

NSA는 다양한 운영체제에 강제 접근 통제를 구현했고 이를 리눅스 커널에도 포팅해서 결과물을 리눅스 커뮤니티에 기증해서 2.6 버전의 커널에 공식 포함되었습니다.

RHEL 기반의 배포판에는 4 버전부터 공식적으로 포함되었으며 다양한 제품들이 현재는 SELinux를 지원합니다.

그러면 SELinux 를 이해하기 위해 필수적인 지식인 강제 접근 통제와 임의 접근 통제에 대해서 알아 봅시다.

### 접근 통제

운영체제에서 접근 통제(Access Control)은 디렉터니라 파일, 네트워크 소켓 같은 시스템 자원을 적절한 권한을 가진 사용자나 그룹이 접근하고 사용할 수 있게 통제하는 것을 의미합니다.

접근 통제애서는 시스템 자원을 객체(Object)라고 하며 자원에 접근하는 사용자나 프로세스는 주체(Subject)라고 합니다.

즉 /etc/passwd 파일은 객체이고 이 파일에 접근해서 암호를 수정할 수 있게 해주는 passwd 라는 명령어는 주체입니다.

### 임의 접근 통제

임의 접근 통제(DAC;Discretionary Access Control)는 시스템 객체에 대한 접근을 사용자나 또는 그룹의 신분을 기준으로 제한하는 방법입니다.

사용자나 그룹이 객체의 소유자라면 다른 주체에 대해 이 객체에 대한 접근 권한을 설정할 수 있습니다.

여기서 임의적이라는 말은 소유자는 자신의 판단에 의해서 권한을 줄 수 있다는 의미이며 구현이 용이하고 사용이 간편하기 때문에 전통적으로 유닉스나 윈도우등 대부분의 운영체제의 기본 접근 통제 모델로 사용되고 있습니다.

임의적 접근 통제는 사용자가 임의로 접근 권한을 지정하므로 사용자의 권한을 탈취당하면 사용자가 소유하고 있는 모든 객체의 접근 권한을 가질 수 있게 되는 치명적인 문제가 있습니다.

임의적 접근 통제 방식의 보안 취약점에 대해 설명하기 위해 유닉스의 두 가지 주요 보안상 취약점에 대해서 알아 봅시다.

#### setuid 문제

사용자들은 자신의 암호를 passwd 명령어를 실행하여 변경할 수 있는데 사용자의 암호는 /etc/shadow 에 저장되며 이 파일은 루트만 접근 가능하지만 일반 사용자도 passwd 명령어로 자신의 암호를 변경할 수 있습니다.

또 상대방 호스트가 동작하는지 확인하기 위해 ping 명령어를 입력할 경우 ping 은 ICMP(Internet Control Message Protocol) 패킷을 사용하므로 루트 권한이 필요하지만 일반 사용자도 ping 명령어를 사용하여 상대 호스트의 이상 여부를 확인할 수 있습니다.

이는 passwd 나 ping 같이 실행시 루트 권한이 필요한 프로그램에는 setuid 비트라는 것을 설정하여 실행한 사용자가 누구든지 setuid 비트가 설정된 프로그램은 루트로 동작하도록 설계하였으므로 가능한 일입니다.
 
 ```
 ls -l /bin/ping/ /usr/bin/passwd
 ```

위 명령어를 실행하면 파일의 퍼미션 부분에 's' 표시가 있는 프로그램들은 setuid 비트가 켜졌다고 하며 이런 프로그램들은 실행시 루트 권한을 갖고 구동됩니다.

그러므로 일반 사용자들도 자신의 암호를 변경할 수 있지만 만약 setuid 비트가 붙은 프로그램에 보안 취약점이 있을 경우 공격자는 손쉽게 루트 권한을 획득할 수 있는 문제가 있습니다. 

이때문에 setuid 비트는 필요하지만 유닉스 시스템의 주요 보안 취약점이었으며 시스템 관리자의 골칫 덩어리이었습니다.

시스템에 있는 setuid 비트가 붙은 프로그램은 다음 명령어로 찾을 수 있습니다.

```sh
find /bin /usr/bin /sbin -perm -4000 -exec ls -ldb {} \;
```

#### 잘 알려진 포트 daemon 문제

잘 알려진 포트(well-known port) 는 특정한 쓰임새를 위해서 IANA에서 할당한 TCP 및 UDP 포트 번호의 일부로 1024 미만의 포트 번호를 갖게 됩니다.

예로 웹에 사용되는 http(80), 메일 전송에 사용되는 smtp(25), 파일 전송에 사용되는 ftp(20, 21) 등이 잘 알려진 포트입니다.

전통적으로 잘 알려진 포트는 루트만이 사용할 수 있으므로 데몬 서비스는 모두 루트의 권한으로 기동됩니다.

보안 문제는 여기에서 발생하며 루트로 구동되었으므로 만약 서비스 데몬이 보안 취약점이 있거나 잘못된 설정이 있을 경우 서비스 데몬을 통해서 공격자는 루트 권한을 획득하게 되며 시스템의 모든 자원에 접근이 가능해 집니다.

### 강제 접근 통제

강제 접근 통제(MAC - Mandatory Access Control)는 미리 정해진 정책과 보안 등급에 의거하여 주체에게 허용된 접근 권한과 객체에게 부여된 허용 등급을 비교하여 접근을 통제하는 모델입니다.

높은 보안을 요구하는 정보는 낮은 보안 수준의 주체가 접근할 수 없으며 소유자라고 할 지라도 정책에 어긋나면 객체에 접근할 수 없으므로 강력한 보안을 제공합니다.

MAC 정책에서는 루트로 구동한 http 서버라도 접근 가능한 파일과 포트가 제한됩니다.
즉 취약점을 이용하여 httpd 의 권한을 획득했어도 */var/www/html*, */etc/httpd* 등의 사전에 허용한 폴더에만 접근 가능하며 80, 443, 8080 등의 포트만 접근이 허용되므로 ssh로 다른 서버로 접근을 시도하는등 이차 피해가 최소화 됩니다.

단점으로는 구현이 복잡하고 어려우며 모든 주체와 객체에 대해서 보안 등급과 허용 등급을 부여하여야 하므로 설정이 복잡하고 시스템 관리자가 접근 통제 모델에 대해 잘 이해하고 있어야 합니다.

### SELinux 사용하기

SELinux는 아래와 같이 커널의 기본 기능으로 동작하며 모든 시스템 콜이 보안 정책을 확인해 접근 허용 여부를 판단하며  빠르게 처리하기 위해 보안 정책은 AVC(Access Vector Cache)라는 이름으로 커널 내부에서 캐싱합니다.

![SELinux 아키텍처](https://cloud.githubusercontent.com/assets/404534/12506805/d187db34-c134-11e5-85e3-76a71fd3ea9a.png "SELinux 아키텍처")


SELinux의 사용 여부는 *sestatus* 명령어를 통해 *Current mode* 에 enforcing 이 있는지 여부로 확인할 수 있습니다.

```
# sestatus

ELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28

```

SELinux 는 enforce, permissive, disable 3 가지 모드가 있으며 기본 설정은 enforce 이고 *setenforce 0* 명령어로 permissive 모드로 전환할 수 있습니다.

enforce 모드에서는 보안 정책에 위배되는 모든 액션이 차단되며 permissive mode 일 경우 경고 메시지만 내고 차단되지 않습니다.



### 참고 자료

* [SELinux 에러 메시지 및 문제 해결](https://www.lesstif.com/pages/viewpage.action?pageId=12943496)
* 