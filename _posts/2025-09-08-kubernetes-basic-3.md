---
date: 2025-09-08 20:30:00 +0900
categories:
  - k8s
tags:
  - k8s
  - devops
author: gayoungoh
description: k8s study based on udemy class`(DevOps (데브옵스) Kubernetes 완전 정복)`
---
# kubernetes basic
## pod
### pod state
* pod status
  * running(실행중)
  * pending(보류중): accept인데 실행이 안되는 상태
    * 이미지가 다운로드 중인 경우
    * 리소스 제약이 걸리는 경우 (CPU, 메모리 이슈)
    * pvc가 필요한데 storage가 없어서 생성되지 않는 경우
  * succeeded(성공): 성공적으로 종료되어 재시작되지 않음
  * Failed(실패): 컨테이너가 종료되면서 exit code를 반환
    * 종료코드가 0인 경우 정상
  * Unknown: pod의 상태가 결정되지 않은 경우
    * 네트워크 오류 발생
```
# pod 조건 확인 가능
kubectl describe pod ${PODNAME}
# 결과값 중 Conditions에 나오는 조건은 pod가 통과한 조건!
```
* pod condition
  * PodScheduled: 노드에 스케쥴 됨 (성공)
  * Ready: pod가 요청을 받아서 매칭 서비스에 추가되는 경우
  * Initialized: 컨테이너 초기화
  * Unschedulable: 리소스 제한 등의 이유로 스케쥴 될 수 없는 경우
  * ContainersReady: 모든 컨테이너 준비 완료
* container state
  * containerStatuses 내에 containerID, image, imageID등이 있음
  * 해당 섹션 안에 상태값에 대한 정보 포함됨

### pod lifecycle
* <img width="842" height="420" alt="Image" src="https://github.com/user-attachments/assets/cec33807-0849-42f0-9eb0-b27b2440b6fc" />
* pre stop hook: pod가 중지될 때 실행하는 것

## secret
* secret data: credentials, keys, passwords 등 다양한 크리덴션 데이터를 제공
* 쿠버네티스에 네이티브, 시크릿을 전달하는 방법
  * 외부 vault 서비스를 이용하면 앱 계층에서 시크릿 전달 가능해짐
* 환경 변수로 활용
* 파드 내에서 파일로 활용
  * 컨테이너에 마운트된 볼륨에 활용
  * dotenv 파일 활용
* 공개 이미지에는 시크릿을 넣을 수 없지만 외부 이미지에 secret을 넣을 수 있음
* `kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt`
* `kubectl create secret generic ssl-certificate --from-file=ssh-privatekey=~/.ssh/id_rsa --ssl-cert=ssl-cert=mysslcert.crt`

```yaml
spec:
  containers:
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: SECRET_PASSWORD
      ...
```

## web UI
* 클러스터에서 실행중인 어플리케이션 개요 확인
* 개별 쿠버네티스 리소스 및 워크로드를 생성/수정 가능
* 일반적으로 `https://<kubernetes-master>/ui` 주소에서 확인 가능
* 수동으로 설치하는 경우 `kubectl create -f https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml`
* 비밀번호 확인이 필요한 경우 `kubectl config view`
