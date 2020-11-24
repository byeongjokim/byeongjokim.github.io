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
    - if not status == 1:
        jobs.append(job)
    """

if __name__ == "__main__":
    app.run(host="0.0.0.0", port="8080")
```

### Dockerfile
```dockerfile
FROM python:3.6
RUN pip install flask
COPY . /app
WORKDIR /app
ENTRYPOINT ["python"]
CMD ["run.py"]
```

### master.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: blur-master-service
  namespace: blur
spec:
  selector:
    app: blur-master
  ports:
  - name: http
    protocol: TCP
    port: AAAA
    targetPort: BBBB
  type: LoadBalancer

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blur-master
  namespace: blur
spec:
  selector:
    matchLabels:
      app: blur-master
  replicas: 1
  template:
    metadata:
      labels:
        app: blur-master
    spec:
      containers:
      - name: blur-master
        image: blur-master:latest
        ports:
        - containerPort: BBBB
        volumeMounts:
        - mountPath: /data
          name: videos
      volumes:
      - name: videos
        hostPath:
          path: /videos/path/
```

## Worker
### run.py
```python
# load DL model
MODEL_WEIGHT = os.getenv('MODEL_WEIGHT', "/MODEL/WEIGHT.pth")
detector = load_model(MODEL_WEIGHT)

# set ip&port of master service
master_ip = os.getenv('MASTER_IP', 'http://127.0.0.1:8080')

def get_job():
    # master_ip + "/get_job" 으로 GET 요청
    # return job

def set_fin_job(job, status):
    # master_ip + '/fin_job?job={job}&status={status}'.format(job=job, status=status) 으로 GET 요청

def inference(job):
    # Blur all faces in video
    # blur 처리된 모든 frame과 기존 영상 속 오디오를 이용하여 새 영상 생성

def main():
    while True:
        job = get_job()
        status = 0
        
        if not job:
            break
        
        try:
            # 새로운 영상 생성 성공시
            inference(job)
            status = 1
        except:
            # 실패시
            status = -1
        
        set_fin_job(job, status)
```

### Dockerfile
```dockerfile
FROM pytorch/pytorch:1.4-cuda10.1-cudnn7-runtime
RUN apt-get update
RUN apt-get install -y libgl1-mesa-dev
RUN apt-get install -y libgtk2.0-dev
RUN apt-get install -y ffmpeg
RUN pip install opencv-python cmake
RUN pip install dlib
RUN pip install ujson
COPY . /app
WORKDIR /app
ENTRYPOINT ["python"]
CMD ["run.py"]
```

### worker.yaml
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: blur-worker
  namespace: blur
spec:
  parallelism: 10
  template:
    spec:
      containers:
      - name: blur-worker
        image: blur-worker:latest
        env:
        - name: MASTER_IP
          value: "http://xxx.xxx.xxx.xxx:AAAA"
        - name: MODEL_WEIGHT
          value: "/MODEL/WEIGHT.pth"
        - name: OUTPUT
          value: "/OUTPUT/PATH"
        resources:
          limits:
            nvidia.com/gpu: 1
        volumeMounts:
        - mountPath: /data
          name: videos
      restartPolicy: Never
      volumes:
      - name: videos
        hostPath:
          path: /videos/path/
  backoffLimit: 2
```

## References
- [커피고래의 노트](https://coffeewhale.com/)
- [핵심만 콕! 쿠버네티스](http://www.yes24.com/Product/Goods/92426926)
