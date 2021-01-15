---
title: "Github.io 블로그에 댓글창 만들기 (minimal-mistakes)"
date: 2021-01-15
categories: [Git]
tags: [Github, Github.io, disqus]
comments: true
---

친구에게 블로그를 보여줬더니 댓글을 달고싶다고 했다.  
다른 블로그보고 따라하면 되겠지~ 했는데 그 사이에 많이 바뀐건지.. 좀 헤맸다.  

### 1. Disqus 계정 만들기
Github에서 댓글기능을 제공해주지는 않는다. Jekyll에서 지원해준다.  
그래서 대부분 Disqus에서 제공하는 댓글 기능을 이용한다.  
일단 [Disqus](https://disqus.com/) 에서 계정을 만들어야한다.

![start](https://user-images.githubusercontent.com/77476913/104708485-20488580-5761-11eb-9d99-41bfb35280d8.PNG)

Get started를 클릭해서 계정을 만든다. 나는 구글계정이랑 연동시켰다.  

![start2](https://user-images.githubusercontent.com/77476913/104708521-2c344780-5761-11eb-9b9c-7593bc4b0f51.PNG)

I want to install Disqus on my site 선택한다.
이게 바로 안뜬다면 왼쪽 상단 프로필 > Add Disqus To Site > 최하단 Get Started 선택하면 된다.

![shortname](https://user-images.githubusercontent.com/77476913/104708533-30606500-5761-11eb-959b-a3829ff1b260.PNG)

여기 website name을 적으면 되는데 아래 your unique ~~~ : ydydydy.disqus.com  
이 ydydydy 부분이 나중에 shortname이 된다.  
카테고리와 언어를 정해주고 Create site 선택  
그 다음에 유료 결제를 유도하는데 우리는 돈이 없으니 그냥 무료인 basic을 선택하면 된다.  

![jekyll](https://user-images.githubusercontent.com/77476913/104708544-348c8280-5761-11eb-96cb-2989e2d7b1c4.PNG)

어느 플랫폼에 설치하는지 물어보는 항목에서는 Jekyll을 찾아서 선택

![config](https://user-images.githubusercontent.com/77476913/104708562-39e9cd00-5761-11eb-947c-8c43ff95e7f3.PNG)

Website URL 넣어주고 다음으로 넘어가면 된다. 

![shortname2](https://user-images.githubusercontent.com/77476913/104709618-8255ba80-5762-11eb-9f14-db6858c36e74.png)

이 화면에서도 내 숏네임이 뭔지 확인 가능하다.
<br>
<br>

### 2. Config 수정하기
여기서부터 좀 꼬였는데... 다른 블로그들이랑 나의 config가 차이가 났다.
그래서 혼자 어찌저찌 열심히 해봤다.

먼저 _config.yml을 연다.

```
comments:
  provider               : "disqus" # false (default), "disqus", "discourse", "facebook", "staticman", "staticman_v2", "utterances", "custom"
  disqus:
    shortname            : "ydydydy" # https://help.disqus.com/customer/portal/articles/466208-what-s-a-shortname-
  discourse:
    server               : # https://meta.discourse.org/t/embedding-discourse-comments-via-javascript/31963 , e.g.: meta.discourse.org
  facebook:
    # https://developers.facebook.com/docs/plugins/comments
```
여기서는 두 부분을 고쳐야 하는데  
먼저 provider가 처음에는 주석처리 되어있는데 이걸 disqus로 변경해준다.  
그리고 disqus의 shortname에 아까 계정만들면서 확인한 shortname을 넣어준다.

그리고 맨 아래부분에 Default 부분에서 comments가 default false인데 이걸 true로 바꿔준다.

```
# Defaults
defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: true
      related: true

```

config 수정은 끝
<br>
<br>

### 3. 본문 수정하기
마지막으로 포스팅의 헤더에서 comments: true를 추가해준다.

```
---
title: "Github.io로 블로그 만들기"
date: 2021-01-15 14:08:00 -0400
categories: Git
tags: [Github, Github.io]
comments: true
---
```

그러면 이렇게 포스팅 아래에 댓글창이 달린다.

![check](https://user-images.githubusercontent.com/77476913/104708585-3eae8100-5761-11eb-9398-40c27b858183.PNG)

각자 사용하는 Jekyll theme에 따라 방법이 조금씩 다를 것 같고  
같은 테마여도 버전에 따라 좀 다르지싶다.  
일단 난 위의 방법으로 성공했다.

다음은 카테고리 만들기



