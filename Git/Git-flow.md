# Git 브랜치 전략
Git을 이용한 브랜치 전략에는 크게 Github-flow와 git-flow 전략이 있습니다.

두 개의 전략이 어떤게 다르고 언제 어떤걸 사용하는게 좋은지 알아보겠습니다.

<br><br>

# Github-flow

![image1](/Img/Git-Flow/github-flow.png)

Github-flow는 1개의 master 브랜치와 PR을 활용한 단순하고 민첩한 브랜치 전략이다.

기능 개발, 버그 픽스 등 어떤 이유로든 새로운 브랜치를 생성하는 것으로 시작된다. 단 이때 브랜치 하나에 의존하게 되기 때문에 **브랜치 이름을 통해 의도를 명확하게 드러내는 것이 매우 중요하다**.

<br>

## 작업 과정

1. master 브랜치에서 작업 브랜치 (bfm-100_login)를 생성합니다.
> (master)]$ git checkout -b bfm-100_login

2. 작업 브랜치에 변경사항을 커밋합니다.
> (bfm-100_login)]$ git commit -m "BFM-100 로그인 구현"

3. 배포할 기능을 모두 구현했으면 push 후 Pull Request를 생성합니다.
> (bfm-100_login)]$ git push origin bfm-100_login

4. 병합되면 서버에 배포되는거와 다름 없으므로 상세한 리뷰와 토의를 거칩니다.
5. 리뷰와 토의가 끝났으면 라이브 서버(혹은 테스트 서버)에 배포해 테스트를 합니다.

    5.1 배포시 문제가 발생한다면 다시 rollback 시킵니다.

<br>

## 특징

- 브랜치 전략이 단순하며 git을 처음 접하는 사람도 쉽게 접근할 수 잇다.
- CI/CD가 자연스럽게 이루어 진다.

<br><br>

# Git-flow 전략

Git-flow에는 5가지 종류의 브랜치가 존재합니다. 항상 유지되는 메인 브랜치들(master, develop)과 일정 기간동안만 유지되는 보조 브랜치들(feature, release, hotfix)이 있습니다.

- master : 제품으로 출시될 수 있는 브랜치
- develop : 다음 출시 버전을 개발하는 브랜치
- feature : 기능을 개발하는 브랜치
- release : 이번 출시 버전을 준비하는 브랜치
- hotfix : 출시 버전에서 발생한 버그를 수정하는 브랜치

![image2](/Img/Git-Flow/gitflow.png)

- 처음에는 master와 develop 브랜치가 존재합니다.
- develop 브랜치에는 상시로 버그를 수정한 커밋들이 추가됩니다.
- 새로운 기능 추가 작업이 있는 경우 develop 브랜치에서 feature 브랜치를 생성합니다.
(feature 브랜치는 언제나 develop 브랜치에서 시작하게 됩니다.)
- 기능 추가 작업이 완료되면 feature 브랜치를 develop 브랜치로 merge 합니다.
- develop에 이번 버전에 포함되는 모든 기능이 merge 되었다면 QA를 하기 위해 develop 브랜치에서 release 브랜치를 생성합니다.
- QA를 진행하면서 발생한 버그들은 release 브랜치에 수정됩니다.
- 무사히 통과했다면 release 브랜치를 master와 develop 브랜치로 merge 합니다.
- 마지막으로 출시된 master 브랜치에서 버전 태그를 추가합니다.

<br><br>

## 작업을 할 때 지켜야할 약속

1. 작업을 시작하기 전에 JIRA 티켓을 생성합니다.
2. 하나의 티켓은 되도록 하나의 커밋으로 합니다.
3. 커밋 그래프는 최대한 단순하게 가져갑니다.
4. 서로 공유하는 브랜치의 커밋 그래프는 함부로 변경하지 않습니다.
5. 리뷰어에게 꼭 리뷰를 받습니다.
6. 자신의 Pull Request는 스스로 merge 합니다.

<br>

## 실제 작업 과정

환경 설정

