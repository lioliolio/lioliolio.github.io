---
layout: post
title: pyenv, pyenv-virtualenv, autoenv로 파이썬 개발환경 설정하기
date: 2018-06-10
---
파이썬을 사용하다보면 특정 버전을 사용해야만 하는 경우가 있다(라이브러리 지원, 또는 과거 aws elb의 파이썬 3.4 지원 등). 그때그때 파이썬 버전을 설치해도 되지만 파이썬 버전을 쉽게 관리할 수 있게 도와주는 `pyenv`가 있다.

또한 프로젝트 별 패키지 관리를 위해 사용하는 `virtualenv`를 `pyenv`와 쉽게 연동해 사용할 수 있는 `pyenv-virtualenv`를 사용하여 파이썬 개발환경을 설정해보자. 

### [pyenv](https://github.com/pyenv/pyenv) 설치하기
- [pyenv](https://github.com/pyenv/pyenv) 문서를 참고했다.
- [git](https://git-scm.com/)이 설치되어 있지 않다면 [git](https://git-scm.com/)부터 설치하자.

```
$ git clone https://github.com/pyenv/pyenv.git ~/.pyenv
```
- Mac 사용자라면 [HomeBrew](https://brew.sh/index_ko) 라는 Mac용 패키지 관리자를 이용해 손쉽게 설치 가능하다.

```
$ brew update
$ brew install pyenv
```

- `bash`를 사용하면 `.bashrc` 또는 `.bash_profile`, `zsh`을 사용하면 `.zshrc`로 변경한다.

```
$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
```

- shell을 재실행한다.

```
$ exec "$SHELL"
```

### [pyenv](https://github.com/pyenv/pyenv)를 사용하여 파이썬 설치하기

- 설치 가능한 파이썬 버전 확인하기

```
$ pyenv install -l
```

- 파이썬 설치하기

```
$ pyenv install 3.6.5
```

- 파이썬 삭제하기

```
$ pyenv uninstall 3.6.5
```

### [pyenv-virtaulenv](https://github.com/pyenv/pyenv-virtualenv) 설치하기

```
$ git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
```

- Mac 유저는 [HomeBrew](https://brew.sh/index_ko) 를 사용하여 쉽게 설치할 수 있다.

```
$ brew install pyenv-virtualenv
```

- `bash`를 사용하면 `.bashrc` 또는 `.bash_profile`, `zsh`을 사용하면 `.zshrc`로 변경한다.

```
$ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
```

### [pyenv-virtaulenv](https://github.com/pyenv/pyenv-virtualenv) 사용하기

- `pyenv virtualenv 파이썬버전 가상환경이름` 으로 가상환경을 만들 수 있다.
- 아래의 명령어는 파이썬 3.6.5 버전을 사용하여 new-django-project라는 가상환경을 만드는 명령어이다.

```
$ pyenv virtualenv 3.6.5 new-django-project
```

- `pyenv activate new-django-project`는 new-django-project 가상환경을 활성화 시키는 명령어이다.

- 가상환경이 활성화되면 아래처럼 터미널 앞쪽에 `(new-django-project)`가 표시된다.

![virtualenv activate Image]({{site.baseurl}}/assets/img/virtualenv-activate.png)

- `pyenv deactivate`는 가상환경을 비활성화 하는 명령어다.

- 가상환경이 비활성화되면 터미널 앞쪽에 `(new-django-project)`가 사라진다

![virtualenv deactivate Image]({{site.baseurl}}/assets/img/virtualenv-deactivate.png)

### [autoenv](https://github.com/kennethreitz/autoenv) 설치하기

[autoenv](https://github.com/kennethreitz/autoenv)는 특정 디렉토리에 `cd`로 이동하면 `.env` 파일을 실행하는 프로그램이다. [autoenv](https://github.com/kennethreitz/autoenv)를 사용하면 손쉽게 가상환경 활성화 및 특정 환경변수 설정을 할 수 있다.

```
$ git clone git://github.com/kennethreitz/autoenv.git ~/.autoenv
$ echo 'source ~/.autoenv/activate.sh' >> ~/.bashrc
```

- Mac 유저는 [HomeBrew](https://brew.sh/index_ko) 를 사용하여 쉽게 설치할 수 있다.

```
$ brew install autoenv
$ echo "source $(brew --prefix autoenv)/activate.sh" >> ~/.bash_profile
```

- `bash`를 사용하면 `.bashrc` 또는 `.bash_profile`, `zsh`을 사용하면 `.zshrc`로 변경한다.

![autoenv Image]({{site.baseurl}}/assets/img/autoenv.png)

- `new-django-project` 폴더 안에 `.env`파일을 만들었다.
- `.env`파일 안에는 `pyenv activate new-django-project`를 추가했다.
- `cd`로 `new-django-project` 폴더로 이동하면 `.env` 파일이 자동으로 실행되면서 `new-django-project` 이름의 가상환경이 활성화 된다. 

### 참고
[pyenv](https://github.com/pyenv/pyenv)

[pyenv-virtaulenv](https://github.com/pyenv/pyenv-virtualenv)

[autoenv](https://github.com/kennethreitz/autoenv)