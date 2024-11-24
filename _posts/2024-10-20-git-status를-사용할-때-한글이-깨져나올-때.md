---
layout: post
title: git status를 사용할 때 한글이 깨져나올 때
categories:
  - 개발
  - Linux
tags:
  - Linux
date: 2024-10-20 22:38 +0900
---

![picture 0](/assets/img/posts/d75bdef54493a101e187b120b6a2582fb545a5cc1e5f2422cfc9fb1473fed209.jpeg)

사진에서 보이는 것과 같이 한글이 저런식으로 깨져서 나온다.  
어떻게 검색해야 할 지도 몰라서 망설이다가 개발자 디스코드 방이 질문했더니 친절하게 답변해주셨다.

```bash
git config --global core.quotepath false
```

> git이 0x80 이상의 character 값을 핸들링할 때 해당 옵션이 true이면 unusual value로 취급해서 제대로 표현 안하고 유니코드 값으로 출력한다고 합니다  
> false로 해줘서 이것도 문자다~ 라고 알려주는 거에요

...라고 하셨는데 뭔가 한글을 정상 문자라고 표시해준다는 내용인 것 같다.

아무튼
![picture 1](/assets/img/posts/d8e048a4638ab4c3a96ab589440a0bce22a6375d441996c91f5ccb26a2eb94c9.jpeg)  
해결~!
