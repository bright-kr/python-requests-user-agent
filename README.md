# Setting and Changing User Agent with Python Requests

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 안전하고 성공적인 Webスクレイピング을 위해 Python Requests에서 User-Agent ヘッダー를 설정하고 로ーテ이션하는 방법을 설명합니다:

- [Why You Should Always Set the User Agent Header](#why-you-should-always-set-the-user-agent-header)
- [What Is the Default Requests Python User Agent?](#what-is-the-default-requests-python-user-agent)
- [How to Change the Python Requests User Agent](#how-to-change-the-python-requests-user-agent)
- [Implement User Agent Rotation in Requests](#implement-user-agent-rotation-in-requests)

## Why You Should Always Set the User Agent Header

User-Agent HTTP ヘッダー는 리クエスト를 수행하는 클라이언트 소프트웨어를 식별합니다. 일반적으로 브라우저 유형, 운영 체제, 아키텍처에 대한 세부 정보를 포함합니다.

다음은 Chrome user agent의 예시입니다:

```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36
```

이 문자열에는 다음이 포함됩니다:

* `Mozilla/5.0`: Mozilla 호환성을 나타내는 역사적 접두사입니다
* `Windows NT 10.0; Win64; x64`: 운영 체제, 플랫폼, 아키텍처입니다
* `AppleWebKit/537.36`: 브라우저 엔진입니다
* `KHTML, like Gecko`: 호환성 표시입니다
* `Chrome/125.0.0.0`: 브라우저 이름과 버전입니다
* `Safari/537.36`: Safari 호환성입니다

user agent는 웹사이트가 리クエスト가 정상적인 브라우저에서 오는지, 혹은 자동화된 소프트웨어에서 오는지 판단하는 데 도움이 됩니다. スクレイピング 방지 시스템은 종종 이 ヘッダー를 확인하여 봇을 식별하고 차단하는데, 봇은 일반적으로 기본값이거나 일관되지 않은 user agent 문자열을 사용합니다.

## What Is the Default Requests Python User Agent?

Python Requests 라이브러리는 다음 형식으로 기본 User-Agent ヘッダー를 설정합니다:

`python-requests/X.Y.Z`

여기서 X.Y.Z는 설치된 Requests 버전입니다. 테스트 리クエ스트를 수행하여 이를 확인할 수 있습니다:

```python
import requests

# make an HTTP GET request to the specified URL
response = requests.get('https://httpbin.io/user-agent')

# parse the API response as JSON and print it
print(response.json())
```

출력은 다음과 같이 표시됩니다:

```
{'user-agent': 'python-requests/2.32.3'}
```

이는 브라우저가 아니라 자동화 도구에서 오는 리クエ스트임을 명확히 식별하므로, 웹사이트가 スクレイピング 시도를 감지하고 차단하기가 쉬워집니다.

## How to Change the Python Requests User Agent

### Set a Custom User Agent

headers 딕셔너리에 전달하여 User-Agent를 사용자 정의합니다:

```python
import requests

# custom user agent header

headers = {

  'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36'

}

# make an HTTP GET request to the specified URL

# setting custom headers

response = requests.get('https://httpbin.io/user-agent', headers=headers)

# parse the API response as JSON and print it

print(response.json())
```

セッション 내 모든 리クエ스트에 대해 user agent를 설정하려면 다음을 사용합니다:

```python
import requests

# initialize an HTTP session

session = requests.Session()

# set a custom header in the session

session.headers['User-Agent'] = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36'

# perform a GET request within the HTTP session

response = session.get('https://httpbin.io/user-agent')

# print the data returned by the API

print(response.json())

# other requests with a custom user agent within the session ...
```

이는 이전과 동일한 출력을 생성합니다.

### Unset the User Agent

권장되지는 않지만, 때때로 User-Agent ヘッダー를 제거해야 할 수도 있습니다. 이를 `None`으로 설정해도 기대한 대로 동작하지 않습니다:

```python
import requests

# custom user agent header

headers = {

  'user-agent': None

}

# make an HTTP GET request to the specified URL

# setting custom headers

response = requests.get('https://httpbin.io/user-agent', headers=headers)

# parse the API response as JSON and print it

print(response.json())
```

이 방식은 여전히 urllib3 기본 user agent를 반환합니다:

```
{'user-agent': 'python-urllib3/2.2.1'}
```

ヘッダー를 완전히 제거하려면 `urllib3.util.SKIP_HEADER`를 사용합니다:

```python
import requests

import urllib3

# exclude the default user agent value

headers = { 

  'user-agent': urllib3.util.SKIP_HEADER 

}

# prepare the HTTP request to make

req = requests.Request('GET', 'https://httpbin.io/headers')

prepared_request = req.prepare()

# set the custom headers with no user agent

prepared_request.headers = headers

# create a requests session and perform

# the request

session = requests.Session()

response = session.send(prepared_request)

# print the returned data

print(response.json())
```

위 Python 코드를 실행하면 다음을 받게 됩니다:

```
{'headers': {'Accept-Encoding': ['identity'], 'Host': ['httpbin.io']}}
```

## Implement User Agent Rotation in Requests

여러 리クエ스트에 단일 user agent를 사용하면 アンチボット 시스템이 여전히 트리거될 수 있습니다. user agent 로ーテ이션은 리クエ스트가 서로 다른 브라우저에서 오는 것처럼 보이게 합니다.

구현 방법은 다음과 같습니다:

### Step 1: Create a User Agent List

```python
user_agents = [

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36",

    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0",

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0"

    # other user agents...

]
```

### Step 2: Select a Random User Agent

[`random.choice()`](https://docs.python.org/3/library/random.html#random.choice)를 사용하여 배열에서 user agent 문자열을 무작위로 추출합니다:

```python
random_user_agent = random.choice(user_agents)
```

위 줄에는 다음 import가 필요합니다:

```python
import random
```

### Step 3: Use the Random User Agent

무작위 user agent로 ヘッダー 딕셔너리를 정의하고 `requests` 리クエ스트에서 사용합니다:

```python
headers = {

  'user-agent': random_user_agent

}

response = requests.get('https://httpbin.io/user-agent', headers=headers)

print(response.json())
```

이 지침에는 다음 import가 필요합니다:

```python
import requests
```

### Step 4: Complete Implementation

다음은 전체 스크립트 코드입니다:

```python
import random

import requests

# list of user agents

user_agents = [

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36",

    "Mozilla/5.0 (Macintosh; Intel Mac OS X 14.5; rv:126.0) Gecko/20100101 Firefox/126.0",

    "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:126.0) Gecko/20100101 Firefox/126.0"

    # other user agents...

]

# pick a random user agent from the list

random_user_agent = random.choice(user_agents)

# set the random user agent

headers = {

  'user-agent': random_user_agent

}

# perform a GET request to the specified URL

# and print the response data

response = requests.get('https://httpbin.io/user-agent', headers=headers)

print(response.json())
```

이 스크립트를 여러 번 실행하여 서로 다른 user agent가 동작하는 것을 확인해 보십시오.

## Conclusion

적절한 User-Agent ヘッダー를 설정하는 것은 Python Requests로 성공적인 Webスクレイピング을 수행하는 데 필수적입니다. user agent를 사용자 정의하고 로ーテ이션함으로써, 자동화된 리クエ스트가 더 자연스럽게 보이도록 만들고 기본적인 アンチボット 탐지를 피할 수 있습니다.

하지만 정교한 スクレイピング 방지 시스템은 다른 방법을 통해 자동화를 여전히 감지할 수 있습니다. 보다 견고한 スクレイピング을 위해 user agent 로ーテーション과 함께 プロキシ 로ーテーション을 사용하는 것을 고려하시거나, 이러한 복잡성을 대신 처리해 주는 전용 [Web Scraper API](https://brightdata.co.kr/products/web-scraper)를 살펴보시기 바랍니다.