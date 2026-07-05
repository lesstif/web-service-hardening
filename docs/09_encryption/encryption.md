# 대칭키 암호화

<!-- toc -->


## 대칭키 알고리즘


대칭키(Symmetric key) 방식은 암호문을 생성(암호화)할 때 사용하는 키와 암호문으로부터 평문을 복원(복호화)할 때 사용하는 키가 동일한 암호 시스템으로 일반적으로 알고 있는 암호 시스템입니다.

하나의 키로 암호화와 복호화를 수행하므로 대칭키(Symmetric key) 또는 비밀키(Secret Key) 방식의 암호화라고 합니다.

대칭키 방식의 암호화는 데이타를 블록(Block) 단위로 처리하는 Block cipher 와 연속된 스트림으로 처리하는 Stream cipher 가 있으며 일반적으로 대칭키 암호화라고 하면 블록 방식의 암호화를 의미합니다.

블록 방식의 주요 알고리즘으로는 예전에 유닉스의 패스워드 파일을 암호화하는데 사용했지만 오래전에 뚫려서 이제는 절대 사용하면 안 되는 DES(Data Encryption Standard)와 현재 미국의 표준 암호 알고리즘인 AES가 있으며 국내에서 개발한 SEED와 ARIA , 그리고 유럽에서 개발한 IDEA 알고리즘 등이 있습니다.

> [!NOTE]
> 미국의 국가표준기술원(NIST)에서는 미정부와 군에서 사용할 암호화 알고리즘을 공모했습니다.
> 벨기에의 암호학자 둘이 자신들이 설계한 Rijndael 알고리즘을 제출했고 미국은 이를 표준 알고리즘으로 채택하고 개선하여 AES(Advanced Encryption Standard)가 되었습니다.
> 정부가 사용할 암호 알고리즘을 공모하고 외국인이 개발한 제품을 채택하는 열린 자세가 미국의 강력함의 원천인듯 합니다.

블록 암호화는 정해진 길이, 또는 가변 길이의 암/복호화 키를 사용하며 알고리즘마다 다릅니다.

예로 AES 는 128, 192, 256 비트의 키를 사용할 수 있으며 SEED 는 128를 OpenBSD 에서 많이 사용하는 Blowfish 는 32 ~ 448 비트의 키 길이를 사용합니다.

대칭키 암호는 공개키 방식에 대비하여 구현이 용이하고 CPU/메모리를 적게 사용하고 매우 빠른 암/복호화 속도를 장점으로 갖고 있으며 Windows의 BitLocker 와 OS X 의 FileVault 같이 디스크 전체를 암호화할 때도 대칭키 방식(AES) 을 사용합니다. 

하지만 대칭키 방식은 하나의 키만 사용하므로 상대방과 대칭키 기반으로 암호화 통신을 할 경우 상대방도 사전에 같은 키를 갖고 있어야 합니다.

