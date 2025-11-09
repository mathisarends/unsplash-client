Unsplash Wrapper
================

Typed async Python client for the [Unsplash API](https://unsplash.com/documentation) focused on the photo search endpoint.

Features:

- Async HTTP via `httpx`
- Pydantic models (requests & responses)
- Fluent builder (`UnsplashSearchParamsBuilder`) OR direct kwargs
- Returns a simple `list[UnsplashPhoto]`
- Automatic retry of 429 (rate limit) responses

Installation
------------

Requires Python 3.13+.

```bash
pip install unsplash-wrapper
```

Set your Unsplash access key (Create one in your Unsplash developer dashboard):

```bash
export UNSPLASH_API_KEY=your_access_key_here  # macOS/Linux
$env:UNSPLASH_API_KEY='your_access_key_here'  # PowerShell
```

Quick Start (async)
-------------------

```python
from unsplash_wrapper import UnsplashClient

client = UnsplashClient()
photos = await client.search_photos(query="mountains")
for photo in photos:
	print(photo.id, photo.url)
```

Sync Usage Helper
-----------------

If you're not already inside an async context:

```python
import asyncio
from unsplash_wrapper import UnsplashClient

def main() -> None:
	client = UnsplashClient()
	photos = asyncio.run(client.search_photos(query="ocean"))
	for p in photos:
		print(p.id, p.url)

if __name__ == "__main__":
	main()
```

Search Parameter Options
------------------------

You can provide parameters in two styles: Builder or Raw kwargs.

### 1. Builder style

```python
from unsplash_wrapper import (
	UnsplashClient,
	UnsplashSearchParamsBuilder,
	Orientation,
	ContentFilter,
	OrderBy,
)

builder = (
	UnsplashSearchParamsBuilder()
	.with_query("sunset beach")
	.with_limit(20)
	.with_landscape_orientation()
	.with_high_quality()
	.with_order_by_latest()
	.with_page(2)
)

params = builder.build()
client = UnsplashClient()
photos = await client.search_photos(params)

for photo in photos:
	print(photo.id, photo.description, photo.url)
```

### 2. Direct kwargs style

All fields mirror the Pydantic model `UnsplashSearchParams`:

```python
from unsplash_wrapper import UnsplashClient, Orientation, ContentFilter, OrderBy

client = UnsplashClient()
photos = await client.search_photos(
    query="architecture",
    per_page=15,
    orientation=Orientation.PORTRAIT,
    content_filter=ContentFilter.HIGH,
    page=1,
    order_by=OrderBy.RELEVANT,
)

for photo in photos:
    print(photo.id, photo.user.username)
```

Models Overview
---------------

`UnsplashPhoto` attributes:

```
id: str
description: str | None
alt_description: str | None
urls: UnsplashUrls (raw/full/regular/small/thumb)
user: UnsplashUser (username, name, portfolio_url, ...)
width, height, color, likes, created_at
url (property -> regular URL string)
```

Error Handling
--------------

Exceptions from `unsplash_wrapper.exceptions`:

- `UnsplashAuthenticationException` (401)
- `UnsplashNotFoundException` (404)
- `UnsplashRateLimitException` (429) — includes `retry_after` if provided
- `UnsplashServerException` (5xx)
- `UnsplashClientException` (other 4xx)
- `UnsplashTimeoutException` (request timeout)

```python
from unsplash_wrapper import (
    UnsplashClient,
    UnsplashRateLimitException,
    UnsplashAuthenticationException,
)

client = UnsplashClient()
try:
    photos = await client.search_photos(query="forest")
except UnsplashRateLimitException as e:
    msg = f"retry after {e.retry_after}s" if e.retry_after else "no retry window"
    print("Rate limited:", msg)
except UnsplashAuthenticationException:
    print("Invalid API key configured")
```

Retry Behavior
--------------

429 responses are retried up to 3 times (delays: 1s → 2s → 4s). Other failures propagate immediately.

Logging
-------

Logger name: `unsplash_wrapper.UnsplashClient`

```python
import logging
logging.basicConfig(level=logging.INFO)
```

Advanced: Manual Params
-----------------------

```python
from unsplash_wrapper import UnsplashSearchParams, UnsplashClient, Orientation

params = UnsplashSearchParams(query="minimal", per_page=5, orientation=Orientation.SQUARISH)
client = UnsplashClient()
photos = await client.search_photos(params)
```

Development
-----------

Run tests:

```bash
uv run pytest -q
```

Type check (if mypy configured):

```bash
mypy unsplash_wrapper
```
