---
draft: false
date: '2025-05-08T14:09:41+09:00'
title: '자주 사용할 것 같은 git 명령어'
summary: "평소에 자주 사용할 것 같은 git 명령어들을 정리하였다."
tags: ["post", "git"]
searchHidden: false
disableShare: false
canonicalURL: 'https://jhlee7854.github.io/posts/frequently-used-git-commands'
ShowCanonicalLink: true
CanonicalLinkText: "Originally published at"
cover:
    image: "images/Git-Logo-2Color.svg" # image path/url
    alt: "git logo" # alt text
    caption: "https://git-scm.com/downloads/logos" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false
---
## git config

git 설정을 위한 커맨드로 다음과 같이 사용자와 사용자 email을 설정할 수 있으며, 이외에 다양한 설정을 할 수 있다.

```sh
# git config user.name ${USERNAME}
# git config user.email ${USER_EMAIL}
git config user.name john
git config user.email doe@example.com
```

`--global` 옵션을 사용하여 관련 설정을 전역적인 설정으로 만들 수 있다.

```sh
git config --global user.name john
git config --global user.email doe@example.com
```

**NOTE**: 본인이 자주 사용하거나 개인적으로 사용하는 계정은 전역으로 설정하고, 회사 또는 잠시 사용할 목적의 계정은 해당 저장소 한정으로 설정하자.

## git init

로컬 저장소를 생성한다. 특정 저장소를 위한 디렉터리 생성 후 해당 디렉터리로 이동하여 다음 명령을 사용하면 해당 디렉터리는 git이 관리하는 저장소가 되며, 저장소의 메타 정보 관리를 위해 `.git` 디렉터리가 생성된다. .git 디렉터리를 제거하면 해당 디렉터리는 git이 관리하지 않는 일반 디렉터리가 되고, 원격 저장소에 반영되지 않은 커밋 및 브랜치와 같은 메타 정보는 모두 없어지게 된다.

```sh
git init
```

## git remote

등록된 원격 저장소 목록을 확인한다. `-v` 옵션을 사용하면 원격 저장소 이름과 함께 원격 저장소의 주소 정보도 함께 출력한다.

```sh
git remote
```

```sh
git remote -v
```

`add` 서브 커맨드는 원격 저장소를 새롭게 등록한다.

```sh
# git remote add ${NAME} ${REMOTE_REPOSITORY}
git remote add origin git@github.com:john/hello.git
```

`rename` 서브 커맨드를 사용하면 등록된 원격 저장소의 이름을 변경할 수 있다.

```sh
# git remote rename ${OLD_NAME} ${NEW_NAME}
git remote rename origin myrepo
```

`remove` 서브 커맨드는 등록된 원격 저장소를 제거한다.

```sh
git remote remove origin
```

```sh
git remote rm origin
```


## git clone

원격 저장소를 로컬로 복제한다. 명령어를 실행하는 디렉터리 하위에 저장소 이름과 동일한 디렉터리를 만들고 해당 디렉터리에 원격 저장소의 내용을 복제한다.

```sh
# git clone ${REMOTE_REPOSITORY}
git clone git@github.com:john/hello.git
```

다음과 같이 별도의 이름을 지정하면 원격 저장소의 이름이 아닌 지정된 이름으로 디렉터리를 만들고 해당 디렉터리에 원격 저장소의 내용을 복사한다.

```sh
# git clone ${REMOTE_REPOSITORY} ${REPOSITORY_NAME}
git clone git@github.com:john/hello.git hello-project
```

## git branch

로컬 브랜치 정보를 확인한다. 출력의 `*` 표시는 현재 바라보고 있는 branch를 나타낸다.

```sh
git branch
```

원격 브랜치 정보를 확인한다.

```sh
git branch -r
```

원격 브랜치를 포함한 모든 브랜치 정보를 확인한다.

```sh
git branch -a
```

지정한 이름의 브랜치를 생성한다.

```sh
# git branch ${BRANCH_NAME}
git branch develop
```

지정한 브랜치를 삭제한다.

```sh
# git branch -d ${BRANCH_NAME}
git branch -d develop
```

**NOTE**: `-r` 옵션과 함께 `-d` 옵션을 사용하면 원격 브랜치를 삭제할 수 있다.

## git checkout

작업 브랜치를 변경한다.

