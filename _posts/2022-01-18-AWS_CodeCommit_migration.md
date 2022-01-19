---
title: AWS CodeCommit으로 Git repo 옮기기
data: 2022-01-19 21:00:00 +09:00
categories: [AWS]
tag: [dev]
---

기존에 사용하던 Git repository를 AWS CodeCommit으로 옮기고 credential helper를 이용해서 remote에 연결하는 과정을 정리하였습니다.

# 1. Git repository를 AWS CodeCommit으로 옮기기

: [AWS Documentation](https://docs.aws.amazon.com/codecommit/latest/userguide/how-to-migrate-repository-existing.html)을 따라가며 작성하였습니다.

1. AWS User에게 적절한 권한을 준 뒤, CodeCommit을 위한 HTTPS Git credentials을 생성합니다.
2. CodeCommit 콘솔에서 새로운 repository를 만들어줍니다.
3. aws-codecommit-demo이라는 폴더에 bare repo를 생성합니다.
    
    `git clone --mirror ssh://git_repo.git aws-codecommit-demo` 
    
4. bare repo 폴더로 이동하여 CodeCommit 콘솔에서 확인한 URL로 bare repo를 push해줍니다.
    
    `cd aws-codecommit-demo`
    
    `git push https://git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/<new_repo> --all`
    
    `git push https://git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/<new_repo> --tags`
    
    - `--all` 옵션을 통해 모든 브랜치를 push할 수 있습니다. 하지만 tags는 push되지 않기 때문에 `--tags` 옵션을 통해 한 번 더 push해줍니다.
5. 위에서 생성한 aws-codecommit-demo이라는 폴더는 더 이상 필요하지 않기에 삭제해줍니다.
6. 끝!


# 2. Git credential helper 설정하기

credential helper는 HTTP 프로토콜을 사용해서 remote repo에 접근할 때 매번 인증을 거치지 않게 해주는 시스템입니다. git에서는 기본적으로 cache, store, wincred와 같은 옵션을 제공하고 자세한 내용은 [여기](https://git-scm.com/book/ko/v2/Git-%EB%8F%84%EA%B5%AC-Credential-%EC%A0%80%EC%9E%A5%EC%86%8C)서 확인할 수 있습니다.

AWS CLI에서는 HTTPS를 통한 CodeCommit 접근을 위해 CLI config의 정보를 이용할 수 있게 해주는 credential helper를 제공합니다. helper를 설정하는 방법은 [document](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-https-windows.html)에 제시되어있습니다.

저는 기본값으로 설정시 403에러가 발생해서 아래와 같이 .gitconfig파일을 직접 수정해주었습니다.

```
[credential "https://git-codecommit.ap-northeast-2.amazonaws.com"]
	helper = !aws codecommit credential-helper --profile carvi $@
	UseHttpPath = true

[credential]
	helper = wincred
```


# 3. 새로 생성한 CodeCommit repo를 remote로 추가하기

1. upstream추가에 앞서 지금 등록된 remote 목록을 확인해봅니다.
    
    `git remote -v`
    
2. CodeCommit repo 주소를 remote에 “upstream”이름으로 등록합니다.
    
    `git remote add upstream https://git-codecommit.ap-northeast-2.amazonaws.com/v1/repos/<new_repo>`
    
3. 끝!