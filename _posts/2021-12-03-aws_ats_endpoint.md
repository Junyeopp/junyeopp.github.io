---
title: AWS - ATS signed data endpoint
data: 2021-12-03 19:00:00 +09:00
categories: [AWS]
tag: [AWS]
---
# 1. 문제

AWS IoT shadow값을 업데이트하는 과정에서 SSL validation 에러가 발생하였습니다.
- 코드

    ```python
    client_iot = boto3.client("iot-data")
    client_iot.update_thing_shadow(
        thingName=thing_name,
        payload=json.dumps(shadow_state_desired)
    )
    ```

- 발생한 에러 메시지

    ```python
    'SSL validation failed for https://data.iot.ap-northeast-2.amazonaws.com/things/../shadow [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1131)'
    ```

- 인증서정보

    ![cert1]({{ site.baseurl }}/assets/images/2021-12-03-aws_ats_endpoint_cert1.png)

    ![cert1]({{ site.baseurl }}/assets/images/2021-12-03-aws_ats_endpoint_cert2.png)


# 2. 원인

사용중이던 Lambda layer를 업데이트하고 인증 문제가 발생하였기 때문에 달라진 점을 찾아보았습니다.

[`Requests`](https://docs.python-requests.org/en/master/) library가 문제가 된 것으로 보였고, 버전이 `requests.version='2.24.0'` 에서 `requests.version='2.26.0'`으로 변경된 것을 확인하였습니다. 이에 따라 [`certifi`](https://certifiio.readthedocs.io/en/latest/)의 버전이 `certifi.version='2020.06.20'`에서 `certifi.version='2021.10.08'` 으로 변경된 것 또한 확인할 수 있었습니다.

`certifi`는 Mozilla’s carefully curated collection of Root Certificates을 제공해줍니다. 그래서 Mozilla의 Root CA 목록에 변화가 있는지를 찾아보았고, [distrust of Symantec root certificates 이슈](https://wiki.mozilla.org/CA:Symantec_Issues)를 확인하였습니다.

- 이슈는 Symantec이 SSL인증서를 부적절하게 발급한 것이 Google과 Mozilla에 의해 확인되었고, 업계에서는 Symantec을 신뢰하지 않게되었다는 내용이었습니다.

AWS IoT shadow 업데이트 과정에서 문제가 되었던 인증서의 발급자인 `VeriSign Class 3 Public Primary Certification Authority - G5` 는 2020년 12월에 [Mozilla의 목록](https://wiki.mozilla.org/CA:IncludedCAs)에서 [제거](https://wiki.mozilla.org/CA/Additional_Trust_Changes#Symantec)된 것을 확인할 수 있었습니다.

# 3. 해결

AWS는 IoT의 data endpoint로 기존의 VeriSign signed data endpoint 대신 ATS signed data endpoint를 사용할 것을 [강력하게 권장](https://docs.aws.amazon.com/iot/latest/apireference/API_DescribeEndpoint.html#API_DescribeEndpoint)하고 있습니다.

ATS endpoint는 다양한 방법으로 얻을 수 있으며 저는 boto3의 [describe_endpoint](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iot.html#IoT.Client.describe_endpoint)를 통해서 ATS endpoint 주소를 구하였고, 기존의 코드를 아래와 같이 변경하였습니다.

```python
def get_ATS_endpoint():
		return boto3.client("iot").describe_endpoint(
		    endpointType="iot:Data-ATS"
		)["endpointAddress"]

client_iot = boto3.client("iot-data", endpoint_url=f"https://{get_ATS_endpoint()}")
client_iot.update_thing_shadow(
    thingName=thing_name,
    payload=json.dumps(shadow_state_desired)
)
```

그 결과 SSL validation을 실패하지않고 정상적으로 shadow를 업데이트하는 것을 확인할 수 있었습니다.
