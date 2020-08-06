---
layout: post
title:  "MLOps - Kubeflow 설치시 error"
description: Kubeflow 설치
date:   2020-07-23 20:03:00 +0900
categories: Kubernetes
use_math: true
---

> 본 포스팅은 [지구별 여행자](https://www.kangwoo.kr/2020/02/17/pc%EC%97%90-kubeflow-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-1%EB%B6%80-nvidia-%EB%93%9C%EB%9D%BC%EC%9D%B4%EB%B2%84-docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/)를 보고 겪은 시행착오를 담았습니다.

1 master node, 3 worker node로 구성된 Kubernetes 환경이 있었다. MLOps에 관심이 생기고, 이 환경에 Kubeflow을 설치해서 ML training 부터 serving 까지의 pipeline을 직접 해보고 싶었다.

Kubeflow 설치를 위해 블로그, medium 등 많은 자료를 뒤적이며 설치해봤지만, 항상 에러를 맛보았다. 본 포스팅에는 대표적으로 어떤 에러가 있었고, 어떻게 해결했는지에 대해 적어본다.

### Kubeflow 설치 시 반복되는 Error
kfctl을 설치 후, **kfctl apply -f kfctl_istio_~~.yaml** 을 통해 istio 기반 kubeflow 설치 해야하는 단계가 있다.

하지만 **webhook.cert-manager.io** 설치가 진행되는 도중 에러가 생겨, 계속 retry 할 때가 있다. 이 retry가 5번 이상 반복되는 것을 기다리면 자동으로 해결이 된다. 하지만 **config.webhook.serving.knative.dev** 설치 중 무한히 에러가 반복될 때가 있다. 

결론부터 얘기를 하자면, 이는 주로 kubeflow를 재설치 할 때 생기는 문제이다. kubeflow을 재설치 하기위해 **kfctl delete -f kfctl_istio_~~.yaml**를 할 경우 관련 모든 resource들이 꺠끗하게 지워지지 않는다.

우선 namespace 부터 terminating stuck이 생긴다. 이는 [delete namespace](https://www.ibm.com/support/knowledgecenter/SSBS6K_3.2.0/troubleshoot/ns_terminating.html)을 참고하면 해결할 수 있다.

하지만, namespace에 관련된 모든 pod, svc, deploy 등을 지웠다 해도, kube-system 등의 namespace에 존재하는 자원들, 혹은 pvc, crd 등은 또 따로 삭제를 해야한다. [Kubeflow Github Issue](https://github.com/kubeflow/kubeflow/issues/3767)를 보면 어느정도 삭제를 할 수 있지만, [쿠브플로우 쿠버네티스에서 머신러닝이 처음이라면!](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&barcode=9788960883055) 책을 통해 좋은 [code](https://github.com/mojokb/kubeflow-book/blob/master/uninstall/kubeflow-uninstall.txt)를 발견하였다.

위 code를 통해 kubeflow, istio 등 과 관련된 resource를 지우고 다시 설치해보면 **config.webhook.serving.knative.dev** 에러 없이 설치가 완료되는 것을 확인할 수 있다.

### istio-proxy OOMKilled
Kubeflow 설치가 완료된 후 설치된 pod 등을 확인해보면, istio-system namespace 안에 주기적인 에러로 계속 재시작되는 pod(istio-policy를 포함한)을 확인할 수 있다. 이는 공통된 하나의 container에서 OOMKilled error가 발생해서 생긴 일이다. 해당 pod 들을 kubectl describe 명령어로 살펴보면 모두 **istio-proxy** container를 사용한다. 

```
istio-proxy:
    Container ID:  ...
    Image:         docker.io/istio/proxyv2:1.1.6
    Image ID:      ...
    Ports:         9091/TCP, 15004/TCP, 15090/TCP
    Host Ports:    0/TCP, 0/TCP, 0/TCP
    Args:
      proxy
      --domain
      $(POD_NAMESPACE).svc.cluster.local
      --serviceCluster
      istio-telemetry
      --templateFile
      /etc/istio/proxy/envoy_telemetry.yaml.tmpl
      --controlPlaneAuthPolicy
      NONE
    State:          Running
      Started:      Wed, 22 Jul 2020 09:31:43 +0900
    Last State:     Terminated
      Reason:       OOMKilled
      Exit Code:    255
      Started:      Wed, 22 Jul 2020 09:23:00 +0900
      Finished:     Wed, 22 Jul 2020 09:26:39 +0900
    Ready:          True
    Restart Count:  81
    ...
```

이를 해결하기 위해 몇 번이나 재설치를 해봤지만 해결되지 않았고, istio를 따로 설치 해보았다. istio 설치는 [URL](https://sarc.io/index.php/cloud/1834-install-istio-on-minikube)을 참고하였다. 그 후, istio 제외 한 kubeflow을 설치 해주면 된다. 

**kfctl_istio_~~.yaml** 속 밑의 두 kustomizeConfig을 지우고 설치를 하면 모든 것이 정상 설치되며, istio-ingressgateway를 통해 kubeflow gui 또한 접속이 가능해진다.

```
apiVersion: kfdef.apps.kubeflow.org/v1
kind: KfDef
metadata:
  namespace: kubeflow
spec:
  applications:

  - kustomizeConfig:
      parameters:
      - name: namespace
        value: istio-system
      repoRef:
        name: manifests
        path: istio/istio-crds
    name: istio-crds
  - kustomizeConfig:
      parameters:
      - name: namespace
        value: istio-system
      repoRef:
        name: manifests
        path: istio/istio-install
    name: istio-install

...
```

## References
- [Kubernetes](https://kubernetes.io/docs/home/)
- [Kubeflow](https://www.kubeflow.org/docs/)
- 쿠버네티스 설치
    - [지구별 여행자](https://www.kangwoo.kr/2020/02/17/pc%EC%97%90-kubeflow-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-1%EB%B6%80-nvidia-%EB%93%9C%EB%9D%BC%EC%9D%B4%EB%B2%84-docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0/)