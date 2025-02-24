---
title: AWS Gitlab Install
date: 2025-02-24
categories: [aws, gitlab]
tags: [aws, devops, gitlab, cicd]     # TAG names should always be lowercase
author: gayoungoh
description: Gitlab page will be based on CICD pipeline
---

## ec2 setting for gitlab
- AWS 이미지는 **ubuntu 22.04**로 세팅하고, **gitlab_test**라는 이름으로 ec2 생성
<img width="927" alt="Image" src="https://github.com/user-attachments/assets/ea9667ab-a9b0-4f37-af8b-018c94d4b450" />
- **인스턴스 유형**: t3.large
  (개인적으로 t3.medium이면 gitlab 동작에 문제 없다고 생각하는데, 초기 세팅시 문제가 있어서 중간에 t3.large로 업그레이드 함)

- **키페어**: 기존 보유 키페어 or 신규 키페어 생성
<img width="930" alt="Image" src="https://github.com/user-attachments/assets/f27aff3a-cd50-42e8-8859-663650ff99e2" />

- (신규) 키페어 이름 쓰고 RSA, .pem 선택 후 키 페어 생성, 생성 버튼을 누르면 자동으로 **gitlat_test.pem** 파일이 저장됨

<img width="608" alt="Image" src="https://github.com/user-attachments/assets/452f73cf-93ae-4a01-aec5-097815b6e7fb" />

- 보안그룹 생성 or 기존 보안그룹 선택
  (초기 세팅에는 0.0.0.0/0 으로 적용하고 사용하면서 보안 인바운드 규칙을 설정하는 편이 편했음)
<img width="930" alt="Image" src="https://github.com/user-attachments/assets/892519a3-3320-44bd-b7a0-fa21674263a5" />

- 원활한 ec2 접속을 위해 사전에 인바운드 규칙을 설정
	* Type: HTTP, Protocol: TCP, Port Range: 80, Source: 0.0.0.0/0
	* Type: HTTPS, Protocol: TCP, Port Range: 443, Source: 0.0.0.0/0

- 스토리지 세팅 후 인스턴스 생성

## ec2 connection for gitlab install
- EC2 -> 인스턴스 id 선택, 연결 버튼을 누르고 EC2 인스턴스 연결, 하단의 연결 버튼 선택
- 윈도우에서는 복사, 붙여넣기가 안됐는데 맥에서는 복사 붙여넣기가 되서 굳이 Mobaxterm 없이 연결해도 괜찮았음

1. 초기 세팅을 위해 시스템 패키지를 업데이트 함
```bash
sudo apt update
sudo apt upgrade -y
```
2. 필요한 종속성 설치
```bash
sudo apt install -y curl openssh-server ca-certificates tzdata perl
```
3. gitlab 패키지 다운로드 & 설치
```bash
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
sudo apt install gitlab-ee
```
4. gitlab 설정
```bash
sudo gitlab-ctl reconfigure
```
5. 방화벽 설정
```bash
sudo ufw status # 방화벽 포트 확인
sudo ufw allow http
sudo ufw allow https
sudo ufw reload
```

- 검색했을 때는 gitlab 설정 후 웹브라우저에서 초기 비밀번호 세팅이 가능하다고 했는데, 내 경우에는 로그인 화면이 바로 떴다

* 오류 발생시 로그 파일을 확인하여 에러 메세지를 찾는다
```bash
sudo gitlab-ctl tail
sudo gitlab-ctl tail nginx # 특정 에러 메세지만 찾는 명령어
```

이렇게 하면 **gitlab 설치 완료!** 다음에는 gitlab.rb를 수정해서 설정을 맞추는 방법을 정리할 예정이다.
