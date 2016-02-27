---
layout: post
title: Github Page로 블로그 호스팅
excerpt: "누구나 쉽게 Github Page로 블로그 서비스하는 방법에 대한 설명"
tags: [sample post, code, highlighting]
modified: 2016-02-22
comments: true
category : env
---

{% include _toc.html %}

Github에서 Repository에 html혹은 markdown .md 파일을 commit하는것만으로 본 블로그처럼 웹페이지를 별도의 서버와 도메인을 갖고 있지 않아도 사용 할 수 있게 해준다.



Github 가입하기
---------------

제일먼저 Githib에 계정을 가지고 있어야 한다. [http://www.github.com](http://www.github.com)에서 유니크한 ID를 선택하고 메일인증통해서 가입완료한다.  
가입을 완료하면 repository를 만들면 내 git 저장소를 사용할 수 있게되는데 향후 jekyll기능을 사용위해 repository이름은 *{계정이름}.github.io*로 생성한다.(계정repository 생성) 그래야 Context path없이 저 이름 규칙대로 url을 사용 할 수 있다.  
(만약 저거 외에 다른 이름을 주면 project url이라고해서 *http://{계정이름}.github.io/{repsitory이름}* 으로 사용할 수 있지만 github에 내장된 page서비스인 jekyll 기능을 사용에 제약이있다)

#### Github repository
![repo_img](http://gjchoi.github.io/images/github-page/repo_img1.png)


Github page 설정하기
---------------

이때 github에서 제공해주는 jekyll이라는 static 웹페이지 서비스를 이용하여 손쉽게 미리만들어진 템플릿을 사용하면 .md파일만 작성해서 github에 commit하는 것만으로 블로그 글을 올릴 수 있다.  
계정 repository에 setting 메뉴에 들어가보면 automatic page generator와 html or jekyll 사용을 고를 수 있는 메뉴가 나오는데 자동생성하면 github markdown을 사용하여 내용물을 만들고 템플릿도 바로 선택해서 사용 가능하여 편리하지만 메인페이지에 국한되고 블로그처럼 기능을 이용하기에는 제약이 따르므로 jekyll기능을 사용하기로하자.

#### Github page setting 메뉴
![setting그림](http://gjchoi.github.io/images/github-page/setting_img1.png)


Jekyll 이란?
---------------

Jekyll은 ruby기반으로 만들어진 손쉽게 blog스타일의 정적 site를 생성해주는 도구로서
사용자들은 markerdown이나 text기반의 내용만 만들어 업로드하는 정도로 블로그를 운영 할 수 있도록 해준다. <u>특히 GitHub Page에 engine으로 사용되어 github website로서 서비스 할 수 있게 해준다.</u> **바로 이 기능을 사용하여 블로그를 구성할 것이다!**  
*※ 참고로 jekyll은 스펠링이 이상하지만 우리가 잘아는 지킬앤 하이드에 지킬이다*

[Jekyll 사이트](https://jekyllrb.com/)에서는 아래와 같이 소개하고 있다.

> Jekyll is a simple, blog-aware, static site generator. It takes a template directory containing raw text files in various formats, runs it through a converter (like Markdown) and our Liquid renderer, and spits out a complete, ready-to-publish static website suitable for serving with your favorite web server. Jekyll also happens to be the engine behind GitHub Pages, which means you can use Jekyll to host your project’s page, blog, or website from GitHub’s servers for free.


Jekyll 사용하기
---------------

앞선 설명과 같이 Github Page에 background에서는 jekyll이 돌고있다. Github repository에 page, css 등을 변경시키면 자동적으로 배포된다는 의미다. jekyll 사이트의 설명을 보면 console명령어로 ruby를 설치하고 jekyll build하고 이것저것 복잡한 과정이 나오는데, 이는 local이나 자체서버에 기동하기 위함이지 github page기능을 이용한다면 단순히 jekyll구조로된 소스만 가져다가 repository에 옮겨두기만 하면된다.  
이미 구축된 page의 github소스를 fork하여 사용하는 방법은 아래 ilmol님이 포스팅해놓은 글에 자세히 설명되어있다. (필자도 처음에는 이글을 보고 따라하며 파악할 수 있었음)  
[Jekyll,Git 을 몰라도 무료 Github Pages 즐기기](http://ilmol.com/2015/01/Jekyll,Git%20%EC%9D%84%20%EB%AA%B0%EB%9D%BC%EB%8F%84%20%EB%AC%B4%EB%A3%8C%20Github%20Pages%20%EC%A6%90%EA%B8%B0%EA%B8%B0.html)

Jekyll theme 마켓 사용하여 배포해보기
---------------

##### Jekyll themes
![마켓플레이스그림](http://gjchoi.github.io/images/github-page/theme_market_img1.png)

사실 Jekyll theme를 모아놓은 마켓플레이스가 존재한다. 이 싸이트에 들어가서 맘에드는 theme 눈으로 보고 골라 사용할 수 있어서 너무 유용하다!  
[Jekyll theme 사이트](http://jekyllthemes.org/)


##### Jekyll theme 선택
![테마선택](http://gjchoi.github.io/images/github-page/theme_market_img2.png)  

그 중에 마음에 드는 theme를 선택했다면 Homepage버튼을 선택해서 fork를 하는 방법과 Download를 눌러 나온 데이터를 github에 commit하는 방법 두가지가 있는데 필자는 그중에 2번째 방법으로 선택했다. 구체적으로 설명하면 다음과 같다.

- 앞서 생성한 본인의 github의 repository 아까 얘기했던 {계정이름}.github.io의 repository 주소로 git clone해온다.

- Jekyll구조의 theme를 다운받아서 git clone한 디렉토리에 압축해제한다.

- 해당 디렉토리내의 모든 파일을 git add하고 commit하고 push한다.  
(혹시 git사용법이 익숙하지 않은 사람이라면 fork해서 복사해오는 방법을 추천한다. 필자는..뭔가 남에꺼를 훔쳐오는 것 같아서 download해서 새로 올리는 방식을 선택했다.)

- Github에 올린 후에(기왕이면 올리기전이 좋음) jekyll의 주요 설정파일인 _config.yml이라는 파일이 존재한다. theme 템플릿을 사용한 것이므로 나만의 blog를 만들기 위해서는 해당파일에 내용물을 내 정보로 바꿔줘야 한다. 주로 블로그 title, 이름, email, sns 주소, 블로그 main site path 등일 것이다.

#### Jekyll _config.yml
![설정정보 변경 그림](http://gjchoi.github.io/images/github-page/jekyll_conf_img1.png)


이정도만 해주면 싸이트에 생성은 어느정도 완성된 것이다. 최초 배포이므로 시간이 좀 소요되는데 배포가 완료되었다면 *http://{계정이름}.github.io*로 접근하면 아까 선택했던 theme로 블로그가 완성된 모습을 볼 수 있다.  

**!주의 : 만약 시간이 30분도 넘게 흘렀는데 아무런 반응이없다면 github가입시 사용한 email에 가서 메일이 와있는지 확인해봐야 한다. 왜냐하면 github page의 jekyll이 배포에 실패하였거나 warning사항이 있을때 메일로 메시지를 보내주기 때문이다.**  
  
필자의 경험상 메일이오는 경우는 크게 2가지다.


#### Warnning 
- _config.yml에 markdown의 종류를 설정하는 곳이있는데 그곳이 `'kramdown'`이 아닌 경우 발생
현재 github에서는 markdown을 `'kramdown'`만 지원하는데 예전에 만들어진 theme에는 `'redcarpet'` 같은 걸로 설정되어있는 경우가 있다. 기능상 문제는없지만 warnning 메일이 계속온다는 불편함이있다..

#### Build fail
- jekyll은 markdown파일을 html로 변환해주고 css를 입혀 보여주는 기능이 주이므로 주로 css를 수정하다가 잘못된 경우에 발생한다. 구체적인 메시지가 제공되진 않아서 알아서 잘 추적해야한다.....


Jekyll 디렉토리 구조
---------------

Jakyll 디렉토리 구조를 보면 보통 아래와 같은 디렉토리들을 가지고 있다.


**_includes**

: footer, header 등 html flagment들

**_layout**

: page생성시 페이지 앞부분에 선언하여 선택하는 layout (jsp의 tile같은 느낌)

**_posts**

: markdown(.md) 등 블로그 글들이 저장되는 디렉토리

**_sass**

: css모음 (scss)


Jekyll 페이지 만들어보기
---------------

Jekyll페이지는 _posts에 .md파일을 만들어 넣는 것만으로 페이지만들기는 끝이다. 대신 jekyll의 markdown은 `'kramdown'`을 사용하므로 `'kramdown'` 문법에 맞추어 작성해야 한다. `'kramdown'`에 대한 자세한 사용법은 다음 posting에서 다루기로 하자.

#### .md파일 샘플사진
![.md파일 샘플사진](http://gjchoi.github.io/images/github-page/md_sample_img1.png)  

~~~
---
layout: post
title: You're up and running!
---
~~~

윗부분에 나와있는 layout은 디렉토리에 있던 것중 1가지로 선택해야 함 (없으면 default)
title은 layout내에 변수 매핑되어 사용된다.  

page를 변경하고 이상이없다면 수초내에 바로 배포/반영된다. 바로 반영되지 않는 경우도 있으니 조금만 더 기다려보고 안되면 메시지가 왔는지 메일을 뒤져보자.


페이지 디자인 수정하기
-----------------------

Jekyll theme를 다운 받아 사용했다면 이미 만들어진 css가 적용되어있을 것이다. `kramdown`을 사용하면 html태그로 전환해주는 것 뿐이지 해당 html 엘리먼트에 대한 디자인은 css가 담당하므로 css를 수정해서 디자인을 customizing 할 수 있다. css는 _sass라는 디렉토리에 모여있는데 이 디렉토리내에 css에 필요요소를 추가하던가 수정하면된다.
필자가 사용한 theme에는 code block에 대한 background 디자인을 변경하고자 다른데서 _highlights.scss파일을 가져와서 적용했다. css를 일일이 수정하는 일은 전문적으로 하는 사람이 아닌이상 시간을 많이 잡아먹는 일이다. 처음부터 꼼꼼히 살펴보고 맞는 jekyll theme를 마켓에서 다운 받아 사용하는게 중요한 것 같다.

  
모바일에서 블로깅하기
--------------------
앞서 언급되었듯이 github의 _posts디렉토리내에 .md파일을 위치시키는 것만으로 글을 업로드 할 수 있으므로 필자는 출퇴근간 폰으로 글을 업로드하기 위해 모바일용 git과 markdown을 작성할 수 있는 텍스트 에디터를 사용하였다. markdown을 작성하고 바로 github에 share할 수 있는 앱이 있으면 좋으련만… 아직 찾지 못했다. 때문에 필자는 sgit프로그램과 해당 sgit이 clone한 로컬레파지토리에 직접 데이터를 수정할 수있는 에디터를 사용하여 글을 생성/수정하고 바로 git hub에 올리는 식으로 작업 중이다.
(사실 모바일 브라우저로 github에 접근해서 직접 .md파일을 수정할 수도 있지만 preview가 안되므로 markdown에 익숙하지 않은 사람은 불편 할 수있다.)

