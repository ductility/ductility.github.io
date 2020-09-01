---
layout: post
title: iCampusCheck(아캠체크)
subtitle: 성균관대 과제/강의 확인 크롬확장프로그램
background: '/img/posts/07.jpg'
comments: true
---

# iCampusCheck(아캠체크)

<img src="https://raw.githubusercontent.com/ductility/images/master/iCampusCheck(0.1.0).gif" width="100%">

2020년 1학기, 코로나19 바이러스로 인해 성균관대는 전면 온라인 수업을 실시했다. 그리고 Learning-X를 이용한 [차세대 아이캠퍼스](https://icampus.skku.edu/)를 도입했다. 차세대 아이캠퍼스의 디자인은 깔끔해 졌지만, 오히려 들어야 할 강의와 해야 할 과제를 확인하는 방법은 더 불편해졌다. 그래서 버튼 클릭 한번으로 한눈에 할 일을 확인할 수 있는 크롬확장프로그램 [iCampusCheck(아캠체크)](https://chrome.google.com/webstore/detail/icampus-check/hackfjdbiccajlckgjnkejepipjjbepm?hl=ko)를 만들었다.
<br>
<br>

## 설치 및 사용법

#### 1. 크롬 웹스토어에서 [iCampusCheck(아캠체크)](https://chrome.google.com/webstore/detail/icampus-check/hackfjdbiccajlckgjnkejepipjjbepm?hl=ko)를 설치한다.

![_config.yml]({{ site.baseurl }}/img/posts/200902/설치.jpg){: width="100%"}

#### 2. 아이캠퍼스(canvas.skku.edu)에서 대시보드 표기 과목 설정

![_config.yml]({{ site.baseurl }}/img/posts/200902/모든과목.jpg){: width="100%"}

<br>

#### 3. 프로그램 사용

canvas.skku.edu에 로그인 한 뒤 확장프로그램 아이콘을 누르면 실행된다. 잠시 로딩을 기다리면 마감기한이 남은 강의와 과제를 남은시간이 적은 순으로 보여준다. 또한 강의/과제를 클릭하면 상세보기 페이지를 새 창에서 연다.
<br>
<br>

## 주의사항
**"강의콘텐츠"**에 속해있는 자료의 출결/제출 여부를 **"출결/학습현황"**에서 받아오는 것이기 때문에 **"과제 및 평가"**항목이 따로 있는 강의는 과제를 받아올 수 없다. 그런 과목은 따로 확인해야 한다.

그리고 새 창에 어떤 과목의 **"출결/학습현황"** 목록이 뜨는 것은 api 사용을 위한 토큰 쿠키를 발행하기 위한 과정이며 오류가 아니다.
<br>
<br>
<br>

## 간단한 제작 과정

### Postman을 이용한 웹사이트 분석
본격적으로 어플리케이션을 만들기에 앞서 과목별 강의/과제 데이터를 받아오는 방법을 알아내야 한다. Postman의 **Capture requests and cookies** 기능을 이용하면 쉽게 그 방법을 알아낼 수 있다. [핑크곰님의 브런치 포스팅, Postman 사용하기](https://brunch.co.kr/@joypinkgom/86)를 참고했다.

#### 1. Postman과 Postman Intercepter 설치

[Postman](https://www.postman.com/)을 설치하고 크롬 확장프로그램인 [Postman Intercepter](https://chrome.google.com/webstore/detail/postman-interceptor/aicmkgpgakddgnaphhhpliifpcfhicfo)를 설치한다.
<br>
<br>

#### 2. Capture requests and cookies를 이용한 사이트 분석

![_config.yml]({{ site.baseurl }}/img/posts/200902/postman.jpg){: width="100%"}

Capture requests and cookies 기능을 이용하면 History에 상세 request들이 나오고, 거기에서 특정 기능을 하는 request를 골라 사용하면 된다.

예를들어 수강 과목을 받아오려면 Get으로 https://canvas.skku.edu/api/v1/users/self/favorites/courses 에 request 하면 된다.

![_config.yml]({{ site.baseurl }}/img/posts/200902/courses.jpg){: width="100%"}

json으로 수강 과목에 대한 데이터가 나타난다.

<br>

### Chrome Extension 만들기

Postman을 이용해 아이캠퍼스 사이트에서 데이터를 가져오는 방법을 파악했으니, Chrome Extension을 제작한다.

#### 1. 데이터 가져오기
jquery ajax를 이용해서 브라우저에 과제/강의에 대한 데이터를 가져온다. 

**executescript.js 중 일부**
```javascript
var get_courses = {
    "url": "https://canvas.skku.edu/api/v1/users/self/favorites/courses",
    "method": "GET",
    "timeout": 0,
    "async": false,
    "dataType": "json"
};
$.ajax(get_courses).done(function (response) {
    for(var i=0; i<response.length; i++) {
        if(true){
            var courseData = {
                "name":response[i].name,
                "id":response[i].id
            }
            if(userID==null) userID = response[i].enrollments[0].user_id;
            course_Array.push(courseData);
        }
    }
    console.log(course_Array);
});
```

수강 과목을 가져오는 API를 이용해 json데이터를 얻은 뒤, 과목의 이름과 id 처럼 필요한 데이터만 수집하는 코드이다. 비슷한 방법으로 과목별로 강의/과제에 대한 데이터를 수집한다. 이어서 데이터를 POPUP 창으로 넘겨준다.

<br>

#### 2. POPUP 창에 강의/과제 표기

넘겨받은 데이터를 정렬해서 POPUP 창에 들어야 할 강의와 해야 할 과제를 나타낸다.

![_config.yml]({{ site.baseurl }}/img/posts/200902/popup.png){: width="100%"}

<br>

## 통계
![_config.yml]({{ site.baseurl }}/img/posts/200902/statistics.jpg){: width="100%"}

크롬 개발자 대시보드 탭에서 사용량 통계를 볼 수 있다. 5월 2일에 첫 등록을 했고 5월 22일 에브리타임 정보게시판에 아캠체크에 대한 게시물을 올리고 나서 폭발적으로 사용자 수가 늘었다. 가장 많을 때에는 838명이 사용하기도 했다.

<br>

## 후기

처음에는 금방 뚝딱뚝딱 만들수 있겠다 싶어 만만하게 보고 시작했던 프로젝트였는데, 만들다 보니 어려운 부분이 많아 한 달을 꼬박 투자해서 겨우 완성시켰다. 우선 Chrome Extension 개발 관련 자료들이 생각보다 많지 않아서 고생했다. 그리고 웹 개발 초심자여서 크로스도메인 이슈 등 몰라서 생기는 오류가 많이 생겼고, 구글링하며 해결하는데 많은 시간이 걸렸다.

![_config.yml]({{ site.baseurl }}/img/posts/200902/everytime.png){: width="100%"}

시간과 노력을 듬뿍 쏟아서인지, 완성해서 작동시켰을 때 그만큼 큰 뿌듯함을 느꼈다. 800명이 넘는 많은 사람들이 내가 만든 어플리케이션을 사용한다는 게 너무 기뻤고, 또 그 사람들이 고마움을 표하는 댓글을 많이 달아줘서 며칠동안이나 신났다. 

내가 만든 건 별로 대단한 건 아니지만 창작하는 사람들 마음이 이런걸까? 코드를 짜면서 에러가 생기고 실행이 안 되면 짜증이 나지만서도, 완성했을 때의 뿌듯함과 사람들에게 받는 인정이 좋아 끊을 수가 없다. 

<br>

## 참고한 링크
* Chrome Extension, Getting Started Tutorial   
    <https://developer.chrome.com/extensions/getstarted>
* Chrome Extension, Page Action
    <https://developer.chrome.com/extensions/pageAction>
* 생활코딩, 웹페이지에서 공부한 단어의 수를 세기 (크롬 확장 기능 만들기)   
    <https://opentutorials.org/module/2503/14051>
* 유튜브 서기, 크롬확장프로그램 만들기 #1. 특정 사이트의 input값 변경하기   
    <https://www.youtube.com/watch?v=f3NLUDVB23Q>
* Postman을 이용한 크롤링   
    <https://brunch.co.kr/@joypinkgom/86>
* 코딩팩토리, Ajax를 활용하여 다른페이지에 있는 데이터 받아오기   
    <https://coding-factory.tistory.com/144>
* Stack OverFlow, Chrome Extension “Refused to load the script because it violates the following Content Security Policy directive”   
    <https://stackoverflow.com/questions/34950009/chrome-extension-refused-to-load-the-script-because-it-violates-the-following-c>

<br>

## Github
<https://github.com/ductility/iCampusCheck>

<br>