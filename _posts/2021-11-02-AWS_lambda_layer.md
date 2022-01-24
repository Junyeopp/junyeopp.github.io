---
title: AWS - Lambda Layer 만들기
data: 2021-11-02 19:00:00 +09:00
categories: [AWS]
tag: [AWS]
---

Lamdba에서 numpy와 같은 외부 라이브러리를 사용하기위해서 layer를 사용합니다. 필요한 라이브러리를 패키징해서 layer로 올려놓으면 Lambda 함수에서 해당 layer를 사용해서 함수를 실행할 수 있습니다.

AWS에서 제공하는 runtime 언어별로 layer를 만드는 [방법](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html)을 참고하여 layer를 생성해보겠습니다. 저는 AmazonLinux2 기반의 Docker Image에서 Python을 기준으로 layer를 생성하였습니다.

1. zip파일 구조 확인하기

    : [AWS Guide](https://docs.aws.amazon.com/lambda/latest/dg/configuration-layers.html) 에 따르면 layer 코드를 .zip file archive로 만들어야합니다. 만약 layer가 native code binary를 포함한다면 Amazon Linux와 호환되도록 Linux machine에서 compile & build 해야합니다.

    아래와 같은 구조로 .zip file archive를 생성하면 됩니다.

    ![paths]({{ site.baseurl }}/assets/images/2021-11-02-AWS_lambda_layer_paths.png)

    ![structure]({{ site.baseurl }}/assets/images/2021-11-02-AWS_lambda_layer_structure.png)

2. pipenv를 이용해서 가상환경에 필요한 패키지 설치하기

    : pipenv install <package> 또는 .Pipfile을 이용해서 패키지를 설치해줍니다.

3. 설치된 패키지 위치 확인하기

    : `pipenv --venv` 를 이용하면 가상환경이 만들어진 위치를 확인할 수 있고 `/root/.local/share/virtualenvs/try-Hjhggp_2/lib/python3.9/site-packages` 위치에 패키지들이 설치된 것을 확인할 수 있습니다.

    ```bash
    pipenv --venv
    ->
    /root/.local/share/virtualenvs/try-Hjhggp_2
    ```

4. 설치된 패키지를 zip파일로 만들기

    : 아래 스크립트를 통해 pipenv에서 만들어진 site-packages를 새로 생성한 tmp폴더안에 구조를 지켜서 복사해서 tmp폴더를 zip해줍니다.

    ```bash
    mkdir tmp
    mkdir -p ./tmp/python/lib/python3.9
    cp -r /root/.local/share/virtualenvs/try-Hjhggp_2/lib/python3.9/site-packages ./tmp/python/lib/python3.9/site-packages

    cd tmp
    zip -r9 ../my-package.zip .
    ```

5. 끝! 이제 생성된 zip파일을 layer에 업로드하면 됩니다.
