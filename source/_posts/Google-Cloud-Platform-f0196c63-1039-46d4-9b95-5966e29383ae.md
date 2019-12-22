---
title: Google Cloud Platform 가입하기
categories: [클라우드,GCP]
date: Aug 03, 2018 12:09 AM
permalink: sign-in-google-cloud-platform
tags: [GCP,VPS,cloud-computing]
updated: Jan 15, 2019 1:40 AM
description: Google Cloud Platform (이하 GCP) 가입하는 방법에 대해 알아보자. 클라우드 서비스는 Amazon AWS가 대세긴 하지만 요즘 다른 곳도 많다. Microsoft Azure, Google Cloud Platform 나 한국 서비스도 많이 있음
---

# GCP 소개

GCP의 최대 장점은 1년간 $300 크레딧이 주어진다는 것. 기간도 길고 크레딧도 많은 편. **무엇보다 사용자의 허락(업그레이드 수락)없이는 절대 과금되지 않는다!** ~~예전 AWS를 사용하다가 몇 달 뒤 신용카드 결제 문자와 이메일을 받았을때의 당혹스러움은 잊을 수 없다.~~

지금은 GCP도 가입하는 튜토리얼이 포스트가 많아졌다. 당장 구글에서만 쳐봐도...

![](/images/Untitled-6adacd92-2d1f-457b-b3f5-9c42717c43ab.png)

~~양질의 튜토리얼이 많이 있어서... 내가 굳이 포스팅을 해야 하나 하는 생각이-_-;;~~

그냥 이런 튜토리얼을 누르고 따라하면 된다. 참조했던 소개 페이지를 하단 참고에 붙인다.

# GCP 가입

일단 페이지 접속

- [https://cloud.google.com](https://cloud.google.com)
- 구글계정이 있어야 함(없으면 생성 - 생략)

시작화면(지금은 또 바뀌어 있는 것 같음)

![](/images/Capture2018-07-12at3-53e66e6f-3b32-4253-ac0b-12966d1a8c83.54.02PM.jpg)

가입항목을 성실히 입력해야 공짜로 받을 수 있음

![](/images/Capture2018-07-12at3-5a3f360e-97a3-4033-a897-b5f976d7790c.54.16PM.jpg)

신용카드 정보를 영혼과 함께 제공

![](/images/Capture2018-07-12at3-f08f71ff-2559-4c76-b053-757d5c727c18.54.29PM.jpg)

공짜로 생성중이니 무한 대기

![](/images/Capture2018-07-12at3-751bc2fb-2f57-40c9-985b-b5ec01c1fc7d.56.38PM.jpg)

환영합니다!

![](/images/Capture2018-07-12at3-31b6988a-c5e7-4f08-b372-ebe422b7309f.57.04PM.jpg)

# VM 인스턴스 생성

이곳에선 서버를 VM 인스턴스라고 부른다. 대략 virtual machine instance인듯.

![](/images/Capture2018-07-12at3-df7c3705-bbca-4d24-9ad0-ef43dff30d0d.57.39PM.jpg)

만들기 고고

![](/images/Capture2018-07-12at3-c5551799-d1b9-4e1b-b7bb-fc47e7adb488.57.48PM.jpg)

항목을 적당히 채워넣는다.

- 이름 : 머신의 hostname이 되므로 참고
- 지역 : 현재 우리나라는 없고, 제일 가까운 지역은 도쿄(asia-northeast1)임
- 머신유형 : 머신의 성능, CPU / RAM 설정
- 부팅 디스크 : 보통 OS 이미지를 제공하고, 특정 패키지나 프로그램이 설치된(pre-installed) OS도 선택할 수 있다.

![](/images/Capture2018-07-12at4-54e3665d-1675-41cb-8c8d-4fe64ddd8669.09.18PM.jpg)

조금 있으면 인스턴스가 **똭** 생성됨

![](/images/Capture2018-07-12at4-c47bc08a-02ad-4f76-83da-1557e6cb5bda.11.26PM.jpg)

SSH 버튼을 누르면 호스트에 접속할 수 있다.

![](/images/Capture2018-07-12at6-14af2636-3fb9-4fd6-9097-f21e4868fa00.15.24PM.jpg)

끝.

# 참고

- [GCP(Google Cloud Platform) 시작하기 - Jaeyeon Baek - Tistory](http://jybaek.tistory.com/606)
- [[GCP]가난한 개발자를 위한 GCP free tier 활용 방법 1/2 – 이정운](https://medium.com/@jwlee98/gcp-가난한-개발자를-위한-gcp-free-tier-활용-방법-1-2-3022348e1103)
- [Google Cloud Platform 가입 및 소개 · 어쩐지 오늘은](https://zzsza.github.io/gcp/2018/01/01/gcp-intro/)
