---
title: DynamoDB - Attribute vs Key
data: 2021-08-09 19:00:00 +09:00
categories: [AWS]
tag: [AWS]
---
DynamoDB scan에서 FilterExpression으로 Attr와 Key 둘 다 작동하였고 어떤 점이 다른지 궁금해서 찾아보았습니다. 두 개는 비슷하지만 목적이 다른 class였습니다.

# DynamoDB의 기본 구성요소

DynamoDB는 다음과 같은 기본적인 요소들로 구성되어 있습니다.

- Tables

    : data의 collection이며 0개 이상의 item을 가지고 있음

- Items

    : attributes의 group이마 0개 이상의 attributes로 구성되어 있음

- Attributes

    : 기초적인 data element로 다른 DB system의 fields 또는 columns와 비슷함


- Primary Key

    : table에서 각각의 item을 유일하게 결정해주는 값으로 다른 item은 같은 primary key를 가질 수 없습니다. primary key에는 다음 두 종류가 있습니다.

    - Partition key

        : 하나의 attribute로 구성됩니다. (partition key)

    - Partition key and sort key

        : 두 개의 attribute로 구성됩니다. (partition key, sort key)

        이 경우 다른 item이 같은 partition key를 가질 수는 있으나, sort key까지 같을 수는 없습니다.


# Attr vs Key

[Boto3 문서](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/dynamodb.html?#querying-and-scanning)를 참고하면 다음과 같은 설명을 볼 수 있습니다.

> The boto3.dynamodb.conditions.Key should be used when the condition is related to the key of the item. The boto3.dynamodb.conditions.Attr should be used when the condition is related to an attribute of the item:
> 

Key는 key와 관련되더 있을 때, Attr는 attribute와 관련되어 있을 때 사용하라고 합니다.

query, scan에 대한 설명에서도 다음과 같이 key에 대해서는 Key를 사용하고 attribute에 대해서는 Attr를 사용하라고 합니다.

> FilterExpression (condition from boto3.dynamodb.conditions.Attr method)
> 

> KeyConditionExpression (condition from boto3.dynamodb.conditions.Key method)
> 

하지만 이 글의 시작부분처럼 Attr와 Key class가 모두 사용가능할 때가 있었습니다. 그래서 소스코드를 좀 더 살펴보았습니다.

- boto3.dynamodb.conditions.Key

    ```python
    class Key(AttributeBase):
        pass
    ```

- boto3.dynamodb.conditions.Attr

    ```python
    class Attr(AttributeBase):
        """Represents an DynamoDB item's attribute."""
        def ne(self, value):
            """Creates a condition where the attribute is not equal to the value

            :param value: The value that the attribute is not equal to.
            """
            return NotEquals(self, value)

        def is_in(self, value):
            """Creates a condition where the attribute is in the value,

            :type value: list
            :param value: The value that the attribute is in.
            """
            return In(self, value)

        def exists(self):
            """Creates a condition where the attribute exists."""
            return AttributeExists(self)

        def not_exists(self):
            """Creates a condition where the attribute does not exist."""
            return AttributeNotExists(self)

        def contains(self, value):
            """Creates a condition where the attribute contains the value.

            :param value: The value the attribute contains.
            """
            return Contains(self, value)

        def size(self):
            """Creates a condition for the attribute size.

            Note another AttributeBase method must be called on the returned
            size condition to be a valid DynamoDB condition.
            """
            return Size(self)

        def attribute_type(self, value):
            """Creates a condition for the attribute type.

            :param value: The type of the attribute.
            """
            return AttributeType(self, value)
    ```


**두 class 모두 AttributeBase class를 사용하고 있었습니다!**

Key class는 AttributeBase class를 그대로 사용하고, Attr class는 몇 가지를 추가해서 사용하고 있었습니다. 그래서 만약 AttributeBase의 method를 사용할 때에는 두 class 모두 다 사용이 가능했던 것이었습니다.

하지만 가독성을 고려해서 원래 목적에 맞게 사용하는 것이 좋을 것이라고 생각합니다.
