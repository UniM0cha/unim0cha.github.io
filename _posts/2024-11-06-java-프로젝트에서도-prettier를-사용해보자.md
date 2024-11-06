---
layout: post
title: Java 프로젝트에서도 Prettier를 사용해보자
categories:
  - 개발
  - Java
tags:
  - Java
image: "/assets/img/posts/fd1a01f2bffe5fedb8be6883abd090a9758c0b425f32f88e9aad6991d39a921d.png"
date: 2024-11-06 22:14 +0900
---

## Prettier?

Prettier는 node 진영에서 코드 포맷을 포맷팅 하기 위해 사용하는 라이브러리입니다.  
스캔 대상에 등록된 파일 전체를 스캔하여 설정한 코드 포맷 규칙에 따라서 코드를 포맷팅 해줍니다.

물론, 이 라이브러리는 포맷팅에 node를 사용하기에 보통 Javascript 프로젝트에만 사용되지만,  
Java 언어를 포맷팅 해주는 `prettier-java` 플러그인이 존재하기에, Java 프로젝트에도 적용해보도록 하겠습니다.

## Prettier-Java 설치

Java 프로젝트가 있는 곳에서 아래 명령어를 실행하여 `prettier`와 `prettier-plugin-java`를 설치해줍니다.

```bash
npm install prettier prettier-plugin-java
```

그 후, 프로젝트 루트 디렉토리에 `.prettierrc.json` 파일을 만들어 플러그인을 적용시킴과 함께 포맷팅 규칙을 설정해줍니다.

```json
{
  "plugins": ["prettier-plugin-java"],
  "printWidth": 120
}
```

> 만약 git을 사용하고 있다면, Java 프로젝트를 만든 후 node_modules 디렉토리가 ignore 되어있지 않을테니  
> 꼭 .gitignore에 추가해줍시다.

## Prettier-Java 활성화

설치 후 사용하는 IDE에서 Prettier를 활성화 해주어야합니다.  
워낙 유명한 라이브러리이다보니 웬만한 IDE에서는 Prettier를 지원해줍니다.  
다만, Java에서 사용하려면 약간 설정을 해주어야 합니다.  
IntelliJ를 기준으로 설명합니다.

1. [설정 > 도구 > 저장 시 액션] 에서 `Prettier 실행`에 체크 표시를 해준 후 마우스를 올려 `구성...`에 들어갑니다.
   ![picture 0](/assets/img/posts/94ea179c82b51ae74da1c51718955b019f441a5352def626f821679bc45ebcd0.png)
2. `자동 Prettier 구성`을 선택해주고,  
   `다음 파일에 대해 실행:` 부분에 있는 확장자들을 모두 지워서 `**/*`가 되도록 합니다.  
   (기본 설정은 Javascript 언어에 대해서만 실행하기 때문에 Java 파일에서도 적용되도록 한 것입니다.)
   ![picture 1](/assets/img/posts/616fa634aab0208e7d68a64bb9a4d81ad944ccb7a6e13a807ebb0138c7397512.png)

완료가 되었다면, 기존 파일에서 줄바꿈 등 변화를 준 뒤 저장하여 포맷팅이 제대로 되는지 확인할 수 있습니다.

포맷팅이 적용되는것을 확인했다면, 아래 명령어를 사용하여 모든 파일에 대하여 Prettier 포맷팅을 적용하도록 할 수 있습니다.

```bash
npx prettier --write "**/*"
```

## 번외) 포맷팅 잘 적용되었는지 Github Action으로 검증하기

`.github/workflows/prettier-check.yml` 파일에 아래와 같이 작성하여  
PR을 올릴 때 자동으로 Prettier가 잘 적용되었는지 검증할 수 있습니다.

```yaml
name: Prettier Format Check

on:
  pull_request:
    branches: ["develop"]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "20"

      - name: Install dependencies
        run: npm install

      - name: Run Prettier check
        run: npm run prettier:check

      - name: Fail if unformatted code is found
        if: failure()
        run: exit 1
```
