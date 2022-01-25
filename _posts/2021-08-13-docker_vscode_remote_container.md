---
title: Docker + VSCode Remote Container로 개발환경 설정하기
data: 2021-08-13 19:00:00 +09:00
categories: [Tools]
tag: [Docker]
---
다음과 같은 장점들을 기대하며 Docker를 사용해서 개발환경을 설정해보았습니다.

- 개발하는 코드가 서비스될 환경과 비슷한 환경에서 개발할 수 있다.
- 여러 사람이 동일한 환경에서 개발할 수 있다.
- 쉽게 수정하고 재구성할 수 있다.

아래의 필요한 조건들에 맞게 구성하였습니다.

- OS: AmazonLinux2
- 설치할 항목:
    - Python3.8 + 라이브러리들
    - AWS CLI, serverless
    - VSCode, JupyterLab
- 로컬환경의 Git working directory를 마운트
- 로컬환경의 `~/.ssh`, `~/.aws`에 있는 설정을 사용하기


# 1. Docker Image 만들기

Docker file을 이용해서 amazonlinux_dev라는 이름의 Image를 생성해줍니다.

`docker build --tag=amazonlinux_dev .`

- Dockerfile

    ```yaml
    FROM amazonlinux
    ENV LANG=en_US.UTF-8

    # Update yum and install packages
    RUN yum update -y && yum install -y zip gzip unzip which tar vim git tar wget openssl util-linux-user shadow-utils sudo

    # Install awscli2
    RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" \
        && unzip awscliv2.zip \
        && ./aws/install

    # Install python3.8
    RUN amazon-linux-extras install -y python3.8

    # Install fish shell
    RUN cd /etc/yum.repos.d/ \
        && wget https://download.opensuse.org/repositories/shells:fish/CentOS_7/shells:fish.repo \
        && yum install -y fish \
        && chsh -s /usr/bin/fish

    # Set git
    RUN git config --global core.autocrlf input

    # Set fish
    SHELL ["/usr/bin/fish", "--command"]
    RUN curl https://raw.githubusercontent.com/oh-my-fish/oh-my-fish/master/bin/install > omf-install \
        && fish omf-install -y --noninteractive \
        && mkdir -p ~/.config/fish/functions/ \
        && touch ~/.config/fish/functions/fish_right_prompt.fish \
        && printf "function fish_right_prompt -d \"Write out the right prompt\"\n    date '+%%m-%%d %%H:%%M:%%S'\nend\n" > ~/.config/fish/functions/fish_right_prompt.fish \
        && source ~/.config/fish/conf.d/omf.fish

    # Install Node.js & npm & serverless
    RUN curl -fsSL https://rpm.nodesource.com/setup_lts.x | bash - \
        && yum install -y nodejs \
        && npm install -g serverless

    # Install pipenv
    WORKDIR /app/
    COPY Pipfile Pipfile.lock .env /app/
    RUN python3.8 -m pip install --no-cache-dir --upgrade pip pipenv \
        && pipenv install --deploy --ignore-pipfile --dev \
        && pipenv --clear

    COPY ./set_ssh_and_aws.sh ./set_ssh_and_aws.sh
    ```

    1. amazonlinux를 기반으로 docker image 생성
    2. yum을 통해 필요한 package들을 설치
    3. awscli2 설치
    4. amazon-linux-extras를 통해 제공되는 python3.8을 설치
    5. CentOS_7를 위해 제공되는 방법으로 fish shell 설치(amazon linux 2는 centos7, amazon linux 1은 centos6 으로), oh-my-fish 설치
    6. git autocrlf 설정
    7. oh-my-fish 설치 및 설정
        - shell의 라인 오른쪽에 시간을 표시하기 위해 fish function을 추가 생성
    8. Node.js, npm, serverless 설치
    9. pipenv 설치 및 /app/ 위치에 pipenv install
        - Pipfile을 이용해 미리 생성된 Pipfile.lock을 사용
        - Pipenv를 실행할 때 사용할 환경변수를 저장한 .env를 사용
    10. ssh, aws 관련 파일을 복사하는 script를 복사

# 2. VSCode에서 실행할 docker-compose.yaml 만들기

생성한 amazonlinux_dev 이미지를 기반으로 docker-compose yaml을 작성합니다.

