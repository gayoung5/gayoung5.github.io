---
date: 2025-09-03 00:00:00 +0900
categories:
  - k8s
tags:
  - k8s
  - devops
author: gayoungoh
description: k8s study based on udemy class`(DevOps (데브옵스) Kubernetes 완전 정복)`
---
# kubernetes basic
## services
* 새 서비스를 생성하면 pod의 endpoint가 생성됨
### service 종류
* **ClusterIP**: 클러스터 내부에서만 연결 가능한 가상 IP
* **NodePort**: 외부에서도 연결 가능한 포트, 노드의 IP를 사용함, 외부에서 파드에 접속하기 위해 필요
* **LoadBalancer**: 외부 트래픽을 노드포트의 모든 노드로 라우팅하는 클라우드 공급자에 의해 배포됨 (ELB on AWS)
  * 이런 옵션들은 virtual IP나 포트들이 있어야 함
  * DNS 활용 가능 (DNS add-on)

### service 상세
* `spec.ports.nodePort, spec.ports.port`는 설정 안하면 자동으로 배정됨
* ```spec.ports.targetPort```를 nodejs-port로 설정한 경우에 pod도 ```spec.containers.ports.name```에 같은 내용을 넣어줘야함
* 설정하는 경우에는 충돌 방지를 위해 기존 서비스들 확인이 필요
* 서비스 포트는 기본적으로 30000-32767 사이에서 사용함
* 예외로 ```--service-node-port-range```인수를 kube-apiserver 끝에 넣어서 동작 변경도 가능함

## Labels
* key/value 쌍으로 구성되며 AWS의 tag와 비슷한 역할을 함\
    ex) key: environment - value: dev / staging / qa / prod\
        key: department - value: engineering / finance / marketing
* 라벨은 unique하지 않고 여러개의 라벨을 붙일 수도 있음

### Label selectors
* 라벨이 붙으면, 해당 라벨 기준으로 필터링을 할 수 있음
* 정규표현식을 통해서 특정 노드에서는 특정 라벨이 있는 파드만 실행할 수 있도록 함

### Node Label
* 라벨은 다양한 오브젝트 외에도 노드에도 사용 가능
* 노드에 라벨 태그 -> `pod configuration`에 `nodeSelector` 추가
```
# 노드에 라벨 태그
kubectl label nodes node1 harware=high-spec
kubectl label nodes node2 hardware=low-spec
# pod에 nodeSelector 추가
--- 중략
spec.nodeSelector:
  hardware: high-spec
```

## health Checks
* pod 내 문제가 발생했을 때 문제를 발견하고 해결하기위해 헬스체크를 진행
* 두가지 타입의 헬스체크
  * 컨테이너 내에서 주기적으로 명령어를 실행
  * HTTP를 사용해서 URL을 주기적으로 확인
### livenessprobe
* 컨테이너가 실행중인지 여부를 판단
* fail인 경우, 컨테이너가 재시작됨
```yaml
# ---중략
spec:
  livenessProbe:
    httpGet:
      path: /
      port: 3000
    initialDelaySeconds: 15 # 초 단위의 초기 지연 발생
    timeoutSeconds: 30 # 타임아웃
```
### readinessprobe
* 컨테이너가 요청을 처리할 수 있는지를 판단
* 재시작되지는 않지만, pod의 IP 주소가 서비스로부터 제거됨
* 이 테스트가 성공해야 pod가 트래픽을 수신할 수 있음
* 일반적으로는 livenessProbe, readinessProbe 둘 다 구성함
