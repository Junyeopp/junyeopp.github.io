---
title: Git - 알아두면 좋을 것들
data: 2021-06-10 19:00:00 +09:00
categories: [Tools]
tag: [Git]
---
# bare repository

### [bare repo](https://git-scm.com/book/en/v2/Git-on-the-Server-Getting-Git-on-a-Server)란?

working directory를 가지지 않는 repository입니다. remote origin을 가지지 않으며 자신이 remote origin 역할을 수행합니다. bare repository의 이름은 `.git`으로 끝나도록 짓는 것이 일반적입니다.

- 만약 non-bare repository로 push를 시도한다면?

    → 아래와 같은 에러가 발생합니다.

    ```
    remote: error: By default, updating the current branch in a non-bare repository is denied, because it will make the index and work tree inconsistent with what you pushed, and will require 'git reset --hard' to match the work tree to HEAD.
    ```


### bare repo 만들기

- bare repository 만들기

    ```bash
    git init --bare new_repo_name.git
    ```

- bare repository로 clone하기

    ```bash
    git clone --bare my_project my_project.git
    ```


# SSH를 이용해 remote 공유하기

SSH를 통해 공유할 수 있는 [방법](https://git-scm.com/book/en/v2/Git-on-the-Server-Getting-Git-on-a-Server) 두 가지을 알아보겠습니다.

1. 사용자들의 개별 계정을 서버에 생성하고 OS의 권한설정을 통해 접근제어하기. 사용자들은 개인 계정으로 git repo에 접근해서 사용할 수 있습니다.
2. 하나의 git 계정을 생성하고 사용자별로 SSH public key를 받아 `~/.ssh/authorized_keys`에 추가하기. 사용자들은 git 계정을 통해서 git repo에 접근해서 사용할 수 있습니다.

    하지만, 그냥 public key만 등록해서 git 계정으로 접속하면 git 명령어외에도 다른 작업들도 할 수 있습니다. 이를 제한하기 위해 추가적인 설정을 하였습니다.

    1. 방법1. [forced command](http://man.openbsd.org/OpenBSD-current/man5/sshd_config.5#ForceCommand)로 명령어 제한하기

        `~/.ssh/authorized_keys`에 public key를 아래처럼 등록하면 추가적인 기능을 제공합니다.

        ```bash
        command="git-shell -c \"$SSH_ORIGINAL_COMMAND\"",no-port-forwarding,no-agent-forwarding,no-X11-forwarding,no-pty ssh-rsa <ssh public key> <comment>
        ```

        - `$SSH_ORIGINAL_COMMAND`값에 입력한 명령어가 들어갑니다.
        - no-port-forwarding,no-agent-forwarding,no-X11-forwarding,no-pty 옵션을 통해 포트포워딩을 막을 수 있고 자세한 설명은 [이곳](https://man.openbsd.org/OpenBSD-current/man8/sshd.8#no-agent-forwarding)에서 찾아볼 수 있습니다.
    2. 방법2. login shell을 [git-shell](https://git-scm.com/docs/git-shell)로 변경하기
        
        git-shell은 git과 관련된 명령어만 사용할 수 있도록 제한된 쉘입니다. 다음과 같이 설정할 수 있습니다.

        ```bash
        cat /etc/shells   # see if git-shell is already in there. If not...
        sudo sh -c "echo $(which git-shell) >> /etc/shells"  # and add the path

        sudo chsh -s $(command -v git-shell) <user>  # change shell
        ```

        이렇게 변경하면 `su - git`으로 git 계정으로 전환할 때도 git-shell을 기본적으로 사용해서 `su --shell /bin/bash - git` 등과 같이 shell을 변경해서 전환해야 합니다.


# Autocrlf 설정하기

- **`core.autocrlf`**

    : OS에 따라 다른 줄바꿈 문자 문제를 위해 아래와 같이 [설정](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)하여 사용합니다.

    이렇게 하면 Windows에서는 CRLF를 사용하고 Mac, Linux, 저장소에서는 LF를 사용할 수 있습니다.

    - Windows

        `git config --global core.autocrlf true`

    - Linux, Mac

        `git config --global core.autocrlf input`


# merge할 때 fast-forward 끄기

[git merge](https://git-scm.com/docs/git-merge)의 fast-forword관련 기본 옵션은 `--ff` 로 merge commit을 생성하지 않고 branch pointer만 움직입니다.

저는 필요한 경우가 아니라면 기록을 남기는 것을 선호해서 `--no-ff` 옵션을 통해 항상 merge commit을 생성하도록 하고 있습니다.

# 참고자료

- [https://git-scm.com/book/en/v2/Git-on-the-Server-Getting-Git-on-a-Server](https://git-scm.com/book/en/v2/Git-on-the-Server-Getting-Git-on-a-Server)
- [https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration](https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration)