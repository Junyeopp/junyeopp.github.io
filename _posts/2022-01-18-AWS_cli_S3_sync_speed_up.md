---
title: AWS CLI - S3 sync 속도 개선하기
data: 2022-01-18 20:00:00 +09:00
categories: [AWS]
tag: [dev]
---

AWS CLI의 `sync` 명령어를 통해서 S3의 데이터를 백업할 때, `aws s3 sync s3://$bucket .` 의 형태로 버킷단위로 `sync`를 진행하면 시간이 많이 걸렸습니다.

`sync`를 할 때 파일 전송시간보다 어떤 파일을 옮겨야 할지 찾는데 시간이 더 많이 걸리는 것 같았고 네트워크 대역폭은 널널했었습니다. 이 대역폭을 가득 채우고 싶었습니다.

AWS에서 아래의 [두 가지 제안](https://aws.amazon.com/premiumsupport/knowledge-center/s3-improve-transfer-sync-command/)을 찾을 수 있었습니다.

# 1. prefix별로 `aws s3 sync` 를 동시에 여러 개 실행하기

기존에는 PowerShell script의 `foreach`를 통해 여러 개의 버킷을 순차적으로 sync하고 있었습니다.

병렬적으로 실행하기 위해서 PowerShell 7부터 지원하는 `ForEach-Object` 명령어의 `-Parallel` 옵션을 사용하기로 하였고, [Winget을 사용하여 PowersShell 7을 설치](https://docs.microsoft.com/ko-kr/powershell/scripting/install/installing-powershell-on-windows?view=powershell-7.2#install-powershell-using-winget)하여 script를 작성해보았습니다.

(윈도우에 기본적으로 설치되어있던 PowerShell이 6.0부터 .NET Core를 기반으로 다양한 플랫폼을 지원하는 오픈 소스 프로젝트가 되었습니다. 그래서 PowerShell 7은 추가적으로 설치가 필요합니다.)

```powershell
$bucket = "싱크 대상 버킷"
$prefix = "싱크 대상 폴더"
$start_time = Get-Date -UFormat "%Y-%m-%d %T%Z"
$start_date = Get-Date -UFormat "%Y%m%d"

aws configure set s3.max_concurrent_requests 30

[array] $parts = aws s3 ls s3://$bucket/$prefix/

$cur_path = "현재 디렉토리 경로"
New-Item -Path $cur_path -Name "${start_date}_${bucket}_${prefix}" -ItemType "directory"
$log_path = "$cur_path\${start_date}_${bucket}_${prefix}"

Write-Output "Syncing bucket '$bucket' ($start_time)..." | Out-File -FilePath "$log_path\log.log" -Append
foreach ($p in $parts)
{
    if ($p -match "part")
    {
        $part = (-split $p | Select-Object -Last 1).Trim("/")
        [array] $nums = aws s3 ls s3://$bucket/$prefix/$part/

        foreach ($n in $nums)
        {
            $num = (-split $n | Select-Object -Last 1).Trim("/")
            if ((($num -split "=") | Select-Object -Last 1) -gt "130")
            {
                Write-Output "    $bucket)/$prefix)/$part)/$num" | Out-File -FilePath "$log_path\log.log" -Append
            }
        }

        $nums | ForEach-Object -Parallel {
            $n = $_
            if ($n -match "num")
            {
                $num = (-split $n | Select-Object -Last 1).Trim("/")
                if ((($num -split "=") | Select-Object -Last 1) -gt "130")
                {
                    aws s3 sync "s3://$($using:bucket)/$($using:prefix)/$($using:part)/$num" "로컬 디렉토리 경로\$($using:bucket)\$($using:prefix)\$($using:part)\$num" 2>&1 > "$($using:log_path)\$($using:part)_$date.log"
                }
            }
        } -ThrottleLimit 20

        $mid_time = Get-Date -UFormat "%Y-%m-%d %T%Z"
        Write-Output "current time: $mid_time, $part" | Out-File -FilePath "$log_path\log.log" -Append
    }
}

$end_time = Get-Date -UFormat "%Y-%m-%d %T%Z"
Write-Output "done ($end_time)"
Write-Output ""
```

- S3 대상 버킷의 prefix의 구조는 `$bucket/$prefix/part=123/num=123/..`의 형태입니다.
    - `../num=123/..` 단위로 병렬적으로 `sync`를 하도록 구성하였습니다.
- `aws s3 sync`의 출력값들을 시간정보와 함께 파일로 저장하기위해서 prefix별로 디렉토리를 만들고 그 안에 출력값들을 저장하도록 하였습니다.
- num=130보다 큰 값들만 `sync`하도록 설정하였습니다.
- `ForEach-Object`에서 `-Parallel` 옵션을 통해 병렬처리를, `-ThrottleLimit`을 통해 동시에 실행될 수 있는 스크립트 수를 제한하였습니다.
- `ForEach-Object` 안으로 밖의 변수를 전달하기 위해 `$using:`을 사용하였습니다. ($bucket의 형태로는 ForEach-Object 밖의 변수를 사용할 수 없습니다.)

# 2. AWS CLI의 max_concurrent_requests값을 수정하기
max_concurrent_requests의 기본값은 10이며, S3에 한 번에 전송할 수 있는 요청수를 나타냅니다.

기본값에서 실행하였을 때 시스템에 부하가 없었기에 max_concurrent_requests를 30정도로 수정하였습니다.

`aws configure set s3.max_concurrent_requests 30`