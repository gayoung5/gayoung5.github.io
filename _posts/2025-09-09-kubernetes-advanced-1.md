---
date: 2025-09-09 08:30:00 +0900
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
## DNS
* addon 관리자를 사용해서 자동으로 시작되는 기본 제공 서비스
* 마스터 노드의 `/etc/kubernetes/addon` 폴더 내에 내장됨
* 같은 클러스터 내에 실행되는 다른 서비스를 찾기 위해서 파드 내 DNS를 활용함
* 같은 pod 내 서비스는 포트만 알면되니까 DNS를 쓸 필요가 없음
* DNS를 사용하려면 서비스 정의가 필수

## configmap
* key-value 쌍으로 이루이진 형태
* 어플리케이션이 구성 파일로 예상하는 볼륨을 사용해서 이 파일을 마운트할 수 있음
* 이미지는 변경하지 않고 설정값만 변경 가능
```
cat <<EOF > app.properties
driver=jdbc
database=postgres
lookandfeel=1
otherparams=xyz
param.with.hierachy=xyz
EOF

kubectl create configmap app-config --from-file=app.properties
```

* 컨피그맵을 사용하면 구성 파일을 삽입해서 어플리케이션 내에서 구성 데이터를 더할 수 있음

## ingress
* 외부 로드밸런서와 노드포트의 대안
  * 외부에서 클러스터로 액세스해야 하는 서비스를 쉽게 노출할 수 있음
* 쿠버네티스 클러스터 안에 고유 인그레스 컨트롤러 실행 가능
* <img width="1117" height="525" alt="Image" src="https://github.com/user-attachments/assets/e8651c54-335e-4943-bad3-4a67a7bdd01a" />
* HTTP 기반의 트래픽들을 라우팅할 수 있도록 구성, **HTTP(s)-based**

### external DNS
* route53 서비스처럼 DNS서버에서 만들 수 있음

## volumes
* 컨테이너 외부에 데이터를 저장하기 위해서 쿠버네티스 볼륨을 활용
* stateless 상태의 앱을 활용하면 컨테이너 중지 시 모든 데이터가 사라짐
* AWS 클라우드에서는 EBS를 활용, 아예 노드 외부에 있는 스토리지를 활용 가능 -> 클라우드를 활용하는 가장 좋은 점

```
// AWS 볼륨 생성
aws ec2 create-volume --size 10 --region eu-west-1 --availability-zone eu-west-1a --volume-type gp2 --tag-specifications 'ResourceType=volume, Tags=[{Key=KubernetesCluster, Value=<kubernetes_name>}]'

kubernetes get node
kubernetes create -f <<volumes_test.yaml>>
```

### volumes provisioning
* storageclass 객체를 사용해서 볼륨 프로비저닝을 할 수 있음
``` yaml
kind: StarageClass
apiVersion: storage.k8s.io/v1beta1
metadata:
  name: standard
# aws-ebs 프로비저너 사용 가능
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  zone: us-east-1
```
* PVC(persistentVolumeClaim)에서 볼륨 크기(size)도 설정 가능
