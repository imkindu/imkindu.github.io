---
title: 安全和加密（三）：CA
author: imkindu
date: 2017-09-08 11:30:00
categories: linux
tags:
- 	CA
---

CA认证


<!--more-->

---

## 搭建私有CA

openssl 配置文件：/etc/pki/tls/openssl.cnf
     
该配置文件中以 "[配置段]",的形式配置相关信息

```nohighlight
[ root@centos69 ~ ]# cat /etc/pki/tls/openssl.cnf 
####################################################################
[ ca ]
default_ca	= CA_default		# The default ca section            默认CA配置段
####################################################################
[ CA_default ]
dir		= /etc/pki/CA		# Where everything is kept              CA工作目录
certs		= $dir/certs		# Where the issued certs are kept   签发过的证书
crl_dir		= $dir/crl		# Where the issued crl are kept         吊销列表
database	= $dir/index.txt	# database index file.              数据库索引文件
#unique_subject	= no			# Set to 'no' to allow creation of  
					# several ctificates with same subject.
new_certs_dir	= $dir/newcerts		# default place for new certs.  新颁发的证书存放位置

certificate	= $dir/cacert.pem 	# The CA certificate                CA自己的证书
serial		= $dir/serial 		# The current serial number         CA证书序列号（颁发到第几个了）
crlnumber	= $dir/crlnumber	# the current crl number            吊销证书序列号
					# must be commented out to leave a V1 CRL
crl		= $dir/crl.pem 		# The current CRL                       
private_key	= $dir/private/cakey.pem# The private key               CA自己的私钥
RANDFILE	= $dir/private/.rand	# private random number file

x509_extensions	= usr_cert		# The extentions to add to the cert

# Comment out the following two lines for the "traditional"
# (and highly broken) format.
name_opt 	= ca_default		# Subject Name options
cert_opt 	= ca_default		# Certificate field options

# Extension copying option: use with caution.
# copy_extensions = copy

# Extensions to add to a CRL. Note: Netscape communicator chokes on V2 CRLs
# so this is commented out by default to leave a V1 CRL.
# crlnumber must also be commented out to leave a V1 CRL.
# crl_extensions	= crl_ext
default_days	= 365			# how long to certify for           证书有限期
default_crl_days= 30			# how long before next CRL          吊销列表的有限期限
default_md	= default		# use public key default MD
preserve	= no			# keep passed DN ordering
# A few difference way of specifying how similar the request should look
# For type CA, the listed attributes must be the same, and the optional
# and supplied fields are just that :-)
policy		= policy_match
# For the CA policy
[ policy_match ]
countryName		= match
stateOrProvinceName	= match
organizationName	= match
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional

# For the 'anything' policy
# At this point in time, you must list all acceptable 'object'
# types.
[ policy_anything ]
countryName		= optional
stateOrProvinceName	= optional
localityName		= optional
organizationName	= optional
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional

####################################################################
```


### 创建所需要的文件

```nohighlight
[ root@centos69 ~ ]# cd /etc/pki/CA/
[ root@centos69 CA ]# ll
total 16
drwxr-xr-x. 2 root root 4096 Mar 23 05:46 certs
drwxr-xr-x. 2 root root 4096 Mar 23 05:46 crl
drwxr-xr-x. 2 root root 4096 Mar 23 05:46 newcerts
drwx------. 2 root root 4096 Mar 23 05:46 private


[ root@centos69 CA ]# touch index.txt       # 生成证书索引数据库文件
[ root@centos69 CA ]# 
[ root@centos69 CA ]# echo 01 > serial      # 指定第一个颁发证书的序列号
[ root@centos69 CA ]# 
[ root@centos69 CA ]# ll
total 20
drwxr-xr-x. 2 root root 4096 Mar 23 05:46 certs
drwxr-xr-x. 2 root root 4096 Mar 23 05:46 crl
-rw-r--r--. 1 root root    0 Sep  1 14:26 index.txt
drwxr-xr-x. 2 root root 4096 Mar 23 05:46 newcerts
drwx------. 2 root root 4096 Mar 23 05:46 private
-rw-r--r--. 1 root root    3 Sep  1 14:26 serial
```


### 生成私钥


