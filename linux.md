# 리눅스 설치와 패키지 관리 
<!-- toc -->

## 패키지 최소 설치

리눅스를 설치할 때 보안을 위해 고려해야 할 사항은 **운영 체제의 사용 용도**(웹 서버, 애플리케이션 서버, DB 서버등)에 맞게 필요한 패키지만 설치하는 것입니다.

예로 DB 전용 서버라면 Web Server 나 Samba 서버같은 사용하지 않는 패키지를 설치하지 않도록 해야 합니다.

FTP나 Samba, Bind 같이 사용하지 않는 서버 프로세스를 구동하는 것은 보안상 문제를 일으킬 수 있으며 운영체제 업그레이드나 패치가 힘들어 지며 재부팅시 오래 걸리는 문제가 있습니다.

특히 웹 서버나 메일 서버같이 DMZ 에 설치되는 서버는 최소 설치후 해당 패키지만 설치하는 것이 좋습니다.

![CentOS 최소 설치](https://cloud.githubusercontent.com/assets/404534/12508730/fe394fa8-c140-11e5-914c-ffc30fa64078.png)

**같이 보기**

* [CentOS minimal 설치후 추가 패키지 설치](https://www.lesstif.com/pages/viewpage.action?pageId=6979710)

## 패키지 매니저 사용

패키지를 설치할 때 source 를 다운 받아서 컴파일 해서 설치하는 것은 리눅스의 빌드 환경을 이해할 수 있고 최신 버전의 패키지를 사용할 수 있다는 장점이 있지만 버그나 보안 패치를 적용하기 어렵고 특히 서버의 대수가 늘어나면 많은 시간이 소요되는 단점이 있습니다.

새로운 패키지가 필요하거나 높은 버전의 패키지가 필요하다면 외부 저장소에서 먼저 찾아보는 것을 권장합니다.

저장소를 찾을 때는 제조사에서 직접 제공하는 저장소가 있는지 확인한 후에 없을 경우 3rd party 저장소를 찾는 것을 권장합니다.

예로 nginx 나  [MySQL](https://dev.mysql.com/downloads/mysql/) 은 RHEL/CentOS 와 Ubuntu용 저장소를 제공합니다.

RHEL/CentOS 는 다음 저장소가 유명합니다.

* [ELEL - Extra Packages for Enterprise Linux](http://fedoraproject.org/wiki/EPEL)
* [REMI](http://rpms.remirepo.net/)
* [Web Tatic](http://www.webtatic.com/)

Ubuntu는 [PPA - Personal Package Archives](https://launchpad.net/ubuntu/+ppas) 에서 패키지를 설치하면 됩니다. 


**같이 보기**

* [RHEL에 EPEL 과 Remi/WebTatic Repository 설치하기](https://www.lesstif.com/pages/viewpage.action?pageId=6979743)
* [nginx 패키지로 설치](https://www.lesstif.com/pages/viewpage.action?pageId=25100304#RHEL/CentOS에nginx설치-NginxRepo에서설치(추천))

## 미사용 패키지 삭제

이미 설치해서 사용중인 시스템이라면 사용하지 않는 패키지를 삭제하는 것도 좋습니다.

다음 명령어로 현재 설치된 패키지의 목록을 확인할 수 있습니다.

**RHEL**

```sh
yum list installed
```

**Ubuntu**

```sh
apt --installed list
```

특히 운영 시스템은 보안을 위해서 컴파일러등의 개발 도구를 삭제하는 것이 좋습니다.

**RHEL**
```sh
yum groupremove "Development tools"
```

** Ubuntu**
```bash
sudo apt-get purge gcc g++ gdb
```


## X Windows 삭제

이미 설치되어 운영을 하고 있는 서버라면 X-Windows 가 설치되어 있는지 확인하고 설치되어 있다면 삭제하는 게 좋습니다.

X-Windows 는 용량이 크고 의존성 있는 패키지가 많으므로 잦은 업데이트가 발생하게 되며 XDM 은 보안상 문제가 발생할 수 있습니다.

만약 X-Windows 가 없으면 명령어를 입력하지 못하는 운영자라면 계속 같이 일하는 것을 고민해 보기 바랍니다.

X-Windows 가 설치되어 있다면 먼저 *telinit 3* 명령어로 런 레벨을 3으로 변경한 후에 삭제해야 합니다.

우분투 서버는 X Windows 를 포함하고 있지 않으며 CentOS 에서는 다음 명령어로 X-Windows 패키지를 삭제할 수 있습니다.
```sh
yum groupremove "X Window System"
```
 
**같이 보기**

* [yum 주요 사용법 및 고급 사용법 (history 관리, plugin 사용, 트랜잭션 undo 등)](https://www.lesstif.com/pages/viewpage.action?pageId=6979667)


## 구동 프로세스 최소화

사용하지 않는데 부팅시 자동 구동되는 데몬 프로세스가 있는지 확인하고  있다면 중지하고 자동 구동을 끄는 것이 좋습니다.

CentOS 6 에서는 chkconfig 명령으로 서비스를 제어할 수 있습니다.

```bash
# chkconfig --list
 
auditd          0:off   1:off   2:on    3:on    4:on    5:on    6:off
blk-availability        0:off   1:on    2:on    3:on    4:on    5:on    6:off
cgconfig        0:off   1:off   2:off   3:off   4:off   5:off   6:off
cgred           0:off   1:off   2:off   3:off   4:off   5:off   6:off
crond           0:off   1:off   2:on    3:on    4:on    5:on    6:off
exim            0:off   1:off   2:on    3:on    4:on    5:on    6:off
```

*0:off   1:off   2:on    3:on    4:on    5:on    6:off* 의 의미는 run level 이 0, 1 일때는 구동되지 않고 2,3,4,5 일때는 구동된다는 의미입니다.

자동 시작을 끄려면 *chkconfig* 명령어 뒤에 서비스 명을 입력하고 off 옵션을 주면 됩니다.

```sh
chkconfig mysqld off
```

CentOS 7 은 systemd 관리 명령어인 systemctl 를 사용하여 서비스 목록을 확인할 수 있습니다.

```bash
# systemctl list-unit-files

UNIT FILE                                 STATE   
proc-sys-fs-binfmt_misc.automount         static  
dev-hugepages.mount                       static  
dev-mqueue.mount                          static  
proc-fs-nfsd.mount                        static  
proc-sys-fs-binfmt_misc.mount             static  
sys-fs-fuse-connections.mount             static  
sys-kernel-config.mount                   static  
sys-kernel-debug.mount                    static  
tmp.mount                                 masked
var-lib-nfs-rpc_pipefs.mount              static  
brandbot.path                             disabled
systemd-ask-password-console.path         static  
systemd-ask-password-wall.path            static  
session-1.scope                           static  
session-3.scope                           static  
arp-ethers.service                        disabled
auditd.service                            enabled 
```

불필요한 서비스는 *systemctl disable 서비스명* 을 사용하여 자동 시작을 중지할 수 있으며 활성화할 경우 *disable* 대신 *enable* 을 사용하면 됩니다.

다음 명령어는 mariadb 서비스의 자동 실행을 중지합니다.

```bash
systemctl enable mariadb
```

특정 서비스의 자동 실행 여부는 *systemctl is-enabled 서비스명* 을 사용하면 되며 다음 명령어는 nginx 웹 서버의 자동 구동 여부를 출력합니다.

```bash
> systemctl is-enabled nginx

disabled
```

우분투는 *sysv-rc-conf* 명령어로 프로세스를 확인할 수 있습니다

```bash
$ sudo sysv-rc-conf --list             
apparmor     S:on
beanstalkd   0:off      1:off   2:on    3:on    4:on    5:on    6:off
blackfire-ag 0:off      1:off   2:on    3:on    4:on    5:on    6:off
console-setu
cron        
cryptdisks  
cryptdisks-e 0:on       6:on    S:on
```

## 시스템을 최신 상태로 유지

패키지 관리자를 사용하여 시스템을 상시 업데이트하여 최신 상태로 유지해야 보안 취약점을 통한 공격에 대응할 수 있습니다.

RHEL은 yum 으로 시스템을 최신 상태로 유지할 수 있습니다. 

```bash
yum update
```

우분투는 아래 명령으로 패키지를 업데이트 할 수 있습니다.

```bash
apt-get update
apt-get upgrade
```


## 참고 자료

* [20 Linux Server Hardening Security Tips](http://www.cyberciti.biz/tips/linux-security.html)