- Repositories
    - upstream (Upstream Repository)
    - origin (Origin Repository)
- Branches
    - feature-user (사용자 관련 기능을 구현하는 feature branch)
    - bfm-100_login_layout (사용자 관련 기능 중 레이아웃 작업 branch)

<br>

## 1. 티켓 처리하기

요구사항 : '로그인 레이아웃 생성' 이라는 티켓을 처리

1. upstream/feature-user 브랜치에서 작업 브랜치 (bfm-100_login_layout)를 생성합니다.
> (feature-user)]$ git fetch upstream  
(feature-user)]$ git checkout -b bfm-100_login_layout -track upstream/feature-user

2. 작업 브랜치에서 소스코드를 수정합니다.
3. 작업 브랜치에서 변경사항을 커밋합니다.
> (bfm-100_login_layout)]$ git commit -m "BFM-100 로그인 화면 레이아웃 생성"

4. 커밋이 불필요하게 여러개로 나뉘어져 있다면 squash를 합니다.
> (bfm-100_login_layout)]$ git rebase -i HEAD~2

5. 작업 브랜치를 upstream/feature-user에 rebase합니다.
> (bfm-100_login_layout)]$ git pull -rebase upstream feature-user

6. 작업 브랜치를 origin에 push합니다.
> (bfm-100_login_layout)]$ git push origin bfm-100_login_layout

7. Github에서 bfm-100_login_layout 브랜치를 feature-user에 merge하는 Pull Request를 생성합니다.
8. 같은 feature를 개발하는 동료에게 리뷰 승인을 받은 후 자신의 Pull Request를 merge 합니다.

위의 절차에서 4,5번 과정을 수행하는 이유를 모르신다면 [여기](https://charliezip.tistory.com/5)를 들어가서 글을 한번 읽어보시면 좋습니다.

<br>

---

Git은 자동으로 `master` 브랜치를 `origin/master` 브랜치의 트래킹 브랜치로 만든다. 트래킹 브랜치를 직접 만들 수 있는데 리모트를 `origin`이 아닌 다른 리모트로 할 수도 있고, 브랜치도 `master` 가 아닌 다른 브랜치를 추적하게 할 수 있다. `git checkout -b <branch> <remote>/<branch>` 명령으로 간단히 트래킹 브랜치를 만들수 있다.

[https://git-scm.com/book/ko/v2/Git-브랜치-리모트-브랜치#_tracking_branches](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%A6%AC%EB%AA%A8%ED%8A%B8-%EB%B8%8C%EB%9E%9C%EC%B9%98#_tracking_branches)

---

<br><br>

## 2. develop 변경사항을 feature로 가져오기 (Optional)

작업을 할 때 브랜치의 수명은 되도록 짧게 가져가는 게 좋지만, feature 브랜치에서 기능을 완료하는데 시간이 오래 걸리는 경우가 있습니다. 그러다 보면 develop에 추가된 기능들이 필요한 경우가 종종 생기게 됩니다. 그럴 때는 feature 브랜치에 develop의 변경사항들을 가져와야 합니다.

1. feature-user 브랜치에 upstream/develop 브랜치를 merge합니다.
> (feature-user)]$ git fetch upstream  
(feature-user)]$ git merge -no-ff upstream/develop

2. upstream/feature-user 기능이 merge된 develop를 upstream에 push 합니다.
> (feature-user)]$ git push upstream feature-user

<br>

---

병합(merge)에는 ff , no-ff 두가지 옵션이 있습니다.

먼저 병합(merge)전 상태입니다.

![image3](/Img/Git-Flow/setting.png)

<br>

1. **—ff**

**기본 설정 옵션**입니다. 병합대상 브랜치가 fast-forward 관게인 경우 병합커밋을 만들지 않고 브랜치 태킹만 변경하게 됩니다.

```bash
$ git merge topic
```

![image4](/Img/Git-Flow/ff.png)

2. **—no-ff**

fast-foward 관계인 경우에도 반드시 병합커밋을 만듭니다.

```bash
$ git merge --no-ff topic
```