```nohighlight
[ root@centos69 CA ]# (umask 066; openssl genrsa -out private/cakey.pem 2048)
Generating RSA private key, 2048 bit long modulus
...................+++
..................................................................................................+++
e is 65537 (0x10001)
[ root@centos69 CA ]# ll private/
total 4
-rw-------. 1 root root 1675 Sep  1 14:30 cakey.pem
```

### 生成自签名证书

`openssl req` 用于生成证书请求，以让第三方权威机构CA来签发，生成我们需要的证书。

> openssl req -new -x509 –key /etc/pki/CA/private/cakey.pem-days 7300 -out /etc/pki/CA/cacert.pem

- -new: 生成新证书签署请求
- -X509: 专用于CA生成自签证书
- -key: 生成请求时用到的私钥文件
- -days n：证书的有效期限
- -out /PATH/TO/SOMECERTFILE: 证书的保存路径

```nohighlight
[ root@centos69 CA ]# openssl req -new -x509 -key private/cakey.pem -days 3650 -out cacert.pem 
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN                # 国家
State or Province Name (full name) []:hubei         # 省名
Locality Name (eg, city) [Default City]:wuhan       # 城市
Organization Name (eg, company) [Default Company Ltd]:tianyu        # 公司
Organizational Unit Name (eg, section) []:development               # 部门
Common Name (eg, your name or your server's hostname) []:ca.imkindu.com     # 服务器名，要和自己的域名保持一次，这里是给自己颁的
Email Address []:caadmin@imkindu.com                # 邮箱
```

### 查看证书

证书的文件名和配置文件相同

```nohighlight
[ root@centos69 CA ]# cat cacert.pem 
-----BEGIN CERTIFICATE-----
MIID9zCCAt+gAwIBAgIJAIWv3Pdkig3GMA0GCSqGSIb3DQEBBQUAMIGRMQswCQYD
VQQGEwJDTjEOMAwGA1UECAwFaHViZWkxDjAMBgNVBAcMBXd1aGFuMQ8wDQYDVQQK
DAZ0aWFueXUxFDASBgNVBAsMC2RldmVsb3BtZW50MRcwFQYDVQQDDA5jYS5pbWtp
bmR1LmNvbTEiMCAGCSqGSIb3DQEJARYTY2FhZG1pbkBpbWtpbmR1LmNvbTAeFw0x
NzA5MDEwNjQzNThaFw0yNzA4MzAwNjQzNThaMIGRMQswCQYDVQQGEwJDTjEOMAwG
A1UECAwFaHViZWkxDjAMBgNVBAcMBXd1aGFuMQ8wDQYDVQQKDAZ0aWFueXUxFDAS
BgNVBAsMC2RldmVsb3BtZW50MRcwFQYDVQQDDA5jYS5pbWtpbmR1LmNvbTEiMCAG
CSqGSIb3DQEJARYTY2FhZG1pbkBpbWtpbmR1LmNvbTCCASIwDQYJKoZIhvcNAQEB
BQADggEPADCCAQoCggEBANnYPqqf0TykyCgUm6Y2HYdrtRmiDLd1UZPMCniF+S95
Vr4E9RFCXZ89q9jZG1wIPK05EF4v56KSNUH3jTjWx021l99r3iXxvipUm3fnDlCA
yVh6CZqDRy0uPEI87lxZPQ0lYk1O3g/IGRTZJDBD1RBhE1A4sz87jabgmVhAmeFm
iwfSSIcOz1gXVotcfTJYv3FCK85XqbpkKVt9jlPd0KsOaXdzR5h7IxksQaVJ7YBZ
/bSK8BnCN3oUF4MrzKghYB+nrqwlA+f3Im8pJQzh1YQM09asEuj/O9PxfywiWn/9
5kMnfn1CXtfw1BCrjj5bMjtBs9XJbO834OKAJ6vw//kCAwEAAaNQME4wHQYDVR0O
BBYEFHYIkdBVCGVRzOHXk7XtOFYFto2ZMB8GA1UdIwQYMBaAFHYIkdBVCGVRzOHX
k7XtOFYFto2ZMAwGA1UdEwQFMAMBAf8wDQYJKoZIhvcNAQEFBQADggEBAD1inoC2
bRbzm2xJ/Ev8oNn4oXtoS6taabHx6juCKWbN42XtsBHEpsxyDTBq4nzn6cHO6yzd
c81+HTYMzywGzpNrSOemS7hnlWVr0bSJZFgVgblakt4vLQO+PB8CDnUd6Atao+sj
FRnT5yyFnyrXILOxyKxraYatwj0VhvAqx0uYqftxsHYA8sARwuzrqfKEdM7I/re7
A/TGSRgWBuK9fanoQ7hcuRAt4dt2ukly8a3+3RGCxnkCU99/cnzS8nANdBTb0QxR
6+VLAaALuOFolqw5T3Ks8kQQpCFk287X4cTgFEME278nIMtAUtFsFoBHfoC36EKT
evQ4wjERSJrOiWU=
-----END CERTIFICATE-----
```

