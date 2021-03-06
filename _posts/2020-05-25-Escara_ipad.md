---
layout: post
title: 아이패드와 로봇팔로 그림 그리기
subtitle: 아이패드에 그린 그림 로봇팔이 따라 그리게하기
background: '/img/posts/05.jpg'
comments: true

date:   2020-05-25 12:00:00 
lastmod : 2020-07-20 12:00:00
sitemap :
  changefreq : weekly
  priority : 1.0
---

## 아이패드 + 로봇팔로 그림 그리기
Node.Js 로 만든 서버는 같은 네트워크를 사용하는 기기에서 접속할 수 있다. 휴대폰과 아이패드로 그 서버에 접속해서 그림을 그렸는데, 예상과는 다르게 그림이 그려지지 않았다. 왜 그런가 알아보니 모바일 기기는 마우스를 사용하지 않아서, Mouse 이벤트를 사용하여 Canvas에 그림을 그리려 하니 작동하지 않았던 것이었다! 모바일 기기에서는 Touch 이벤트를 사용해야 한다는 것을 깨닫고, Touch 이벤트에서 마우스 클릭, 움직임 등에 대응되는 이벤트 객체가 무엇인지 찾아 *main.html*을 수정했다.

그리고 아이패드에서 "홈 화면에 추가"를 이용, 간편하게 접근할 수 있으며 주소창에 가리지 않고 웹 어플리케이션을 사용할 수 있게 하는 코드를 ```<head>```태그 안에 넣어 주었다.

```html
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<meta name="apple-mobile-web-app-title" content="ESCARA">
```


이 부분은 정상 작동하는 것을 확인했다. 하지만 다른 문제가 생겼다.
<br>
<br>

## G-Code 데이터 스트링이 길어지면 오류 발생
아이패드에 근사한 그림을 그려 로봇팔이 따라 그리는 영상을 촬영하려고 했다. 한두획을 그은 뒤 전송하는 테스트는 항상 성공했지만, 제대로 된 그림을 그려 전송하는 실제 촬영에는 예외없이 실패했다. 이유를 몰라 몇 번이고 도전한 끝에 생성된 g-code파일이 이상하다는 것을 발견했다. 일부러 긴 데이터 스트링을 만들어 전송했는데, g-code의 뒷부분이 잘린다는 것을 발견했다. POST는 데이터 전송시 길이 제한을 두지 않는 것으로 알고 있었는데, 단일 스트링에는 제한이 있는 것 같다. 그래서 이 오류를 잡기 위해서 스트링을 특정 문자 수로 나눠 Array에 담아 보낸 뒤 다시 합치는 방법을 선택했다.


```js
function slicegcode(data) {
    var num = 10000;
    var data_array = [];
    while(data.length >= num) {
        data_array.push(data.substr(0, num));
        data = data.substr(num);
    }
    data_array.push(data);

    return data_array;
}
```

이런 식으로 데이터 스트링을 잘라 전송하고, 그것을 다시 모아 g-code파일을 만들었다. 잘린 스트링을 다시 모을 때에는, ```decodeURIComponent```를 이용해 G-code에 사용할 수 있게 만들어 주었다. 그리고 정상적으로 G-code파일이 잘 생성됨을 확인했다.   

<br>

## 작동 영상
[![_config.yml]({{ site.baseurl }}/img/posts/200525/유튜브썸네일.jpg){: width="100%"}](https://www.youtube.com/watch?v=pEVLhL1p7I8=0s){: target="_blank"}   
*이미지를 누르면 해당 동영상 링크가 새 창에서 열립니다.*

아이패드에 애플펜슬을 이용하여 미니언즈를 그리면 이어서 로봇팔이 그린 그림을 따라 그린다.

영상은 4월에 진작 찍어뒀는데, 너무 귀찮아서 미루다보니 벌써 5월 말이 되었다. 이번 작업중에는 예상치 못한 오류들을 만나 고생을 좀 했다. 하지만 해결하는 과정에서 많은 것을 배울 수 있었다. 그림 연습을 좀 더 해서 다음번에는 더 근사한 그림을 그려봐야겠다. 
