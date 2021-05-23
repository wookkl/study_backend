## git pull이란?

다른 레포지토리 또는 로컬 브랜치를 가져와서 통합하는 것이다.
쉽게 말하면 `git pull`은 `git fetch` 후 `git merge`하는 것이다.

> git pull = git fetch + git merge

실제로 git pull에 대한 도움말을 보면

```shell
git pull --help

DESCRIPTION

Incorporates changes from a remote repository into the current branch. In its default mode, git pull is shorthand for git fetch followed by git merge FETCH_HEAD.
```

원격 저장소의 변화들을 현재 브랜치로 통합한다. `git pull`은 `git fetch`와 `git merge FETCH_HEAD`의 약자이다.

> FETCH_HEAD란?
> 원격 저장소로 부터 가져온 모든 브랜치의 HEAD를 .git/FETCH_HEAD에 기록한다. FETCH_HEAD는 원격 저장소로부터 가져온 브랜치의 HEAD를 의미한다.

여기에 문제가 있다. git pull은 종종 merge commit을 생성하여 merge를 진행한다. 그럼 이전에 존재하지 않았던 commit 하나가 생기는 것이다. 치명적이진 않지만, 깃 초보자에게 혼동을 줄 수 있다. 그래서 깃은 사용자가 git pull하는 방법을 구성하지 않았다면 아래처럼 경고의 문구를 표시한다.

```
warning: Pulling without specifying how to reconcile divergent branches is
discouraged. You can squelch this message by running one of the following
commands sometime before your next pull:

  git config pull.rebase false  # merge (the default strategy)
  git config pull.rebase true   # rebase
  git config pull.ff only       # fast-forward only

You can replace "git config" with "git config --global" to set a default
preference for all repositories. You can also pass --rebase, --no-rebase,
or --ff-only on the command line to override the configured default per
invocation.
```

여기서 git pull을 하는 방식중에 세가지가 소개된다.

1. `git config pull.rebase false`
2. `git config pull.rebase true`
3. `git config pull.ff only`

**rebase** 란 나뉘어진 두 브랜치가 나뉘기 전인 공통 커밋으로 이동하고 나서 그 커밋 부터 지금 checkout한 브랜치가 가리키는 커밋까지 diff를 차례로 만들어 나가면서 임시로 저장해 놓는다. Rebase할 브랜치가 합칠 브랜치가 가리키는 커밋을 가리키게 하고 아까 저장해 놓았던 변경사항을 차례대로 적용한다.

**fast-forward**
현재의 브랜치가 가지고 있는 커밋들을 충돌 없이 따라갈 수 있는 관계를 말한다.
원격의 브랜치를 로컬 브랜치로 머지하고 싶을때 원격 브랜치의 커밋 히스토리가 로컬 브랜치의 커밋 히스토리에 완전히 포함되어 있을 경우를 말한다.

**`git config pull.rebase false`**

- pull 할 때 rebase를 하지 않고 merge한다.

**`git config pull.rebase true`**

- pull 할 때 rebase를 한다.

**`git config pull.ff only`**

- fast-foward 일때만 pull을 허용한다.
