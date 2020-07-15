---
title: "Go - Using Macaroons"
date: 2020-02-15T14:36:17-06:00
draft: false
description: "Serializing, deserializing macaroons using Go-lang"
categories: [
  "go"
]
tags: [
    "security",
]
---
 ---

  Macaroons are a flavor of authorizations. We're using them at Brave to verify the identity during payment flows. It's not always clear how to use them in more static languages like GoLang.
 <!--more-->
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

We can generated the macaroon pretty easily in Ruby. But if we want to do this in Go it's a little more complex. 

That's why we at Brave wrote this [macaroon-cli tool](https://github.com/brave-intl/bat-go/tree/master/bin/macaroon-gen). This helps you generate valid tokens off of YAML files.

With a valid macaroon token we can now parse, deserialize, and validate it.

```go

token := "MDAxN2xvY2F0aW9uIGJyYXZlLmNvbQowMDFhaWRlbnRpZmllciBwdWJsaWMga2V5CjAwMzJjaWQgaWQgPSA1Yzg0NmRhMS04M2NkLTRlMTUtOThkZC04ZTE0N2E1NmI2ZmEKMDAxN2NpZCBjdXJyZW5jeSA9IEJBVAowMDE1Y2lkIHByaWNlID0gMC4yNQowMDJlY2lkIGV4cGlyZXNfYXQgPSAyMDIwLTAxLTAyVDIzOjA2OjEwKzAwMDAKMDAyZnNpZ25hdHVyZSDBV0h4Fl3Vh9SSJVnbNZOW5zIrR"

macBytes, err := macaroon.Base64Decode([]byte(token))
if err != nil {
  panic(err)
}
mac := &macaroon.Macaroon{}
err = mac.UnmarshalBinary(macBytes)
if err != nil {
  panic(err)
}

// Now we can look at the caveats of the Macaroon
caveats := mac.Caveats()

for i := 0; i < len(caveats); i++ {
  caveat := mac.Caveats()[i]
  values := strings.Split(string(caveat.Id), "=")
  key := strings.TrimSpace(values[0])
  value := strings.TrimSpace(values[1])
}
```


In the loop above we can get all the key value pairs that have been set in the Macaroon but we need to validate that those values were set by the person we expected to sign the macaroon.

With the `mac` variable we defined above we can validate that the secret matches the secret used generating the token.

```go
secret := "secret"
var discharges []*macaroon.Macaroon
err = mac.Verify([]byte(secret), CheckCaveat, discharges)

// Macaroon is signed with the wrong secret Fishy! 🐟
if err != nil {
  return nil, err
}

// CheckCaveat validates a caveat based on existing conditions
func CheckCaveat(caveat string) error {
	values := strings.Split(string(caveat), "=")
	key := strings.TrimSpace(values[0])
	value := strings.TrimSpace(values[1])

	switch key {
	case "currency":
		{
      // Methods for validation for each of the properties.
			err := validateCurrency(value)
			if err != nil {
				return err
			}
		}
	}

	return nil
}
```

With the above code we've validated the macaroon token and have functions to validate the caveats are the expected values. 

A good blog post with better validation functions is [API Input Validation in Golang](https://husobee.github.io/golang/validation/2016/01/08/input-validation.html). This walks through adding validation methods for all the structs that we could be using.

