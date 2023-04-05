---
title:  "AWS EC2 Amazon Linux 2023 Code Server 연동" 
excerpt: "AWS EC2에 Code Server를 연동해보자"

categories:
  - aws
tags:
  - [aws]

toc: false
toc_sticky: false
published: false
 
date: 2023-04-05
last_modified_at: 2023-04-05
---

# 1. 생성 동기

집이나 회사가 아니라면 노트북으로 이동식으로 개발은 하면 된다. 그런데 이번엔 태블릿을 하나 구했는데 개발을 할 순 없을까 고민을 하고 개발이 가능한 방법을 찾아 보았는데 앱을 통산 스크립트 언어(python)를 제외하고 컴파일이 필요한 언어는 전부 개발이 현재로썬 불가능했다. 그래서 예전 하던 방식대로 개인 PC를 서버로 해서 원격으로 접속해서 개발을 하면 되지 않을까해서 검색해보았다. code-server라는 것을 찾았다. code-server는 MS의 VS Code를 서버에서 동작시키고 VS CODE 클라이언트 또는 웹을 통해 접속해서 터미널또는 파일에 접근해서 수정이 가능하다.
즉 웹에서 파일을 생성하고 코드를 짜고 C, C++, Java 모두 컴파일이 가능하며 실행된다. 개인 PC에 구축해볼까도 했는데 AWS EC2를 사용하는 중이라 AWS에서 구축해보기로 하였다.

# 2. Code-server 설치 

구축환경 : AWS EC2 Amazonlinux 2023
<br>
가이드문서 : https://github.com/coder/code-server

구축 요구사항
-
![image](https://user-images.githubusercontent.com/13990392/229999855-d993fe9b-213a-4382-8d9f-3cb195fed467.png)

다행히 생성해둔 EC2는 vCPU2, ram 1G 로 만족했었다.

## 2-1 code-server install 스크립트 실행

```
curl -fsSL https://code-server.dev/install.sh | sh

Amazon Linux 2023
Installing v4.11.0 of the amd64 rpm package from GitHub.

mkdir -p ~/.cache/code-server
curl -#fL -o ~/.cache/code-server/code-server-4.11.0-amd64.rpm.incomplete -C - https://github.com/coder/code-server/releases/download/v4.11.0/code-server-4.11.0-amd64.rpm
######################################################################## 100.0%
mv ~/.cache/code-server/code-server-4.11.0-amd64.rpm.incomplete ~/.cache/code-server/code-server-4.11.0-amd64.rpm
sudo rpm -U ~/.cache/code-server/code-server-4.11.0-amd64.rpm
        package code-server-0:4.11.0-1.x86_64 is already installed
```
가이드 문서대로 실행을 했는데도 Time out error, Protocol error 등 에러가 전부 서버측 에러이기 때문에 계속 재시도하여 설치를 진행하였다.
인스톨 명령어를 실행하면 OS에 맞는 패키지파일의 url이 반환되어 자동으로 설치까지 이루어 진다.


## 2-2 code-server Service 실행
```
# code-server systemd으로 등록
sudo systemctl enable --now code-server@$ec2-user
# 시작 스크립트
sudo systemctl start code-server@ec2-user
# 종료 스크립트
sudo systemctl stop code-server@ec2-user
# 프로세스 동작 확인
ps -ef | grep code-server
ec2-user   41023   37036  0 00:00 ?        00:00:00 /usr/lib/code-server/lib/node /usr/lib/code-server
```
실행시 ec2-user 이외 다른 계정을 사용했다면 유저명을 변경한다.

서비스가 정상적으로 실행되었다면 실행중인 서비스가 확인된다.

## 2-3 code-server 환경 설정 확인

설치가 되고 동작이 되었으면 접속을 해서 확인을 해야한다. 
```
cat ~/.config/code-server/config.yaml 
bind-addr: 127.0.0.1:8080
auth: password
password: 패스워드
cert: false
```
설정을 확인하면 접속 패스워드를 설정해야한다. 앞으로 사용할 패스워드를 저장한다.
그리고 접속 정보가 127.0.0.1:8080의 localhost로 기본 설정되어 있다. 원격에서 접속을 해야 되는데 로컬 동작만 가능하다. [공식 가이드 문서](https://github.com/coder/code-server/blob/main/docs/guide.md#3-expose-code-server)를 확인해보자
![image](https://user-images.githubusercontent.com/13990392/230012928-9fc38cb2-f67c-4b5c-8778-e45c628860d3.png)

별도의 인증 절차가 없다면 localhost로만 동작하며 아래 4개의 방식으로 보안을 권장하며 가이드를 한다.
- Port forwarding via SSH : SSH를 통한 포트 포워딩
- Using Let's Encrypt with Caddy : Caddy를 사용한 암호화
- Using Let's Encrypt with NGINX : NGINX를 사용한 암호화
- Using a self-signed certificate : 자체 서명된 인증서 사용

이 중에 NIGNX를 사용해서 처리하고자 한다. 
![image](https://user-images.githubusercontent.com/13990392/230014091-0ae2556a-3d4f-484e-8283-fd6ebca908a3.png)

가이드 문서에 친절하게 NGINX 설치 및 설정이 설명되어 있지만 결론부터 말하자면 
RHEL(Rathat)계열서만 가능하고 안된다. 그럼 RHEL RELEASE를 설치해서 처리하면 될꺼 같아서 시도를 몇번이나 해보았지만 AWS에서 자체 차단한것인지 RHEL로 접속하는 어떤 저장소도 접속되지 않았고 설치도 불가능했다.

## 2-4 NGINX 설치

amazon linux [NGINX설치 문서](https://cs.nginx.com/static/instr/amazonlinux2023.html)를 참조해서 설치를 진행하였다.
```

sudo yum install ca-certificates

sudo wget -P /etc/yum.repos.d https://cs.nginx.com/static/files/plus-amazonlinux2023.repo

sudo yum install nginx

```

정상적으로 설치가 되었다면 code-server의 가이드 문서대로 환경설정을 진행한다.

```
# 2개의 폴더를 관리자 권한으로 생성한다.
# 환경설정 임시파일 폴더
sudo mkdir /etc/nginx/sites-available
# 환경설정 적용파일 폴더
sudo mkdir /etc/nginx/sites-enabled
```

sudo vi /etc/nginx/sites-available//code-server.conf
를 실행하여 가이드 문서의 내용을 그대로 입력한다.
```
server {
    listen 80;
    listen [::]:80;

    location / {
      proxy_pass http://localhost:8080/;
      proxy_set_header Host $host;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection upgrade;
      proxy_set_header Accept-Encoding gzip;
    }
}
```

```
sudo ln -s ../sites-available/code-server /etc/nginx/sites-enabled/code-server

#nginx 서비스 실행
sudo systemctl start nginx
``` 
설정을 완료하고 서비스를 실행후 AWS의 외부 접속 ip를 통해 접속하면 
![image](https://user-images.githubusercontent.com/13990392/230030731-8301ac51-7faa-486a-adfb-977866c189e2.png)

드디어 원격에서 code-server로 접근이 가능하다.
패스워드는 초기설정한 유저의 .config/code-server/config.yaml 에서 확인 가능하고 설정되지 않았다면 패스워드를 설정한다.

패스워드 입력 후 즐거운 원격 코딩을 시작하자

***
    개인 공부 기록용 블로그입니다. 오류나 틀린 부분이 있을 경우 
    댓글 또는 메일로 알려주시면 감사하겠습니다.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}