![image5](/Img/Git-Flow/no-ff.png)

출처: [https://koreabigname.tistory.com/65](https://koreabigname.tistory.com/65)

---

<br><br>

## 3. 완료된 기능을 이번 출시 버전에 포함시키기

1. develop 브랜치에 upstream/feature-user 브랜치를 merge합니다.
> (develop)]$ git fetch upstream  
(develop)]$ git merge —no-ff upstream/feature-user

2. upstream/feature-user 기능이 merge 된 develop를 upstream에 push 합니다.
> (develop)]$ git push upstream develop

<br>

## 4. QA 시작하기

출시를 시작하기 위해 먼저 release 브랜치를 생성하고 upstream에 push하여 release 브랜치를 공유합니다.

1. release-1.0.0 브랜치를 생성합니다.
> (develop)]$ git fetch upstream  
(develop)]$ git checkout -b release-1.0.0 —track upstream/develop

2. release-1.0.0 브랜치를 upstream에 push합니다.
> (release-1.0.0)]$ git push upstream release-1.0.0

<br>

## 5. QA 중 버그 수정하기

개발을 완료한 후 QA중 버그가 발생하면 예외 상황이 발생할 때마다 버그 티켓이 하나씩 생성됩니다. 버그 티켓들도 티켓이기 때문에 '1. 티켓 처리하기' 와 같은 방법으로 처리합니다.

1. release 브랜치에서 버그 티켓에 대한 브랜치를 생성합니다.
> (release-1.0.0)]$ git checkout -b bfm-100_bug_login_id_max_length

2. 버그를 수정합니다.
3. 작업 브랜치에 버그 수정사항을 커밋합니다.
> (bfm-100_bug_login_id_max_length)]$ git commit -m "BFM-101 로그인 아이디 길이 제한 버그 수정"

4. 작업 브랜치를 origin에 push 합니다.
> (bfm-100_bug_login_id_max_length)]$ git push origin bfm-100_bug_login_id_max_length

5. Github에서 bfm-100_bug_login_id_max_length 브랜치를 release-1.0.0에 merge하는 Pull Request를 생성합니다.
6. 동료에게 리뷰 승인을 받은 후 자신의 Pull Request를 merge 합니다.

<br>

## 6. 앱 출시

발생하는 버그들을 모두 수정했다면 이젠 출시를 준비할 때 이다. release 브랜치를 master 브랜치와 develop 브랜치에 merge하고 마지막으로 master 브랜치에서 버전 태그를 달아줍니다.

1. release 브랜치를 최신 상태로 갱신합니다.
> (release-1.0.0)]$ git pull upstream release-1.0.0

2. release 브랜치를 develop 브랜치에 merge 합니다.
> (release-1.0.0)]$ git checkout develop  
(develop)]$ git pull upstream develop  
(develop)]$ git merge —no-ff release-1.0.0

3. develop 브랜치를 upstream에 push 합니다.
> (develop)]$ git push upstream develop

4. release 브랜치를 master 브랜치에 merge 합니다.
> (develop)]$ git checkout master  
(master)]$ git pull upstream master  
(master)]$ git merge —no-ff release-1.0.0

5. 1.0.0 태그를 추가합니다.
> (master)]$ git tag 1.0.0

6. master 브랜치와 1.0.0 태그를 upstream에 push 합니다.
> (master)]$ git push upstream master 1.0.0

<br><br>

# 마무리

`Github-flow` 와 `Git-flow` 전략에 대해 알아보았습니다.

두 전략에는 각각의 장단점이 있지만 어느 하나가 딱 정답인 것은 아니라고 생각합니다.

개인적으로는 개인프로젝트, 소규모 프로젝트는 `Github-flow`가 더 적합해보이고

규모가 있는 프로젝트는 `Git-flow` 전략이 적합해 보이지만

중요한건 같이 작업하는 사람들과 약속을 통해 실행한다는 것으로 꼭 이 두가지 전략방법이 아니더라고 프로젝트에 알맞은 전략이 있다면 상관없을거 같습니다.