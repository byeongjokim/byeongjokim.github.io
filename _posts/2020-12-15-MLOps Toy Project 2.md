---
layout: post
title:  "거대한 데이터인 척, mnist로 MLOps 구축하기 #2"
description: CI/CD using Github Action
date:   2020-12-15 00:00:00 +0900
categories: MLOps
use_math: true
---

- [프로젝트 소개 및 정보](https://byeongjokim.github.io/posts/MLOps-Toy-Project-0/)
- [MLOps System Design](https://byeongjokim.github.io/posts/MLOps-Toy-Project-1/)
- [CI/CD using Github Action](https://byeongjokim.github.io/posts/MLOps-Toy-Project-2/)

![pipeline](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/mlops2/pipeline.png){: width="100%"}

이 포스팅에는 위의 CI - CD 단계에 관련된 내용을 담고 있다.

## Github Action
이 프로젝트는 Gtihub을 통해 관리 되었으며, Jenkins를 사용하여 CI/CD를 구축 하려다가 평소에 써보고 싶었던 Github Action을 사용하기로 결정하였다. 깃헙 메뉴의 Actions을 통해 이를 설정할 수 있다. 각 Workflows는 YML 파일을 통해 관리가 가능하며, 이 프로젝트에서는 **CI**, **CD** 두 workflow을 설정하였다. 

![github action](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/mlops2/githubaction1.PNG){: width="100%"}

### CI

```yaml
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
```

CI workflow는 main brunch 로 push가 될 때 발생하는 workflow 이다. 위 코드부분과 같이 **on**으로 설정할 수 있다. push, pull_requets 뿐만 아니라 workflow_run 을 통해 타 workflow의 상태를 trigger로 사용할 수도 있다.

```yaml
build:
    runs-on: ubuntu-latest
    steps:          
      - name: Check out
        uses: actions/checkout@v2
      
      - name: Docker Login
        uses: docker/login-action@v1.8.0
        with:
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}

      - name: Build Images
        run: |
          docker build kubeflow_pipeline/0_data -t byeongjokim/mnist-pre-data
          docker push byeongjokim/mnist-pre-data
          (중략)
```

CI workflow 에서는 docker 이미지를 빌드 후 push 하는 과정이 이루어진다. github action의 장점으로는 여러가지 오픈소스 actions들을 사용할 수 있다. [url](https://github.com/marketplace?type=actions)에서 사용하고 싶은 action을 검색 후 **uses**를 통해 사용이 가능하다.

![github action](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/mlops2/githubaction2.PNG){: width="100%"}

```yaml
      - name: Slack Notification
        if: always()
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_ICON_EMOJI: ':bell:'
          SLACK_CHANNEL: mnist-project
          SLACK_MESSAGE: 'Build/Push Images :building_construction: - ${{job.status}}'
          SLACK_USERNAME: Github
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
```

그리고 CI의 결과를 Slack으로 알릴 수 있다. 실패할 때도 알려야 하기 때문에, **if: always()**를 설정하였다. 전체 CI Workflow YML 파일은 아래와 같다.

```yaml
name: CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:          
      - name: Check out
        uses: actions/checkout@v2
      
      - name: Docker Login
        uses: docker/login-action@v1.8.0
        with:
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}

      - name: Build Images
        run: |
          docker build kubeflow_pipeline/0_data -t byeongjokim/mnist-pre-data
          docker push byeongjokim/mnist-pre-data
          (중략)
      
      - name: Slack Notification
        if: always()
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_ICON_EMOJI: ':bell:'
          SLACK_CHANNEL: mnist-project
          SLACK_MESSAGE: 'Build/Push Images :building_construction: - ${{job.status}}'
          SLACK_USERNAME: Github
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### CD

```yaml
on:
  workflow_run:
    workflows: ["ci"]
    branches: [main]
    types: 
      - completed
```

CD Workflow의 경우 ci worfklow가 완료 된 후 시작이 된다.

```yaml
      - uses: actions/setup-python@v2
        with:
          python-version: '3.6.12'
          architecture: x64
      
      - uses: BSFishy/pip-action@v1
        with:
          packages: |
            kfp==1.1.2

      - name: run pipeline to kubeflow
        run: python kubeflow_pipeline/pipeline.py
```

우선 kubeflow pipeline python SDK를 실행하기 위해 kfp를 설치하였다. 그 후 배포는 **python kubeflow_pipeline/pipeline.py** 로 이루어진다. 다음 포스팅에서 설명하겠지만 pipeline.py를 실행하면 자동으로 지정된 서버의 kubeflow로 pipeline으로 업로드(배포) 되도록 개발이 되어있다. CD workflow에도 마찬가지로 Slack으로 상태를 알리도록 하였다. 다음은 CD Workflow YML 파일의 전체이다.

```yaml
name: CD

on:
  workflow_run:
    workflows: ["ci"]
    branches: [main]
    types: 
      - completed

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:          
      - name: Check out
        uses: actions/checkout@v2
          
      - uses: actions/setup-python@v2
        with:
          python-version: '3.6.12'
          architecture: x64
      
      - uses: BSFishy/pip-action@v1
        with:
          packages: |
            kfp==1.1.2

      - name: run pipeline to kubeflow
        run: python kubeflow_pipeline/pipeline.py

      - name: Slack Notification
        if: always()
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_ICON_EMOJI: ':bell:'
          SLACK_CHANNEL: mnist-project
          SLACK_MESSAGE: 'Upload & Run pipeline :rocket: - ${{job.status}}'
          SLACK_USERNAME: Github
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK_URL }}
```

![github action](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/mlops2/githubaction3.PNG){: width="100%"}

CI 또는 CD의 Workflow는 위와 같이 단계별로 에러와 상태를 확인 할 수 있다. 

이 프로젝트는 위의 CI와 CD를 통해 파이프라인이 배포가 되며 학습이 이루어진다. 다음 포스팅에서는 kubeflow pipeline을 이용한 학습에 대한 내용을 나눠서 올릴 예정이다.

## References
- [Github Action](https://docs.github.com/en/free-pro-team@latest/actions)