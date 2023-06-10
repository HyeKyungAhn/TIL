# Git Command

## commit 취소를 commit 취소해야할 때

        git reset HEAD@{1} or git reset [키값] 

</br>

## 마지막 커밋을 취소하는 방법

        git reset --soft HEAD~1 
        //commit 이전으로 돌아감. 변경사항이 손실되지 않고 staged 상태로 워킹 디랙토리에 남아있음

        git reset --mixed HEAD^
        //기본옵션. commit이전으로 돌아감. 변경사항이 unstage 상태로 남아있음
        git reset HEAD^
        //위와 동일
        git reset HEAD~2
        //마지막 commit 2개를 취소 unstages상태로 남아있음

        git reset --hard HEAD~1
        //commit 취소, 변경사항들은 unstaged 상태로 디렉터리에서 삭제됨
[참조](https://gmlwjd9405.github.io/2018/05/25/git-add-cancle.html)

</br>

## .gitignore에 추가한 파일이 계속 unstaged list에 등장할 때

캐시 삭제 전, 현재 변경 사항이자 commit해야할 변화들은 반드시 stash 또는 commit을 하자. 아니면 삭제된다. 

        git rm -r --cached .
        git add .
        git commit -m "fixed untracked files"

프로젝트 초반에 .gitignore파일 잘 추가하기. 미래의 나야 잘하자.

[참조](https://stackoverflow.com/questions/11451535/gitignore-is-ignored-by-git)

## merge를 취소하기

merge전의 상태로 돌아가려면, 

1 . git log 또는 git reflog를 실행하여 돌아가려는 merge 직전의 commit의 hash를 얻는다.

2-1. 

    git reset --hard commit-hash-before-the-merge
       
- --hard는 현재 커밋되지 않은 변경사항을 제거한다.

</br>   
2-2. 

    git reset --merge commit-hash-before-the-merge

-  --merge는 커밋되지 않은 변경사항을 유지한다.

#### 참고 1