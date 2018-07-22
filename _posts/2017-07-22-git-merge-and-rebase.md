---
layout: post
title: "Git Merge 와 Rebase"
date: 2018-07-21
---
[Git](https://git-scm.com/)을 사용하면 보통 기능 또는 이슈 별로 브랜치를 나눠서 사용하게 된다. 이때 작업을 끝낸 브랜치를 메인 브랜치에 합쳐야 하는데 이때 사용하는 방법으로는 `Merge`와 `Rebase`가 있다. 개인 프로젝트나 팀 프로젝트에서 이 두 가지를 다 사용하는 중인데 정확하게 어떤 차이가 있는지 잘 모르는 관계로 이번 기회에 정리해 보려고 한다.

### Git Merge

`merge`의 경우 `Fast-forword`와 `3-way Merge` 두 가지 방식이 있다.

#### Fast-forward

![git-merge-example-01]({{site.baseurl}}/assets/img/git-merge-example-01.png)

`master` 브랜치에서 `hotfix-01` 브랜치와 `issue-01` 브랜치를 만들었다. 이때 `master` 브랜치에 `hotfix-01` 브랜치를 `merge` 하면 다음과 같이 나온다.

![git-merge-fast-forward-01]({{site.baseurl}}/assets/img/git-merge-fast-forward-01.png)

`hotfix-01`는 `master` 브랜치의 커밋인 `Commit1`을 포함하고 있다. 따라서 `master` 브랜치가 `hotfix-01` 커밋을 가리키도록만 하면 된다. 이때 이 방식을 `Fast-forward` 방식이고 `merge` 메세지에서도 확인 할 수 있다.

#### 3-way Merge

![git-merge-fast-forward-02]({{site.baseurl}}/assets/img/git-merge-fast-forward-02.png)

이제 `issue-01` 브랜치를 `master` 브랜치에 `merge` 해보자. 위의 예제와 다른 점은 `issue-01` 브랜치는 `master` 브랜치의 커밋인 `Commit2`를 포함하고 있지 않다는 것이다.

![git-merge-3-way-merge-01]({{site.baseurl}}/assets/img/git-merge-3-way-01.png)

이때는 **[3-way-merge](https://en.wikipedia.org/wiki/Merge_(version_control)#Recursive_three-way_merge)** 를 사용한다.

![git-merge-3-way-merge-02]({{site.baseurl}}/assets/img/git-merge-3-way-02.png)

`master` 브랜치의 커밋인 `Commit2`와 `issue-01` 브랜치의 `Commit3`, 그리고 두 브랜치의 공통 조상(Common Ancestor) 커밋인 `Commit1` 이 세 커밋을 사용하여 새로운 커밋인 `Merge branch 'issue-01'`을 만든다. 그리고 `master` 브랜치가 이 커밋을 가리키도록 변경한다.

### Git Rebase

![git-rebase-01]({{site.baseurl}}/assets/img/git-rebase-01.png)

`master` 브랜치에 `issue-01` 브랜치을 `merge`하는 경우 새로운 커밋을 만든다.
`rebase`를 사용하면 `issue-01` 브랜치에 `master` 브랜치의 변경된 사항을 적용할 수 있다.

```
$ git checkout issue-01
$ git rebase master
```

![git-rebase-02]({{site.baseurl}}/assets/img/git-rebase-02.png)

`rebase` 하기 전 `issue-01` 브랜치의 커밋이다.

![git-rebase-03]({{site.baseurl}}/assets/img/git-rebase-03.png)

`rebase` 이후 `issue-01` 브랜치의 커밋이다. 

`master` 브랜치의 `Commit2`, `Commit4` 커밋이 `issue-01` 브랜치에 적용되었다. 그 이후 `Commit3`, `Commit5` 커밋이 있는데 `rebase` 적용 전과 커밋 이름, Author, Date는 같지만 sha-1 해시 값을 변경 되었다.

`rebase` 전 커밋과 같은 내용의 새로운 커밋을 만든 것이다.

![git-rebase-04]({{site.baseurl}}/assets/img/git-rebase-04.png)

```
$ git checkout master
$ git merge issue-01
```

![git-rebase-06]({{site.baseurl}}/assets/img/git-rebase-06.png)

이렇게 변경된 `issue-01` 브랜치를 `master` 브랜치에 `merge`를 하면 `fast-forward` 시킨다. `3-way-merge`에 비해 깔끔한 이력을 확인을 할 수 있다.

![git-rebase-07]({{site.baseurl}}/assets/img/git-rebase-07.png)

### 참고
- [3.2 Git 브랜치 - 브랜치와 Merge 의 기초](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EC%99%80-Merge-%EC%9D%98-%EA%B8%B0%EC%B4%88)
- [3.6 Git 브랜치 - Rebase 하기](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0)
- [git rebase | Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase)
