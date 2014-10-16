---
layout: post
title:  "OmniAuth::StageBloc"
date:   2014-10-15 20:22:29
categories: omniauth-stagebloc
---

We've released an [OmniAuth strategy](https://github.com/TheCodeDeli/omniauth-stagebloc) to connect your
Ruby application with StageBloc. Doing so allows you to connect with your
StageBloc fans and keep data centralized.

I'm going to go through the steps of setting up a simple Ruby on Rails
application to pull a StageBloc user's information.

### Create the Application

The first step is to [register an
application](http://stagebloc.com/account/admin/management/developers/) with
your StageBloc account. The only thing to pay attention to here is the
<strong>Redirect URI</strong>. It needs match the following. This is where
OmniAuth expects to find the provider, which is `stagebloc` in this case.

```
http://localhost:3000/auth/stagebloc/callback
```

Once you've created your application you will receive your **client id** and
**client secret**.

![StageBloc App](http://cdn.stagebloc.com/production/photos/5123/original/20141016_014523_5123_678318.png)

### Adding OmniAuth with OmniAuth::StageBloc

Now that we've registered our application with StageBloc, the next step is to
add the authentication method to our Rails app. You'll need to add the
following gems to your `Gemfile` and run `bundle
install`.

```ruby
gem 'omniauth', '~> 1.2.2'
gem 'omniauth-stagebloc', '~> 0.1.0'
```

The next step is add StageBloc as an OmniAuth provider.

```ruby
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :stagebloc, ENV['STAGEBLOC_CLIENT_ID'], ENV['STAGEBLOC_SECRET'],
           :parse => :stagebloc_parser
end
```

### Adding the OmniAuth endpoints to our Rails app

We have now configured everything with StageBloc. The final step is adding the
endpoints in our Rails app to store the response from StageBloc.

It's very easy and requires few code additions. We'll add our callback route to
match what we set up in our StageBloc application. Then we'll create the
`SessionsController` that will store the values form creation.

```ruby
get '/auth/:provider/callback', to: 'sessions#create'
```

This is where StageBloc will send the user after authentication. A quick
tidbit, the `:provider` in the path gives us a hook if we want to allow
multiple OAuth providers. I.e StageBloc, Facebook, and Twitter could all be
options when creating an account in one application.

With our route in place we need to create the sessions controller. For the sake
of simplicity we're going to render the response as JSON for us to inspect.

```ruby
class SessionsController &lt; ApplicationController
  def create
    render json: auth_hash.info
  end
  protected
  def auth_hash
    request.env['omniauth.auth']
  end
end
```

### Starting the Authorization Process

There's something that hasn't been covered. How do we start the authentication
process. We've put all this code in place for handling the response, but we
haven't done anything towards sending the user to StageBloc for authorization.

OmniAuth adds a route to our application that takes care of this for us. All we
need to do is visit http://localhost:3000/auth/stagebloc

Why is it done this way? The answer is best described in OmniAuth's README.

> It is designed to be a black box that you can send your application's users
> into when you need authentication and then get information back.

I think this is one of the coolest features of OmniAuth. The fact we can log in
through a 3rd party service by simply adding the client id and secret is pretty
awesome.

![Example Response](http://cdn.stagebloc.com/production/photos/5123/original/20141016_030929_5123_678321.png)

### The Response

OmniAuth collects the information from StageBloc with a Ruby-esque DSL.
Something that isn't shown in the screenshot is the access token. This is very
important as it's used to act as the logged in user.

```ruby
auth_hash.credentials.token #=> RWNrRlhnSnhia1d6U09Wbm1xUTNIZzBEenF
```

### Sample Application

I've pushed the sample application from this article on GitHub. Feel free to
check it out and download it for your own use.

https://github.com/TheCodeDeli/omniauth-stagebloc-demo