### cer查看

导出到windows，注意后缀是cer可以查看证书内容

<div align="center">
![](/images/linux/ca01.png)
</div>

## 申请CA

### 创建密钥对

```nohighlight
[ root@centos69 CA ]# (umask 066; openssl genrsa -out imkindu.prikey 2048)
Generating RSA private key, 2048 bit long modulus
..................+++
......................................+++
e is 65537 (0x10001)
[ root@centos69 CA ]# ll
total 28
-rw-r--r--. 1 root root 1436 Sep  1 14:43 cacert.pem
drwxr-xr-x. 2 root root 4096 Mar 23 05:46 certs
drwxr-xr-x. 2 root root 4096 Mar 23 05:46 crl
-rw-------. 1 root root 1675 Sep  1 15:12 imkindu.prikey
-rw-r--r--. 1 root root    0 Sep  1 14:26 index.txt
drwxr-xr-x. 2 root root 4096 Mar 23 05:46 newcerts
drwx------. 2 root root 4096 Sep  1 14:30 private
-rw-r--r--. 1 root root    3 Sep  1 14:26 serial

[ root@centos69 CA ]# cat imkindu.prikey
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEApwdAA2rKYkwmRbp3qOI/k1OdwrSVXARghdKuAId+LODMQ5lB
mEZ8Ucw2n03vvZ4YAMgATdxA8n+xkgkCUMBqsjTnV4D+Ame3Wi0YcZmI71UW3pod
75lpCUwkK5TD4UFvD5PchvfziWx0rh7doWOe0cD6/BaL+5BlGHebEF+8iemFS7Xj
6tPNz9wuoYP5QkJ1wxnO6k6LyXWtHOk4bg4leFt/Oy7wSLLj4cM6frvYoV0r1t8s
tpvLSWnsCt9/PT2kzcqNq03pCDwuQTotoWX176Y9Vcd7sZl7RXLjGmQ6zRF0o5Fx
Jq2GfRJIn5NkDN/kXwwe3WuiG7tgrwogV4qb5QIDAQABAoIBAAasGXiJeZA3roe2
jTUn5JZEDtdKU3Ubj6eI5P6MaxPr3v0MUDx/BFRYLg5rFJqkiBzv4GM72zRUuYk1
5uvG4/w+dMdgFcWO0xo9Fu7izT+STJmT2oJJxJJkgkVjaffDn2Yl5/dUTFw/AuI5
xWy/CAclCGGtnOXtvLwfewhKasOvieYPs2To9z/WtdbKwma9CsFTE0H+Mq+vqCok
gyiVHf7CCpySxDp3OJqVCHErsBVq0BMrkr/9FnOCZY+Kd4Qcz2IlHDMER4f3+jS8
pCJdjBOBf1qb9ENAFOCswsYe6g1fTItGGuxr9kmDBorTZSG5RdjCGArf/Eql9tsw
ItPMkDkCgYEA1Y5WlCZywGVR6B8afyMQ5wjFWlluEfemFaiSXv66GHiXj5K02lnB
2SV3bSyNR9taFZFK/+KBNVRPOQESbtHcmQoaSWmsb2nml7Tp/UDW+A8tn8yWw0kH
AJK6eFUqGWKjJu8XuMv0H97BqV4So+rhde8CprB23bFeEjnKDoIn/ZsCgYEAyDmZ
DsdHBrAqrHMZX2DDgGpb1HxlM20PKorDdWRZeGD1rCBr7SJuGgJYafO1lMW6lJxy
J2U7ZfwP1qNR8qLjijRJwTZvvzeDVYyfHRzsPCkRqXSHE58aMRcVK7YAz9XcnNq+
WHlXjAYIpblkl7x0NL3hGplMdrZqdUN6ixe+JH8CgYB9zMF3uEZ0y7q6MEhdiHyW
fHY1SOUsNGRj8c93ojph2/f8HYHn9mPY1NdLOqlnIPIqLlKt9fIDRkz82YLQQVPf
2zGs+VEYuJub1njYNO/tZJONxOky1LwJPGYYKKMKHS7a6pFgzNRcSc5vRPlaEi0K
WeeH5f+/jJJLzjsW3NlN7QKBgFtfja3k21D+DDtuu2F/czijUQ0DR9vUJVuwv8pO
5VW+Sd8nXJl3YO+VqmuPwIoIQkGXs7CuzhCYm1HEbp1gIJ7thcsa4JxO5SyhY+uR
S22ZAGpot0wJC5bjhdHQ2UX/vxIF8V/G4GEST9fxZyqn4hA/pv7QfsieLq8dAEuB
plBZAoGAdHnKQFlwSdy2oDCo9dWHiVWusHLBdZ218BU/hkfCzBMi/M1OrAWyeFU3
/267deY+uKhEUJgUfrqQw5n+nmZyUOqzcYY1yMXMHbyiXhH1kEkwtPlEDAa7w9we
+o59IxSMuUNtBTg5tl1oYLULoEwIj27RKVOn43UQvqPXfGKfXVw=
-----END RSA PRIVATE KEY-----
```

