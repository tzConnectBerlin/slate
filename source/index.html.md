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
    "userAddress": "$user_address",
    "signedPayload": "$user_password",
}
EOF
```

> To authorize at authorization guarded endpoints, eg `/auth/logged_in`, set the "Authorization" header alike:

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

## Get NFTs (optionally with filters)

```shell
http "/nfts"
```

> The above command returns JSON structured alike:

```json
{
    "currentPage": 1,
    "numberOfPages": 4,
    "lowerPriceBound": "1.12",
    "upperPriceBound": "105.00",
    "nfts": [
        {
            "artifactUri": "https://kanvas-admin-files.s3.amazonaws.com/NFT_FILE__6_image.png",
            "categories": [
                {
                    "description": "Not actually visual",
                    "id": 3,
                    "name": "Applied Art"
                }
            ],
            "createdAt": 1645710471,
            "description": "Nice for breakfast",
            "displayUri": "https://kanvas-admin-files.s3.amazonaws.com/NFT_FILE__6_image.png",
            "editionsAvailable": 0,
            "editionsSize": 1,
            "id": 6,
            "ipfsHash": "ipfs://QmQx1zaVfbXk1JfPGhHo3pRaBgcHBzsHC1EVzXEkG2k1rz",
            "launchAt": 1645743600,
            "name": "baguette",
            "price": "1.68",
            "thumbnailUri": "https://kanvas-admin-files.s3.eu-central-1.amazonaws.com/NFT_FILE__6_thumbnail.png"
        }
    ]
}
```

Results are paginated.

Optionally, a set of filters can be specified. NFTs can be filtered by: `categories`, `userAddress`, `priceAtLeast`, `priceAtMost`, `availability` (for more info see the `Query parameters` section below).

For each filter, if not specified, it's not applied (so, for example, if no availability is specified, NFTs of all availability status' are returned).

If multiple filters are specified, NFTs that fit each criteria are returned (AND logic, not OR).

The return includes additional general info:

- `currentPage` (simply reflects the requested `page`)
- `numberOfPages` (takes into account the selected filters and `pageSize`)
- `lowerPriceBound` and `upperPriceBound` (takes into account the selected filters, except for the price filters)

Note: `ipfsHash` may be `null` in the response. It's only set (not `null`) for NFTs that have already been purchased at least once. We do this to allow remaining uncommitted to the existence of an NFT for as long as possible. This would be useful in case for some reason an NFT no longer is deemed appropriate and should be delisted. Delisting is always possible, in the sense of blacklisting it from the store, however once the IPFS hash is shared and the NFT is minted on the blockchain, it is no longer possible to erase the NFT entirely.

### HTTP Request

`GET /nfts`

### HTTP Request (with some filters set)

`GET /nfts?userAddress=tz1..&availability=onSale,upcoming`

### Query parameters

Parameter | Default | Description
--------- | ------- | -----------
page | 1 | Paginate through results, first page is 1
pageSize | 10 | Number of NFTs to fit on a single page
orderBy | 'id' | must be one of ['id', 'createdAt', 'price', 'name', 'views'].
orderDirection | 'asc' | must be one of ['asc', 'desc'].
categories | undefined | Get NFTs that belong to any of a comma separated set of category identifiers (eg: `1,4,10`). Multiple entries here means: find NFTs that belong to at least one of the given categories.
userAddress | undefined | Get NFTs that are owned by some tezos address.
availability | undefined | Comma separated enable-list of availability status (each entry being one of `'onSale','soldOut','upcoming'`). Multiple entries means find NFTs that belong to one of the entries.
priceAtLeast | undefined | Get NFTs that at least cost `x` amount (`x` is inclusive).
priceAtMost | undefined | Get NFTs that at most cost `x` amount (`x` is inclusive).
firstRequestAt | undefined | In seconds since UNIX, the timestamp of first pagination action (note: responsibility of setting this is at the caller). If set, only NFTs will be returned that were listed on the store at or before this timestamp.


## Search NFTs

```shell
http "/nfts/search" searchString=='rock'
```
> The above command returns JSON structured alike:

```json