```sh
# git checkout ${BRANCH_NAME}
git checkout develop
```

다음과 같이 `-b` 옵션을 사용하면 지정한 이름의 브랜치를 생성한 후 해당 브랜치로 작업 브랜치를 변경한다.

```sh
# git checkout ${NEW_BRNACH_NAME}
git checkout -b develop
```

## git status

현재 working tree의 파일 상태를 확인한다.

```sh
git status
```

## git add

커밋에 포함될 파일을 등록한다. 공백 구분자로 한번에 여러 파일을 등록할 수 있다.

```sh
# git add ${FILE_PATH}
git add .gitignore
```

특정 디렉터리를 지정하면 해당 디렉터리 하위의 모든 파일들이 등록된다.

```sh
git add .
```

## git commit

변경사항들을 묶은 새로운 커밋을 생성한다. git 설정에 지정된 editor를 통해 커밋 메시지를 작성하고 저장하면 새로운 커밋이 생성된다.

```sh
git commit
```

다음과 같이 `-m` 옵션을 사용하여 커맨드 라인 상에서 바로 커밋 메시지를 작성할 수 있다.

```sh
git commit -m "Initial commit"
```

변경사항들을 커밋할 때 설정된 저자 정보 대신 다른 저자 정보를 사용하고자 한다면 다음과 같이 `--author` 옵션을 사용한다.

```sh
# git commit --author "${USERNAME} <${USER_EMAIL}>"
git commit --author "John Doe <doe@example.com>"
```

```sh
git commit --author "John Doe <doe@example.com>" -m "Initial commit"
```

방금 작성한 커밋 메시지를 수정하고 싶을 경우 `--amend` 옵션을 사용한다. 지정된 에디터 또는 커맨드 라인에서 수정된 메시지를 이용해 새로운 커밋을 생성하고 바로 이전의 커밋을 대체한다.

```sh
git commit --amend
```

```sh
git commit --amend -m "First commit"
```

방금 작성한 커밋의 저자 정보를 수정하고 싶을 경우 --amend 옵션과 함께 `--author` 옵션을 사용한다.

```sh
# git commit --amend --author "${USERNAME} <${USER_EMAIL}>"
git commit --amend --author "John Doe <doe@example.com>"
```

## git reset

현재 커밋을 가리키는 `HEAD`를 이전 커밋을 가리키도록 변경한다. 즉, 지금까지의 변경사항은 없던 내용이 되고 지정한 이전 커밋으로 되돌아 간다. 기존에 추적하던 변경사항은 워킹트리에 그대로 남게 되며 `Untracked` 또는 `Modified` 상태가 된다.

```sh
# git reset ${COMMIT_HASH}
git retset 6531a0764fd4f39761f8f0f404e90a70f19efae7
```

다음과 같이 HEAD를 기준으로 되돌아갈 커밋을 지정할 수 있다.

```sh
# git reset HEAD
# git reset HEAD~${NUMBER}
git reset HEAD # 바로 이전 커밋으로 되돌아 간다.
git reset HEAD~2 # 이전 2번째 커밋으로 되돌아 간다.
```

**NOTE**: 다음과 같이 옵션을 사용하여 되돌아가는 시점의 상태를 지정할 수 있다.

```sh
git reset --soft HEAD~3 # 커밋 이력은 모두 없어지지만 변경 내역은 stagign area에 남아 있다.
git reset --mixed HEAD~3 # default 모드 이며 커밋 이력은 모두 없어지지만 변경 내역은 Untracked 또는 Modified 상태로 워킹트리에 남아 있다.
git reset --hard HEAD~3 # 커밋 이력과 함께 변경 내역도 모두 사라진다.
```

**NOTE**: 특정 커밋 이후의 내역을 없던 일로 되돌리는 것이기 때문에 reset을 이용하여 이미 원격지에 반영된 커밋 내역 이전으로 되돌아가는 것은 협업에 혼란을 불러온다.

## git revert

특정 커밋 이력을 제거하고 제거한 내역을 커밋한다. 특정 커밋이력은 제거 되지만 이후 커밋한 내역은 계속 보존되고, 특정 커밋의 내역을 제거했다는 내역이 신규 커밋으로 추가된다.

```sh
# git revert ${COMMIT_HASH}
git revert 6531a0764fd4f39761f8f0f404e90a70f19efae7
# 위 명령어가 수행되면 커밋 메시지를 입력할 수 있는 editor가 열린다.
```

