---
layout: post
title: "Bitbucket Pipelines 사용시 주의점"
date: 2018-12-23
---
[Bitbucket Pipelines](https://ko.atlassian.com/software/bitbucket/features/pipelines) 을 사용하면서 경험했던 문제를 공유하고자 한다.

### Bitbucket Pipelines을 사용하기 
[Bitbucket Pipelines](https://ko.atlassian.com/software/bitbucket/features/pipelines) 은 빌드, 테스트 및 배포 자동화를 위한 툴이다.

파이썬 프로젝트를 Bitbucket Pipelines을 사용하여 배포하기 위해 `bitbucket-pipelines.yml` 파일을 다음과 같은 형식으로 만든다.

```
# 예시
# bitbucket-pipelines.yml

image: python:3.6.1

pipelines:
  default:
    - step:
        script:
          - apt-get update && apt-get install -y python-dev
          - python3 -m venv venv
          - source venv/bin/activate
          - pip install -r requirements.txt
```

### 비공개 파이썬 저장소 사용하기

회사 내부에서 사용하기 위해 AWS S3를 사용하여 비공개 파이썬 패키지 저장소를 만들었다. 그리고 `requirements.txt` 파일 안에 비공개 파이썬 저장소의 패키지를 설치하도록 다음 내용을 추가했다.

```
# requirements.txt

--extra-index-url http://pypi.test-python.com.s3-website.us-east-1.amazonaws.com
--trusted-host pypi.test-python.com.s3-website.us-east-1.amazonaws.com
test-python-package
```

[--extra-index-url](https://pip.pypa.io/en/stable/reference/pip_wheel/#extra-index-url) 옵션은 파이썬 패키지 인덱스 URL을 추가하는 옵션이다. 기본으로 설정되어 있는 `https://pypi.org/simple` 인덱스 URL에 `http://pypi.test-python.com.s3-website.us-east-1.amazonaws.com` 인덱스 URL을 추가한다.

[--trusted-host](https://pip.pypa.io/en/stable/reference/pip/#cmdoption-trusted-host) 옵션을 사용하면 `https`가 아니더라도 해당 URL을 사용할 수 있게 한다.

위의 AWS S3 버킷은 특정 IP 대역에서만 접근하도록 설정했다. 

Bitbucket Pipelines에서 AWS S3 버킷을 사용하도록 하기 위해 AWS S3 정책에 [What are the Bitbucket Cloud IP addresses I should use to configure my corporate firewall?](https://confluence.atlassian.com/bitbucket/what-are-the-bitbucket-cloud-ip-addresses-i-should-use-to-configure-my-corporate-firewall-343343385.html) 문서에 있는 IP를 추가했다.

Bitbucket Pipelines을 통해 패키지를 설치하니 `403` 에러가 나면서 `test-python-package`를 설치하지 못하고 실패했다.

### 원인은 무엇일까?
해당 내용을 찾을 수가 없었기 때문에 Bitbucket 팀에 직접 문의를 했다. 답변은 다음과 같았다.

```
We appreciate that you let us know about it.
We've looked into the issue with our team and we believe that this is due to Pipelines infrastructure on AWS.
For your information, Pipelines uses a VPC when making requests to S3 or DynamoDB.
We do this to significantly increase the performance of caches and artefacts (among other things) and reduce build times for our users.
Unfortunately, this also means that requests from those services do not come from the documented public IP ranges.
 
The internal IPs from AWS service are unpredictable and subject to change so whitelisting these services by source IP is only possible if you bypass the VPC by proxying the request over the public internet.
As a workaround, it's possible to set up a proxy for S3 using AWS API Gateway: https://docs.aws.amazon.com/apigateway/latest/developerguide/integrating-api-with-aws-services-s3.html
Or you can host the S3 bucket which is not on the same aws region as Pipelines(us-west-1 and us-east-1)
 
I hope this explains.
```

Bitbucket Pipelines은 AWS S3나 Dynamo DB에 요청을 보내는 경우에는 VPC를 사용하기 때문에 Bitbucket Pipeline과 S3나 Dynamo DB가 동일한 리전에 있는 경우 문서에 있는 IP가 아닌 AWS 내부 IP를 사용한다는 것이다.

위의 `requirements.txt`에 추가한 URL을 보면 AWS S3 버킷의 위치는 `us-east-1` 이다.

따라서 Bitbucket Pipelines과 AWS S3 버킷이 동일한 리전에 있었고 AWS S3 정책에 문서에 있는 IP를 추가해도 AWS S3 버킷에 접근할 수 없었던 것이다.

### 결론
Bitbucket Pipelines에서 AWS S3나 DynamoDB에 직접 요청을 보내는 경우 제대로 동작을 하지 않는다면 AWS S3와 DynamoDB 리전을 확인해야 한다.

Bitbucket Pipelines은 `us-east-1`과 `us-west-1`을 사용하므로 해당 리전과 다른 리전을 사용하던가 아니면 프록시를 사용하여 VPC가 아닌 퍼블릭 인터넷을 사용하도록 해야 한다.


### 참고
- [Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines)
- [Python with Bitbucket Pipelines](https://confluence.atlassian.com/bitbucket/python-with-bitbucket-pipelines-873891271.html)
- [pip --extra-index-url 옵션](https://pip.pypa.io/en/stable/reference/pip_wheel/#extra-index-url)
- [pip --trusted-host 옵션](https://pip.pypa.io/en/stable/reference/pip/#cmdoption-trusted-host)