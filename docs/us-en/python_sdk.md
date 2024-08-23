# Python SDK

> The Cloudbypass Python SDK only supports Python 3.6 and above.

The CloudCross SDK, which is encapsulated on the basis of `psf/requests`, supports the call of Scrapingbypass API service. Through the built-in session manager, session requests can be automatically processed without manually managing information such as cookies.

[![cloudbypass](https://img.shields.io/pypi/pyversions/cloudbypass)](https://pypi.org/project/cloudbypass/ ":no-zoom")
[![cloudbypass](https://img.shields.io/pypi/v/cloudbypass)](https://pypi.org/project/cloudbypass/ ":no-zoom")
[![cloudbypass](https://img.shields.io/pypi/dd/cloudbypass)](https://pypi.org/project/cloudbypass/#files ":no-zoom")
[![cloudbypass](https://img.shields.io/pypi/wheel/cloudbypass)](https://pypi.org/project/cloudbypass/ ":no-zoom")

### Install

```shell
python3 -m pip install cloudbypass -i https://pypi.org/simple
```

### Send Request

The `Session` class inherits from `requests.Session` and supports all methods of `requests`.

Added initialization parameters `apikey` and `proxy` to set the Scrapingbypass API service key and proxy IP respectively.

Custom users can specify the service address by setting the `api_host` parameter.

> The above parameters can be configured using the environment variables `CB_APIKEY`, `CB_PROXY`, and `CB_APIHOST`.

```python
from cloudbypass import Session

if __name__ == '__main__':
    with Session(apikey="<APIKEY>", proxy="http://proxy:port") as session:
        resp = session.get("https://opensea.io/category/memberships")
        print(resp.status_code, resp.headers.get("x-cb-status"))
        print(resp.text)
```

#### async

The `AsyncSession` class inherits from `aiohttp.ClientSession` and supports all methods of `aiohttp`.

```python
import asyncio
from cloudbypass import AsyncSession


async def main():
    async with AsyncSession(apikey="<APIKEY>", proxy="http://proxy:port") as session:
        resp = await session.get("https://opensea.io/category/memberships")
        print("Status:", resp.status)
        print("Content-type:", resp.headers['content-type'])
        html = await resp.text()
        print("Body:", html[:15], "...")


if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())
```

### Using V2

Scrapingbypass API V2 is suitable for websites that need to pass JS challenge verification. For example, visit https://etherscan.io/accounts/label/lido and request an example:

```python
from cloudbypass import Session

if __name__ == '__main__':
    with Session(apikey="<APIKEY>", proxy="http://proxy:port") as session:
        resp = session.get("https://etherscan.io/accounts/label/lido", part="0")
        print(resp.status_code, resp.headers.get("x-cb-status"))
        print(resp.text)
```

#### async

```python
import asyncio
from cloudbypass import AsyncSession


async def main():
    async with AsyncSession(apikey="<APIKEY>", proxy="http://proxy:port") as session:
        resp = await session.get("https://etherscan.io/accounts/label/lido", part="0")
        print("Status:", resp.status)
        print("Content-type:", resp.headers['content-type'])
        html = await resp.text()
        print("Body:", html[:15], "...")


if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())
```

### Check balance

Use the `get_balance` method to query the current account balance.

```python
from cloudbypass import get_balance

if __name__ == '__main__':
    print(get_balance("<APIKEY>"))

```

#### async

```python
import asyncio
from cloudbypass import async_get_balance


async def main():
    print(await async_get_balance("<APIKEY>"))


if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())

```

### Extraction Proxy

The `Proxy` class can be used to extract the cloud-bypass dynamic proxy IP and time-sensitive proxy IP.

+ `copy()` Duplicate the current object so that the original object will not be affected.
+ `set_dynamic()` Set as dynamic proxy.
+ `set_expire(int)` Set to time-limited proxy, the parameter is the IP expiration time, in seconds.
+ `set_region(str)` Set the proxy IP region.
+ `clear_region()` Clear the area of the agent.
+ `format(str)` Format proxy IP. The parameter is a formatted string, for example, `{username}:{password}@gateway`.
+ `limit(int, str)` Returns a proxy IP string iterator, with the parameters being the extraction quantity and the proxy format string.
+ `loop(int, str)` Returns a proxy IP string loop iterator, the parameters are the actual number and the proxy format string.

```python
from cloudbypass import Proxy

if __name__ == '__main__':
    proxy = Proxy("username-res:password")

    # Extracting dynamic proxies
    print("Extract dynamic proxy: ")
    print(str(proxy.set_dynamic()))
    print(str(proxy.set_region('US')))

    # Extract time agent and specify region
    print("Extract proxy with expire and region: ")
    print(str(proxy.copy().set_expire(60 * 30).set_region('US')))

    # Batch Extraction
    print("Extract five 10-minute aging proxies: ")
    pool = proxy.copy().set_expire(60 * 10).set_region('US').limit(5)
    for _ in pool:
        print(_)

    # Cycle Extraction
    print("Loop two 10-minute aging proxies: ")
    loop = proxy.copy().set_expire(60 * 10).set_region('US').loop(2)
    for _ in range(10):
        print(loop.__next__())
```

### About redirection issues

When using the SDK to initiate a request, the redirection operation is automatically handled without manual processing. The redirection response will also consume credits.