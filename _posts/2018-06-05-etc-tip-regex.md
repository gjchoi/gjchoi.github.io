
---
layout: post
title: 정규식 Tip
excerpt: "개발하며 필요한 정규식 tip 모음"
tags: [tip, regex]
modified: 2018-06-05
comments: true
category : etc
---

{% include _toc.html %}

## Capturing Group Reference

특정 패턴부 Group을 변수화하여, 구부분에 대해 처리함

~~~ java
String testUrl = "http://test.domain.com?t=1&data1=vvvv&f=3344";
testUrl.replaceAll("(?<G1>[?|&]data1=)(?<G2>[^&]*)", "${G1}ssss")
~~~

or 

~~~ java
Pattern.compile("(?<G1>[?|&]data1=)(?<G2>[^&]*)")
                .matcher(testUrl)
                .replaceAll("${G1}ssss");
~~~                
