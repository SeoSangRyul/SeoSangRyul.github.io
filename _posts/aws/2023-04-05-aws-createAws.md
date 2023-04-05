---
title:  "AWS EC2 Amazon Linux 2023 프리웨어 생성" 
excerpt: "AWS EC2 프리웨어 생성을 해보았다."

categories:
  - aws
tags:
  - [aws]

toc: false
toc_sticky: false
 
date: 2023-04-05
last_modified_at: 2023-04-05
---


<br>

# 1. AWS 계정을 생성한다.

AWS에 접속 하여 계을 생성한다.

AWS : `aws.amazon.com`
![image](https://user-images.githubusercontent.com/13990392/229956998-2e2c221f-ef5f-49b7-b401-ce13b0e687aa.png)

컨솔 로그인 클릭

![image](https://user-images.githubusercontent.com/13990392/229957182-ad449004-d3c2-4ab0-90d9-4f682ba69824.png)

AWS계정 새로 만들기 클릭

![image](https://user-images.githubusercontent.com/13990392/229957342-6ea36239-3cbb-4fc2-96eb-74c81861faa1.png)

이메일 주소 확인을 클릭하고 메일 인증까지 완료한다.


# 2. AWS EC2 생성한다.

아래의 주소를 통해 프리티어 라이센스를 확인한다.

AWS프리티어 : `aws.amazon.com/ko/free'

- 프리티어는 주기적으로 제품이나 조건이 변하니 확인하고 만들어야함
<br>

![image](https://user-images.githubusercontent.com/13990392/229958755-c4442ead-b62c-4850-a3c1-09ebadabd4f5.png)

프리 티어 유형
- 12개월 무료 : Amazon EC2처럼 계정 생성 후 첫 Ec2에 한해 12개월을 무료로 사용가능
- 언제나 무료 : 정책이 변경되기 전까지 비용발생이 없음
- 평가판 : 서비스에 따라 용량, 기간 등이 정해진 만큼만 무료로 사용가능

라이센스를 확인했다면 Amazon EC2를 클릭한다.

![image](https://user-images.githubusercontent.com/13990392/229959652-f4e1210a-e014-40c4-b41f-155dd127c578.png)

Amazon EC2 시작하기를 클릭한다.

![image](https://user-images.githubusercontent.com/13990392/229959826-1fa86692-85ca-4291-8652-c6be8536e4d6.png) 
![image](https://user-images.githubusercontent.com/13990392/229959931-6ae02934-f14a-4bdb-8dff-73b9547b4904.png)

컨솔화면화면에서 좌측메뉴의 EC2대시보드를 클릭하고, 본문의 인스턴스 시작을 클릭한다.

![image](https://user-images.githubusercontent.com/13990392/229960582-360060e2-22e8-4ce0-bd6d-3476594b8c63.png)
EC2에 사용한 이름을 지정한다.

![image](https://user-images.githubusercontent.com/13990392/229960773-4e33ed4e-e8c3-4048-ac8a-13498efb527b.png)
사용한 OS를 지정한다.
- 프리티어 사용가능을 반드시 확인한다.
- Ubuntu나 RedHat의 경우 자료가 많아서 장애처리 및 각종 설치가 편리하지만 추가 요금이 발생한다.
- 글 작성 시점기준 Amazon linux 2023이 반영되어서 그것을 실습하기로 한다.

![image](https://user-images.githubusercontent.com/13990392/229961304-7a61b45f-98e4-4375-9f83-ce7af5d65e5a.png)
![image](https://user-images.githubusercontent.com/13990392/229961695-be917010-4658-42e2-b14b-80d21c8df48f.png)
인스턴스 유형을 지정한다.
 - 프리 티어 사용 가능을 확인한다.
 - 인스턴스 유형에 따라 성능과 비용이 달라진다.
 - 인스텀스 유형에 따라 지원하는 OS와 금액이 달라진다. (프리티어 1년 사용은 문제 없음)

    ✨ 24시간 30일 사용 월 720시간 기준 
    - RHEL(RedHat) : 0.0708 * 720 = 50.976$ (약 6만5천원)
    - Azmzon Linux : 0.0108 * 720 = 7.776$ (약 1만)


![image](https://user-images.githubusercontent.com/13990392/229963120-dad5179c-1189-4afa-b092-8818d9661ae5.png)
키페어는 개발이나 운영을 위해 SSH접속을 위해 로컬PC 저장된다.
해킹위험에서 보호하기 위해서 키페어 생성을 권장하며 발급시 관리에 유의해야한다.

![image](https://user-images.githubusercontent.com/13990392/229963543-77c569a9-0d83-45b1-9940-392429b60d4a.png)
네트워크 설정은 기본 값을 설정해도 되지만 AWS해킹 위험에서 보호 하고자 SSH트래픽을 자주사용하는 PC를 고정IP로 사용하는 것이 안전하다.


![image](https://user-images.githubusercontent.com/13990392/229964162-803607a1-4c80-470c-ae9e-fd58478b3738.png)
스토리지 용량을 구성한다.
- 1년 프리티어 사용시 GP3타입의 30GB까지 무료
- 용량에 비례하여 프리티어 종료와 동시에 요금이 발생하니 주의 필요


![image](https://user-images.githubusercontent.com/13990392/229964454-7ccb43a1-61f6-4e9e-9f60-02494f7c4361.png)
설정이 완료되었다면 우측의 인스턴스 시작을 클릭한다.

# 3. AWS EC2 인스턴스를 실행한다.

관리자 컨솔로 접속하여 좌측의 인스턴스를 클릭한다.

![image](https://user-images.githubusercontent.com/13990392/229986261-106b181c-123e-4703-9978-148603e530e7.png)

인스턴스가 실행중인지 확인한다.

![image](https://user-images.githubusercontent.com/13990392/229986540-eca7a53b-7682-4e2e-9e42-f381fe9bc9bc.png)

실행중이 아니라면 인스턴스 시작을 클릭 후 인스턴스 상태를 확인한다.


# 4. AWS EC2 인스턴스에 SSH 접속한다.

![image](https://user-images.githubusercontent.com/13990392/229986685-68ab22f7-bd50-4919-8459-c23cedfc1bd3.png)

인스턴스를 선택 후 연결을 클릭한다.

![image](https://user-images.githubusercontent.com/13990392/229987008-accda453-efa4-4d5d-a7b0-2d8c994838ea.png)

퍼블릭 IP 주소를 통해 외부에서 접속이 가능하며 연결을 클릭하여 접속한다.
- 페어키를 설정하였다면 저장된 PC에서만 접속이 가능하다.
- 설정 단계에서 IP를 지정하였다면 해당 IP에서만 접속이 가능하다.

![image](https://user-images.githubusercontent.com/13990392/229987411-37c0f3b9-508d-4ee2-97b3-c3562cece70b.png)

접속 후 터미널 화면이 출력 된다면 완료

***
    개인 공부 기록용 블로그입니다. 오류나 틀린 부분이 있을 경우 
    댓글 또는 메일로 알려주시면 감사하겠습니다.

[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}




