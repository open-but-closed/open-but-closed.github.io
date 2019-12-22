---
title: Hexo에 Google Analytics를 연동
categories: [IT,웹서비스]
date: Sep 22, 2019 19:00 PM
permalink: hexo-adds-google-analytics
tags: [Google Analytics,hexo]
updated: Sep 23, 2019 12:00 AM
description: 그래도 방문자 통계등은 궁금할 수 있지. 다음과 같이 Google Analytics를 연동해보자.
---

# Google Analytics 가입 및 Tracking ID 확인

처음

![](/images/Untitled-3685cd51-d5e4-46e7-a9d8-da97bbd238ff.png)

1단계

![](/images/Untitled-502ed0de-7388-4ab7-a169-6500ed3dac4c.png)

2단계

![](/images/Untitled-30553d89-c805-4436-931e-bf4a80c47cee.png)

3단계

![](/images/Untitled-12aaa2fa-9a7e-4a82-a364-daa0724817b5.png)

최종동의

![](/images/Untitled-aed4f636-9708-44f7-8463-0dbf0f3db267.png)

Tracking ID가 생성되고 대시보드가 나온다.

![](/images/Untitled-0808e7b2-8ee5-46b2-a877-72b9b92a2fb7.png)

# Hexo에 셋업

Hexo의 Theme에 따라 지원 여부가 다를 수 있다. 만약 해당 테마에서 셋업을 지원한다면 다음과 같이 `config.yml`에 다음 부분만 추가해주면 된다고 한다.
`/_config.yml` 파일

    google_analytics: <<트래킹ID>>

하지만 [어떤 블로그](https://urstory.github.io/2019/05/26/append-google-analytics/)에선 검색이 제대로 되지 않기 때문에, 추가적인 설정이 필요하다고. 그래서 이 블로그에서 참조하는 [github](https://github.com/hackjutsu/hexo-theme-next-modified/blob/master/layout/_layout.swig)을 참조해서 다음을 수정한다.

`/themes/next/layout/_scripts/`에 `analytics.swig` 생성

    {% include 'analytics/google-analytics.swig' %}

`/themes/next/layout/_scripts/`이하로 `analytics/google-analytics.swig`폴더와 파일을 생성

    {% if theme.google_analytics %}
    <script>
      (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
                (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
              m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
      })(window,document,'script','//www.google-analytics.com/analytics.js','ga');
      ga('create', '{{ theme.google_analytics }}', 'auto');
      ga('send', 'pageview');
    </script>
    {% endif %}

`/themes/next/layout/_layout.swig` 수정

    <body ...>
    
        {% include '_scripts/analytics.swig' %}
    
    ...

실제 돌아가는지는, 시간이 좀 지난 다음에 확인이 가능하다고 한다.

# Google Analytics에서 확인

대시보드에서 테스트 트래픽을 날리면 잡히는게 생기는듯

![](/images/Capture_2019-09-22_at_23-0242d62f-bf0f-45ee-9139-0b2a92297092.57.38.jpg)

.끝.