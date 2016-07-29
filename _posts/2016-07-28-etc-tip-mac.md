---
layout: post
title: Mac Tip
excerpt: "개발하며 필요한 mac 명렁어나 설정등에 대한 tip 모음"
tags: [tip, linux]
modified: 2016-07-28
comments: true
category : etc
---

{% include _toc.html %}

Mac Jenkins 서비스 설정 및 시작/종료
------------------

~~~
sudo defaults write /Library/Preferences/org.jenkins-ci httpPort 7070

sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist
sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist
~~~

[자세한 설정](https://wiki.jenkins-ci.org/display/JENKINS/Starting+and+Accessing+Jenkins)
