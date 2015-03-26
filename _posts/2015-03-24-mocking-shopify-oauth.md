---
layout: post
title:  "Mocking Shopify OAuth for Testing"
date:   2015-03-24 18:01:24
categories: omniauth
---

When writing acceptance or integration tests though it's useful to mock this
connection to simply the test itself and keep it local. This can be done in
OmniAuth by declaring `OmniAuth.test_mode = true`. Like so.

```ruby
# Mock OmniAuth logins
OmniAuth.config.test_mode = true
OmniAuth.config.add_mock(:shopify, {
  provider: 'shopify',
  credentials: {
    token: ENV.fetch('SHOPIFY_STORE_TOKEN')
  }
})
```

The problem is that this won't include a shop domain in the query string, which
is how Shopify identifies the authorized shop in the [shopify_app gem](https://github.com/Shopify/shopify_app).
To get around this we need to monkey patch the Shopify strategy with a custom
`callback_url`.

```ruby
# Patch to use custom callback url with shop domain
class OmniAuth::Strategies::Shopify
  option :callback_url, "shopify/callback?shop=#{ENV.fetch('SHOPIFY_STORE_DOMAIN')}"
end

# Mock OmniAuth logins
OmniAuth.config.test_mode = true
OmniAuth.config.add_mock(:shopify, {
  provider: 'shopify',
  credentials: {
    token: ENV.fetch('SHOPIFY_STORE_TOKEN')
  }
})
```

There you go, problem solved!
