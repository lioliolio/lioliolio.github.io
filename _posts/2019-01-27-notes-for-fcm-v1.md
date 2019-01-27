---
layout: post
title: "FCM HTTP v1 주의점"
date: 2019-01-27
---
[FCM(Firebase Cloud Messaging)](https://firebase.google.com/docs/cloud-messaging/)를 사용하면 안드로이드, ios 앱 및 모바일 웹에 푸쉬 메시지를 전송할 수 있습니다. 

구글에서 제공하는 FCM 서버 프로토콜 중 [FCM legacy HTTP API](https://firebase.google.com/docs/cloud-messaging/http-server-ref)에서 [FCM HTTP v1 API](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages)으로 변경하면서 있었던 문제들을 공유하고자 합니다.


### FCM legacy HTTP API와 FCM HTTP v1 API

[이전 HTTP에서 HTTP v1로 이전](https://firebase.google.com/docs/cloud-messaging/migrate-v1) 문서에서
FCM HTTP v1 API는 FCM legacy HTTP API에 비해 다음과 같은 장점이 있다고 설명합니다.

- 액세스 토큰을 통한 보안 향상
- 여러 플랫폼에서 보다 효율적인 메시지 맞춤설정
- 새 클라이언트 플랫폼 버전을 위한 확장성 강화 및 미래 경쟁력 확보

하지만 기기 그룹 메시징, 멀티캐스트 메시징을 사용하는 경우, 해당 기능은 FCM HTTP v1 API에서는 지원하지 않기 때문에 FCM legacy HTTP API를 사용해야 합니다.

### FCM HTTP v1 API로 변경하면서 겪었던 문제들

기존 앱에서 기기 그룹 메시징, 멀티캐스트 메시징 기능은 사용하고 있지 않았기 때문에 FCM HTTP v1 API로 변경해도 큰 문제는 없을거라 생각했습니다. 모든 일이 그렇지만 예상대로만 흘러 가진 않았습니다...ㅠㅠ

##### FCM legacy HTTP API와 다르게 FCM HTTP v1 API 는 콘솔에서 API를 enable 해야합니다.

[API Dashboard - Google Cloud Console](https://console.cloud.google.com/apis/dashboard)에서 enable 해야 합니다.

##### FCM legacy HTTP API의 경우 토큰이 리프레시 되면 리프레시 된 토큰 값을 알 수 있지만, FCM HTTP v1 API는 리프레시 된 토큰 값을 알 수 없습니다.

```
# FCM legacy HTTP API Response example
{
    "multicast_id": 57677857360,
    "success": 1,
    "failure": 0,
    "canonical_ids": 1,
    "results": [
        {
            "registration_id": "NEW_REGISTRATION_ID",
            "message_id": "0:1539658517630439"
        }
    ]
}
```

토큰이 리프레시 되는 경우가 종종 있습니다. FCM legacy HTTP API의 경우 위의 예시 응답처럼 `registration_id`에 새로운 토큰 값을 보내줍니다. 하지만 FCM HTTP v1 API의 경우 error 관련 메시지만 보내줄 뿐 변경된 토큰 값을 알 수 없습니다. 

따라서, 변경된 토큰 값에 대한 문제는 FCM HTTP v1 API 만으로는 처리할 수 없습니다. 기존 FCM legacy HTTP API를 사용하던지, 다른 방법을 찾아야 합니다.

##### FCM HTTP API v1 문서 내용이 너무 빈약하여 정확한 응답 메시지 형식을 알기 힘들었습니다.

문서만 봐서는 어떤 형식으로 응답 메시지를 주는지 알기가 쉽지 않았습니다. 직접 요청 메시지를 보내서 응답 메시지를 확인하거나, 다른 구글 라이브러리 코드를 확인하는 방식으로 응답 메시지를 확인했습니다.

```
# details가 없는 error 응답 메시지
{
    "error": {
        "code": 403,
        "message": "The caller does not have permission",
        "status": "PERMISSION_DENIED"
    }
}
```

```
# details가 있는 error 응답 메시지
{
    "error": {
        "code": 400,
        "message": "Request contains an invalid argument.",
        "status": "INVALID_ARGUMENT",
        "details": [
            {
                "@type": "type.googleapis.com/google.firebase.fcm.v1.FcmError",
                "errorCode": "INVALID_ARGUMENT",
            },
            {
                "@type": "type.googleapis.com/google.rpc.BadRequest",
                "fieldViolations": [
                    {
                        "field": "message.token",
                        "description": "The registration token is not a valid FCM registration token",
                    }
                ]
            }
        ]
    }
}
```

error 응답 메시지가 위와 같이 `details`가 빠져서 올 수 있다는 것을 사전에 인지하지 못하는 경우도 있었습니다.

문서를 보고 이해가 안되면 바로 구글 검색이나 Firebase Admin Python SDK 및 다른 구글 라이브러리 구현 코드를 확인하는 게 시간을 아낄 수 있을거라 생각합니다.

### 참고
- [FCM(Firebase Cloud Messaging)](https://firebase.google.com/docs/cloud-messaging/)
- [FCM legacy HTTP API](https://firebase.google.com/docs/cloud-messaging/http-server-ref)
- [FCM HTTP v1 API](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages)
- [이전 HTTP에서 HTTP v1로 이전](https://firebase.google.com/docs/cloud-messaging/migrate-v1)