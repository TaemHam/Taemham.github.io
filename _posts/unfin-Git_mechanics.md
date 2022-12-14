---
title: "[IT|Git] Git은 어떻게 작동하는가?"
author: TaemHam
date: 2022-08-31 16:32:00 +0900
categories: [IT, General, Git]
tags: [Git, Mechanics, Repository, Git Objects, ]
---

## 개요
***

<img src="https://git-scm.com/images/logos/downloads/Git-Logo-2Color.png" width="50%" height="50%"/><em>출처: https://git-scm.com/downloads/logos</em>

소스코드의 버전들을 효과적으로 관리하기 위해 필수적인 시스템인 Git. 하지만 아직 협업 경험이 없는 나는, 왜 그렇게 커밋을 자주 하라고 하는지, 커밋 메시지를 자세히 작성하라고 하는지 잘 모른 채로 개발 공부를 해왔다. 개발자 지망생으로서 이런 걸 모른 채 있을 수는 없기에, 이번에는 Git이 소스코드를 언제, 어디에, 어떻게 저장하는지 알아보려 한다.

## 정의

Git은 **소스코드를 효과적으로 관리하기 위해 개발된 DVCS(분산형 버전 관리 시스템)**이다.

DVCS가 무엇인지 설명하기 전에, 앞의 D를 뺀 VCS(버전 관리 시스템)가 무엇인지부터 알아보자. 예를 들어, 어떤 파일을 편집하고 저장하는 작업을 여러 번 한다고 하자. 만약 마지막에 저장한 편집 내용을 무르고, 이전 내용으로 돌리고 싶을 때, 가장 간단한 방법은 저장 내용을 다른 곳에 복사해 두는 것이다. 하지만 이렇게 하면 저장할 때마다 새롭게 복사하는 번거로움은 물론, 복사본들의 용량까지 감당해야 한다. 이런 걱정 없이 편리하게 수정 내역을 왔다 갔다 할 수 있도록 도와주는 것이 바로 VCS 이다. 
이 외에도, 여럿이서 한 파일을 수정하는 작업을 할 때에는, 한 사람이 수정한 내용이 뒷 사람의 수정 내용 때문에 없어져버릴 위험이 생긴다. 이런 위험을 막아주려면 수정할 파일을 '분산'시켜서 각각의 컴퓨터에서 수정한 후, 어떤 내용이 바뀌었는지 비교하며 저장해주어야 한다. 이런 작업을 해주는 것이 DVCS이다.

Git 외에도 소스 코드의 버전을 관리하는 소프트웨어는 CVS, Subversion, Perforce, Bazaar 등 여러가지가 있다. 하지만 Git이 이들과 다른 점은 **스냅샷** 방식으로 내용을 관리한다는 것이다. 스냅샷 방식으로 저장한다는 것은, 파일이 수정 되었을 때 파일 하나하나의 수정 사항들만 저장하는 것이 아닌, 프로젝트에 포함된 모든 파일들을 통째로 저장해놓고 수정된 내용을 비교한다. 이 방식이 용량이 커질까 걱정되겠지만, 어느정도 저장하고 나서는 가장 마지막 파일을 제외한 나머지를 각 시점의 파일들이 어떻게 수정 되었는지, 즉 "델타" 파일로 변환한다. 이렇게 시점별로 저장해 놓은 것들을 모아서 보면, 사진을 연속해서 찍는 것과 비슷해 스냅샷이라 이름이 붙여졌다고 한다.

## 동작 원리
***

먼저 Git이 어떻게 프로젝트를 관리하는지 간단하게 서술하면, 다음과 같다.
1. 프로젝트를 관리할 로컬 저장소를 만든다.
2. 프로젝트를 주시하며, 바뀐 내용들을 통째로 저장한다.
3. 저장한 내용을 탐방하며 버전을 오간다.

이제 이 간략하게 정리된 내용을 한줄 한줄 들여다보자.

### 로컬 저장소
***

내 컴퓨터(local)에 있는 프로젝트에서 `git init`을 할때, 혹은 원격 서버(remote)에 저장된 프로젝트를 내 컴퓨터에 가져오려고 `git clone`을 할 때, 해당 폴더 내에 `.git`이라는 폴더가 생성된다. 이 폴더에는 개발 과정에서의 수많은 커밋과 브랜치, 관리되는 파일의 정보들이 저장된다.

<img src= "https://user-images.githubusercontent.com/95671168/188423521-fe39398c-7da6-404a-a56b-79b378bac972.png" width="70%" height="70%"/><em>.git 폴더 내부 구조</em>

.git 폴더 내부는 여러 폴더들과 확장자명도 없는 파일들로 가득하다. git의 내부 구조를 이해하기 위해 모든 것을 전부 살펴볼 필요는 없고, Branch와 Commit에 기여하는 것들만 이해해도 된다.

#### Commit 관련

