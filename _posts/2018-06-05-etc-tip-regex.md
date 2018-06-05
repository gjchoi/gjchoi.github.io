
### Capturing Group Reference

특정 패턴부 Group을 변수화하여, 구부분에 대해 처리함

~~~
String testUrl = "http://test.domain.com?t=1&data1=vvvv&f=3344";
testUrl.replaceAll("(?<G1>[?|&]data1=)(?<G2>[^&]*)", "${G1}ssss")
~~~

or 

~~~
Pattern.compile("(?<G1>[?|&]data1=)(?<G2>[^&]*)")
                .matcher(testUrl)
                .replaceAll("${G1}ssss");
~~~                
