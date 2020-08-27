---
layout: post
title:  "MLOps - Toy Project #2"
description: Jenkins CI/CD using Docker
date:   2020-08-14 00:00:00 +0900
categories: MLOps_Project
use_math: true
---

Dev 서버는 총 3가지를 위해 사용된다.

- Development + Jenkins + Kafka Server
    - 소스 코드 개발
    - CI/CD 수행
    - 신규 학습 데이터 전송

Jenkins와 Kafka는 Docker Image 로 실행을 할 것이며, 소스 코드는 Github와 Docker hub push 를 통해 CI/CD가 이루어 질것이다. 따라서 Docker CE를 우선 설치하였다.

Docker 설치는 [Post](https://byeongjokim.github.io/posts/install-kubeflow/)를 참고하면 된다.

```
$ yum install -y yum-utils device-mapper-persistent-data lvm2
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ yum install docker-ce
$ sudo systemctl start docker && systemctl enable docker
```

## Jenkins install using Dockerfile
```
FROM jenkins/jenkins:lts
USER 0
RUN apt-get update && \
    apt-get -y install apt-transport-https \
        ca-certificates \
        curl \
        gnupg2 \
        software-properties-common && \
    curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
    add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
        $(lsb_release -cs) \
        stable" && \
    apt-get update && \
    apt-get -y install docker-ce
```
```
# docker build/run jenkins images
# jenkins container에 tcp로 접속하기 위해 docker.sock 설정
$ sudo docker build -t byeongjokim/jenkins .
$ sudo docker run -d -p 8080:8080 -v /var/run/docker.sock:/var/run/docker.sock --name jenkins byeongjokim/jenkins
```

## CI/CD to Docker Hub using Github Webhook
Github Master branch에 PUSH가 이루어지면 자동으로 Docker Build/Push 가 이루어지게 하기 위해 Jenkins Pipeline을 설정하였다.

### Github Webhook Setting
![settings0](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/toy2/github_settings1.PNG)

### Jenkins Pipeline Setting
![settings1](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/toy2/pipline_settings1.PNG)
![settings2](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/toy2/pipline_settings2.PNG)

### Install Jenkins Plugin
- CloudBees Docker Build and Publish plugin
- GitHub Integration Plugin
- Docker

### Jenkins Credentials Setting
![settings1](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/toy2/jenkins_settings.PNG)

### Create Jenkinsfile
```
pipeline{
    agent any
    stages{
        stage("Checkout Code"){
            steps{
                script{
                    checkout scm
                }
            }
        }
        stage("Unit Test"){
            agent { docker { image 'python:3.6-alpine' } }
            steps{
                sh 'pip install flask'
                sh 'python test.py'
            }
        }
        stage("Docker Build"){
            steps{
                script{
                    app = docker.build("byeongjokim/toy-project")
                }
            }
        }
        stage("Docker Push"){
            steps{
                script{
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerHub'){
                        app.push()
                    }
                }
            }
        }
    }
}
```

### Create dockerfile
```
FROM python:3.6-alpine
RUN pip install flask
COPY . /app
WORKDIR /app
ENTRYPOINT ["python"]
CMD ["app.py"]
```

이제 Github에 push를 하면 자동으로 Docker build가 이루어지며, Docker Hub에 push가 이루어진다.
![result](https://raw.githubusercontent.com/byeongjokim/byeongjokim.github.io/master/assets/images/toy2/result.PNG)

## References
- Jenkins
    - [Install](https://shmoon.tistory.com/11)
    - [How to](https://www.youtube.com/watch?v=nMLQgXf8tZ0)
    - [Docker Build](https://www.youtube.com/watch?v=z32yzy4TrKM)
    - [Pipeline](https://dzone.com/articles/adding-a-github-webhook-in-your-jenkins-pipeline)


