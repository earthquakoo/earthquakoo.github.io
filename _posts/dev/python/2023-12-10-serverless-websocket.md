---
layout: post
title:  "Python serverless wesocket 서버 만들기 with AWS"
subtitle:   "Python serverless wesocket 서버 만들기 with AWS"
categories: dev
tags: python
comments: true
---


0. Introduction
1. AWS Configure
2. 프로젝트 생성
3. Serverless Ping 함수 구현
4. 디버깅
5. 데이터베이스 생성
6. 실제 Websocket handler 작성

---

# 0. Introduction

이 포스팅은 [Creating a Chat App with Serverless, WebSockets, and Python: A Tutorial](https://levelup.gitconnected.com/creating-a-chat-app-with-serverless-websockets-and-python-a-tutorial-54cbc432e4f) 을 참고했기에 해당 포스팅을 들어가면 더 상세한 정보를 얻을 수 있습니다.(해당 정보를 제공해준 Lance Goodridge에게 무한한 감사를 표합니다!)

아래의 사전 준비가 필요합니다.

- AWS 계정
- python3.9
- npm, node 설치 유무
- AWS CLi 설치 유무

# 1. AWS Configure

위 과정은 aws 계정이 필요합니다. 계정을 생성한 후 AWS console에서 [IAM](https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/home) 에 접속해 사용자로 이동하여 사용자 생성을 해줍니다.

사용자의 이름을 지정합니다.

![img](/assets/img/dev/python/aws-configure-1.png)

이후 직접 정책 연결을 선택하고 사용자에게 **AdministratorAccess** 권한을 부여합니다.
**AdministratorAccess**는 대부분의 권한을 허용해주므로 실제 애플리케이션의 경우에는 필요한 권한을 직접 선택해줘야 합니다.

![img](/assets/img/dev/python/aws-configure-2.png)

권한 설정이 끝나면 사용자가 생성됩니다. 생성된 사용자를 클릭하고 "보안 자격 증명"에 들어가 액세스 키를 만들어줍니다.

액세스 키의 사용 사례는 **Command Line Interfacec(CLI)** 로 생성합니다. 이후 생성된 액세스 키는 CSV 파일이나 개인적인 저장소에 저장해줍니다.

액세스 키의 접근 방식을 Command Line Interface(CLI)로 생성했기 때문에 aws-cli 설치를 필요로 합니다. [AWS-CLI](Z0P1XTvqsxXGjrYF3baN+SvYu1QljXbEVhmZN71V) 해당 페이지에서 aws-cli를 다운로드 후 설정을 진행합니다.

터미널에서 아래의 설정해줍니다.

```bash
$ aws configure

-> AWS Access Key ID [None]: <Acess Key id>
-> AWS Secret Access Key [None]: <Secret Access Key>
-> Default region name [None]: <Your region name>
-> Default output format [None]:
```

# 2. 프로젝트 생성

```bash
mkdir serverless-websocket
cd serverless-websocket
```

프로젝트 폴더를 생성했다면 가상환경을 설정해줍니다.(os: window, terminal: git bash)

```bash
python -m venv venv

source venv/scripts/activate
```

AWS 배포를 위한 serverless를 설치해줍니다.
```bash
npm install -g serverless
```

# 3. Serverless Ping 함수 구현

아래의 커맨드를 입력하면 **serverless-websocket** 이라는 서비스의 템플릿을 생생합니다.

```bash
serverless create --template aws-python3 --name serverless-websocket
```

템플릿이 생성되면 프로젝트 폴더는 아래와 같은 구조가 됩니다.

```
>venv
.gitignore
handler.py
serverless.yml
```

`handler.py` 는 다음과 같습니다. 아직은 ping 함수를 구현하는 과정이므로 매우 단순한 기능인 것을 확인할 수 있습니다.

```python
import json


def hello(event, context):
    body = {
        "message": "Go Serverless v1.0! Your function executed successfully!",
        "input": event
    }

    response = {
        "statusCode": 200,
        "body": json.dumps(body)
    }

    return response
```

이 템플릿을 배포해서 간단한 테스트를 진행해보겠습니다.

```bash
serverless deploy
```

아래의 명령어로 `handler.py`의 함수를 호출할 수 있습니다.

```bash
serverless invoke -f hello
```

정상적으로 배포되고 호출이 되었다면 아래와 같은 반환값을 얻어야합니다.

```bash
{
    "statusCode": 200,
    "body": "{\"message\": \"Go Serverless v1.0! Your function executed successfully!\", \"input\": {}}"
}
```

간단한 테스트를 완료했으니 원래 구현하려고 했던 Ping 함수를 구현해보겠습니다. `handler.py`는 아래와 같이 정상적으로 수신했다는 statuscode과 pong이라는 값을 반환하는 함수로 바꿔줍니다.

```python
import json


def ping(event, context):

    response = {
        "statusCode": 200,
        "body": "PONG!"
    }
    
    return response
```

`serverless.yml`에서는 원래의 `hello` 함수를 `ping` 함수로 업데이트합니다. 또한 URL 경로가 요청될 때 함수를 트리거하는 이벤트를 추가합니다.

```yml
service: serverless-websocket

frameworkVersion: '3'

provider:
  name: aws
  runtime: python3.9

functions:
    ping:
        handler: handler.ping
        events:
            - http:
                path: ping
                method: get
```

이후 재배포를 해줍니다.

```bash
serverless depoly
```

배포가 끝나면 엔드포인트가 제시되는데 이 엔드포인트로 접근하면 아래와 같은 결과를 얻을 수 있습니다.

```bash
endpoint: GET - <Your endpoint>
```

```bash
curl <Your endpoint>

-> PONG!
```

# 4. 디버깅

Serverless 서비스를 이용하여 로그를 확인하는 방법은 [AWS CloudWatch](https://us-east-1.console.aws.amazon.com/cloudwatch/home?region=us-east-1#home:) 에서 가능합니다. Lambda 함수에서 출력값이나 **Logger** 메서드는  [AWS CloudWatch](https://us-east-1.console.aws.amazon.com/cloudwatch/home?region=us-east-1#home:)에 로그로 생성됩니다. Logger 메서드를 이용하면 AWS Cloudwatch에 확인가능한 로그의 형태로 제공받을 수 있습니다.

`handler.py` 에서 아래와 같이 변경해 로그를 생성시킵니다.

```python
import logging

logger = logging.getLogger("handler_logger")
logger.setLevel(logging.DEBUG)


def ping(event, context):

    logger.info("Ping requested.")
    response = {
        "statusCode": 200,
        "body": "PONG!"
    }
    return response
```

재배포 후 [AWS CloudWatch](https://us-east-1.console.aws.amazon.com/cloudwatch/home?region=us-east-1#home:)에 접속하면 새로운 로그그룹이 생성된 것을 확인할 수 있습니다.

```bash
serverless deploy
```

![img](/assets/img/dev/python/serverless-ping-function-1.png)

# 5. 데이터베이스 생성

이제 Websocket의 정보를 저장할 데이터베이스를 생성해야합니다. Websocket connection id를 저장하기 위한 테이블과 채팅방 이름, 메시지 송신자와 메시지를 저장할 테이블이 필요합니다.

Serverless라는 목적에 맞게 DynamoDB를 사용하겠습니다. [DynamoDB](https://us-east-1.console.aws.amazon.com/dynamodbv2/home?region=us-east-1#)에서 테이블 생성합니다.

테이블 이름과 각각의 키를 아래와 같이 설정합니다.

![img](/assets/img/dev/python/database-1.png)

![img](/assets/img/dev/python/database-2.png)

이후 `serverless.yml`에서 DynamoDB와의 상호 작용을 허용하기 위한 코드를 추가합니다.

```yml
provider:
  name: aws
  runtime: python3.9
  iamRoleStatements:
      - Effect: Allow
        Action:
            - "dynamodb:PutItem"
            - "dynamodb:GetItem"
            - "dynamodb:UpdateItem"
            - "dynamodb:DeleteItem"
            - "dynamodb:BatchGetItem"
            - "dynamodb:BatchWriteItem"
            - "dynamodb:Scan"
            - "dynamodb:Query"
        Resource:
            - "arn:aws:dynamodb:us-east-1:*:*"
```

Python에서 사용할 의존성을 위해서 serverless plugin을 설치합니다. 해당 플러그인은 `requirements.txt` 파일을 읽고 핸들러 기능을 AWS Lambda에 푸시하기 전에 이를 설치합니다.

```bash
serverless plugin install -n serverless-python-requirements
```

설치가 완료되었으면 `serverless.yml`에 python-requirements 플러그인을 사용할 수 있게 하는 코드를 추가합니다.

```yml
plugins:
  - serverless-python-requirements

custom:
  pythonRequirements:
    dockerizePip: false
    noDeploy: []
```

위 작업이 완료되었다면 `serverless.yml` 파일을 아래와 같아야합니다.

```yml
service: serverless-websocket

frameworkVersion: '3'

provider:
  name: aws
  runtime: python3.9
  iamRoleStatements:
      - Effect: Allow
        Action:
            - "dynamodb:PutItem"
            - "dynamodb:GetItem"
            - "dynamodb:UpdateItem"
            - "dynamodb:DeleteItem"
            - "dynamodb:BatchGetItem"
            - "dynamodb:BatchWriteItem"
            - "dynamodb:Scan"
            - "dynamodb:Query"
        Resource:
            - "arn:aws:dynamodb:us-east-1:*:*"
functions:
    ping:
        handler: handler.ping
        events:
            - http:
                path: ping
                method: get

plugins:
  - serverless-python-requirements
custom:
  pythonRequirements:
    dockerizePip: false
    noDeploy: []
```

이제 데이터베이스(dynamodb) 연결 설정을 해야합니다. python에서는 [boto3](https://aws.amazon.com/ko/sdk-for-python/) 모듈을 통해 AWS 서비스에 접근할 수 있습니다.

```bash
pip install boto3
```

이후 `requirements.txt` 파일을 생성해 종속성을 저장합니다.

```bash
pip freeze > requirements.txt
```

앞선 작업들이 모두 완료되었다면 프로젝트 폴더 구조는 아래와 같을 것 입니다.

```bash
> .serverless
> node_modules
> venv
.gitignore
handler.py
package-lock.json
package.json
requirements.txt
serverless.yml
```

이제 python handler에서 **boto3**를 이용해 데이터베이스를 읽고 업데이트할 수 있습니다.  **boto3** 를 이용해 DynamoDB 리소스를 가져와 생성한 다음 테이블을 업데이트하는 코드를 작성하면 아래와 같습니다.

```python
import boto3
import json
import logging
import time

logger = logging.getLogger("handler_logger")
logger.setLevel(logging.DEBUG)

dynamodb = boto3.resource("dynamodb")


def ping(event, context):

    logger.info("Ping requested.")

    table = dynamodb.Table("serverless-websocket-message")
    create_at = int(time.time())
    table.put_item(Item={"message_type": "broadcast", "index": 0,
            "create_at": create_at, "user_name": "ping-user",
            "content": "PING!", "room_name": "python"})
    logger.debug("Item added to the database.")

    response = {
        "statusCode": 200,
        "body": "PONG!"
    }
    return response
```

- **boto3** 를 통해 DynamoDB의 리소스를 가져온 다음 이전에 만든 `serverless-websocket-message` 라는 테이블을 가져옵니다.
- 해당 테이블에는 `room` 과 `index` 키만 생성했지만 여기서 메시지 생성시간인 `create_at`, 메시지를 작성한 사용자의 이름 `user_name`, 메시지 내용 `content` 를 추가했습니다.

이제 재배포 후 함수를 호출하면 DynamoDB의 `serverless-websocket-message` 테이블로 이동해 "표 항목 탐색"을 누르면 생성된 메시지를 확인할 수 있습니다.

```bash
serverless deploy

curl <Your endpoint>

-> PONG!
```

![img](/assets/img/dev/python/database-3.png)

# 6. 실제 Websocket handler 작성

지금까지는 간단한 ping pong 함수를 만들어 테스트를 해보았습니다. 이제는 실용적인 websocket handler를 만들 차례입니다!

## 6-1. Connection manager function

먼저 websocket의 연결 및 연결 해제를 처리하는 `connection_manager` 함수를 생성해보겠습니다.

```python
def _get_response(status_code, body):
    if not isinstance(body, str):
        body = json.dumps(body)
    return {"status_code": status_code, "body": body}


def connection_manager(event, context):

    connection_id = event["requestContext"].get("connectionId")
    room_name = event.get("queryStringParameters", {}).get("room_name")
    
    if event["requestContext"]["eventType"] == "CONNECT":
        logger.info("Connect requested")

        table = dynamodb.Table("serverless-websocket-connections")
        table.put_item(Item={"connection_id": connection_id,
                             "room_name": room_name})
        return _get_response(200, "Connect successful.")

    elif event["requestContext"]["eventType"] == "DISCONNECT":
        logger.info("Disconnect requested")

        table = dynamodb.Table("serverless-websocket-connections")
        table.delete_item(Key={"connection_id": connection_id})
        return _get_response(200, "Disconnect successful.")

    else:
        logger.error("Connection manager received unrecognized eventType '{}'")
        return _get_response(500, "Unrecognized eventType.")
```

- Websocket 연결 시 DynamoDB의 `serverless-websocket-connections` 테이블을 불러와 `connection_id`를 저장하고 연결 해제 시 제거하는 코드를 생성했습니다.
	- 또한 `room_name`을 query parameter로 받아 이를 `connection_id` 와 함께 저장했습니다.
- 함수가 종료될 때마다 응답 메시지를 보내야하기 때문에 `_get_response`라는 helper function을 생성했습니다.

## 6-2. Send message function

메시지 전송을 위한 함수를 생성합니다. 클라이언트 측에서는 메시지의 `content`와 메시지를 보내는 사용자의 이름 `user_name`, 채팅방의 이름 `room_name` 과같은 데이터를 제공해야하며 서버 측은 이를 해석해 데이터베이스에 저장합니다.

```python
def _send_to_connection(connection_id, data, event):
    gatewayapi = boto3.client("apigatewaymanagementapi",
            endpoint_url = "https://" + event["requestContext"]["domainName"] +
                    "/" + event["requestContext"]["stage"])
    return gatewayapi.post_to_connection(ConnectionId=connection_id,
            Data=json.dumps(data).encode('utf-8'))


def send_message(event, context):

    logger.info("Message sent on WebSocket.")

    body = _get_body(event)
    for attribute in ["user_name", "content", "room_name"]:
        if attribute not in body:
            logger.debug(f"Failed: {attribute} not in message dict."\
                    .format(attribute))
            return _get_response(400, f"{attribute} not in message dict"\
                    .format(attribute))
            

    message_table = dynamodb.Table("serverless-websocket-message")
    message_table_response = message_table.query(
        KeyConditionExpression="message_type = :message_type",
        ExpressionAttributeValues={":message_type": "broadcast"},
        Limit=1, ScanIndexForward=False
        )
    message_table_items = message_table_response.get("Items", [])
    index = message_table_items[0]["index"] + 1 if len(message_table_items) > 0 else 0
    
    create_at = int(time.time())
    user_name = body["user_name"]
    content = body["content"]
    room_name = body["room_name"]
    
    message_table.put_item(Item={
        "message_type": "broadcast",
        "index": index,
        "room_name": room_name,
        "create_at": create_at,
        "user_name": user_name,
        "content": content
        })
    
    connection_table = dynamodb.Table("serverless-websocket-connections")
    connection_table_response = connection_table.scan(
        FilterExpression=Attr('room_name').eq(room_name)
    )
    items = connection_table_response.get("Items", [])
    connections = [x["connection_id"] for x in items if "connection_id" in x]

    message = {"user_name": user_name, "content": content}
    logger.debug("Broadcasting message: {}".format(message))
    data = {"messages": [message]}
    for connection_id in connections:
        _send_to_connection(connection_id, data, event)

    return _get_response(200, "Message sent to all connections.")
```


```python
    message_table = dynamodb.Table("serverless-websocket-message")
    message_table_response = message_table.query(
        KeyConditionExpression="message_type = :message_type",
        ExpressionAttributeValues={":message_type": "broadcast"},
        Limit=1, ScanIndexForward=False
        )
    message_table_items = message_table_response.get("Items", [])
    index = message_table_items[0]["index"] + 1 if len(message_table_items) > 0 else 0
```

- 먼저 DynamoDB의 `serverless-websocket-message` 테이블을 불러옵니다.
- 테이블의 index를 수동으로 계산하기 위해 table의 마지막 값을 가져와 index + 1을 해줍니다.

```python
    create_at = int(time.time())
    user_name = body["user_name"]
    content = body["content"]
    room_name = body["room_name"]
    
    message_table.put_item(Item={
        "message_type": "broadcast",
        "index": index,
        "room_name": room_name,
        "create_at": create_at,
        "user_name": user_name,
        "content": content
        })
```

- `create_at`은 `time` 메소드로 직접 설정해줍니다.
- `user_name`, `content`, `room_name` 은 클라이언트 측에서 제공받은 데이터를 저장해줍니다.

```python
    connection_table = dynamodb.Table("serverless-websocket-connections")
    connection_table_response = connection_table.scan(
        FilterExpression=Attr('room_name').eq(room_name)
    )
    items = connection_table_response.get("Items", [])
    connections = [x["connection_id"] for x in items if "connection_id" in x]

    message = {"user_name": user_name, "content": content}
    logger.debug("Broadcasting message: {}".format(message))
    data = {"messages": [message]}
    for connection_id in connections:
        _send_to_connection(connection_id, data, event)
```

- 이제 `serverless-websocket-connections` 테이블을 가져와 클라이언트 측에서 제공받은 `room_name` 과 동일한 `connection_id`를 가져옵니다.
- 마지막으로 이 메시지는 websocket에 연결된 모든 이에게 브로드캐스트 되어야 합니다.
	- [ApiGatewayManagementApi](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/apigatewaymanagementapi/client/post_to_connection.html#) 의 `post_to_connection`으로 메시지를 전송할 수 있습니다.

## 6-3. get recent messages

다음으론 가장 최근에 보내진 10개의 메시지를 가져오는 함수를 생성해보겠습니다.

```python
def get_recent_messages(event, context):

    logger.info("Retrieving most recent messages.")
    connection_id = event["requestContext"].get("connectionId")
    
    body = _get_body(event)
    room_name = body["room_name"]

    table = dynamodb.Table("serverless-websocket-message")
    response = table.query(
        KeyConditionExpression="message_type = :message_type",
        ExpressionAttributeValues={":message_type": "broadcast"},
        FilterExpression=Attr('room_name').eq(room_name),
        Limit=10, ScanIndexForward=False)
    items = response.get("Items", [])

    messages = [{"user_name": x["user_name"], "content": x["content"]}
            for x in items]
    messages.reverse()

    data = {"messages": messages}
    _send_to_connection(connection_id, data, event)

    return _get_response(200, "Sent recent messages.")
```

- Endpoint에 입력된 query parameter인 `room_name`과 `event['requestContext']`에 있는 `conncetion_id`를 가져옵니다.
- DynamoDB의 `serverless-websocket-message` 테이블을 가져와 key값인 `message_type`의 `broadcast`와 입력받은 `room_name`과 메시지의 `room_name`이 같은 메시지 10개를 가져옵니다.
	- `scanIndexForward=False`로 index의 역순인 10개의 항목을 가져오는데 시간의 역순이 되기 때문에 `message.reverse()`를 통해 원래의 순서로 업데이트해줍니다.


이제 모든 핸들러 함수들이 생성되었으므로 `serverless.yml`의 구성을 추가해주면 됩니다. 아래는 최종 `serverless.yml`의 구성입니다.

```yml
service: serverless-websocket

provider:
    name: aws
    runtime: python3.9
    websocketApiName: serverless-websocket-api
    websocketApiRouteSelectionExpression: $request.body.action
    iamRoleStatements:
        - Effect: Allow
          Action:
              - "execute-api:ManageConnections"
          Resource:
              - "arn:aws:execute-api:*:*:**/@connections/*"
        - Effect: Allow
          Action:
              - "dynamodb:PutItem"
              - "dynamodb:GetItem"
              - "dynamodb:UpdateItem"
              - "dynamodb:DeleteItem"
              - "dynamodb:BatchGetItem"
              - "dynamodb:BatchWriteItem"
              - "dynamodb:Scan"
              - "dynamodb:Query"
          Resource:
              - "arn:aws:dynamodb:us-east-1:*:*"

plugins:
    - serverless-python-requirements

custom:
    pythonRequirements:
        dockerizePip: False
        noDeploy: []

functions:
    connectionManager:
        handler: handler.connection_manager
        events:
            - websocket:
                route: $connect
            - websocket:
                route: $disconnect
    defaultMessage:
        handler: handler.default_message
        events:
            - websocket:
                route: $default
    sendMessage:
        handler: handler.send_message
        events:
            - websocket:
                route: sendMessage
    getRecentMessages:
        handler: handler.get_recent_messages
        events:
            - websocket:
                route: getRecentMessages
    ping:
        handler: handler.ping
        events:
            - http:
                path: ping
                method: get
```

# 7. 테스트

제대로 동작하는지 확인해보기 위해선 [Websocket cat](https://github.com/websockets/wscat)을 이용할 수 있습니다. 아래의 커맨드로 `wscat`을 설치합니다.

```bash
npm install -g wscat
```

이후 재배포합니다.

```bash
serverless depoly
```

제공받은 endpoint로 connection을 요청합니다.

```bash
wscat -c <Your endpoint>.execute-api.us-east-1.amazonaws.com/dev?room_name=<Your room name>
```

아래와 같이 요청을 보낸 후

```bash
> {"action": "sendMessage", "user_name": "earthquake", "content": "Websocket API test.", "room_name": "<Your room name>"}
```

아래와 같은 응답이 온다면 성공입니다!!

```bash
< {"messages": [{"user_name": "test-websocket-user", "content": "Websocket API 
test."}]}
```