다음과 같이 `--no-edit` 옵션을 사용하여 별도의 커밋 메시지 입력을 생략할 수 있다.

```sh
git revert 6531a0764fd4f39761f8f0f404e90a70f19efae7 --no-edit
```

커밋 해시 부분을 HEAD를 기준으로 사용할 수 있다.

```sh
# git revert HEAD
# git revert HEAD~${NUMBER}
git revert HEAD # 바로 이전 커밋을 제거한다.
git revert HEAD~2 # 이전 2번째 커밋을 제거한다.
```

## git cherry-pick

다른 브랜치의 특정 커밋을 현재 작업 브랜치에 적용한다.

```sh
# git cherry-pick ${COMMIT_HASH}
git cherry-pick aafd6a004a1ddc803f2abfe95e562823496ec3e7
```

커밋 해시를 열거하여 여러개의 커밋을 한번에 적용할 수 있다.

**NOTE**: cherry-pick을 사용하여 다른 commit을 가져올 때 conflict가 발생한다면 충돌을 해결하여 저장하고 `git cherry-pick --continue`를 실행하여 cherry-pick을 완료할 수 있다. 만약 수정없이 cherry-pick을 취소하고 싶다면 `git cherry-pick --abort`를 실행한다.

## git rebase

다른 브랜치의 변경 내역을 가져와 현재 브랜치의 이력을 재정렬 한다. 변경 내역을 가져오려는 브랜치의 커밋 내역이 현재 작업 브랜치에 업데이트 및 재정렬 되므로, 작업이 끝난 후 변경 내역을 가져온 브랜치에서 현재 작업 브랜치를 병합하면 3-way merge 방식이 아닌 Fast-forward merge방식이 되어 커밋 히스토리가 깔끔해 진다.

```sh
# git rebase ${BRANCH_NAME}
git rebase feature # 현재 브랜치의 변경 내역에 feature 브랜치의 변경 내역이 반영되어 현재 브랜치의 변경 내역이 재정렬 된다.
```

**NOTE**: rebase는 커밋의 이력을 재정렬 하는 작업이므로 원격 저장소에 누군가 이미 push한 commit 이력을 재정렬할 경우 매우 큰 혼란을 초래할 수 있다.

## git log

작업 브랜치의 커밋 로그들을 확인한다.

```sh
git log
```

다음과 같은 옵션을 통해 간소화 해서 볼 수 있다.

```sh
git log --pretty=oneline
```

`--graph` 옵션을 사용하여 커밋 내역들 사이의 관계를 시각적으로 확인할 수 있다.

```sh
git log --graph
```

```sh
git log --pretty=oneline --graph
```

다음과 같이 포맷을 사용하여 원하는 형태로 커밋 내역을 볼 수 있다.

```sh
git log --pretty='format:%C(auto)%h %d %s %Cblue%an %Cgreen%ah' --graph
```

## git merge

다른 브랜치의 커밋들을 현재 작업 브랜치에 병합한다.

```sh
# git merge ${BRANCH_NAME_TO_MERGE}
git merge develop
```

## git fetch

원격 저장소의 변경 내역을 로컬 저장소로 가져온다.

```sh
# git fetch ${REMOTE_NAME} ${REMOTE_BRANCH}
git fetch origin main
```

**NOTE**: fetch 커맨드는 원격 저장소의 변경 내역을 가져오기만 한다. `FETCH_HEAD`는 가져온 변경 내역을 가리키고 있으며 이를 이용하여 다음과 같이 원격 저장소의 변경 내역을 확인 및 병합할 수 있다.

```sh
git log FETCH_HEAD # 변경 내역에 대한 로그를 확인한다.
git diff ...FETCH_HEAD # 변경 내역을 비교한다.
git merge FETCH_HEAD # 변경 내역을 병합한다.
```

## git pull

원격 저장소의 변경 내역을 로컬 저장소로 가져오고 병합한다.

```sh
# git pull ${REMOTE_NAME} ${REMOTE_BRANCH}
git pull origin main
```

## git push

로컬 저장소의 변경 내역을 원격 저장소에 반영한다.

```sh
# git push ${REMOTE_NAME} ${REMOTE_BRANCH}
git push origin main
```

## References

- [git-scm.com](https://git-scm.com/docs)
