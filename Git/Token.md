갑자기 git push를 하는데

<br>

![Token1](/Img/Token/Token1.png)


위와같은 **Support for password authentication was removed on August 13, 2021. Please use a personal access token instead.** 에러가 발생하면서 push가 되질 않았다.

<br>

git이 알려주는 https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/  링크를 들어가보니 더이상 password 인증은 허용하지 않고 token을 통한 인증을 사용하라는 것이었다.

<br>

이걸 해결하기 위해 많이 찾아보았는데 해결 방법을 공유하겠습니다.

참고로, 저는 Window환경에서 작업했으니 Mac에서 하시는 분들은 참고만 하시면 좋을거 같습니다.

<br>

# [ Token 생성 ]
먼저 token 인증을 하기 위해서는 personal access token 을 만들어야 합니다.

<br>

![Token2](/Img/Token/Token2.png)

github에 들어가 Setting에 들어갑니다.

<br>

![Token3](/Img/Token/Token3.png) 

Developer settings 로 들어가 줍니다.

<br>

![Token4](/Img/Token/Token4.png)

Personal access tokens 메뉴로 들어가면 저는 이미 토큰이 생성되어 있지만, 하나도 없으신 분도 있으실 겁니다.

이미 있으신분들은 기존에 있는걸 사용하셔도 가능하지만, 토큰 Key를 까먹으셨다면 새로 생성하시는걸 추천드립니다. Generate new token을 눌러 토큰을 새로 생성해 줍니다.

<br>

![Token5](/Img/Token/Token5.png)

github 비밀번호를 입력해서 들어오면 위와같은 화면이 뜨게 됩니다.

<br>

Note - Token의 이름을 정해줍니다. 저는 "CodePush"라 하겠습니다.

Expiration - 토큰 유효기간을 설정해줍니다. 개인의 사용목적에 맞게 설정하시면 됩니다.

Select scopes - 토큰 접근 범위를 설정해줍니다. 기본적인 레포지토리 관리를 사용하실거라면 repo, read:org, gist 는 체크해주시는것이 좋습니다. 나머지는 사용목적에 맞게 체크하여 사용하시면 되고 좀 더 자세한 정보가 궁금하신 분들은 여기에서 정보를 찾아 보시면 도움을 얻을 수 있을거 같습니다.

<br>

![Token6](/Img/Token/Token6.png)

마지막으로 Generate Token을 하시면 토큰이 생성됩니다. 빨간 네모 부분이 토큰 키 입니다.

> **토큰키는 생성한 직후인 지금 화면에서만 확인하실수 있고, 이후로는 볼 수 없기 때문에 지금 다른 안전한 곳에 저장해 두시길 추천드립니다. 나중에 필요하니 꼭 까먹지 말고 저장해 두세요!**
 
<br>

# [ 임시 해결 방법 ]
이제 토큰을 생성했습니다. 그리고 다시 아까 오류화면으로 돌아가 보겠습니다.

<br>

![Token1](/Img/Token/Token1.png)

위의 오류가 발생했고, 토큰을 생성했다면 현재 연결되어 있는 원격 저장소를 삭제 한뒤, 다시 토큰 인증 방식으로 저장소를 연결하면 됩니다.

```Bash
$ git remote remove origin
$ git remote add origin https://<USERNAME>:<TOKEN>@<GIT_URL>.git
$ git pull
```

만약 git pull을 하는데 아래와 같은 메시지가 발생한다면

![Token7](/Img/Token/Token7.png)

```Bash
$ git branch --set-upstream-to=origin/master master
이와 같이 해결하면 됩니다.
```

<br>

추가로 만약 저장소를 토큰 인증을 통해 복제할거라면

```Bash
$ git clone https://<USERNAME>:<TOKEN>@<GIT_URL>.git
```
위 방식을 사용하면 됩니다.

<br>

# [ 자격 증명 수정 ]
근데 위의 방법을 사용하면 모든 레포지토리를 각각 변경해 줘야 합니다.

그건 너무 귀찮은 일이기 때문에 인증 정보를 캐시하는 방법이 있습니다.

macOS 나 Linux는 crediential Cache를 사용해 문제를 해결 할 수 있습니다. 

<br>

(macOS 나 Linux 분들은 
[여기](https://dolsup.work/tech-blog/different-ways-to-access-github-repository/)
에서 crediential 부분을 참고해보세요!)
 
<br>

저는 Window 환경에서 해결 방법을 알려 드리겠습니다.

<br>

![Token8](/Img/Token/Token8.png)

먼저 설정에서 자격 증명 관리자로 들어갑니다.

<br>

![Token9](/Img/Token/Token9.png)

Windows 자격 증명을 선택하고 일반 자격 증명 에서 git 자격 증명을 찾습니다.

<br>

![Token10](/Img/Token/Token10.png)

편집을 눌러 들어갑니다.

<br>

![Token11](/Img/Token/Token11.png)

암호를 우리가 생성한 Token으로 변경해 줍니다.

그러면 기존과 같이 git push가 정상 작동하는 것을 확인 할 수 있을겁니다.

> 추가로, Token 인증대신 SSH를 이용하셔도 됩니다.