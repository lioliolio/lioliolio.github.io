---
layout: post
title: "AWS Elastic Beanstalk 구성 파일(.ebextensions)의 commands와 container_commands"
date: 2018-12-09
---
### .ebextensions?
`.ebextensions` 란 AWS Elastic Beanstalk 구성 파일이다. 이 구성 파일을 통해 Elastic Beanstalk를 커스터 마이징 할 수 있다.

소스번들 루트에 `.ebextensions` 폴더를 만들고 그 안에 `*.config`의 형식으로 파일을 만들어 준다.

```
# ~/project/test-app
.ebextensions/
    db-migrate.config
```

`*.config` 파일의 형식은 `JSON` 또는 `YAML` 형식이다. 보통 `YAML` 형식을 더 많이 사용하는 듯하다.

```
# db-migrate.config (AWS 예제 코드)
container_commands:
  01_migrate:
    command: "django-admin.py migrate"
    leader_only: true
option_settings:
  aws:elasticbeanstalk:application:environment:
    DJANGO_SETTINGS_MODULE: ebdjango.settings
```
### commands와 container_commands
`*.config` 파일을 구성하는 키는 [Linux 서버에서 소프트웨어 사용자 지정](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/customize-containers-ec2.html) 문서에서 확인할 수 있다. 이 중 `commands`와 `container_commands`에 대해서 정리할 생각이다.

[commands](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/customize-containers-ec2.html#linux-commands) 키 EC2 인스턴스에서 명령을 실행할 수 있다. 

[container_commands](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/customize-containers-ec2.html#linux-container-commands)는 애플리케이션 소스 코드에 영향을 주는 명령을 실행할 수 있다.

두 키의 차이점을 뭘까?

바로 명령을 실행하는 시점이다.

`commands`는 어플리케이션 및 웹 서버가 설정되고 애플리케이션 버전 파일이 추출되기 전에 실행하며, `container_commands`는 애플리케이션과 웹 서버를 설정하고 애플리케이션 버전 아카이브의 압축을 푼 후 애플리케이션 버전을 배포하기 이전에 실행한다.

### commands와 container_commands 실행 시점
Elastic Beanstalk 파이썬 플랫폼에서 pip를 업그레이드하는 예제를 통해 비교해보자.

Elastic Beanstalk 파이썬 3.4.3 의 경우 pip 버전이 낮아서(7.1.2) `--trusted-host` 옵션이 `requirements.txt` 파일 내부에 있으면 패키지를 설치할 수 없다. 따라서 패키지를 정상적으로 설치하기 위해서는 `requirements.txt` 패키지를 설치하기 전에 pip 업그레이드를 해야 한다.

```
# pip-upgrade.config
container_commands:
  pip_upgrade:
    command: "sudo /opt/python/run/venv/bin/pip install --upgrade pip"
```

`container_commands`로 pip를 업그레이드하는 명령어를 추가하면 배포에 실패한다. `eb-activity.log`를 보자.

```
[2018-12-05T12:47:36.929Z] ERROR [8657]  : Command execution failed: Activity failed. (ElasticBeanstalk::ActivityFatalError)
caused by: Usage: pip [options]
  
  pip: error: no such option: --trusted-host
  
  You are using pip version 7.1.2, however version 18.1 is available.
  You should consider upgrading via the 'pip install --upgrade pip' command.
  2018-12-05 12:47:36,922 ERROR    Error installing dependencies: Command '/opt/python/run/venv/bin/pip install -r /opt/python/ondeck/app/requirements.txt' returned non-zero exit status 1
  Traceback (most recent call last):
    File "/opt/elasticbeanstalk/hooks/appdeploy/pre/03deploy.py", line 22, in main
      install_dependencies()
    File "/opt/elasticbeanstalk/hooks/appdeploy/pre/03deploy.py", line 18, in install_dependencies
      check_call('%s install -r %s' % (os.path.join(APP_VIRTUAL_ENV, 'bin', 'pip'), requirements_file), shell=True)
    File "/usr/lib64/python2.7/subprocess.py", line 540, in check_call
      raise CalledProcessError(retcode, cmd)
  CalledProcessError: Command '/opt/python/run/venv/bin/pip install -r /opt/python/ondeck/app/requirements.txt' returned non-zero exit status 1 (ElasticBeanstalk::ExternalInvocationError)
caused by: Usage: pip [options]
...
```

배포시 `hooks/` 안에 있는 스크립트가 실행하는 순서는 다음과 같다.

```
/opt/elasticbeanstalk/hooks/appdeploy/pre
/opt/elasticbeanstalk/hooks/appdeploy/enact
/opt/elasticbeanstalk/hooks/appdeploy/post
```

`container_commands`를 실행하는 스크립트 파일은 `/opt/elasticbeanstalk/hooks/appdeploy/post/` 폴더 안에 존재한다. 

`requirements.txt` 파일을 설치하는 스크립트는 `/opt/elasticbeanstalk/hooks/appdeploy/pre/` 폴더 안에 존재한다.

따라서 pip 설치 명령어가 실행하는 시점이 requirements.txt 파일을 설치하는 시점보다 뒤였기 때문에 배포에 실패한 것이다.

`commands`로 pip 업그레이드 명령어를 추가해보자.

```
# pip-upgrade.config
commands:
  pip_upgrade:
    command: "sudo /opt/python/run/venv/bin/pip install --upgrade pip"
```

```
[2018-12-06T09:58:50.745Z] INFO  [3245]  - [Application update test_app/AppDeployStage0/EbExtensionPreBuild/Infra-EmbeddedPreBuild/prebuild_1_test_app/Command pip_u       pgrade] : Starting activity...

...

194456 [2018-12-06T09:58:53.055Z] INFO  [3245]  - [Application update test_app/AppDeployStage0/AppDeployPreHook/01new.py] : Starting activity...
194457 [2018-12-06T09:58:53.574Z] INFO  [3245]  - [Application update test_app/AppDeployStage0/AppDeployPreHook/01new.py] : Completed activity.
194458 [2018-12-06T09:58:53.574Z] INFO  [3245]  - [Application update test_app/AppDeployStage0/AppDeployPreHook/02unzip.py] : Starting activity...
194459 [2018-12-06T09:58:58.686Z] INFO  [3245]  - [Application update test_app/AppDeployStage0/AppDeployPreHook/02unzip.py] : Completed activity. Result:

...

[2018-12-06T09:58:58.687Z] INFO  [3245]  - [Application update test_app/AppDeployStage0/AppDeployPreHook/03deploy.py] : Starting activity...
```

`pip_upgrade` 명령어를 소스 번들 압축 풀기 전에 실행한다. 따라서, `container_commands`와 달리 정상적으로 배포하는 것을 확인할 수 있다.

### 참고
- [Linux 서버에서 소프트웨어 사용자 지정](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/customize-containers-ec2.html)
- [Elastic Beanstalk Configuration files(.ebextensions)](http://woowabros.github.io/woowabros/2017/08/07/ebextension.html)
- [Amazon Elastic Beanstalk Hooks](http://www.jmccc.com/blog/archives/2015/06/17/amazon-elastic-beanstalk-hooks/)