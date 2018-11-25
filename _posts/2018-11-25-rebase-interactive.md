---
layout: post
title: "대화형 리베이스(interactive rebase)"
date: 2018-11-25
---
최근 아주 큰 PR을 올리면서 코드 리뷰 하는 분들을 고통받게 했다. 가장 좋은건 적당한 크기로 PR을 올리는 건데...종종 그걸 넘어서는 경우가 있다. 커밋을 잘 작성한다면 코드 리뷰 하는 분들이 고통을 덜 받지 않을까 하는 생각에 대화형 리베이스(interactive rebase)에 관심을 갖게 되었고 이번 기회에 정리를 해볼 생각이다.

### 대화형 리베이스(interactive rebase)
대화형 리베이스(interactive rebase)는 커밋 히스토리를 수정 할 때 아주 유용하다. 

터미널에서 다음과 같은 명령어로 실행할 수 있다.

`git rebase -i HEAD~3`

HEAD를 포함한 마지막 세 개의 커밋을 리베이스 한다.

`git rebase -i --root`

위의 명령어는 모든 커밋을 리베이스 한다.

### 커밋 순서 변경하기
![interactive-rebase-01]({{site.baseurl}}/assets/img/interactive-rebase-01.png)

네 개의 커밋이 존재한다. test_1.txt에 대한 커밋과 test_2.txt에 대한 커밋끼리 묶기 위해 순서를 변경해 보자.

`git rebase -i --root`를 실행하면 에디터에 다음처럼 커밋 내역이 나온다.

![interactive-rebase-02]({{site.baseurl}}/assets/img/interactive-rebase-02.png)

![interactive-rebase-03]({{site.baseurl}}/assets/img/interactive-rebase-03.png)

위와 같이 위치를 변경하고 저장을 한다.

![interactive-rebase-04]({{site.baseurl}}/assets/img/interactive-rebase-04.png)

위와 같이 순서가 변경된 것을 볼수 있다.

### 커밋 수정하기
커밋을 수정하는 방법은 `reword`, `edit` 두 가지가 있다.

![interactive-rebase-05]({{site.baseurl}}/assets/img/interactive-rebase-05.png)

`edit`의 경우 위의 그림 처럼 파일을 추가하거나 수정할 수 있다. `reword`는 파일은 수정할 수 없고 커밋 메시지만 수정 가능하기 때문에 에디트만 실행된다.

### 커밋 합치기
커밋을 합치는 방법은 `squash`를 사용하면 된다.

![interactive-rebase-06]({{site.baseurl}}/assets/img/interactive-rebase-06.png)

`squash`는 이전 커밋 메시지와 합친다. 위와 같이 변경한 뒤 저장을 하면

![interactive-rebase-07]({{site.baseurl}}/assets/img/interactive-rebase-07.png)

위와 같이 에디터가 실행된다. 여기에서 커밋 이름을 변경한 뒤 저장한다.

![interactive-rebase-08]({{site.baseurl}}/assets/img/interactive-rebase-08.png)

커밋 두개가 합쳐진 것을 확인 할 수 있다.

### 커밋 분리하기
커밋을 분리하기 위해서는 `edit`를 사용하면 된다.

![interactive-rebase-09]({{site.baseurl}}/assets/img/interactive-rebase-09.png)

`edit`를 사용한 후 `git reset HEAD^`를 사용하면 커밋을 해제할 수 있다. 그 뒤 원하는 대로 커밋을 작성할 수 있다.

![interactive-rebase-10]({{site.baseurl}}/assets/img/interactive-rebase-10.png)

### 주의점
첨부한 이미지를 잘 보면 커밋 값이 변경된 걸 볼 수 있다. 리베이스는 커밋을 변경하기 때문에 원격 저장소에 올라간 코드를 리베이스 해서는 안된다. 꼭 기억하자.

### 참고
- [7.6 Git 도구 - 히스토리 단장하기](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-%ED%9E%88%EC%8A%A4%ED%86%A0%EB%A6%AC-%EB%8B%A8%EC%9E%A5%ED%95%98%EA%B8%B0)