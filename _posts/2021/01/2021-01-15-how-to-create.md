---
title: "Github.io로 블로그 만들기"
date: 2021-01-15 14:08:00 -0400
categories: [Git]
tags: [Github, Github.io]
comments: true
---

Github.io에 블로그를 만들고 처음 쓰는 포스팅  
  
개발 경력 무려 8년차  
하지만 나에게 8년동안 뭘 했냐고 물어보면 그냥 일 열심히 했는데요? 라는 말 외에는 떠오는 말이 없어서  
더 늦기 전에 개발 기록 겸 포트폴리오 제작을 시작하기로 했다.   
티스토리랑 github.io 중에 뭘로 시작할까 고민했는데 그냥 느낌 가는대로 깃헙으로...  
겸사겸사 지금 현업에서는 쓸 일이 전무한 HTML이랑 마크다운 연습도 해볼 생각이다.  
(예전에 지인이 github.io로 모바일청첩장을 만들었던 적이 있었는데 정말 개발자스럽다는 생각을 많이 했었다.)  
  
<br>
<br>


### 1. Repository 만들기  
[Repository](https://github.com/new) 에서 새로 만들면 되고  
왼쪽 상단 프로필 > Your repositories > New 클릭해도 된다.  

![시작하기](https://user-images.githubusercontent.com/77476913/104692858-03558780-574c-11eb-9feb-37cc73d7a96b.PNG)  

첫번째로 Repository 이름을 넣어주면 되는데 ownername.github.io 형식으로 작성해야한다.  
나의 경우 ydydydyc가 owner 이름이기 때문에 ydydydyc.github.io로 작성하면 된다.  
이 때, owner가 아닌 다른 이름으로 작성하면 404 not found error를 보게 된다.  
**<mark style='background-color: "ffdce0'>반드시 같은 이름으로 할것!</mark>**  
Public, Add a README file 선택한 후 Create repository를 선택하면 생성 끝  
[owner].github.io url로 접속해서 블로그가 잘 뜨면 성공.  
<br>
<br>
  
### 2. Local에 다운받기
로컬에 다운받지 않아도 상관은 없는데, 테마를 사용 할 경우 한계도 있고 강력한 기능들을 제공해주고 있는 다양한 에디터를 사용할 수 없는 점이 여러모로 불편하다.  
만약 git을 사용할 줄 모르는 사람이라면 commit, push 정도만 익혀도 포스팅하는데는 문제는 없다. (하지만 굳이..?)  
여튼 git의 기본적인 사용법을 알고 개발 환경에 git이 install 되어있고, ssh key까지 등록되어있다는 가정 하에!  

![clone](https://user-images.githubusercontent.com/77476913/104693252-b0c89b00-574c-11eb-9556-daad3008787f.PNG)  

Repository 왼쪽에 Code를 눌러서 나오는 링크를 카피하고

```
$ git clone git@github.com:ydydydyc/ydydydyc.github.io.git
```
clone 해서 다운받으면 내 로컬에서 수정 및 포스팅이 가능하다.  
난 IntelliJ와 Sublime Text를 주로 사용하는데 IntelliJ에서는 마크다운(.md)의 경우 프리뷰로 바로 보여준다.

<br>
<br>

### 3. 테마 설정하기
처음 만들어진 블로그는 굉장히 밋밋하게 생겼다.  
다행히 Github에는 공개된 Jekyll theme이 많고, 내가 원하는 테마로 설정하면 된다.  
Jekyll은 Ruby 기반의 웹사이트 생성기고 자세한 설명은 생략한다. 난 루비 안해봄  
그래서 로컬에 Ruby가 설치되어있어야 하지만 좀 찾아보니 그럴 필요 없이 그냥 소스 가져다 넣어도 될 것 같아서 난 루비 설치 없이 진행했다.  
다만 여전히 git은 있어야함  
공개된 [Jekyll theme](https://github.com/topics/jekyll-theme) 중에서 마음에 드는 테마를 고른다.  
난 그냥 제일 상위에 있고 깔끔해 보이고, 많이 쓰는 것 같은 minimal-mistake를 선택했다.  

![minimal](https://user-images.githubusercontent.com/77476913/104693248-ae664100-574c-11eb-8d38-5ebb4835d9d0.PNG)  

github 들어가서 똑같이 프로젝트를 clone 한다.

![minimal-clone](https://user-images.githubusercontent.com/77476913/104693249-af976e00-574c-11eb-83cc-030d0bc107d3.PNG)

fork 해도 되는데 굳이 history까지 내가 알 필요는 없을 것 같고(귀찮아서) 그냥 clone했다.  
clone마저 귀찮다면 zip download 해도 된다. 근데 clone이 편함  
pull 해서 받은 모든 소스를 내 repository 안에 넣는다. **(README.md 제외)**  
```
$ git add .
$ git commit -m "change theme"
$ git push
```
그리고 push 하면 테마 적용 끝

<br>
<br>

### 4. 테마 커스텀하기 
테마를 설정했으니 이제 블로그 이름 등을 수정해주면 된다.  
_config.yml을 오픈한다. 

````
# theme                  : "minimal-mistakes-jekyll"
remote_theme           : "mmistakes/minimal-mistakes"
````

remote_theme에 주석처리가 되어있을텐데 이 라인 주석을 풀어준다.

```
# Site Settings
locale                   : "en-US"
title                    : "YD"
title_separator          : "-"
subtitle                 : # site tagline that appears below site title in masthead
name                     : "Dayoung Choi"
description              : "An amazing website."
url                      : "https://ydydydyc.github.io/" # the base hostname & protocol for your site e.g. "https://mmistakes.github.io"
baseurl                  : ""# the subpath of your site, e.g. "/blog"
```
타이틀 같은 것들은 각자의 판단 하에 적절히 수정하고
url은 만들었던 [owner].github.io로 변경하고, baseurl은 ""로 수정해준다.
```
# Site Author
author:
  name             : "Dayoung Choi"
  avatar           : "/assets/images/bio.png"# path of avatar image, e.g. "/assets/images/bio-photo.jpg"
  bio              : "Software developer"
  location         : "Suwon"
  email            :
  links:
```
Site author를 수정하면

![bio](https://user-images.githubusercontent.com/77476913/104698273-72cf7500-5754-11eb-80cc-c785b520c7db.PNG)

요렇게 변경된다.  
근데 정말 내 소개에 쓸말이 없더라  

<br>
<br>

### 5. 첫 포스팅 하기
루트에 _posts 라는 디렉토리를 하나 생성한다.
그리고 그 하위에 .md 확장자의 파일을 생성해서 적으면 된다. 이렇게
![posting](https://user-images.githubusercontent.com/77476913/104698766-32bcc200-5755-11eb-9854-b78b4b09f72c.PNG)  
<br>

그리고 첫줄은 반드시 아래 형식으로 적어야한다.
```
---
title: "Github.io 블로그 만들기"
date: 2021-01-15 14:08:00 -0400
categories: Git
---
```
다 적었으면 아까처럼 push
```
$ git add .
$ git commit -m "Update my first post"
$ git push
```

다시 나의 깃헙에 들어가보면 업데이트가 되어있다. 

![finish](https://user-images.githubusercontent.com/77476913/104699100-947d2c00-5755-11eb-9d38-8c5be45dce30.PNG)

끝!


+ **전체 폰트 변경하기**  
포스팅 후 들어가보면 노안이 오고있는 개발자를 배려한듯 겁나 큰 폰트 크기에 놀랄것이다.  
  안이뻐서 나는 폰트 크기를 조금 수정했다.  
  /_sass/minimal-mistakes/_reset.scss
  이 파일에서 font-size들을 조금씩 줄여줬다.

```css
html {
  /* apply a natural box layout model to all elements */
  box-sizing: border-box;
  background-color: $background-color;
  font-size: 14px;

  @include breakpoint($medium) {
    font-size: 12px;
  }

  @include breakpoint($large) {
    font-size: 14px;
  }

  @include breakpoint($x-large) {
    font-size: 16px;
  }

  -webkit-text-size-adjust: 100%;
  -ms-text-size-adjust: 100%;
}

```

정말 끝!  
<br>
~~이제라도 티스토리로 가야할까? ^_ㅠ~~

