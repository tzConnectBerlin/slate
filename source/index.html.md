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

Welcome to the Kanvas API specs!

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
http GET "/auth/logged_in" \
  Authorization:"Bearer $access_token"
```

> Make sure to replace `$access_token` with your token.

<aside class="notice">
You must replace <code>$access_token</code> with your personal API key.
</aside>

For more info see the "Users auth" section.

# NFTs

## Get all NFTs

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
            "ipfsHash": "ipfs://somenft",
            "dataUri": "https://d-art.mypinata.cloud/ipfs/Qmf9LEvo73GGeymjxMYWCggc91DKCXcKVAiNoHT6ZVrjq2",
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

### HTTP Request

`GET /nfts`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
page | 1 | Paginate through results, first page is 1
pageSize | 10 | Number of NFTs to fit on a single page
firstRequestAt | undefined | In seconds since UNIX, the timestamp of first pagination action (note: responsibility of setting this is at the caller). If set, only NFTs will be returned that were listed on the store at or before this timestamp.
orderBy | 'id' | must be one of ['id', 'price', 'name', 'views'].
order | 'asc' | must be one of ['asc', 'desc'].

## Get NFTs filtered

Currently, NFTs can be filtered by:
- list of categories (comma separated category id)
- NFT owner address.

At least one filter must be specified (so, at least a category or at least the address). If both are specified, NFTs that fit both filters are returned (so it's AND logic here, not OR).

```shell
http "/nfts/filter" categories==1,2 address==tz1
```

> The above command returns JSON structured like this:

```json
{
    "currentPage": 1,
    "numberOfPages": 2,
    "nfts": [
        {
            "id": 1,
            "name": "SomeNFT",
            "tokenId": null,
            "contract": null,
            "ipfsHash": "ipfs://somenft",
            "dataUri": "https://d-art.mypinata.cloud/ipfs/Qmf9LEvo73GGeymjxMYWCggc91DKCXcKVAiNoHT6ZVrjq2",
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

### HTTP Request

`GET /nfts/filter`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
categories | undefined | Get NFTs that belong to any of a comma separated set of category identifiers (eg: `1,4,10`). Multiple entries here means: find NFTs that belong to at least one of the given categories.
address | undefined | Get NFTs that are owned by some tezos address.
page | 1 | Paginate through results, first page is 1.
pageSize | 10 | Number of NFTs to fit on a single page
firstRequestAt | undefined | In seconds since UNIX, the timestamp of first pagination action (note: responsibility of setting this is at the caller). If set, only NFTs will be returned that were listed on the store at or before this timestamp.
orderBy | 'id' | must be one of ['id', 'price', 'name', 'views'].
order | 'asc' | must be one of ['asc', 'desc'].

## Get a Specific NFT

```shell
http "/nfts/2"
```
> replace `2` with any NFT id

> The above command returns JSON structured like this:

```json
{
    "name": "NFT2",
    "id": 2,
    "tokenId": null,
    "contract": null,
    "ipfsHash": "ipfs://somenft+",
    "dataUri": "https://d-art.mypinata.cloud/ipfs/QmYjTQiHo4imnccru6Ucsv1MwFLgtHTA1NzXYeZZPPTvq1",
    "metadata": {
        "I": "guess?",
        "json": "encoded"
    },
    "categories": [
        {
            "description": "its not actually blue",
            "name": "water"
        }
    ]
}
```

### HTTP Request

`GET /nfts/:id`

### URL Parameters

Parameter | Description
--------- | -----------
id | The id of the NFT to retrieve


# Categories

## Get all categories

```shell
http "/categories"
```

> The above command returns JSON structured like this:

```json
[
    {
        "children": [
            {
                "children": [
                    {
                        "children": [],
                        "description": "its part of rock",
                        "id": 4,
                        "name": "nirvana"
                    }
                ],
                "description": "its part of a mountain",
                "id": 3,
                "name": "rock"
            }
        ],
        "description": "steep hills that go heigh",
        "id": 1,
        "name": "mountains"
    },
    {
        "children": [],
        "description": "its not actually blue",
        "id": 2,
        "name": "water"
    }
]
```

### HTTP Request

`GET /categories`


# Users auth

## Create account

```shell
http POST "/auth/register" <<EOF
{
    "address": "$user_address",
    "signedPayload": "$user_password"
}
EOF
```

### HTTP Request

`POST /auth/register`

### Body parameters

Parameter | Description
--------- | -----------
address | tezos address
signedPayload | password
name | username (currently it's optional)

## Login

```shell
http POST "/auth/login" <<EOF
{
    "address": "$user_address",
    "signedPayload": "$user_password"
}
EOF
```

> The above command returns JSON structured like this:

```json
{
    "address": "addr",
    "id": 1,
    "maxAge": "86400000",
    "name": null,
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwibmFtZSI6bnVsbCwiYWRkcmVzcyI6ImFkZHIiLCJyb2xlcyI6W10sImlhdCI6MTYzNzI0ODM5Nn0.HovGQYf8QaVh0ZEdLhYBzBYqqYTrH34j6MUDBw8Bb2M"
}
```

### HTTP Request

`POST /auth/login`

### Body parameters

Parameter | Description
--------- | -----------
address | tezos address
signedPayload | password

## Logged in user

```shell
http "/auth/logged_user" Authorization:"Bearer $access_token"
```

> The above command returns JSON structured like this:

```json
{
    "address": "addr",
    "id": 1,
    "name": null,
    "roles": []
}
```

### HTTP Request

`GET /auth/logged_user`

# Cart

All endpoints related to shopping cart management.

A cart session is either derived from the cookie session (when not logged in)
or from the cart session that is connected to the logged in user. When logging
in after having done some cart operations on the cookie session, the user (if
not already connected to another cart session) is connected to the cookie based
cart session. From thereon this cart session stays connected to this user until
either it is checked out by the user, or it expires.

The only endpoint that requires being logged in is the checkout endpoint.

Every cart session has an expiration date. Currently this is 30 minutes after
the last POST action (add, remove).

## List NFTs in the cart

```shell
http "/users/cart/list"
```

> The above command returns JSON structured like this:

```json
{
    "expiresAt": "2021-11-18T14:29:59.403Z",
    "nfts": [
        {
            "id": 1,
            "name": "SomeNFT",
            "tokenId": null,
            "contract": null,
            "ipfsHash": "ipfs://somenft",
            "dataUri": "https://d-art.mypinata.cloud/ipfs/Qmf9LEvo73GGeymjxMYWCggc91DKCXcKVAiNoHT6ZVrjq2",
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

### HTTP Request

`GET /users/cart/list`


## Add NFT

Note, this may fail if

* This NFT is already sold out
* All remaining editions of this NFT are already in other active carts
* This NFT is already in the active cart (currently we only allow 1 edition per NFT per active cart)

```shell
http POST "/users/cart/add/2"
```

### HTTP Request

`POST /users/cart/add/:id`

### URL Parameters

Parameter | Description
--------- | -----------
id | The id of the NFT to add to the cart

## Remove NFT

```shell
http POST "/users/cart/remove/2"
```

### HTTP Request

`POST /users/cart/remove/:id`

### URL Parameters

Parameter | Description
--------- | -----------
id | The id of the NFT to remove from the cart

## Checkout cart

Placeholder: for now, all NFTs in the cart are immediately assigned to the
user, without any sort of payment / minting.

```shell
http POST "/users/cart/checkout" Authorization:"Bearer $access_token"
```

### HTTP Request

`POST /users/cart/checkout`
