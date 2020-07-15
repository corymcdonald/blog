---
title: "Go - Using Macaroons"
date: 2019-11-15T14:36:17-06:00
draft: false
description: "Serializing, deserializing macaroons using Go-lang"
categories: [
  "go"
]
tags: [
    "security",
]
---

Macaroons are a flavor of authorizations. We're using them at Brave to verify the identity during payment flows.

In Ruby reading macaroons is very straight forward with the help of [this blog post](http://tech.tmh.io/concept/2016/06/07/macaroons-a-new-flavor-for-authorization.html). However in Go it's a little more challenging.

For example if I'm creating a macaroon with the following properties

```ruby
public_key = 'public-api-key'
secret_key = 'secret-key'
website    = 'https://corywmcdonald.com'
```

With the following first-party caveats

```ruby
macaroon.add_first_party_caveat('expires_at = 2020-01-01T00:00')
macaroon.add_first_party_caveat('currency = USD')
macaroon.add_first_party_caveat('id = 775e1c2d-f41d-4236-9cb8-6baea355cfe6')

```

```go

token := "MDAxN2xvY2F0aW9uIGJyYXZlLmNvbQowMDFhaWRlbnRpZmllciBwdWJsaWMga2V5CjAwMzJjaWQgaWQgPSA1Yzg0NmRhMS04M2NkLTRlMTUtOThkZC04ZTE0N2E1NmI2ZmEKMDAxN2NpZCBjdXJyZW5jeSA9IEJBVAowMDE1Y2lkIHByaWNlID0gMC4yNQowMDJlY2lkIGV4cGlyZXNfYXQgPSAyMDIwLTAxLTAyVDIzOjA2OjEwKzAwMDAKMDAyZnNpZ25hdHVyZSDBV0h4Fl3Vh9SSJVnbNZOW5zIrR"

macBytes, err := macaroon.Base64Decode([]byte(req.Items[0].SKU))
if err != nil {
  panic(err)
}
mac := &macaroon.Macaroon{}
err = mac.UnmarshalBinary(macBytes)
if err != nil {
  panic(err)
}

fmt.Println(string(mac.Id()))
```