[U-571](http://movie.daum.net/moviedb/main?movieId=984)이라는 영화는 2차 대전때 연합군이 독일군의 암호화 통신을 해독하기 위해 [에니그마 장비](https://upload.wikimedia.org/wikipedia/commons/3/3e/EnigmaMachineLabeled.jpg)를 탈취하는 특수 작전에 대해서 다루고 있습니다.

이처럼 대칭키 방식의 최대 단점은 암호화 통신을 위한 키 전달과 관리가 어려운 점입니다. 

가령 대칭키를 보내는 중간에 유출될 우려가 있고 이로 인해 암호화 통신을 도감청 당할 수 있으며 여러 상대방과 통신할 경우 각각의 대칭키를 관리하기(생성, 전달, 유출시 폐기)가 어렵고 여러 상대방과 같은 키를 사용할 경우 키가 노출되면 모든 채널과 새로 키를 전달해야 하는 치명적인 단점이 있습니다.


## 블록 암호 패딩(Padding)

블록 알고리즘은 입력 데이타를 정해진 블록으로 잘라서 암/복호화를 수행하며 블록의 크기는 알고리즘마다 다릅니다.
예로 AES 는 128 bit(16 byte)를 하나의 블록으로 처리합니다.

하지만 입력 데이타가 항상 블록의 배수일리는 없으므로 입력데이타를 블록 크기에 맞춰주는 작업이 필요하며 이를 패딩이라고 합니다.

즉 AES 를 사용할 경우 암호화할 데이타가 19 바이트일 경우 최소 공배수인 32 byte 로 맞춰주어야 하며 이 작업을 패딩이라고 지칭하며 여러 가지 패딩 방법이 있습니다.



## 블록 암호 운영 모드(Operation mode)

암호화를 수행할 때 개별 블록으로 나눠진 데이타를 처리하는 방식을 블록 암호 운영 모드라고 하며 여러 가지 모드가 있으며 대표적인 2가지 모드를 소개하겠습니다.

### ECB(electronic codebook)

가장 간단한 방식이며 나눠진 블록을 각각 암호화하는 방식으로 각 블록마다 같은 암호화 키를 사용하므로 보안이 취약한 문제가 있습니다.

![ECB 방식](https://upload.wikimedia.org/wikipedia/commons/c/c4/Ecb_encryption.png)

### CBC(cipher-block chaining)

CBC 는 ECB 의 단점을 해결하기 위한 방법으로 각 블록은 암호화되기 전에 이전 블록과 XOR 연산을 거치며 유추가 힘들도록 최초의 입력은 IV(initial vector) 라는 데이타와 연산을 하게 됩니다.

이때문에 CBC 암호화를 사용할 경우 대칭키와 IV 가 필요합니다.

![CBC 방식](https://upload.wikimedia.org/wikipedia/commons/d/d3/Cbc_encryption.png)


아래의 펭귄 그림이 암호화할 원본 데이타라고 생각해 봅시다.

![원본 데이타](https://upload.wikimedia.org/wikipedia/commons/5/56/Tux.jpg "원본 데이타")

좌측은 **ECB**, 우측은 **CBC** 방식으로 암호화한 결과이며 그림처럼 ECB 는 원문의 흔적이 남기 때문에 원문을 유추할 수 있으며 특히 원문에 같은 내용의 블록이 있을 경우 키를 알아내고 원문을 손쉽게 해독할 수 있는 취약점이 존재합니다.

![ECB 암호화](https://upload.wikimedia.org/wikipedia/commons/f/f0/Tux_ecb.jpg "ECB 암호화")
![CBC 암호화](https://upload.wikimedia.org/wikipedia/commons/a/a0/Tux_secure.jpg "CBC 암호화")

## 암호화 해시 함수

암호화 해시 함수(Cryptographic hash function)는 해시 함수의 일종으로 입력에 대해 정해진 길이의 출력을 내주며 해시 값으로부터 원래 값을 유추하기 어렵게 설계된 함수입니다.

즉 'abc' 라는 입력이 있을 경우 SHA 256 은 'edeaaff3f1774ad2888673770c6d64097e391bc362d7d6fb34982ddf0efd18cb' 라는 출력을 내게 됩니다.

임의의 입력이 있을 경우 정해진 길이의 출력을 하는 일방향 함수(One Way function)를 의미하며 동일한 입력에 대해 동일한 출력을 내야 하며 다른 값인데 동일한 출력을 내는 경우(해시 충돌)가 극히 드물어야 합니다.

유명한 알고리즘으로는 MD5(보안 문제때문에 암호용으로는 사용 되지 않음), SHA1(현재는 안전하지 않음), SHA-2(SHA256, SHA384, SHA512) 등이 있습니다.

## 비밀번호 암호화

비밀 번호 암호화는 일반적인 암호화와는 달리 **복호화를 할 필요가 없습**니다.

비밀 번호 암호화는 PBKDF 같은 함수를 사용하거나 또는 bcrypt 알고리즘을 구현한 메서드를 사용하면 됩니다,

직접 비밀번호 암호화하는 기능을 구현할 수 있겠지만 검증된 라이브러리를 사용하는 것을 권장하며 직접 만들어 쓸 경우 다음 사항을 꼭 지키십시요.

* 안전한 해시 함수(SHA2 이상) 사용
* Random 값을 생성한 후에 Salt 첨가.

### PBKDF

PBKDF(Password-Based Key Derivation Function) 는 사용자에게 문자열을 입력받아서 random 길이의 salt 를 추가하고 정해진 반복횟수만큼의 해시 함수를 돌려서 일정한 길이의 결과를 내주는 함수입니다. 

단방향 함수이므로 결과로 부터 원문을 유추할 수 없으므로 사용자 암호에 적용하기 좋은 알고리즘입니다.

PHP 는 [hash_pbkdf2](http://php.net/manual/en/function.hash-pbkdf2.php) 를 사용하면 됩니다.


```php
<?php
$password = "mypasswd";

$iterations = 1000;

// $salt = null; 이면 동일한 hash 출력
$salt = openssl_random_pseudo_bytes(16);

$hash = hash_pbkdf2("sha256", $password, $salt, $iterations, $length = 20);

echo $hash . "\n";

```

### bcrypt

bcrypt 는 비밀번호 해시에 사용하기 위해 만들어진 알고리즘으로 OpenBSD 에 기본 탑재되어 있습니다.

PHP 는 [password_hash](http://php.net/manual/en/function.password-hash.php) 의 두 번째 파라미터를 *PASSWORD_BCRYPT* 로 지정하면 됩니다.

> [!NOTE]
> PHP 5.5 이상은 기본 값이 bcrypt 이므로 *PASSWORD_DEFAULT* 로 설정해도 됩니다.

```php
<?php

echo password_hash("strong_password", PASSWORD_DEFAULT )."\n";
```

Java 는 [spring security 에 BCrypt](https://docs.spring.io/spring-security/site/docs/current/apidocs/org/springframework/security/crypto/bcrypt/BCrypt.html) 가 구현되어 있으므로 이것을 사용하면 됩니다.

```java
import import org.springframework.security.crypto.bcrypt.BCrypt;

// 사용자 입력 암호
String plain_password = "password_1234";

// pw_hash 는 bcrypt 로 암호화된 비밀번호가 저장되며 이 값을 DB 에 저장하면 됩니다.
String pw_hash = BCrypt.hashpw(plain_password, BCrypt.gensalt(10));

// 사용자가 입력한 암호 검증
String candidate_password = "password_123";
String stored_hash = pw_hash;

if (BCrypt.checkpw(candidate_password, stored_hash))
    System.out.println("It matches");
else
    System.out.println("It does not match");
```

## 키 관리

데이타를 안전하게 관리하기 위해서 가장 중요한 부분중 하나는 암호화한 키를 어떻게 안전하게 보관하는지 입니다.

키를 데이타 파일로 저장할 경우 키와 데이타만 손에 넣으면 모두 해독 가능한 위험이 있습니다.

신용 카드 정보등 아주 중요한 데이타를 보관해야 한다면 HSM(Hardware Security Module) 처럼 키를 장비에서 생성하고 외부에 유출되지 않는 안전한 장비 도입을 검토해 볼 필요가 있습니다.

![Luna HSM](http://www.tssl.com/tsslweb/wp-content/uploads/2014/11/product_safenet_luna_sp2.png "Luna HSM")

유명한 HSM 제품인 LunaHSM 은 [Amazon Web Service의 CloudHSM](https://aws.amazon.com/ko/cloudhsm/) 에 적용되어 있고 또 다른 제품인 Thales nShield 는 [MS Azure의 Key Vault](https://docs.microsoft.com/ko-kr/azure/key-vault/key-vault-hsm-protected-keys) 에 적용되어 있습니다.


## 데이타 암호화 프로세스

데이타 암호화와 복호화 작업을 누가 수행하는지에 따라 암호화 프로세스가 달라져야 합니다.

### 사용자가 복호화

예로 사이트 로그인 정보를 관리해 주는 lastpass.com 서비스를 생각해 봅시다.
로그인 정보는 lastpass 서버에 암호화해서 보관되며 lastpass가 복호화를 하면 안 됩니다.

이때 필요한 방법은 PBKDF2같이 비밀번호 기반으로 암호화하는 방법으로 사용자로부터 마스터 암호를 입력받고 PBKDF2 함수를 수행해서 
안전한 대칭키와 Initial Vector 를 생성해서 복호화합니다.

암호화와 복호화는 사용자의 비밀번호를 기반으로 이루어지며 사용자만 수행해야 합니다..

이 방법은 암호화된 사용자 데이타마다 비밀번호가 다르므로 서버는 키 관리가 힘들지 않고 실제 데이타/암복호화는 사용자 PC에서 이루어지는 경우가 많으므로 서버의 부담이 적습니다.

### 서버에서 복호화
 
 사용자의 신용 카드 정보를 보관하고 매 달마다 결제한다고 생각해 봅시다.
 
 서버는 정보를 암호화하여 안전하게 보관해야 하며 매달 결제일마다 복호화하여 카드 정보를 알아내야 합니다.
 
 데이타의 암호화/복호화가 서버에서 이루어지므로 보안에 매우 신경써야 하며 특히 암호화 키를 잘 관리해야 합니다.
 
 이 모델을 사용할 경우 HSM을 도입하여 안전하게 키 관리하는것이 좋습니다.
 

## 데이타 암호화 예제

### Java

Java 는 JCE/JCA 라는 암복호를 하기 위한 표준이 있고 이를 구현한 JCE Provider 가 필요합니다.

권장하는 JCE 프로바이더는 [Bouncy Castle](https://www.bouncycastle.org/java.html) 입니다.

> **Tip** 기본 JDK 의 JCE 정책은 미국외에서는 강력한 키 길이(AES256, RSA2048)를 사용할 수 없어서 *"java.lang.SecurityException: Unsupported keysize or algorithm parameters"* 또는 *"java.security.InvalidKeyException: Illegal key size"* 에러가 발생합니다.
 
이를 해결하려면 Oracle 에서 "Unlimited Strength Jurisdiction Policy Files" 파일을 다운로드 받아서 $JRE/lib/security/ 에 복사해야 합니다.

**AES 256 암호화**

```java
String input = "Hello World";

KeyGenerator gen = KeyGenerator.getInstance("AES");
gen.init(256);  // 256 key length

// 대칭키 생성
SecretKey key = gen.generateKey(); 

// Initial Vector 생성
SecureRandom sr = SecureRandom.getInstance("SHA1PRNG");
byte[] iv = new byte[16];
sr.nextBytes(iv);		

// 암호화
Cipher c = Cipher.getInstance("AES/CBC/PKCS5Padding");
                
IvParameterSpec spec = new IvParameterSpec(iv);
        
c.init(Cipher.ENCRYPT_MODE, key, spec);
        
// 암호화된 데이타
byte[] encData = c.doFinal(input.getBytes());

// 복호화
Cipher d = Cipher.getInstance("AES/CBC/PKCS5Padding");

d.init(Cipher.DECRYPT_MODE, key, spec);
byte[] decData = d.doFinal(encData);

System.out.println("ORG: " + input);
System.out.println("DEC: " + new String(decData));

```



### PHP 

PHP는 openssl extension 을 사용하여 대칭키 암호를 할 수 있습니다. (mcrypt 는 오래됐고 버그가 많아서 PHP 7 부터 지원이 중지됩니다.)

```php
<?php

$key = openssl_random_pseudo_bytes(32);

$iv = openssl_random_pseudo_bytes(16);

$data = 'Hello World';

$enc = openssl_encrypt($data, 'AES-256-CBC', $key, 0, $iv);

echo base64_encode($enc) . "\n";

$dec =  openssl_decrypt($enc, 'aes-256-cbc', $key, 0, $iv);

echo $dec . "\n";
```

# 참고 자료

* [KISA 암호 키 관리 안내서](http://seed.kisa.or.kr/iwt/ko/guide/EgovGuideDetail.do?bbsId=BBSMSTR_000000000011&nttId=83&pageIndex=1&searchCnd=&searchWrd=)
