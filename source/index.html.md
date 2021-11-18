---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Kittn API
---

# Introduction

Welcome to the Kanvas API!

The API examples use [http](https://httpie.io/), basically cURL but a bit easier to use.

# Authentication

Endpoints that require a logged in user expect a valid token in the Authorization header. To get new a token, call `/auth/login`.

```shell
# Login, and get an access token:
http POST "/auth/login" <<EOF
{
    "address": "$user_address",
    "signedPayload": "$user_password",
}
EOF
```

> To authorize at authorization guarded endpoints eg `/auth/logged_in`, use this code:

```shell
# With shell, you can just pass the correct header with each request
http GET "/auth/logged_in" \
  Authorization:"Bearer $access_token"
```

> Make sure to replace `$access_token` with your token.

<aside class="notice">
You must replace <code>$access_token</code> with your personal API key.
</aside>


# NFTs

## Get All NFTs

```shell
http "/nfts"
```

> The above command returns JSON structured like this:

```json
{
    "currentPage": 1,
    "numberOfPages": 4,
    "nfts": [
        {
            "id": 1,
            "name": "SomeNFT",
            "tokenId": null,
            "contract": null,
            "dataUri": null,
            "ipfsHash": "https://d-art.mypinata.cloud/ipfs/Qmf9LEvo73GGeymjxMYWCggc91DKCXcKVAiNoHT6ZVrjq2",
            "metadata": {
                "json": "encoded"
            },
            "categories": [
                {
                    "name": "mountains",
                    "description": "steep hills that go heigh"
                }
            ]
        }
    ]
}
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
page | 1 | Paginate through results, first page is 1
pageSize | 10 | Number of NFTs to fit on a single page
firstRequestAt | undefined | In seconds since UNIX, the timestamp of first pagination action (note: responsibility of setting this is at the caller). If set, the value will be cloned to the return json.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -X DELETE \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

