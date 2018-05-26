---
layout: post
title: "Vim 명령모드 진입시 자동으로 영문 키보드로 변경하기(Mac OS)"
date: 2018-03-11
---
## `Vim` 사용시 불편한 점
나는 `Vim`을 자주 사용한다. `Vim`을 사용하면서 불편한 점은

**한글키보드를 사용하다가 `명령 모드`로 진입 하면 영문 키보드로 변경해야 하는 것이다.**

`명령 모드`에서는 거의 영문 키보드를 사용하게 된다. 매번 키보드 레이아웃을 변경하는 것은 매우 귀찮은 일이었기 때문에 명령 모드 진입시 자동으로 키보드 레이아웃을 변경하는 방법을 찾아보았다.

## Input Source Switcher 사용하기 (Mac OS)

[Input Source Switcher](https://github.com/vovkasm/input-source-switcher) 는 MAC OS 커맨드라인 환경에서 쉽게 키보드 레이아웃을 변경할 수 있도록 도와주는 유틸리티 프로그램이다.

### 1. [Input Source Switcher](https://github.com/vovkasm/input-source-switcher) 설치

```
git clone https://github.com/vovkasm/input-source-switcher.git
cd input-source-switcher
mkdir build && cd build
cmake ..
make
make install
```
- `cmake`가 필요하다. 없다면 `brew install cmake`로 설치하자.
- `Input Source Switch`를 설치했다면 두 가지 방법으로 설정할 수 있다.

### 2-1. `.vimrc` 설정 추가

```
if filereadable('/usr/local/lib/libInputSourceSwitcher.dylib')
    autocmd InsertLeave * call libcall('/usr/local/lib/libInputSourceSwitcher.dylib', 'Xkb_Switch_setXkbLayout', 'com.apple.keylayout.ABC')
endif
```
- `.vimrc`에 위의 내용을 추가한다.

#### 주의할 점
위의 코드는 [vim 명령모드에서 영문 키보드로 자동으로 전환되게 만들기](https://sangwook.github.io/2015/01/01/vim-insert-mode-keyboard-switch.html)를 참고했다. [MacVim](http://macvim-dev.github.io/macvim/), [Neovim](https://neovim.io/), [HomeBrew로 설치한 Vim](https://brew.sh/)을 사용하고 있다면 블로그에 있는 코드를 그냥 사용해도 괜찮다.

하지만, `usr/bin/vim` 경로를 가진 `Vim`을 사용하는 중이라면 블로그에 있는 코드가 제대로 동작하지 않는다. `has('mac')`코드에서 0을 리턴하기 때문이다(모든 맥에서 동일한지 잘 모름). 그래서 그 부분을 삭제했다.


### 2-2 [vim-xkbswitch](https://vimawesome.com/plugin/vim-xkbswitch) 플러그인 사용
[vim-airline](https://vimawesome.com/plugin/vim-airline-superman)을 사용하고 있다면 [vim-xkbswitch](https://vimawesome.com/plugin/vim-xkbswitch) 플러그인 설치시 상태바에 현재 키보드 레이아웃이 표시된다. 다른 기능에 대해서는 문서를 참고하도록 하자.

```
let g:XkbSwitchEnabled = 1
let g:XkbSwitchLib = '/usr/local/lib/libInputSourceSwitcher.dylib'
```
- [vim-xkbswitch](https://vimawesome.com/plugin/vim-xkbswitch) 플러그인을 설치한 후 `.vimrc`에 위의 내용을 추가한다.


#### 주의할 점
`usr/bin/vim` 경로를 가진 `Vim`을 사용하는 경우 [vim-xkbswitch](https://vimawesome.com/plugin/vim-xkbswitch) 플러그인이 동작하지 않는다(모든 맥에서 동일한지는 잘 모름). [MacVim](http://macvim-dev.github.io/macvim/), [Neovim](https://neovim.io/), [HomeBrew로 설치한 Vim](https://brew.sh/)은 잘 동작한다.

터미널 기본 `Vim`을 사용하면서 [vim-xkbswitch](https://vimawesome.com/plugin/vim-xkbswitch) 플러그인 사용하고 싶다면 코드를 수정해주어야 한다.

```
# ~/.vim/bundle/vim-xkbswitch/plugin/xkbswitch.vim

if !has('macunix') && !has('osx') && has('unix') && empty($DISPLAY)
```
- 19번째 줄 기존 코드 `if !has('macunix') && has('unix') && empty($DISPLAY)`에 `&& !has('osx')` 추가한다.


### [구름 입력기](http://gureum.io/) 사용하기
[구름 입력기](http://gureum.io/) 에는 `esc 키로 로마자 자판으로 변환` 옵션이 있다고 한다. [구름 입력기](http://gureum.io/)를 사용한다면 더 쉽게 사용할 수 있을거라 생각한다. 직접 써본 것은 아니기 때문에 자세한 내용은 문서를 보도록 하자.

### 참고
- [vim 명령모드에서 영문 키보드로 자동으로 전환되게 만들기](https://sangwook.github.io/2015/01/01/vim-insert-mode-keyboard-switch.html)
- [Input Source Switcher](https://github.com/vovkasm/input-source-switcher)
- [vim-xkbswitch](https://vimawesome.com/plugin/vim-xkbswitch)
