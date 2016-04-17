---
layout: post
title: 리눅스 Tip
excerpt: "개발하며 필요한 linux 명렁어나 설정등에 대한 tip 모음"
tags: [tip, linux]
modified: 2016-04-17
comments: true
category : etc
---


인증서없는 Password모드로 로그인하기 (EC2내에서 주로 사용)
------------------

EC2에서는 instance 생성시 발급한 인증서(pem)을 사용하여 만든 ppk파일을 사용하여 ssh로그인 하도록 되어있다.
개발, 테스트 환경등에서는 불편하므로 ID/PW방식으로 변경하여 접근할 때 유용하다.

`sudo vi /etc/ssh/sshd_config` 파일에 가보면 아래 설정이 있다.

패스워드 인증설정을 *no*에서 *yes*로 변경

~~~
#PasswordAuthentication no
PasswordAuthentication yes
~~~

root계정을 사용하고 싶은 경우 PermitRootLogin을 *yes*로 설정
~~~
#PermitRootLogin no
PermitRootLogin yes
~~~


`sudo reload ssh`로 설정 리로드 or `sudo service sshd restart`로 sshd 재시작



Ubuntu에서 Oracel JAVA 설치
------------------

Ubuntu java 설치

1) OpenJDK 제거
`$ sudo apt-get purge openjdk*`
 
2) repository 추가
`$ sudo add-apt-repository ppa:webupd8team/java`
 
3) repository index 업데이트
`$ sudo apt-get update`
 
4) JDK 설치
아래의 세가지 버전 중에 자신이 필요한 버전을 설치한다.

`$ sudo apt-get install oracle-java8-installer`
`$ sudo apt-get install oracle-java7-installer`
`$ sudo apt-get install oracle-java6-installer`
