---
date: 2025-09-17 18:30:00 +0900
categories:
  - k8s
tags:
  - k8s
  - devops
author: gayoungoh
description: k8s study based on udemy class`(DevOps (데브옵스) Kubernetes 완전 정복)`
render_with_liquid: false
---
# kubernetes advanced
## pod presets
* 런타임 중에 파드에 정보를 넣어줄 수 있음 (시크릿, 컨피그맵, 볼륨, 환경변수 등)
```yaml
apiVersion: settings.k8s.io/v1alpha1
kind: PodPreset
spec:
  selector:
    matchLabels:
      app: myapp # matchlabel이 맞으면 아래 내용이 다 적용됨
  env:
    - name: MY_SECRET
      value: "123456"
  volumeMounts:
    - mountPath: /share
      name: share-volume
  volumes:
    - name: share-volume
      emptyDir: {}
```
* 충돌이 생기면 pod에 적용되지 않음
* 0개 이상의 pod에 적용할 수 있음
  * 현재 적용되는 pod가 없더라도 이후에 적용되는 pod가 생길 수도 있음

## stetefulset
* 기존(1.3)에는 Pet Set이라는 이름이었는데 k8s 1.9 버전부터는 **statefulset**으로 명명됨
* stateful application을 실행할 수 있음
  * 원래 `podname-randomstring`으로 구성된 이름을 갖는데, index를 활용하면 podname-0, podname-1 이런식으로 고정된 값을 가질 수도 있음
  * 연결된 스토리지가 안정적이라 pod가 재시작 되어도 데이터가 보존됨
  * 삭제/생성(스케일링)되어도 데이터가 유지됨
* DNS를 활용해서 다른 peer를 찾을 수 있음, 태그가 고정이라 찾기 쉬움

## daenon sets
* 노드가 클러스터에 추가될 때 새로운 파드가 자동으로 시작됨
* 노드가 제거되도 다른 노드로 스케줄링되지 않음
* ex) Logging aggregators, Monitoring, Load Balancers, Reverse Proxies, API gateways
* 노드마다 필요한 내용을 자세히 작성하면 됨

### 리소스 사용 모니터링
* Heapster: 컨테이너 클러스터 모니터링, 퍼포먼스 분석
  * REST endpoints를 통해서 클러스터 메트릭을 보냄
* `pod auto-scaling`을 하기 위해서 사전에 필요한 부분
* influxdb를 활용해서 힙스터 yaml 파일을 받음
* 모든 노드의 kubelet에서 heapster pod로 metric을 전달하는 형태임
* **1.8 이후부터는 힙스터를 사용하지 않음!!**
  * k8s 1.13 버전부터는 아예 제거됨
* **metrics-server**를 활용
  * `kubectl top node`, `kubectl top pod` 명령어를 통해 CPU, 메모리 사용량을 알 수 있음

## autoscaling
* k8s는 pod, deploy, replication controller, replicaset을 자동으로 확장할 수 있음
* 1.3 버전 이후부터는 CPU 사용량에 따라 확장이 가능
* 타겟 파드의 사용량을 주기적으로 확인함 (query)
* CPU 사용량이 일정 수준 이상인 경우에 자동으로 maxReplicas까지 확장함 (horizontal scaling)