{
    "categories": [
        {
            "id": 2,
            "name": "rock n roll",
            "description": "guitar music"
        }
	],
    "nfts": [
        "nfts": [
        {
            "artifactUri": "https://kanvas-admin-files.s3.amazonaws.com/NFT_FILE__54_image.png",
            "categories": [
                {
                    "description": "Sub photography category",
                    "id": 9,
                    "name": "Abstract"
                }
            ],
            "createdAt": 1653314947,
            "description": "testing nft end date",
            "displayUri": "https://kanvas-admin-files.s3.amazonaws.com/NFT_FILE__54_image.png",
            "editionsAvailable": 33,
            "editionsSize": 34,
            "id": 54,
            "ipfsHash": "ipfs://QmUVVgxmjpBt6SF8pNJZRNJiqy5nczL9xN8jiNJkTDApGW",
            "launchAt": 0,
            "name": "nft with end date",
            "onsaleUntil": 1662019200,
            "price": "13.00",
            "thumbnailUri": "https://kanvas-admin-files.s3.eu-central-1.amazonaws.com/NFT_FILE__54_thumbnail.png"
        }
    ]
}
```

This searches for:

- NFTs by name and description
- For categories by name

Currently returns a maximum of 3 NFTs and a maximum of 5 categories, sorted by most relevant first.

### HTTP Request

`GET /nfts/search`

### Query parameters

Parameter | Description
--------- | -----------
searchString | The text to search for


## Get a specific NFT

```shell
http POST "/nfts/2"
```

> The above command returns JSON structured alike:

```json
{
    "artifactUri": "https://kanvas-admin-files.s3.eu-central-1.amazonaws.com/NFT_FILE__39_image.png",
    "categories": [
        {
            "description": "Sub photography category",
            "id": 16,
            "name": "Black & White"
        }
    ],
    "createdAt": 1649094886,
    "description": "oldschool",
    "displayUri": "https://kanvas-admin-files.s3.eu-central-1.amazonaws.com/NFT_FILE__39_image.png",
    "editionsAvailable": 3,
    "editionsSize": 6,
    "id": 39,
    "ipfsHash": "ipfs://QmYdeHrRH1K6PKP5rmie5KYCHePJYXjn2CGauP5FJWnj2i",
    "launchAt": 0,
    "name": "typewriter",
    "price": "0.04",
    "thumbnailUri": "https://kanvas-admin-files.s3.eu-central-1.amazonaws.com/NFT_FILE__39_thumbnail.png"
}
```

Note: this is a POST, not a GET, because on each call we increment a "views" count. This is then used to allow for sorting `/nfts` on number of views.

### HTTP Request

`POST /nfts/:id`

### URL parameters

Parameter | Description
--------- | -----------
id | The id of the NFT to retrieve


# Categories

## Get all categories

```shell
http "/categories"
```

> The above command returns JSON structured alike:

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
    "userAddress": "$user_address",
    "signedPayload": "$user_password"
}
EOF
```

### HTTP Request

`POST /auth/register`

### Body parameters

Parameter | Description
--------- | -----------
userAddress | tezos address
signedPayload | password
userName | username (currently it's optional)

## Login

```shell
http POST "/auth/login" <<EOF
{
    "userAddress": "$user_address",
    "signedPayload": "$user_password"
}
EOF
```

> The above command returns JSON structured alike:

```json
{
    "userAddress": "addr",
    "id": 1,
    "maxAge": "86400000",
    "userName": null,
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwibmFtZSI6bnVsbCwiYWRkcmVzcyI6ImFkZHIiLCJyb2xlcyI6W10sImlhdCI6MTYzNzI0ODM5Nn0.HovGQYf8QaVh0ZEdLhYBzBYqqYTrH34j6MUDBw8Bb2M"
}
```

The "token" in the response is a JWT, it's the token that can be passed to any endpoints that require authorization by setting the "Authorization" header to "Bearer $token".

### HTTP Request

`POST /auth/login`

### Body parameters

Parameter | Description
--------- | -----------
userAddress | tezos address
signedPayload | password

## Logged in user

```shell
http "/auth/logged_user" Authorization:"Bearer $access_token"
```

> The above command returns JSON structured alike:

```json
{
    "id": 3,
    "userName": "utan_second-hand_10",
    "userAddress": "tz2D1s4VvB8HU8YrYGjKQJ3zXxiJw6QHyZQa",
    "createdAt": 1645716097,
    "profilePicture": null
}
```

### HTTP Request

`GET /auth/logged_user`

## Logout

```shell
http POST "/auth/Logout" Authorization:"Bearer $access_token"
```

### HTTP Request

`POST /auth/logout`

# User profile

## Get profile

```shell
http "/users/profile" userAddress==tz1
```

> The above command returns JSON structured alike:

```json
{
    "nftCount": 2,
    "user": {
        "createdAt": 1638452668,
        "id": 2,
        "profilePicture": "https://some-url (eg may be AWS S3 hosted)",
        "userAddress": "tz1",
        "userName": "test user"
    }
}
```

### HTTP Request

`GET /users/profile`

### Query parameters

Parameter | Description
--------- | -----------
userAddress | tezos address (optional, if undefined the endpoint returns currently logged in user if logged in).

## Edit profile

Edit:

- username (leave undefined if not to be changed)
- profile picture (leave undefined if not to be changed)

```shell
http -f POST "/users/profile/edit" profilePicture@./tg_image_775116946.jpeg userName=test
```

> The above command only returns the status code (201 on success), no body

### HTTP Request

`POST /users/profile/edit (form, content-type: multipart/form-data)`

### Body parameters

Parameter | Description
--------- | -----------
userName | change username

## Check validity of username edit

```shell
http "/users/profile/edit/check" userName==test
```

> The above command returns JSON structured alike:

```json
{
    "available": true,
    "userName": "test"
}
```

Note: must be logged in.

### HTTP Request

`GET /users/profile/edit/check`

### Query parameters

Parameter | Description
--------- | -----------
userName | username to check availability for

## Top buyers

Get the list of top buyers.


```shell
http "/users/topBuyers"
```

> The above command only returns JSON structured alike:

```
{
    "topBuyers": [
        {
            "totalPaid": "3019.91",
            "userAddress": "tz1g3coajkc9N77XDy55pVEgBGWspQfYqMiH",
            "userId": 1,
            "userName": "Setter_agitated_17",
            "userPicture": null
        },
        {
            "totalPaid": "1019.84",
            "userAddress": "tz2D1s4VvB8HU8YrYGjKQJ3zXxiJw6QHyZQa",
            "userId": 3,
            "userName": "utan_second-hand_10",
            "userPicture": null
        },
        {
            "totalPaid": "25.00",
            "userAddress": "tz1WWcuiKUN4Ed5zouETqr7MbVzd3vkC4ubr",
            "userId": 16,
            "userName": "Rattlesnake_naive_7",
            "userPicture": null
        },
        ...
    ]
}
```

### HTTP Request

`POST /users/profile/edit (form, content-type: multipart/form-data)`

# Cart

All endpoints related to shopping cart management.

A cart session is either derived from the cookie session (when not logged in)
or from the cart session that is connected to the logged in user. When logging
in after having done some cart operations on the cookie session, the user is
connected to the cookie based cart session (and if the user was already
connected to another cart session, this other session is canceled).

The only cart related endpoint that requires being logged in is the checkout endpoint.

Every cart session has an expiration date. In our demo deployments this is 30
minutes after the last POST action (add, remove, list), but this can be set to
whichever duration, determined by an environment variable.

## List NFTs in the cart

```shell
http POST "/users/cart/list"
```

> The above command returns JSON structured alike:

```json
{
    "expiresAt": "2021-11-18T14:29:59.403Z",
    "nfts": [
        {
            "id": 1,
            "name": "SomeNFT",
            ...
        }
    ]
}
```

### HTTP Request

`POST /users/cart/list`


## Add NFT

Note, this may fail if

* This NFT is not for sale (eg it's already sold out, or it's not yet released)
* All remaining editions of this NFT are already in other active carts
* This NFT is already in the active cart (currently we only allow 1 edition per NFT per active cart)

```shell
http POST "/users/cart/add/2"
```

### HTTP Request

`POST /users/cart/add/:id`

### URL parameters

Parameter | Description
--------- | -----------
id | The id of the NFT to add to the cart

## Remove NFT

```shell
http POST "/users/cart/remove/2"
```

### HTTP Request

`POST /users/cart/remove/:id`

### URL parameters

Parameter | Description
--------- | -----------
id | The id of the NFT to remove from the cart

## Checkout cart

Checking out the cart involves creating a payment intent, which then has to be
met (finalized) in the frontend application. For example, if paying with Stripe,
the payment intent has to be finalized by applying Stripe's SDK in the frontend.

```shell
http POST "/payment/create-payment-intent" Authorization:"Bearer $access_token"
```

### HTTP Request

`POST /payment/create-payment-intent`

## Nft Ownership status

This endpoint is used to find out after having finished payment whether the
purchased NFTs have arrived to the user's wallet onchain or if this process is
still pending.

Note that a user can own multiple editions of a single NFT (though limited by 1
edition per NFT per cart, users can own multiple editions regardless either through multiple payments
or through onchain transfers). Therefore this endpoint returns per requested nftId
a list of ownerships, eg if someone already fully owns nftId 1, and then purchases again nftId 1,
until the latter purchase has been processed onchain the response for this nftId will be: `{ nftId: "37", ownerStatuses: ["owned", "pending"] }`.

```

```shell
http POST /users/nftOwnership
```

Example response is alike:
```
[
    {
        nftId: "37",
        ownerStatuses: ["owned", "pending"]
    }
]
```

### HTTP Request

`POST /users/nftOwnership?nftIds=5,10,..`

### URL parameters

Parameter | Description
--------- | -----------
nftIds | A comma separated list of nft IDs to check the status of

## Checkout cart

Checking out the cart involves creating a payment intent, which then has to be
met (finalized) in the frontend application. For example, if paying with Stripe,
the payment intent has to be finalized by applying Stripe's SDK in the frontend.

```shell
http POST "/payment/create-payment-intent" Authorization:"Bearer $access_token"
```

### HTTP Request

`POST /payment/create-payment-intent`

