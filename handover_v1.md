# 핸드오버 API (BETA)

하나의 계정에서 챗봇과 톡톡 파트너센터 상담원이 함께 협업하여 응대할 수 있는 기능을 제공합니다.
챗봇이 사용자의 메시지에 응답하지 못하는 경우, 핸드오버 API를 이용하여 실제 상담원에게 연결하여 보다 나은 사용자 경험을 제공할 수 있습니다. 

## 핸드오버 기능 활성화 

핸드오버 기능을 활성화하려면 [톡톡 파트너센터](https://partner.talk.naver.com/)의 챗봇 API 설정에서 핸드오버 API를 ON으로 설정해주세요.
![composite_message](/images/handover-switch.png)

## 파트너 센터와의 연동을 통해 챗봇과 실시간으로 응대하기

 파트너 센터에 연동된 챗봇과 파트너가 응대하는 것을 좀 더 인터랙티브(interactive)하게 사용할 수 있습니다. 기본적으로 챗봇이 응대를 하고 있다가, 실제 파트너의 응대가 필요한 경우에 `passThread`를 파트너에게 합니다.

 `핸드오버`를 켠 상태에서 챗봇이 응대하고 있는 대화는 "완료된 상담" 목록에 표시됩니다. 그러나 `passThread`를 파트너에게 하게 되면, "진행중 상담" 목록에 표시됩니다. 따라서 파트너는 파트너에게 전달된 상담을 바로 응대할 수 있게 됩니다. 이 때 챗봇은 유저가 보내는 메시지에 `standby`속성이 true로 가므로 응답하지 않도록 구현해야 합니다. 

 파트너가 응대를 하다가 "상담 완료하기" 버튼을 누르면 파트너로부터 챗봇에게 `passThread`하게 됩니다. 그럼 대화는 다시 "완료된 상담" 목록에 표시됩니다. 또한 챗봇은 유저가 보내는 메시지에 `standby` 속성이 사라지게 되므로 응답을 하도록 구현해야 합니다.

## echo 이벤트 활용

 대화의 주도권이 파트너에게 있더라도, 챗봇이 특정 시점에 대화의 주도권을 빼앗을 수 있습니다. 이 땐 `takeThread`를 하면 됩니다. 주도권이 파트너에게 있을 땐 standby 속성을 true로 받다가 `takeThread`를 통해서 대화의 주도권을 가져와 챗봇의 시나리오로 진행할 수 있습니다. 

```json
// 파트너가 대화의 주도권을 가질 땐 threadOwnerId가 파트너를 의미하는 1이며, 메시지의 주체인 sourceId도 1입니다.
{
  "echoedEvent": "send",
  "event": "echo",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc8b1i",
  "textContent": {
    "text": "하이",
    "inputType": "typing"
  },
  "options": {
    "mobile": false,
    "sourceId": 1,
    "threadOwnerId": 1,
    "managerNickname": "구매자"
  }
}

// 파트너가 "대화 종료"를 통해 챗봇에 전달되는 핸드오버 데이터
{
  "event": "handover",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc8b1i",
  "options": {
    "control": "passThread",
    "metadata": "{'managerNickname':'구매자','autoEnd':false}"
  }
}

// 대화의 주도권이 챗봇에게 넘어간 뒤 파트너가 대화를 입력했을 때 threadOwnerId는 챗봇의 시퀀스이며, 메시지의 주체인 sourceId는 파트너인 1입니다.
{
  "echoedEvent": "send",
  "event": "echo",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc8b1i",
  "textContent": {
    "text": "하이",
    "inputType": "typing"
  },
  "options": {
    "mobile": false,
    "sourceId": 1,
    "threadOwnerId": 10007,
    "managerNickname": "구매자"
  }
}
```


## 챗봇의 메시지를 수신할 때, standby 속성

 `standby`는 유저가 보낸 메시지에 대한 속성입니다. 대화의 주도권에 따라 true 혹은 생략이 됩니다. 그러나 챗봇이 보낸 메시지를 사용자가 클릭했을 땐, 대화의 주도권이 파트너에게 있더라도 `standby`를 생략하여 보냅니다. 따라서 대화의 주도권과 관계없이 챗봇이 응대해야하는 시나리오를 진행할 수 있습니다. 예를 들어 시나리오에 `상담원과 대화 종료하기`라는 의도의 버튼을 추가하여, 유저가 원하는 시점에 상담원과의 대화를 종료하도록 만들 수도 있습니다. 이 땐 유저가 해당 버튼을 클릭시 `takeThread` 하도록 기능을 구현하면 됩니다.

 챗봇이 파트너에게 주도권을 전달할 땐, 타겟(targetId)을 `1`로 합니다. 아래와 같이 보냅니다.

```json
{
  "event": "handover",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc8b1i",
  "options": {
    "control": "passThread",
    "targetId": 1
  }
}
```

 대화의 주도권이 파트너에게 있기 때문에 챗봇이 수신하는 유저의 메시지에는 아래와 같이 `standby` 속성에 true가 들어갑니다.
 ```json
{
  "standby": true,
  "event": "send",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc8b1i",
  "textContent": {
    "text": "헬로",
    "inputType": "typing"
  },
  "options": {
    "mobile": false
  }
}
```

 아래는 파트너가 "상담 완료하기" 버튼을 눌렀을 경우, 챗봇에게 전송되는 메시지입니다.

```json
{
  "event": "handover",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc1234",
  "options": {
    "control": "passThread",
    "metadata": "{\"managerNickname\":\"파트너닉네임\",\"autoEnd\":false}"
  }
}
```
 
 "상담 완료하기" 버튼을 누르지 않고 대화가 "자동 완료" 되는 시점에는 다음과 같이 전송됩니다.
```json
{
  "event": "handover",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc1234",
  "options": {
    "control": "passThread",
    "metadata": "{\"autoEnd\":true}"
  }
}
```

## 챗봇이 대화의 주도권을 가져오기
 챗봇이 원하는 시점에 파트너의 대화 주도권을 빼앗을 땐, 아래와 같이 보냅니다.
```json
{
    "event": "handover",
    "user": "al-2eGuGr5WQOnco1_V-FQ",
    "partner": "wc1234",
    "options": {
        "control": "takeThread",
        "metadata": ""
    }
}
```
