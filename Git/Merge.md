Git을 사용하다 보면 자주 Merge를 사용하게 되는데요.

Merge의 다양한 방법과 그로 인해 git log가 어떤식으로 변하는지 살펴보겠습니다.

먼저 Merge에 대한 이해를 쉽게 하기 위해

- Local 환경에서만 Git을 사용
- Confilct가 발생하는 하는 경우는 고려하지 않음
- 기본적인 git command(add, commit, branch) 명령어 설명은 생략하겠습니다.

<br>

**제가 사용할 git log 명령어입니다.**

```bash
$ git log --all --graph --oneline
# --all 로그 전체를 보여준다
# --graph 그래프형식을 이용해 표현해준다
# --oneline 한 커밋당 한줄로 표현해준다.
```

다양한 log 명령어가 궁금하시다면 
[여기](https://git-scm.com/docs/git-log)
를 참고해보세요!

<br>

# [ merge-v1 기본 Merge ]

**환경세팅**

![Merge1](/Img/Merge/Merge1.png)

![Merge2](/Img/Merge/Merge2.png)

먼저 우리가 아는 기본적인 merge를 해보겠습니다.

**git merge (브랜치명) 을 통해 merge를 실행합니다.**

```bash
$ git checkout master
$ git merge merge-v1
```

<br>

**결과**

![Merge3](/Img/Merge/Merge3.png)

![Merge4](/Img/Merge/Merge4.png)

그냥 merge를 수행하면 새로운 커밋이 하나 추가되며 합쳐지는 걸 볼 수 있습니다.

<br>

**merge를 완료한 branch는 삭제**

```bash
$ git branch -d merge-v1
```

<br>

하지만 현재는 하나의 브랜치에서만 merge를 했지만 

만약 여러개의 브랜치가 있고, 여러 브랜치에서 merge를 수행한다고 생각하면 우리의 깃 로그 기록이 매우 복잡해 질거라는걸 상상해 볼 수 있습니다.

깃 로그가 복잡해지면 나중에 유지보수측면이나, 프로젝트를 이해할 때 많은 불편함을 겪을 수 있습니다.

그러면 다른 merge 방법들을 한번 살펴보겠습니다.

<br>

# [ merge-v2 Rebase Merge ]

**환경세팅**

![Merge5](/Img/Merge/Merge5.png)

![Merge6](/Img/Merge/Merge6.png)

rebase 명령어를 실행하고 결과를 보면서 설명을 하겠습니다.

```bash
$ git checkout merge-v2
$ git rebase master
```

<br>

**결과**

![Merge7](/Img/Merge/Merge7.png)

![Merge8](/Img/Merge/Merge8.png)

rebase 명령어를 실행하면 

1. 두 브랜치가 나뉘기 전인 공통 커밋 work 3로 이동한뒤
2. 지금 checkout 한 브랜치(merge-v2)가 가리키는 커밋까지 diff를 차례로 만들어 저장해 놓는다.
3. rebase 할 브랜치(merge-v2)가 합칠 브랜치(master)가 가리키는 커밋을 가리키게 하고 아까 저장해 놓았던 변경사항을 차례대로 적용한다.

<br>

음.. 말이 너무 어려운거 같다.

쉽게 생각해보자면 **merge-v2** 에 있는 커밋을 **master**가 가리키는 **`work 4`** 커밋부터 하나씩 **merge**를 한다고 생각하면 된다.

그래서 rebase 명령어는 rebase 할 브랜치(merge-v2)의 커밋 수 만큼 새로운 커밋이 추가된다.

헌데 지금 깃 로그를 보니 master 브랜치는 `work 4` 커밋을 가리키고 있다.

우리가 원하는건 master 브랜치가 `merge-v2 work 2` 를 가리키게 하고 싶다.

rebase 한 merge-v2를 master 브랜치랑 합쳐주자.

<br>

**master 브랜치 Fast-forward**

```bash
$ git checkout master
$ git merge merge-v2
```

![Merge9](/Img/Merge/Merge8-1.png)

드디어 우리가 원하는 결과가 나왔다.

rebase 명령어는 좀 더 복잡하지만 깃 로그를 봤을때 v1 merge 보다 훨씬 깔끔한걸 볼 수 있다.

> **주의**
rebase는 merge와 반대로 `git checkout merge-v2` 를 통해 merge-v2 브랜치로 가서  rebase 명령어를 실행했다.
rebase는 자기자신 브랜치를 변경하는 명령어 이므로 꼭 자기자신 branch로 checkout 한뒤 사용하도록 하자.

<br>

현재는 Local 환경에서만 하는거라 문제가 없지만, github 같은 remote 환경도 같이 사용하고 있다면 rebase를 함부로 사용하면 안된다.

**딱, 한가지 규칙만 명심하면 된다. remote 저장소로 push 한 커밋은 rebase 하지 않는다.**

왜 그런지 이유가 궁금하시다면 
[여기](https://git-scm.com/book/ko/v2/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-Rebase-%ED%95%98%EA%B8%B0) 
링크에서 `Ctrl+F`로 '위험성' 이라 검색해서 읽어보시길 추천한다.

<br>

# [ merge-v3 Squash Merge ]

rebase로 깃로그도 깔끔해졌다. 

하지만 rebase에는 단점이 있었다. 만약 rebase 하는 브랜치에 커밋이 100개가 있다면

rebase 후에 새로 커밋이 100개가 추가된다.

하지만 우리는 100개의 커밋 과정을 모두 알고 싶지 않다. 딱 하나의 커밋으로 합치고 싶다.

그럴때는 squash라는 명령어를 사용하면 된다.

<br>

**환경세팅**

![Merge9](/Img/Merge/Merge9.png)

![Merge10](/Img/Merge/Merge10.png)

```bash
$ git checkout master
$ git merge --squash merge-v3
Automatic merge went well; stopped before committing as requested
Squash commit -- not updating HEAD
```

squash 명령어를 사용하면 자동 병합이 완료되지만 새로운 커밋을 update 해주지는 않습니다.

<br>

```bash
$ git commit -m "squash merge complete"
```

추가적으로 commit을 실행해 줘야 합니다.

<br>

**결과**

![Merge11](/Img/Merge/Merge11.png)

![Merge12](/Img/Merge/Merge12.png)

Merge-v3브랜치의 3개의 커밋이 한개로 합쳐져서 merge 된걸 볼 수 있습니다.

다만, merge —squash 는 Merge-v3 의 변경사항이 그대로 남아있습니다.

또한 커밋을 하나로 모두 줄여 깔끔하게 만들었지만, 모든 경우에 'squash'를 사용하는 것보다

팀의 동의가 있고 'squash'를 했을때 이득이 있을 경우에만 사용하는 것이 좋습니다.

<br>

반대로 커밋 로그가 남아있길 원한다면 이 방법을 사용해서는 안됩니다.

<br>


# [ 커밋 압축하기 ]

추가로 여러개의 커밋을 하나의 커밋으로 합칠 수 있습니다.

```bash
$ git rebase -i HEAD~N
```

N에는 합치고 싶은 커밋의 수를 적습니다.

HEAD~3을 할 경우 3개의 커밋을 합칠 수 있습니다.

![Merge13](/Img/Merge/Merge13.png)

work 9, work 8, work 7 세개의 커밋을 한번 합쳐보겠습니다.

```bash
$ git rebase -i HEAD~3
```

를 입력하면 이러한 창이 뜨게 됩니다.

![Merge14](/Img/Merge/Merge14.png)

rebase에는 여러가지 명령어가 있는데 저희는 squash를 사용하겠습니다.

![Merge15](/Img/Merge/Merge15.png)

pick → squash 로 변경해 줍니다. 

squash로 변경한 커밋들은 합쳐줄 것입니다.

![Merge16](/Img/Merge/Merge16.png)

다음으로 넘어가면 빨간부분에 새로 저장한 커밋 메시지를 작성하면 됩니다.

![Merge17](/Img/Merge/Merge17.png)

실제로 커밋 `work 7, work 8, work 9` 가 합쳐져 하나의 `rebase squash 3 to 1` 커밋으로 바뀐걸 확인할 수 있습니다.

또한 이 방법을 사용해 미리 커밋을 합친후 merge 를 하는 방법도 가능합니다.