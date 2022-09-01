---
title: "[IT|Git] Git은 어떻게 작동하는가?"
author: TaemHam
date: 2022-08-31 16:32:00 +0900
categories: [IT, General, Git]
tags: [Git, Commands, Branch, Switch, Merge, Rebase, Reset, Revert, Reflog, Cherry-pick]
---

## 개요
***

<img src="https://git-scm.com/images/logos/downloads/Git-Logo-2Color.png" width="50%" height="50%"/><em>출처: https://git-scm.com/downloads/logos</em>

소스코드의 버전들을 효과적으로 관리하기 위해 필수적인 시스템인 Git. 하지만 아직 협업 경험이 없는 나는, 왜 그렇게 커밋을 자주 하라고 하는지, 커밋 메시지를 자세히 작성하라고 하는지 잘 모른 채로 개발 공부를 해왔다. 개발자 지망생으로서 이런 걸 모른 채 있을 수는 없기에, 이번에는 Git이 어떻게 버전을 관리하는지, 어떤 명령어들이 자주 쓰이는지에 대해 정리해보려 한다.

## 동작 원리

먼저 Git이 어떻게 동작하는지 간단하게 서술해 보겠다.
1. 소스코드를 버전을 관리할 저장소를 만든다.
2. 소스코드 내용을 주시하며, 바뀐 내용을 해싱하여 저장한다.
3. 