### 创建申请证书

与CA自己认证不同的是，没有-x509，表示申请

CSR是`Certificate Signing Request`的英文缩写，即证书请求文件

```nohighlight
[ root@centos69 CA ]# openssl req -new -key imkindu.prikey -days 365 -out imkindu.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN                    国家
State or Province Name (full name) []:hubei             省份
Locality Name (eg, city) [Default City]:wuhan           城市
Organization Name (eg, company) [Default Company Ltd]:douyu     公司
Organizational Unit Name (eg, section) []:hr            部门
Common Name (eg, your name or your server's hostname) []:www.douyu.tv   # 注意证书和网址域名
Email Address []:douyu@email.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:                                加密证书请求（证书会在网上传输）
An optional company name []:
```


### 请求发送给CA

> scp 到CA服务器

### CA签证

openssl ca 命令就是用于签证证书

> openssl ca -in /file.csr -out /file -days 365

- in 申请
- out 签售之后证书存放位置
- days 签证期限


`crt`是标准的证书后缀

签证时，需要确认证书信息，在openssl.cnf 中配置的有CA必须符合的标准，满足时才能签证

```nohighlight
[ root@centos69 CA ]# openssl ca -in imkindu.csr -out newcerts/imkindu.crt -days 365
Using configuration from /etc/pki/tls/openssl.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 1 (0x1)
        Validity
            Not Before: Sep  1 07:25:38 2017 GMT
            Not After : Sep  1 07:25:38 2018 GMT
        Subject:
            countryName               = CN
            stateOrProvinceName       = hubei
            organizationName          = douyu
            organizationalUnitName    = hr
            commonName                = www.douyu.tv
            emailAddress              = douyu@email.com
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                98:BB:45:C4:12:7E:DE:75:29:70:DB:35:E1:19:2D:D5:3B:72:37:F2
            X509v3 Authority Key Identifier: 
                keyid:76:08:91:D0:55:08:65:51:CC:E1:D7:93:B5:ED:38:56:05:B6:8D:99

Certificate is to be certified until Sep  1 07:25:38 2018 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

数据库更新

```nohighlight
[ root@centos69 CA ]# cat index.txt
V	180901072538Z		01	unknown	/C=CN/ST=hubei/O=douyu/OU=hr/CN=www.douyu.tv/emailAddress=douyu@email.com
```

### 查看证书

```nohighlight
[ root@centos69 CA ]# cat newcerts/imkindu.crt 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 1 (0x1)
    Signature Algorithm: sha1WithRSAEncryption
        Issuer: C=CN, ST=hubei, L=wuhan, O=tianyu, OU=development, CN=ca.imkindu.com/emailAddress=caadmin@imkindu.com
        Validity
            Not Before: Sep  1 07:25:38 2017 GMT
            Not After : Sep  1 07:25:38 2018 GMT
        Subject: C=CN, ST=hubei, O=douyu, OU=hr, CN=www.douyu.tv/emailAddress=douyu@email.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:a7:07:40:03:6a:ca:62:4c:26:45:ba:77:a8:e2:
                    3f:93:53:9d:c2:b4:95:5c:04:60:85:d2:ae:00:87:
                    7e:2c:e0:cc:43:99:41:98:46:7c:51:cc:36:9f:4d:
                    ef:bd:9e:18:00:c8:00:4d:dc:40:f2:7f:b1:92:09:
                    02:50:c0:6a:b2:34:e7:57:80:fe:02:67:b7:5a:2d:
                    18:71:99:88:ef:55:16:de:9a:1d:ef:99:69:09:4c:
                    24:2b:94:c3:e1:41:6f:0f:93:dc:86:f7:f3:89:6c:
                    74:ae:1e:dd:a1:63:9e:d1:c0:fa:fc:16:8b:fb:90:
                    65:18:77:9b:10:5f:bc:89:e9:85:4b:b5:e3:ea:d3:
                    cd:cf:dc:2e:a1:83:f9:42:42:75:c3:19:ce:ea:4e:
                    8b:c9:75:ad:1c:e9:38:6e:0e:25:78:5b:7f:3b:2e:
                    f0:48:b2:e3:e1:c3:3a:7e:bb:d8:a1:5d:2b:d6:df:
                    2c:b6:9b:cb:49:69:ec:0a:df:7f:3d:3d:a4:cd:ca:
                    8d:ab:4d:e9:08:3c:2e:41:3a:2d:a1:65:f5:ef:a6:
                    3d:55:c7:7b:b1:99:7b:45:72:e3:1a:64:3a:cd:11:
                    74:a3:91:71:26:ad:86:7d:12:48:9f:93:64:0c:df:
                    e4:5f:0c:1e:dd:6b:a2:1b:bb:60:af:0a:20:57:8a:
                    9b:e5
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: 
                CA:FALSE
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                98:BB:45:C4:12:7E:DE:75:29:70:DB:35:E1:19:2D:D5:3B:72:37:F2
            X509v3 Authority Key Identifier: 
                keyid:76:08:91:D0:55:08:65:51:CC:E1:D7:93:B5:ED:38:56:05:B6:8D:99

    Signature Algorithm: sha1WithRSAEncryption
         3f:58:52:d6:c5:39:d8:df:12:2f:d6:9a:81:71:4e:ca:2c:25:
         d5:04:fb:e8:f2:f7:20:92:7d:82:8a:6c:64:55:b9:4d:ff:05:
         ff:5c:f7:ea:3d:e9:de:d9:1b:54:54:ab:37:db:2f:e9:48:57:
         d0:47:d1:22:40:fc:e9:b8:dd:f9:b0:be:0c:f1:e1:72:b7:f1:
         fc:bd:5c:00:17:a1:be:fb:28:33:2a:f6:c3:0b:6b:a8:bd:a6:
         c2:ec:bf:7a:71:68:07:12:28:d2:69:a0:42:cf:21:c3:94:ef:
         b7:a9:51:67:a1:b0:27:ef:de:d6:72:1f:4a:67:92:dd:a9:f8:
         27:27:60:62:5d:32:ef:5e:02:5e:49:7c:24:79:f5:14:5a:65:
         3c:d3:df:8b:03:09:5a:fe:05:36:cc:2c:7e:b0:89:aa:78:8a:
         bc:9e:96:72:47:17:91:23:fc:54:9d:23:b6:1f:0f:8d:6d:55:
         a5:6f:81:6f:ec:fd:2d:cc:75:3c:7f:27:c0:3c:d7:fd:ef:fb:
         38:34:ab:f4:74:df:8a:d4:bb:a3:ad:81:f8:54:95:57:60:94:
         bd:22:c6:a1:56:6b:c9:11:ba:00:96:44:62:98:1d:06:64:94:
         7a:d3:46:83:bf:f6:3c:92:4d:60:7e:28:b2:c0:29:aa:07:0a:
         2a:98:1d:2e
