---
title: kubernetes advanced 3
date: 2025-09-26 18:30:00 +0900
categories: [k8s]
tags: [k8s, devops]
author: gayoungoh
description: k8s study based on udemy class`(DevOps (데브옵스) Kubernetes 완전 정복)`
---
# kubernetes advanced
## affinity & anti-affinity
* 특정 노드에 파드가 스케줄링 되도록 하는 방법
* `anti-affinity`는 `nodeselector`랑 비슷한 역할을 함
* 쿠버네티스에서는 node affinity, pod affinity/anti-affinity를 설정할 수 있음
* 규칙을 재적용 하고 싶으면 규칙 생성 후 재시작 해야함
  * node affinity: nodeSelector
    * requiredDuringSchedulingIgnoredDuringExecution: 무조건 만족해야 함
    * preferredDuringSchedulingIgnoredDuringExecution: 시도는 하지만 보장이 안됨, 규칙이 만족되지 않아도 스케쥴링 가능
    * 두가지 조건 모두 사용 가능
    * 가중치를 두는 경우 만족시 점수를 더하는 형태로 노드의 점수 기반으로 적합성 검토
  * pod affinity: 어떻게 파드가 실행중인 다른 파드를 고려해서 스케쥴 될 수 있는지 규칙을 생성
    * 파드 스케쥴링에만 적용되는 규칙
    * requiredDuringSchedulingIgnoredDuringExecution
    * preferredDuringSchedulingIgnoredDuringExecution
    * 다른 파드랑 같은 노드에 위치하는 규칙
    * 가용영역(AZ) 기반으로도 설정 가능
  * pod anti-affinity: 파드가 노드에서 한 번만 스케줄링 되도록 함
    * pod affinity와 반대로 제외 조건을 설정

## taints & tolerations
* 둘 다 파드의 스케줄링에 영향을 줌
* taints: 노드에 마킹
  * pod 생성 시 마스터에 스케줄링 되지 않도록 함
  * 마스터에 설정 - `node-role.kubernetes.io/master:NoSchecule`
  ```
  kubectl taint nodes node1 key=value:NoSchedule
  # node1에 pod가 스케쥴되지 않음
  # 일치하는 toleration이 있는 경우 스케쥴 가능
  ```
  * taint key 삭제할 때 `kubectl taint nodes <node명> <key>-`
* tolerations: 노드가 파드 세트에 접근하지 못하게 함
  * operator가 equal인 경우 key, value를 제공하고, exists인 경우 키의 존재 여부만 확인
  * tolerationSeconds 를 지정하는 경우에 pod를 해당 초만큼만 유지함
* 다양한 설정 예시
```
node.kubernetes.io/not-ready # node가 준비되지 않음
node.kubernetes.io/unreachable # 노드가 노드 컨트롤러에 접근할 수 없음
node.kubernetes.io/out-of-disk #디스크 용량 부족
node.kubernetes.io/memory-pressure # 메모리 할당 압박
node.kubernetes.io/disk-pressure # disk 할당 압박
node.kubernetes.io/network-unavailable # 네트워크 활용 불가
node.kubernetes.io/unschedulable # 노드에 스케쥴 불가
```
## custom resource
* yaml 파일 내에 정의됨
* 관리자 - CRDs(Custom Resource Definitions), 쿠버네티스 API 확장

### Operators (연산자)
* statefulset을 관리할 수 있음
* prometheus, vault, rook, mysql, postgresql 등에 연산자가 있음
* 클라우드에서 좀 더 서비스처럼 활용 가능
