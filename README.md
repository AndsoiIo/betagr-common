# BetAgr-common - base async api-clients for betagr-project wrote with aiohttp

A set of tools, utilities and base async api-classes for maintaining the infrastructure of betagr-project services.

### Requirements

* python>=3.6
* aiohttp>=3.6

### Usage

```
pip install betagr-common

# or only for user
python -m pip install --user betagr-common

# or for pipenv
pipenv install betagr-common
```

BetAgr-common ships with two base api-classes: BaseClient and BaseClientSSO.

##### BaseClient

It is a base class provides a basic REST API methods which are simple wrapper over a aiohttp.request.
Сlass also implements basic logging of client requests and handling cookies within one client, since the connection session exists only during one request.

Api-clients of each  services should be inherited from the BaseClient.

```python
from common.rest_client.base_client import BaseClient

class BakeryClient(BaseClient):

    _host = 'www.some-bakery.com'
    _port = 8000

    def __init__(self, headers):
        super().__init__(headers=headers)

    async def bake_bread(self, bread_flour='first-grade'):
        body = {'flour': bread_flour}
        api_uri = 'api/bake-bread'
        
        return await self.post(api_uri, body=body)
```

```python
import asyncio
import BakeryClient

db_orders = {'John Doe': {'flour': 'first-grade'},
             'Anderson': {'flour': 'second-grade'}}

async def main():
    async for client in db_orders:
        custom_headers = {'Content-Type': 'application/json'}

        bakery_client = BakeryClient(custom_headers)
        
        bread_flour_type = db_orders[client].get('flour')            

        response = await bakery_client.bake_bread(bread_flour_type)
        assert response.status == 200    # bread was bake!
                        

loop = asyncio.get_event_loop()
loop.run_until_complete(main())  
```

##### BaseClientSSO
Api abstraction for sso-client with the release of basic methods:
* sign_up
* sign_in
* sign_out
* reset_password

```python
import os
import asyncio
from common.rest_client.base_client_sso import BaseClientSSO


async def main():
    # env should contains 'SSO_API_HOST' and 'SSO_API_PORT' variables
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}
    
    sso_client = BaseClientSSO(headers)

    body = {'username': 'john doe',
            'password': 'qwerty'}

    response = await sso_client.sign_up(body)
    print(response)    # ClientResponse -> defaults
                       # {'json': {},
                       #  'status': 200,
                       #  'headers': {'Content-Type: "application/json"},
                       #  'raw_content': b''}

    body = {'username': 'john doe',
            'password': 'qwerty'}

    response = await sso_client.sign_in(body)
    print(response)

    body = {'username': 'john doe',
            'old_password': 'qwerty',
            'new_password': 'qwerty'}

    response = await sso_client.reset_password(body)
    print(response)

    response = await sso_client.sign_out()
    print(response)

loop = asyncio.get_event_loop()
loop.run_until_complete(main())

```
Environment variables are used to determine url/connection parameters in base classes:

|            env variable         |             value               |
|               ---               |              ---                |
| SSO_API_HOST                    |    localhost, www.example.com   |
| SSO_API_PORT                    |              8080               |
| PARSER_API_HOST                 |    localhost, www.example.com   |
| PARSER_API_PORT                 |              8080               |
| AGGREGATOR_API_HOST             |    localhost, www.example.com   |
| AGGREGATOR_API_PORT             |              8080               |
| COMMON_API_CLIENT_LOGGING_LEVEL |               40                |

see numeric represents logging level here: https://docs.python.org/3/library/logging.html#logging-levels