-----BEGIN CERTIFICATE-----
MIID+TCCAuGgAwIBAgIBATANBgkqhkiG9w0BAQUFADCBkTELMAkGA1UEBhMCQ04x
DjAMBgNVBAgMBWh1YmVpMQ4wDAYDVQQHDAV3dWhhbjEPMA0GA1UECgwGdGlhbnl1
MRQwEgYDVQQLDAtkZXZlbG9wbWVudDEXMBUGA1UEAwwOY2EuaW1raW5kdS5jb20x
IjAgBgkqhkiG9w0BCQEWE2NhYWRtaW5AaW1raW5kdS5jb20wHhcNMTcwOTAxMDcy
NTM4WhcNMTgwOTAxMDcyNTM4WjBxMQswCQYDVQQGEwJDTjEOMAwGA1UECAwFaHVi
ZWkxDjAMBgNVBAoMBWRvdXl1MQswCQYDVQQLDAJocjEVMBMGA1UEAwwMd3d3LmRv
dXl1LnR2MR4wHAYJKoZIhvcNAQkBFg9kb3V5dUBlbWFpbC5jb20wggEiMA0GCSqG
SIb3DQEBAQUAA4IBDwAwggEKAoIBAQCnB0ADaspiTCZFuneo4j+TU53CtJVcBGCF
0q4Ah34s4MxDmUGYRnxRzDafTe+9nhgAyABN3EDyf7GSCQJQwGqyNOdXgP4CZ7da
LRhxmYjvVRbemh3vmWkJTCQrlMPhQW8Pk9yG9/OJbHSuHt2hY57RwPr8Fov7kGUY
d5sQX7yJ6YVLtePq083P3C6hg/lCQnXDGc7qTovJda0c6ThuDiV4W387LvBIsuPh
wzp+u9ihXSvW3yy2m8tJaewK3389PaTNyo2rTekIPC5BOi2hZfXvpj1Vx3uxmXtF
cuMaZDrNEXSjkXEmrYZ9Ekifk2QM3+RfDB7da6Ibu2CvCiBXipvlAgMBAAGjezB5
MAkGA1UdEwQCMAAwLAYJYIZIAYb4QgENBB8WHU9wZW5TU0wgR2VuZXJhdGVkIENl
cnRpZmljYXRlMB0GA1UdDgQWBBSYu0XEEn7edSlw2zXhGS3VO3I38jAfBgNVHSME
GDAWgBR2CJHQVQhlUczh15O17ThWBbaNmTANBgkqhkiG9w0BAQUFAAOCAQEAP1hS
1sU52N8SL9aagXFOyiwl1QT76PL3IJJ9gopsZFW5Tf8F/1z36j3p3tkbVFSrN9sv
6UhX0EfRIkD86bjd+bC+DPHhcrfx/L1cABehvvsoMyr2wwtrqL2mwuy/enFoBxIo
0mmgQs8hw5Tvt6lRZ6GwJ+/e1nIfSmeS3an4JydgYl0y714CXkl8JHn1FFplPNPf
iwMJWv4FNswsfrCJqniKvJ6WckcXkSP8VJ0jth8PjW1VpW+Bb+z9Lcx1PH8nwDzX
/e/7ODSr9HTfitS7o62B+FSVV2CUvSLGoVZryRG6AJZEYpgdBmSUetNGg7/2PJJN
YH4ossApqgcKKpgdLg==
-----END CERTIFICATE-----
```

<div align="center"> 
![](/images/linux/ca02.png)
</div>

---

## 证书吊销


>1、获取要吊销证书的serial,客户端申请，服务端不能随便吊销

>2、CA根据客户端提交的serial与subject信息，对比检验与index.txt文件信息是否一致

>3、吊销证书
在签售证书时，会在newcerts目录下生成对应序列号的pem文件
openssl ca -revoke /etc/pki/CA/newcerts/SERIAL.pem

>4、生成吊销证书编号(第一次申请一个序列号)
echo 01 > /etc/pki/CA/crlnumber

>5、更新证书的吊销列表
openssl ca -gencrl -out FILENAME.crl
查看crl文件
openssl crl -in /PATH/CRL_FILE.crl -noout -text

