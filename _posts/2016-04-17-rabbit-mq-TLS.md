---
layout: post
title: RabbitMQ - TLS 통신보안
weight: 70
excerpt: "Rabbit MQ의 통신간 보안을 위해 TLS를 적용하는 방법에 대해서 알아본다."
tags: [rabbitMQ, oss]
modified: 2016-04-17
comments: true
category : rabbit
icon : wrench.ico
---


TSL 지원
-----------

RabbitMNQ는 기본적으로 TLS를 지원한다.
(POOLDLE attack[^1]으로 인해 RabbitMQ 3.4.0대에 SSLv3는 disabled되어 있음)

Erlang crypto 어플리케이션이 반드시 설치되고 동자가되어야 한다.


[^1]:  Padding Oracle On Downgraded Legacy Encryption 의 약자로 SSL3.0 버전의 취약점으로서 TLS가 하위 SSL3.0의 호환성을 가지고 있고 CBC mode의 취약점을 이용하여 패킷을 탈취하는 보안적 문제


Key와 Certification / CA Certificates
-------------------------------------

### 테스트용 Certificate Authority 생성

1) 준비 
특정 디렉토리에 다음 디렉토리 구조 및 파일을 만든다.

~~~~
# mkdir testca
# cd testca
# mkdir certs private
# chmod 700 private
# echo 01 > serial
# touch index.txt
~~~~

2) 특정 디렉토리에 *openssl.cnf* 파일 생성

~~~~~
[ ca ]
default_ca = testca

[ testca ]
dir = .
certificate = $dir/cacert.pem
database = $dir/index.txt
new_certs_dir = $dir/certs
private_key = $dir/private/cakey.pem
serial = $dir/serial

default_crl_days = 7
default_days = 365
default_md = sha256

policy = testca_policy
x509_extensions = certificate_extensions

[ testca_policy ]
commonName = supplied
stateOrProvinceName = optional
countryName = optional
emailAddress = optional
organizationName = optional
organizationalUnitName = optional

[ certificate_extensions ]
basicConstraints = CA:false

[ req ]
default_bits = 2048
default_keyfile = ./private/cakey.pem
default_md = sha256
prompt = yes
distinguished_name = root_ca_distinguished_name
x509_extensions = root_ca_extensions

[ root_ca_distinguished_name ]
commonName = hostname

[ root_ca_extensions ]
basicConstraints = CA:true
keyUsage = keyCertSign, cRLSign

[ client_ca_extensions ]
basicConstraints = CA:false
keyUsage = digitalSignature
extendedKeyUsage = 1.3.6.1.5.5.7.3.2

[ server_ca_extensions ]
basicConstraints = CA:false
keyUsage = keyEncipherment
extendedKeyUsage = 1.3.6.1.5.5.7.3.1
~~~~~



3) OpenSSL 명령어로 *.pem*과 *.cert* 생성 (Certificate Authority 생성)

~~~~
# openssl req -x509 -config openssl.cnf -newkey rsa:2048 -days 365 \
    -out cacert.pem -outform PEM -subj /CN=MyTestCA/ -nodes
# openssl x509 -in cacert.pem -out cacert.cer -outform DER
~~~~

*.pem*과 *.cer*은 모두 같은 정보(root certificate)를 가지고 있지만 포맷이 다름.


대부분 **PEM** 포맷 사용하지만, Microsoft진영(Microsoft, Mono)은 **DER** 사용



4) Client와 Server용 key생성

Erlang 클라이언트와 RabbitMQ 브로커는 PEM을 바로 사용.
PEM은 3가지 정보 포함

1. root certificate (인증정보)
2. private key (공인인증의 소유권)
3. public key (공인인증 peer 식별)

JAVA나 .NET은 PKCS#12 store를 제공해서 client의 인증서와 key를 passwd로 보호하여 보관할 수 있다.


#### Server용 key 생성

