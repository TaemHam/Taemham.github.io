---
title: "[백엔드|스프링부트] 동영상 스트리밍 서비스 API 클론코딩 1주차"
author: TaemHam
date: 2022-10-25 14:00:00 +0900
categories: [Backend, SpringBoot]
tags: [Tutorial, Backend, Spring, SpringBoot, Java]
---

## **API 설계 가이드**

[링크](https://velog.io/@couchcoding/%EA%B0%9C%EB%B0%9C-%EC%B4%88%EB%B3%B4%EB%A5%BC-%EC%9C%84%ED%95%9C-RESTful-API-%EC%84%A4%EA%B3%84-%EA%B0%80%EC%9D%B4%EB%93%9C)

## **기능**

* 메인을 로드하면, 메인에 띄울 영상 ID의 리스트를 불러오고, 리스트의 ID들을 각각 조회해 썸네일, 영상 이름 등의 정보를 불러온다.
  * 메인을 로드할 때마다 각각 다른 영상들이 뜬다. 유튜브라면 추천 알고리즘을 사용하겠지만, 현재로선 불가능하므로 랜덤으로 조합한다.
* 구독 페이지를 로드하면, 사용자에 따라 다른 영상 ID 리스트를 불러오고, 메인처럼 각각 조회한다.
* 기록 페이지도 위와 마찬가지.
* 영상을 클릭하면, 영상을 로드함과 동시에 유저의 시청 기록을 생성한다. 시청 중이던 영상이중단 될 경우, 영상 시청 기록을 업데이트 한다.
* 영상의 댓글은 영상에서 스크롤을 내리면 로드 되므로, 따로 조회하도록 한다.
* 좋아요나 싫어요를 누르면 유저 아이디별로 영상 기록을 조회해 안눌렸으면 +1, 이미 눌려있으면 -1을 하도록 한다.
* [영상 다운로드 및 업로드](https://os94.tistory.com/161)는 [AWS S3의 API](https://docs.aws.amazon.com/ko_kr/AmazonS3/latest/userguide/RESTRedirect.html)를 이용해 영상과 썸네일에 대한 CRUD 연산을 하도록 할 것이므로, 서비스할 서버에서는 해당 정보 이외의 정보들만 저장한다.
  * 영상 업로드, 삭제시에 pub-sub 패턴을 이용해 구독자의 구독 리스트에 해당 영상 ID를 추가한다.

## **API 설계**

|Resource|GET|POST|PUT|DELETE|
|--------|---|----|---|------|
|/users|회원 정보 조회|회원 가입|회원 정보 수정|회원 탈퇴|
|/users/login||로그인|||
|/feed/main|메인 페이지 영상 id 조회||||
|/feed/subscription|구독 영상 id 조회||||
|/feed/history|시청 기록 영상 id 조회||||
|/video|영상 정보 조회|영상 업로드|영상 정보 수정|영상 삭제|
|/watch|재생할 영상 로드|영상에 대한 유저 기록 생성|기록 수정|기록 삭제|
|/watch/comment|댓글 조회|댓글 생성|댓글 수정|댓글 삭제|
|/watch/like|좋아요 정보 조회||좋아요 토글||
|/watch/dislike|싫어요 정보 조회||싫어요 토글||
|/subscribe|구독 정보 조회|구독 토글|||

## **궁금한 점**

* 영상 기록을 조회하고 없으면 생성해야하는데, POST 메소드 하나로 해 요청을 하나로 보낼지, 아니면 GET POST 따로 만들어 요청을 두번 보낼지 모르겠다.
