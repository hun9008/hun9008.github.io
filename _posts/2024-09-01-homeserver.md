---
layout: post
title: [홈서버] 개인 Ubuntu 서버 제작
author: Hun
tags:
- DevOps
date: 2024-09-01 09:18 +0800
last_modified_at: 2024-09-01 09:28:25 +0800
toc: true
---

<img src="/DevOps_architecture.png">

## Motivation

평소 AWS EC2나 Azure와 같은 클라우드 서비스를 자주 이용했지만, 요금이 부담되고 자유도가 떨어진다는 생각이 들어 방학때 개인 서버를 만들어봤습니다.

 현재 기숙사에 살고있어 서버 운용에 별다른 비용이 들지 않아, 바로 당근에서 10만원짜리 중고 윈도우 PC를 사서 서버 구축을 시작했습니다.

## Progress 

### Environmental Check

우선 기숙사 공유기 관리 페이지에 접속할 수 있는지 확인했습니다. 

다행히 192.168.0.1 (Private IP address)로 공유기 관리 페이지에 접속할 수 있어, 포트 포워딩이 가능하겠다는 판단이 되어 컴퓨터를 바로 구매했습니다.

구매한 PC는 i5, HDD 1TB, ssd 512GB, RAM 16GB로 AWS 프리티어 EC2에 비하면 훨씬 좋은 사양입니다.

### OS Setting

개인적으로 Linux 명령어가 편하기도 하고, 서버로만 쓸거라 Ubuntu 24.04로 교체하기로 했습니다.

Mac을 사용중이라, BalenaEtcher를 이용해 USB를 포멧하고 Ubuntu iso로 부팅용 USB를 생성했습니다. 

만든 USB로 Ubuntu를 설치하고 Window를 삭제했습니다. 여기서 USB부팅 순서때문에 한참 헤맸었는데, BIOS에서 부팅 순서를 바꿔주니 금방 해결됬습니다.

<img src="/DevOps_remote.png">

### Port Fowarding & UFW

외부에서 SSH로 접속해 서버를 조작하기 위해 공유기 관리 페이지에서 포트포워딩을 진행했습니다.

SSH를 위한 22번포트, http와 https를 위한 80, 443포트 그리고 DB를 위한 3306포트를 우선 포워딩 했습니다. 

포워딩은 단순히 외부 XX포트로의 접속을 설치한 PC의 내부IP의 XX포트로 포워딩 진행했습니다.

보안은 잘 몰라서 기본적인 UFW만 설치하고 사용할 포트만 open하는 방식으로 세팅했습니다.

### NGINX & DNS

다른 기능보다, 리버시 프록시를 위해 NGINX를 사용했습니다.

구축한 서버에는 여러 웹사이트와 백엔드 서버, 모델 서버와 DB까지 모두 서빙할 계획이라, 도메인 네임을 구매하고 특정 도메인으로 들어오면 내부 특정 포트로 포워딩하기 위해 NGINX를 이용했습니다.

Gabia에서 도메인을 하나 구매하고 A tag는 자유롭게 생성할 수 있기에 제가 구매한 도메인의 앞의 A 테그를 보고 NGINX를 이용해 특정 포트로 포워딩해줬습니다. 

> 예를 들어, mydomain.com을 구매했다면,
>
> homepage.mydomain.com으로 접속하면 3000번 포트의 Next.js 웹 페이지를 연결해주고,
>
> server.mydomain.com으로 접속하면 8000번 포트의 FastAPI서버를 연결해주는 식입니다.

nginx의 default conf file을 수정해 위와 같은 포워딩 설정을 완료했습니다. 

> 이후 여러 서비스를 서빙하면서 timeout, https 설정도 conf file을 수정해 해결했습니다.

추가로 nginx의 default index html을 수정해서 제 도메인으로 접속했을 때, 가이드 페이지를 간단히 제작해봤습니다. 

<img src="/DevOps_html.png">

## Review

이번 여름방학동안 총 3개의 서비스 서빙의 대부분을 이 서버에 서빙했었는데, 비용적으로 경험적으로 대만족합니다!

현재도 FastAPI backend 서버 2개 FastAPI AI model 서버 1개, 웹 페이지 2개를 서빙중인데 (물론 실제 많은 유저가 사용하지는 않지만) 오버헤드없이 잘 구동되고 있습니다.