~~~~~
# cd ..
# ls
testca
# mkdir server
# cd server
# openssl genrsa -out key.pem 2048
# openssl req -new -key key.pem -out req.pem -outform PEM \
    -subj /CN=$(hostname)/O=server/ -nodes
# cd ../testca
# openssl ca -config openssl.cnf -in ../server/req.pem -out \
    ../server/cert.pem -notext -batch -extensions server_ca_extensions
# cd ../server
# openssl pkcs12 -export -out keycert.p12 -in cert.pem -inkey key.pem -passout pass:MySecretPassword
~~~~~

#### Client용 key 생성

Server용 key 생성과 거의 동일하지만


~~~~~
# cd ..
# ls
server testca
# mkdir client
# cd client
# openssl genrsa -out key.pem 2048
# openssl req -new -key key.pem -out req.pem -outform PEM \
    -subj /CN=$(hostname)/O=client/ -nodes
# cd ../testca
# openssl ca -config openssl.cnf -in ../client/req.pem -out \
    ../client/cert.pem -notext -batch -extensions client_ca_extensions
# cd ../client
# openssl pkcs12 -export -out keycert.p12 -in cert.pem -inkey key.pem -passout pass:MySecretPassword
~~~~~





### SSL 사용하기

앞서 생성한 3가지 정보 필요

1. Root 인증서 (Root Certificate)

2. 서버 인증서 (Server Certificate)

3. 서버 키 (Server Key)


config file설정하기


~~~
[
  {rabbit, [
     {ssl_listeners, [5671]},				// 5671포트로 ssl기동
     {ssl_options, [{cacertfile,"/path/to/testca/cacert.pem"},		// root 인증서 위치 지정
                    {certfile,"/path/to/server/cert.pem"},			// 서버 인증서 위치 지정
                    {keyfile,"/path/to/server/key.pem"},			// 서버 key 파일 위치지정
                    {verify,verify_peer},				// client가 보낸 인증서를 신뢰하여 통신 
                   #{verify,verify_none},				// client와 인증서 교환 안함
                    {fail_if_no_peer_cert,false}]}		// client가 인증서를 안가지고 있을 시 허용
                   #{password,  "t0p$3kRe7"}			// private key파일에 password걸어서 사용
				   #{ciphers,  [{ecdhe_ecdsa,aes_128_cbc,sha256},
                   #            {ecdhe_ecdsa,aes_256_cbc,sha}]}   // cipher suites 사용

   ]}
].
~~~

시스템에 지원하는 cipher suites나 ssl version 확인하기

~~~
rabbitmqctl eval 'ssl:cipher_suites(openssl).'
rabbitmqctl eval 'ssl:verions().'
~~~



#### 여러 인증서 합쳐서 사용하기

~~~
# cat testca/cacert.pem >> all_cacerts.pem
# cat otherca/cacert.pem >> all_cacerts.pem
~~~


### SSL/TLS 버젼 설정

SSL/TLS 공격으로 알려진 POODLE(SSLv3경우)나 BEAST attack(TLSv1.0경우)를 방지하기 위해
tls, ssl의 일부 버젼을 disable해야하는 경우가 있다.
아래와 같은 ssl설정을 통해 가능하다.

~~~
%% Disable SSLv3.0 support, leaves TLSv1.0 enabled.
[
 {ssl, [{versions, ['tlsv1.2', 'tlsv1.1', tlsv1]}]},
 {rabbit, [
           {ssl_listeners, [5671]},
           {ssl_options, [{cacertfile,"/path/to/ca_certificate.pem"},
                          {certfile,  "/path/to/server_certificate.pem"},
                          {keyfile,   "/path/to/server_key.pem"},
                          {versions, ['tlsv1.2', 'tlsv1.1', tlsv1]}

                         ]}
          ]}
].
~~~


openssl을 통한 연결확인

~~~
# connect using SSLv3
openssl s_client -connect 127.0.0.1:5671 -ssl3
~~~

