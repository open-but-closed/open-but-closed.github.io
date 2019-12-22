---
title: 블로그 플랫폼을 Hexo로 변경 
categories: [IT,웹서비스]
date: Jan 13, 2019 7:00 PM
permalink: hexo-as-a-platform-for-blogging
tags: [hexo,static-website-generator]
updated: Jan 22, 2019 10:39 PM
description: 블로그 글쓰기를 유지하기는 왜 이렇게 힘든 것인가. 이건 내가 게을러서가 아니라 플랫폼이 별로여서다. 그래서 블로그 플랫폼을 Hexo로 변경하기로 했다.
---

# Hexo 소개

## 잡설

오랫동안(?) 블로그를 [티스토리](http://open-but-closed.tistory.com)에서 운영해왔다.

![](/images/Untitled-0af556f2-e151-4c1d-8e9f-296805b37f44.png)

(비장한 각오를 다지던 2019년 처음이자 마지막 포스팅. 평균 월간 포스팅 0.3개, 평균 일별 방문자 0.7명. ~~나조차 들어오지 않았었으니...~~)

이럴 때를 대비해서 블로그 이름을 개장휴업으로 했지만 참 부끄러운 상황이다. 그래서 맘 편하게 남탓을 해보자면 **티스토리 글쓰는 창이 재미 없었기 때문**으로 할 수 있겠다. 

반년 전 쯤 [**Notion**](https://www.notion.so/)이라는 서비스를 알게 되었는데, 자료를 작성하고 구성하는데 매우 유용한 툴이다. 개인 자료들이나 글들은 이곳에 저장하고 있는데, 가능하면 이 자료를 이용해 블로그 글을 관리하고 싶었다. 처음에는 `ctrl+c` + `ctrl+v` 신공을 사용하려 했으나, 생각만큼 잘 되지는 않았다. 물론 가장 큰 문제는 글을 자주 쓰지 않는 것이지만, 글을 배포하는 시간이라도 줄이고 싶은 생각이 들었다.

## Jekyll, Hexo, Hugo

**Notion**에는 페이지를 `export` 할 수 있는 기능이 있는데, 이때 결과물이 **Markdown**형식으로 생성된다. 만약 이 파일을 바로 포스팅으로 연결한다면 글의 배포가 매우 손쉽게 될 것 같다. 처음에 검색을 했던 것은 **Wordpress**의 Markdown 관련 플러그인이었다. 하지만 검색 중 자주 언급되는 키워드가 있었는데 **Jekyll**, **Hexo**, **Hugo** (각각 이름임) 등이었다. 

예전 블로그 플랫폼을 고민하면서 **Jekyll+Github**을 알아봤던 적이 있다. 최근의 개발자 블로그의 대부분을 Jekyll+Github으로 만들 만큼 인기 있는 플랫폼이다. 그럼에도 당시에 큰 관심을 두지 않은 이유가, 사용하기 위한 필요 기술 때문이었다. Jekyll+Github을 사용하려면 **Git**이나 여타 개발 환경에 익숙해야 하는데, 나는 DB엔지니어라 그런 부분이 부족했기 때문.

하지만 Jekyll같은 툴에서 요구하는 기술이 Markdown 문법의 사용인데, 이 부분은 익숙하다. 그리고 Jekyll을 사용하기 어렵다면 **Hexo**를 사용해 보라는 의견을 보았다. 

## Content Management System

일반적으로 게시판이 들어간 시스템을 만들땐 **CMS(Content Management System)**라 불리는 패키지를 사용한다. 글로벌 하게 많이 사용하는 것은 **Wordpress**를 들 수 있고, 우리나라의 과거 **zeroboard**나 현재의 **XpressEngine**도 CMS에 속한다. 시스템 마다 상세한 기술 스택은 다를 수 있으나, 대부분 **DBMS** - **WAS** - **Web Server**의 구성을 가지고 있다.

![](/images/Untitled-e9be5595-bf71-45da-b574-e76e20b5f097.png)

- **DBMS** - 데이터가 저장된다. 글쓴이의 포스팅 뿐 아니라 시스템의 설정 같은 부분이 저장된다. Wordpress의 경우 **MySQL**이나 **PostgreSQL**등을 선택할 수 있다.
- **WAS(Web Application Server)** - 사용자의 요청에 따라 컨텐츠를 동적으로 생성한다. 즉, 사용자가 1번 페이지를 요청하면 1번 페이지의 제목과 내용을 정적인 페이지(html)로 생성한다. 해당 사용자가 권한이 없을 경우 권한이 없는 페이지로 생성한다. 사용자의 요청에 따라 매번 페이지를 생성하기 때문에 동적 생성이라는 용어를 사용한다. **server-side 언어**가 사용되는 곳이며, Wordpress같은 경우 **PHP**가 사용된다.
- **Web Server** - 정적인 파일(html)이 서비스 되는 곳. WAS에서 이 역할을 수행할 수 있지만, 부하 분산등의 이유로 보통 웹서버까지 함께 간다. 과거에는 **Apache**, 요즘엔 **Nginx**가 많이 사용된다.

동적인 사이트라고 하는 이유는 매 요청마다 동적으로 페이지를 생성하기 때문이다. 사용자가 `글번호=3`을 요청하면 매번 DBMS에서 `글번호=3`에 해당하는 **제목**과 **내용**을 읽어서 html 페이지를 생성하는 방식이다. 물론 계속 3번 글을 요청하면 동일한 내용이 생성될 것이다. 웹서버나 DB 등 여러 단계에서 캐싱이란 기술도 사용되긴 하지만, 어쨌건 **DBMS를 읽어서 다시 페이지를 만드는 것이 원칙**이다. `글번호=3`에 해당하는 내용이 언제 어떻게 바뀌었을 지 알 수 없기 때문에.

## Static Website Generator

위에 나열한 **Jekyll**, **Hexo**, **Hugo**등은 **Static website generator**로 불린다. 이들은 모든 html 페이지를 로컬 환경에서 생성한다. 그리고 생성된 **정적(Static)인 파일**을 서버에 올리기만 하면 된다. **이미 모든 페이지들이 만들어져 있기 때문에 서버에는 DBMS와 WAS가 필요하지 않다.** 단순하게 웹 서버만 있으면 홈페이지를 서비스할 수 있는 것이다. 여기서 **Github**이 좋은 궁합으로 많이 언급되는 이유가, Github에서는 이런 정적인 html이나 이미지 같은 것을 무료로 호스팅할 수 있기 때문이다. `userid`**.github.io** 로 레파지토리를 생성하면 아예 홈페이지처럼 호스팅이 된다.

**Hexo**로 예를 들면, 사용자는 몇 가지 셋업을 하고, 각각의 포스트 들은 개별 markdown형식의 파일로 만들면 된다. 그리고 generate하면 서비스 되야 할 모든 html 페이지들이 생성된다. 예를들어 글의 리스트를 볼 `archive`페이지라 하면 `archive`에 대한 내용을 모두 만들어버리고, `tag`페이지라 하면 `tag`페이지를 모두 만들어 버린다. 특정 tag를 눌렀을 때는 누른 tag에 대한 글을 보여줘야 하니, 아예 모든 tag 갯수 만큼 html 파일들을 만들어버린다.

![](/images/Untitled-a0a085b7-279a-4515-ae52-dc6db2d7be35.png)

정적인 페이지는 매우 저렴하게 서비스할 수 있는 장점이 있다. github뿐 아니라, 무료 호스팅에도 올릴 수 있고 (디렉토리 구조만 지켜진다면), 유로 서비스라도 DBMS나 어플리케이션 서버를 구축하는 것 보다 훨씬 간편하고 저렴하다. 성능도 크게 향상된다. 정적인 페이지들을 서비스 하는 것 자체는 페이지를 전송만 하면 되니 서버에 부하가 훨씬 적다. 대부분 시스템에서 부하가 발생하는 부분은 프로그램과 DBMS부분이다.

정적인 페이지를 사용할 때의 문제는 하나다. **내용을 어떻게 변경하나?** 글 하나를 추가하면 글 목록도 바뀌고 태그 목록도 바뀐다. 일반 CMS 툴이라면, 글을 변경하고 확인 버튼을 누를 때, 해당 변경사항이 DBMS에 저장되면서 모든 작업이 끝난다. 반대로 Hexo는 다시 동적 파일을 생성하고 서비스 하는 서버로 배포해야 한다. 즉, 기존의 WAS와 DBMS에서 하던 작업은 훨씬 수고가 많이 들게 되었다

결국 Hexo같은 Static website generator는 글의 조회수 >>>>>>>>>> 글의 수정 횟수 를 만족하는 블로그나 기업 홈페이지에 적합하다.

추가적으로 고민할 수 있는 부분은, 방문자와 소통할 수 있는 comment 창 같은 경우다. Hexo는 정적인 페이지만 생성하기 때문에 댓글 같은 모듈을 적용하는 것은 적절치 않다. 따라서 [**DISQUS**](https://disqus.com/)같은 별도의 댓글 모듈를 정적 페이지에 embed하는 방식으로 사용하게 된다. 

# Hexo 설치

이제 Hexo를 설치해 보자. 아래와 같이 Hexo를 로컬에 설치하면 정적 페이지들을 생성하고 확인할 수 있다.

## Node.js 설치

Hexo는 [**node.js**](https://nodejs.org/) 기반으로 되어 있다. (참고로 **Jekyll**은 **Ruby** 기반으로 되어 있다.) ~~나는 노알못이며 루알못이기 때문에 어느 것을 선택해도 알못이었다.~~

node.js는 [공식사이트](https://nodejs.org/)에서 각자의 운영체제(필자는 MacOS)에 맞게 다운 받아서 설치하면 된다. 설치 후 다음 명령어로 node.js와 **npm**(node 패키지 매니저) 설치를 확인할 수 있다.

    $ node -v
    v11.6.0
    
    $ npm -v
    6.5.0-next.0

## Hexo 설치

hexo는 npm을 통해 받을 수 있다.

    $ npm install -g hexo-cli
    ...

받으면 설치 끝.

## Hexo 작업 디렉토리 설정

블로그 작업 디렉토리를 생성하고 이동한다.

    $ mkdir kjhexo
    $ cd kjhexo

블로그 생성은 `hexo init`으로 할 수 있다. 파라미터가 있으면 해당 디렉토리를 생성하고, 없으면 현재 디렉토리를 작업 디렉토리로 초기화한다. 쿵짝쿵짝 초기화 된 이후 디렉토리를 조회하면 파일들이 생성되어 있다. 참고로 초기화를 하면 첫 포스트인 `hello-world.md`파일이 `source/_posts`에  포함되어 있다.

    $ hexo init
    INFO  Cloning hexo-starter to ...
    ...
    
    $ ls
    _config.yml		node_modules		package.json		source
    db.json			package-lock.json	scaffolds		themes
    
    $ ls source/_posts/
    hello-world.md

## Hexo 빌드

위와 같이 markdown형태의 포스트를 `_posts` 폴더에 놓은 상태에서 `hexo generate` 또는 `hexo g` 명령어로 Hexo를 빌드할 수 있다. 빌드 후 `public` 폴더가 생성되며, 빌드된 파일들은 이 `public`폴더 아래 위치하게 된다. 이 `public`이하의 파일들에 결과물인 **html**, **css**, **jpg**, **png**등의 파일들이 생성되며, 복사해서 웹 서버에 놓으면 그 자체로 웹페이지들을 서비스 할 수 있다.

    $ hexo g
    INFO  Start processing
    INFO  Files loaded in 393 ms
    INFO  Generated: index.html
    ...
    
    $ ls public
    2019		archives	css		fancybox	index.html	js

## Hexo 서버 시작

Hexo에는 자체 서버도 내장되어 있어, 어떤 식으로 생성되는지 테스트를 할 수 있다. 명령어는 `hexo server`또는 `hexo s` 이다.

    $ hexo s
    INFO  Start processing
    INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.

## Hexo 접속

서버가 시작되면 윗 결과 처럼 `http://localhost:4000`으로 접속하면 된다. 다음과 같은 초기 화면을 볼 수 있다.

![](/images/Untitled-37ef6a7c-4d9f-4dbd-b056-17ad5459aa10.png)

(아름다운 Hello World의 등장)

여기까지가 Hexo에 대한 기본 소개이다. 이후엔 주로 Theme를 변경 한다던지, 실제 서비스 될 서버에 **배포(deploy)**를 하는 작업들을 해야 한다.

# 참고

- [Github Page와 Hexo를 통해 30분만에 기술 블로그 만들기 - 안녕 프로그래밍 소개](https://www.holaxprogramming.com/2017/04/16/github-page-and-hexo/)
- [Hexo와 Github page로 시작하는 블로그 만들기 - DY, Kim](https://kdydesign.github.io/2017/06/23/start-hexo-blog/)