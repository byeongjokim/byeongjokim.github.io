---
layout: post
title:  "MLOps - Large Video Processing using Kubernetes"
description: Blur all faces in 300h videos
date:   2020-11-20 00:00:00 +0900
categories: MLOps_Project
use_math: true
---

## Problem
- **300시간**의 비디오 속 모든 인물 비식별화 처리
    - **모든 frame**에 대해 처리
- **Multi-Process**
    - ZMQ(ZeroMQ) 사용하여 통신
    - GPU 2개가 달린 서버
    - **4 프로세스 한계** (GPU 당 2 프로세스)
    - 20개 영상(총 1시간) 전처리 -> 10시간 반 소모
    - 단순 계산으로 300시간 영상 전처리 -> 약 **3000시간(125일)** 소모
- Deadline: 약 **2주**

## Solving
- Kubernetes 사용
- Service, Job을 이용하여 Master, Worker 구현
    - Master: 남은 일 체크 및 일 분배
    - Worker: 받은 일 처리 완료 후 일 요청
- 10 GPU 사용

## Master
### run.py
```python
@app.route("/start")
def start():
    """
    Params(GET):
    - input_path=데이터 위치
    - log_path=로깅 파일 위치
    Descriptions:
    - jobs = make_job_list(input_path)
        - 주어진 input_path의 존재하는 모든 mp4 파일명 리스트를 생성(jobs)
    """ 

@app.route("/get_job")
def get_job():
    """
    Descriptions:
    - job = jobs.pop()
    - return job
    """ 

@app.route("/fin_job")
def fin_job():
    """
    Params(GET):
    - job=해당 job
    - status=상태 (1 or -1)
    Descriptions:
    - if not status == 1
        - jobs.append(job)
    """

if __name__ == "__main__":
    app.run(host="0.0.0.0", port="8080")
```


## References
- [커피고래의 노트](https://coffeewhale.com/)
- [핵심만 콕! 쿠버네티스](http://www.yes24.com/Product/Goods/92426926)