~~~
# connect using TLSv1.0 through v1.2
openssl s_client -connect 127.0.0.1:5671 -tls1
~~~


### JAVA Security 프레임워크의 3가지 Component

1. Key Manager
2. Trust Manager
3. Key Store


Key Manager

: 자기 인증서를 관리. Session이 연결될 때 어떤 인증서를 Remote 호출시 사용할지 결정


Trust Manager

: Remote 인증서 관리. Session이 연결될 때 Remote의 어떤 인증서를 신뢰할지 결정


Key Store

: Java specific binary format이라 불리는 *PKCS#12*으로 보관





### JAVA 코드를 활용한 SSL 호출


#### JAVA 프로그램을 통해 간단한 RabbitMQ 메시지 처리하기

서버 인증서 검증없고 Client 인증서 보내지 않고 간단한 메시지를 보냄

~~~

import java.io.*;
import java.security.*;


import com.rabbitmq.client.*;

public class Example1
{
    public static void main(String[] args) throws Exception
    {

        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setPort(5671);

        factory.useSslProtocol();		//SSL 모드로 
        // Tells the library to setup the default Key and Trust managers for you
        // which do not do any form of remote server trust verification

        Connection conn = factory.newConnection();
        Channel channel = conn.createChannel();

        //non-durable, exclusive, auto-delete queue
        channel.queueDeclare("rabbitmq-java-test", false, true, true, null);
        channel.basicPublish("", "rabbitmq-java-test", null, "Hello, World".getBytes());


        GetResponse chResponse = channel.basicGet("rabbitmq-java-test", false);
        if(chResponse == null) {
            System.out.println("No message retrieved");
        } else {
            byte[] body = chResponse.getBody();
            System.out.println("Recieved: " + new String(body));
        }


        channel.close();
        conn.close();
    }
}

~~~


※ 주의 : Java Version이 1.7이하이 경우 Client인증서 보내지 않고 TLS(SSL) 호출하는 경우 peer 오류가 발생하는 경우가 있다.  1.8이상을 이용해주자.




#### 인증정보를 포함한 JAVA Client 프로그램

~~~

import java.io.*;
  import java.security.*;
  import javax.net.ssl.*;

  import com.rabbitmq.client.*;


  public class Example2
  {
      public static void main(String[] args) throws Exception
      {

      	// Key Manager Setting	(본인의 Client 인증서)
        char[] keyPassphrase = "MySecretPassword".toCharArray();
        KeyStore ks = KeyStore.getInstance("PKCS12");
        ks.load(new FileInputStream("/path/to/client/keycert.p12"), keyPassphrase);

        KeyManagerFactory kmf = KeyManagerFactory.getInstance("SunX509");
        kmf.init(ks, passphrase);


		// Trust Manager Setting (remote 인증서)
        char[] trustPassphrase = "rabbitstore".toCharArray();
        KeyStore tks = KeyStore.getInstance("JKS");
        tks.load(new FileInputStream("/path/to/trustStore"), trustPassphrase);

        TrustManagerFactory tmf = TrustManagerFactory.getInstance("SunX509");
        tmf.init(tks);


        SSLContext c = SSLContext.getInstance("TLSv1.1");
        c.init(kmf.getKeyManagers(), tmf.getTrustManagers(), null);


        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setPort(5671);
        factory.useSslProtocol(c);

        Connection conn = factory.newConnection();
        Channel channel = conn.createChannel();

        channel.queueDeclare("rabbitmq-java-test", false, true, true, null);
        channel.basicPublish("", "rabbitmq-java-test", null, "Hello, World".getBytes());


        GetResponse chResponse = channel.basicGet("rabbitmq-java-test", false);
        if(chResponse == null) {
            System.out.println("No message retrieved");
        } else {
            byte[] body = chResponse.getBody();
            System.out.println("Recieved: " + new String(body));
        }


        channel.close();
        conn.close();
    }
}

~~~
