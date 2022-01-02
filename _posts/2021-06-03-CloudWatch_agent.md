---
title: CloudWatch agent
data: 2021-06-03 15:00:00 +09:00
categories: [AWS]
tag: [AWS, CloudWatch]
---

- EC2의 memory 사용량을 모니터링 하기 위해서 CloudWatch agent를 사용
- docs: [https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html](https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)

1. `CloudWatchAgentServerPolicy`권한을 가지는 Role를 생성
2. 생성한 Role을 agent를 설치하고자하는 EC2에 연결

    EC2 → Security → Modify IAM role → `CloudWatchAgentServerPolicy`을 가진 Role 선택

3. `wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb` : OS가 Ubuntu일 경우 해당 download-link를 사용
4. 위에서 DEB 패키지를 다운로드한 경우 패키지가 있는 디렉터리에서 다음을 실행

    `sudo dpkg -i -E ./amazon-cloudwatch-agent.deb`

5. EC2의 아웃바운드 인터넷 액세스 권한 확인

    → 현재 security group에서 아웃바운드 모두 허용 상태

6. Create the CloudWatch agent configuration file with the agent

    run `sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard`

    ```bash
    =============================================================
    = Welcome to the AWS CloudWatch Agent Configuration Manager =
    =============================================================
    On which OS are you planning to use the agent?
    1. linux
    2. windows
    3. darwin
    default choice: [1]:
    1
    Trying to fetch the default region based on ec2 metadata...
    Are you using EC2 or On-Premises hosts?
    1. EC2
    2. On-Premises
    default choice: [1]:
    1
    Which user are you planning to run the agent?
    1. root
    2. cwagent
    3. others
    default choice: [1]:
    1
    Do you want to turn on StatsD daemon?
    1. yes
    2. no
    default choice: [1]:
    2
    Do you want to monitor metrics from CollectD?
    1. yes
    2. no
    default choice: [1]:
    2
    Do you want to monitor any host metrics? e.g. CPU, memory, etc.
    1. yes
    2. no
    default choice: [1]:
    1
    Do you want to monitor cpu metrics per core? Additional CloudWatch charges may apply.
    1. yes
    2. no
    default choice: [1]:
    2
    Do you want to add ec2 dimensions (ImageId, InstanceId, InstanceType, AutoScalingGroupName) into all of your metrics if the info is available?
    1. yes
    2. no
    default choice: [1]:
    2
    Would you like to collect your metrics at high resolution (sub-minute resolution)? This enables sub-minute resolution for all metrics, but you can customize for specific metrics in the output json file.
    1. 1s
    2. 10s
    3. 30s
    4. 60s
    default choice: [4]:
    4
    Which default metrics config do you want?
    1. Basic
    2. Standard
    3. Advanced
    4. None
    default choice: [1]:
    1
    Current config as follows:
    {
            "agent": {
                    "metrics_collection_interval": 60,
                    "run_as_user": "root"
            },
            "metrics": {
                    "metrics_collected": {
                            "disk": {
                                    "measurement": [
                                            "used_percent"
                                    ],
                                    "metrics_collection_interval": 60,
                                    "resources": [
                                            "*"
                                    ]
                            },
                            "mem": {
                                    "measurement": [
                                            "mem_used_percent"
                                    ],
                                    "metrics_collection_interval": 60
                            }
                    }
            }
    }
    Are you satisfied with the above config? Note: it can be manually customized after the wizard completes to add additional items.
    1. yes
    2. no
    default choice: [1]:
    1
    Do you have any existing CloudWatch Log Agent (http://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/AgentReference.html) configuration file to import for migration?
    1. yes
    2. no
    default choice: [2]:
    2
    Do you want to monitor any log files?
    1. yes
    2. no
    default choice: [1]:
    2
    Saved config file to /opt/aws/amazon-cloudwatch-agent/bin/config.json successfully.
    Current config as follows:
    {
            "agent": {
                    "metrics_collection_interval": 60,
                    "run_as_user": "root"
            },
            "metrics": {
                    "metrics_collected": {
                            "disk": {
                                    "measurement": [
                                            "used_percent"
                                    ],
                                    "metrics_collection_interval": 60,
                                    "resources": [
                                            "*"
                                    ]
                            },
                            "mem": {
                                    "measurement": [
                                            "mem_used_percent"
                                    ],
                                    "metrics_collection_interval": 60
                            }
                    }
            }
    }
    Please check the above content of the config.
    The config file is also located at /opt/aws/amazon-cloudwatch-agent/bin/config.json.
    Edit it manually if needed.
    Do you want to store the config in the SSM parameter store?
    1. yes
    2. no
    default choice: [1]:
    2
    Program exits now.
    ```

    - Basic으로 설정시 `mem_used_percent`, `disk_used_percent`가 포함됨
7. agent 시작

    `sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json`

    - onpremise의 경우 `sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m onPremise -s -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json`

    /opt/aws/amazon-cloudwatch-agent/bin/config.json: config파일이 저장된 위치

    → output:

    ```bash
    ****** processing amazon-cloudwatch-agent ******
    /opt/aws/amazon-cloudwatch-agent/bin/config-downloader --output-dir /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d --download-source file:/opt/aws/amazon-cloudwatch-agent/bin/config.json --mode ec2 --config /opt/aws/amazon-cloudwatch-agent/etc/common-config.toml --multi-config default
    Successfully fetched the config and saved in /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d/file_config.json.tmp
    Start configuration validation...
    /opt/aws/amazon-cloudwatch-agent/bin/config-translator --input /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json --input-dir /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d --output /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.toml --mode ec2 --config /opt/aws/amazon-cloudwatch-agent/etc/common-config.toml --multi-config default
    2021/06/03 06:49:05 Reading json config file path: /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d/file_config.json.tmp ...
    Valid Json input schema.
    I! Detecting run_as_user...
    No csm configuration found.
    No log configuration found.
    Configuration validation first phase succeeded
    /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent -schematest -config /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.toml
    Configuration validation second phase succeeded
    Configuration validation succeeded
    amazon-cloudwatch-agent has already been stopped
    Created symlink /etc/systemd/system/multi-user.target.wants/amazon-cloudwatch-agent.service → /etc/systemd/system/amazon-cloudwatch-agent.service.
    ```

8. (optional) `mem_used_percent`만 기록되도록 `config.json`파일을 수정해서 `fetch-config` 함
- 추가정보
    - 간격이 60초 미만인 데이터는 3시간 동안 유지도며 1분인 데이터는 15일 유지됨

        : [https://aws.amazon.com/ko/cloudwatch/faqs/](https://aws.amazon.com/ko/cloudwatch/faqs/)

- Usage

    `amazon-cloudwatch-agent-ctl -help`