- docker-compose.yaml

    ```yaml
    version: "3.8"

    services:
      analyzer:
        image: amazonlinux_dev:latest
        entrypoint: ["/bin/sh", "-c"]
        command: ["/app/set_ssh_and_aws.sh && pipenv run python3.8 -m jupyter lab --ip=0.0.0.0 --port=8888 --allow-root --no-browser --NotebookApp.password='sha1:d99ebe6398a3:5fbfb3f2133200fad796640c88e5ff5c3d11d9d8'"]
        ports:
          - 8899:8888
        logging:
          driver: json-file  #journald
        restart: always
        volumes:
          - repos:/app/repos
          - C://Users//USER//.ssh:/app/ssh_local:ro
          - C://Users//USER//.aws:/app/aws_local:ro

    volumes:
      repos:
        name: repos
        driver: local
        driver_opts:
          type: none
          o: bind
          device: "C://Users//USER//Documents//mount_dir"
    ```

    : script를 통해 SSH, AWS CLI 설정파일을 복사하고, pipenv를 통해 jupyter lab을 8899포트에서 열도록 하였습니다.

    - --NotebookApp.password는 Jupyter에서 sha1 algorithm으로 생성
        
        ```python
        from notebook.auth import passwd
        passwd(algorithm="sha1")
        -> Enter password: amazon_dev ->
        return: 'sha1:d99ebe6398a3:5fbfb3f2133200fad796640c88e5ff5c3d11d9d8'
        ```

    - --ip 0.0.0.0

        : 모든 IP에  대해 접근을 허용

    - volume.repos.driver_opts.device

        : 마운트할 directory의 경로를 입력

    - ~~secrets~~

        ~~: ssh_config, aws_config, aws_credential 정보를 secrets를 통해 container 내부로 전달~~

- ssh, aws 관련 설정
    - ssh key는 local SSH agent를 통해 공유할 수 있습니다.

        [https://code.visualstudio.com/docs/remote/containers#_using-ssh-keys](https://code.visualstudio.com/docs/remote/containers#_using-ssh-keys)

    - .ssh/config파일은 별도로 생성해 주어야 해서 script를 작성하였습니다.
        - `set_ssh_and_aws.sh`

            ```bash
            #!/bin/bash

            sudo cp -R /app/ssh_local/. ~/.ssh/
            sudo cp -R /app/aws_local/. ~/.aws/

            sudo chmod 0700 ~/.ssh
            sudo chmod 0600 ~/.ssh/*
            sudo find ~/.ssh -name "*.pub" -exec chmod 0644 {} +

            sudo chmod 0700 ~/.aws
            sudo chmod 0600 ~/.aws/*
            ```


# 3. VSCode를 설정해줄 .devcontainer.json 만들기

VSCode를 container에 연결할 때 "Remote - Containers" Extension을 사용합니다.

Container를 연결하는 방법 중 두 가지를 소개하겠습니다.

1. `>Remote-Containers: Open Folder in Container...` 를 통해 .devcontainer.json파일이 있는 폴더를 선택합니다.

    : .devcontainer.json파일을 통해 VSCode를 설정하고 container를 만들어 실행시킬 수 있습니다. [Document](https://code.visualstudio.com/docs/remote/devcontainerjson-reference)를 참조하여 extension들과 사용할 docker-compose파일, work folder, phthon path 등을 설정하였습니다.
    
    ```json
    {
    	"name": "amazon_dev-container",
    	"dockerComposeFile": ["./docker-compose.yaml"],
    	"service": "analyzer",
    	"shutdownAction":"none",
    	"workspaceFolder": "/app/repos",
    	"settings":  {
    		"python.pythonPath": "/usr/local/bin/python"
    	},
    	"extensions": [
    		"ms-python.python",
    		"ms-python.vscode-pylance",
    		"ms-toolsai.jupyter",
    		"mhutchie.git-graph",
    		"eamodio.gitlens"
    	]
    }
    ```

2. `>Remote-Containers: Attach to Running Container...`를 통해 이미 실행 중인 container에 연결합니다.

    : 이미 실행되고 있는 container가 있을 때 attatch하여 사용할 수 있습니다.


# 추가

- 개선하기

    현재 로컬환경의 OS는 Windows이고 WSL2 backend로 Docker가 실행되고 있습니다. Docker container의 메모리 점유율이 높아지면 컴퓨터 사용이 불편해져서, WSL에서 사용할 수 있는 컴퓨터 자원을 제한하였습니다.

    %UserProfile% directory에 있는 [.wslconfig](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#configure-global-options-with-wslconfig) 파일의 내용을 수정해주면 되며, 설치된 모든 WSL2 버전의 Linux distributions에 적용됩니다.

    - .wslconfig

        ```yaml
        [wsl2]
        memory=6GB # Limits VM memory in WSL 2 to 6 GB
        processors=6 # Makes the WSL 2 VM use six virtual processors
        ```
