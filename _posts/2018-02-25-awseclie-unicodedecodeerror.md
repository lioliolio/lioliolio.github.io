---
layout: post
title: "awsebcli 사용시 나타나는 UnicodeDecodeError 해결하기"
date: 2017-02-25
---
파이썬 장고로 프로젝트를 시작하면서 AWS Elastic Beanstalk에 배포를 하기 위해서 [공식 문서](https://docs.aws.amazon.com/ko_kr/elasticbeanstalk/latest/dg/create-deploy-python-django.html)에 따라 설정을 하고 있었다. region 설정, 파이썬 버전 설정 등 잘 진행하고 있었는데 AWS CodeCommit 을 사용할 것인지 묻는 과정에서 `UnicodeDecodeError` 가 나타났다.

![UnicodeDecodeError Image]({{site.baseurl}}/assets/img/unicode-decode-error.png)

여러번 다른 값을 입력하고 다시 실행해도 같은 위치에만 에러가 났기 때문에 debug 모드로 해당 코드를 찾아서 확인해보았다. 

```
# ebcli/objects/sourcecontrol.py

def set_up_ignore_file(self):
    if not os.path.exists('.gitignore'):
        open('.gitignore', 'w')
    else:
        with open('.gitignore', 'r') as f:
            for line in f:
                if line.strip() == git_ignore[0]:
                    return

    with open('.gitignore', 'a') as f:
        f.write(os.linesep)
        for line in git_ignore:
            f.write(line + os.linesep)
```

프로젝트 폴더에는 `.gitignore` 파일이 있었고 `.gitignore`을 연 후 `for line in f:`에서 `UnicodeDecodeError`가 나는 상황이었다. 

문제 해결은 간단했다. `ascii`로 디코딩을 못하는 상황이었으니 `ascii`로 디코딩 할 수 없는 문자가 `.gitignore` 파일에 있다는 건데 그것만 찾아서 삭제하면 된다. 한글로 주석을 추가했던 것이 있었고 해당 주석을 삭제하고 난 뒤에는 에러 없이 잘 실행됐다.

파이썬3에서 open() 함수의 경우 

*"The default encoding is platform dependent (whatever locale.getpreferredencoding() returns), but any text encoding supported by Python can be used."*

디폴트 인코딩은 해당 시스템에 의존한다. 내가 사용하는 컴퓨터에서는 locale 설정이 `en_US.UTF-8` 이었기 때문에 `locale.getpreferredencoding()` 의 리턴값은 `UTF-8`이었다. 디폴트 인코딩을 사용했다면 에러가 나지 않았을 것이다.

`awsebcli` 내의 코드를 확인해보니 `locale.setlocale(locale.LC_ALL, 'C')`로 locale 설정을 하기 때문에 `ascii`로 인코딩 설정이 바뀐것이다. 따라서 `.gitignore`파일의 인코딩은 `UTF-8`이 아닌 `US-ASCII`이었다.

결론은, 

### **`awsebcli` 사용할 때 한글 쓰면 안됩니다.**

 
