---
title: "놀고 있는 AWS IOT Button을 슬랙 메세지 보내기"
date: 2019-06-30T22:20:13+09:00
draft: false
categories: [development, aws]
tags: [aws, iot, terraform, python]
---

# 놀고 있는 AWS IOT Button을 슬랙 메세지 보내기

### 서두

현재 소속되어 있는 팀의 팀장님께서 어느날 AWS IoT 버튼을 주셨습니다.

> 팀장님: 저 보단 영준님이 더 잘 사용할 수 있을 것 같아요.
> 나: 오.. 감사합니다. 재미있어 보이는데 한번 어떻게든 써볼게요.

라고 말했지만 실은 이 글을 작성하고 있는 이 시간에도 어떻게 활용할지 고민에 빠져 있습니다.
처음에는 AWS 자습서를 보다가 테라폼으로 만들어서 언젠간 쉽게 재활용할 날이 오겠지하고 이 글을 적게되었습니다.
처음 구성할 때 AWS IoT 버튼을 활용하기 위해서 여러가지 AWS Console에 들어가 설정을 해야하고 이


## 목표

- 특정 슬랙 채널에 메세지를 보내기 위한 Lambda 함수 정의
- Terraform 으로 IoT Button을 활용할 수 있는 환경 구성

## 필요사항
- terraform 0.11
- python3.6
- 집에서 놀고 있는 AWS IoT 버튼 1개
- AWS 계정 (ACCESS_KEY, SECRET_KEY)


## 설정해보기

```sh
git clone https://github.com/m0ai/IotButton2Slack.git
```
 terraform 과 lambda 를 쉽게 구성할 수 있게 설정된 저장소를 복사합니다.

해당 저장소는 IoT 버튼을 바로 활용하기 위한 Terraform 파일들과 Lambda를 바로 배포할 수 있도록 작은 스크립트를 포함하고 있습니다.

 우리의 목표는 IoT 버튼으로 슬랙에 알람을 보내는 것 입니다. 우
```sh
cd IotButton2Slack
make configure # lambda에서 사용할 requests 패키지를 설치합니다.
make publish   # lambda에 업로드할 zip을 생성합니다.

file lambda.zip
```
람다 함수를 배포하기 위해 필요한 패키지와 업로드할 .zip 파일을 생성해놓는 작업을 진행합니다. 위 스크립트를 실행하여 생성한 lambda.zip 은 단순하게 내가 원하는 슬랙 워크스페이스에 메세지를 보내는 간단한 일을 하고 있습니다. 만약 해당 람다 함수의 코드가 궁금하면 (lambda.py) 을 참고해주세요.
(이 글을 읽는 사람은 빠르게 IoT Button을 사용해보고 싶을테니 우리가 만든 람다에 대해서는 간단하게 설명하였습니다.)

람다 함수를 당장이라도 업로드하여 우리 손 안에 있는 IoT 버튼을 누르고 싶지만, 어떤 슬랙 채널에 메세지를 보낼지 어떤 메세지 내용을 보낼지 정의하지 않았습니다. 해당 자원은 테라폼에서 관리되고 있으므로 `terraform/locals.tf` 파일을 수정하여 우리가 원하는 동작을 하도록 수정해야 합니다.

```hcl
# cat terraform/locals.tf
locals {
  slack_incoming_url = "https://hooks.slack.com/~~~" # 메세지를 전송할 슬랙 채널의 incoming url
  single_click_msg = "한번 눌렀습니다." # 한번 눌렀을 때 슬랙으로 전송되는 메세지
  double_click_msg = "두번 눌렀습니다." # 두번 눌렀을 때 슬랙으로 전송되는 메세지
  long_click_msg = "기~일게 눌렀습니다." # 길게 눌렀을 때 슬랙으로 전송되는 메세지
}
```

위 내용을 참고하여 `terraform/locals.tf` 파일을 수정합니다. 간단하게 확인용으로 밋밋한 문자를 적어 놓았지만 원하시는 메세지로 수정하면 더 맛깔나게 버튼을 누를 수 있을겁니다.

```sh
cd terraform
terraform plan
terraform apply
```

위 작업까지 수정이 완료되었으면 내용을 내 AWS 계정에 IoT Button 을 사용하기 위해 테라폼으로 인프라 구성을 시작합니다. `lambda.tf`, `locals.tf`,


## 마지막 작업
### IoT 버튼과 인프라 연결하기
aws lambda 홈페이지에 접속하여 IotButton2SlackNotifier 람다 함수를 클릭하여 Configuration 매뉴에 접속합니다.

좌측 Designer/트리거 추가/AWS IoT 매뉴를 눌러 트리거 구성 섹션에서 필요한 항목(Device Serial Number)을 채워 인증서 및 키 생성 버튼을 누릅니다 .

버튼을 누른 후 AWS에서 IoT 버튼에서 사용할 키를 구성하라고 안내를 해줍니다. 이를 완료한 후 추가를 누릅니다.

완료되었습니다. 이제 버튼을 눌러 버튼을 사용해보아요.

결과
image

슬랙봇의 이름은 .. 노코멘트

Inspired by
우아한 형제들 - 차임벨
당근마켓 - 화분 물주기 알림이