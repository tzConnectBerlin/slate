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

This endpoint retrieves all kittens.

### HTTP Request

`GET /nfts`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
page | 1 | Paginate through results, first page is 1
pageSize | 10 | Number of NFTs to fit on a single page
firstRequestAt | undefined | In seconds since UNIX, the timestamp of first pagination action (note: responsibility of setting this is at the caller). If set, only NFTs will be returned that were listed on the store at or before this timestamp.

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
categories | undefined | Filter by comma separated list of categories identifiers (eg: `1,4,10`). Multiple entries here means: find NFTs that belong to at least one of the given categories.
address | undefined | Get NFTs that are owned by some tezos address.
page | 1 | Paginate through results, first page is 1.
pageSize | 10 | Number of NFTs to fit on a single page
firstRequestAt | undefined | In seconds since UNIX, the timestamp of first pagination action (note: responsibility of setting this is at the caller). If set, only NFTs will be returned that were listed on the store at or before this timestamp.

## Get a Specific NFT

```shell
http "/nfts/1"
```
> replace `1` with any NFT id

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
