---
title: 이전 버전의 크로미움 브라우저 설치하기
date: 2019-11-19 00:00:00 +0900
category: [개발]
tags: [chromium]
---

개발을 하다보면 버전호환성 확인이나 그 외 다른 이유로 인해 예전 버전의 크로미움 브라우저갸 필요한 경우가 있다. 특히나 안드로이드 웹뷰의 경우 최근에는 시스템 WebView를 Play Store를 통해 업데이트할 수 있게 되어 파편화가 좀 줄어들긴 했으나, 젤리빈이나, 킷캣 버전의 안드로이드를 사용할 경우 시스템 WebView를 업데이트하는 것이 불가하여, 버전별로 웹컨텐츠가 다르게 보이는 경우가 제법 많이 발생한다. 특히나 모바일 환경이 아닌 AVN 등에서 사용하는 안드로이드는 아직도 이정도 버전에 머물러 있는 경우가 많아서, 웹 컨텐츠를 제작할 때 미리 테스트해보기가 쉽지 않다.

<!--more-->

그래서, 이전 버전의 크롬 브라우저를 PC에 다운받아 미리 컨텐츠가 어떻게 동작될런지 확인해보고자 했는데, 이전버전의 크롬 브라우저를 PC에 설치하는 것 또한 쉬운일은 아니더라.

혹시나 나처럼 고생할 사람들을 위해서 조사한 내용을 정리해 본다.

1. 먼저, 다운받길 원하는 크롬의 버전을 정확히 알아야 한다. 크롬의 버전은 4자리로 구성되어 있으며, 4자리를 모두 알아야 한다. 또한, stable 버전이 아니면 다운이 어려울 수 있으니 확인이 필요하다. 버전은 [Wikipedia의 크롬 버전 히스토리 페이지](https://en.wikipedia.org/wiki/Google_Chrome_version_history){:target="_blank"}나 [Google Chrome 릴리즈 블로그](https://chromereleases.googleblog.com/){:target="_blank"} 등을 검색해서 찾을 수 있다.
2. 다음으로 해당버전의 버전 정보를 검색하기 위해 [https://omahaproxy.appspot.com/](https://omahaproxy.appspot.com/) 에서 아래쪽에 Tools 항목에 있는 **Version Information** 을 검색한다. 검색할 값은 1번에서 확인한 버전이다.
3. 조회가 성공하면 여러 정보들이 나오는데 그중에서 우리에게 필요한 값은 **Branch Base Position** 이다. 이 값을 복사하던지 기억하자.
4. 이제 실행파일을 다운로드 하기 위해 [chromium-browser-snapshots](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html){:target="_blank"} 페이지를 연다. 그런다음 원하는 플랫폼을 선택하면 숫자로 된 폴더들이 나올텐데, **Filter** 항목에 전단계에서 복사한 **Branch Base Position**을 넣으면 폴더가 짜잔! 하고 나올 것이다.

참고로, 내가 찾고자 했던 버전은 39.x 버전이었는데 이 버전의 경우 2번에서 검색하니, **Branch Base Position** 값이 나오지 않았다. 크롬 블로그에서 버전을 검색해보니 14년 12월 경의 버전이었고, 결국 폴더를 뒤져가면서 비슷한 시점의 버전을 선택했다. 

아참. 39.x 버전은 32비트밖에 없으니 `Win_x64`가 아닌 `Win` 폴더를 뒤져야 한다. :smile:


## 참고

- [Downloading old builds of Chrome / Chromium](https://www.chromium.org/getting-involved/download-chromium#TOC-Downloading-old-builds-of-Chrome-Chromium){:target="_blank"}